---
layout: default
title: SpringBoot Jackson
parent: SpringBoot
nav_order: 190
---


## SpringBoot jackson 配置

### 1、模块位置：

SpringBoot 引入 `spring-boot-starter-web` 时，会有 `spring-boot-starter-json` 模块。

该模块中，有个`json-databind` 的依赖，再下一层包下有 `json-core` 的依赖。其实在json相关模块都有依赖 `json-core` 和  `json-annotations` 。

包路径是 `com.fasterxml.jackson.core` 。


### 2、配置位置

既然是springBoot项目，那肯定要找自动配置（autoconfigure）相关的包`spring-boot-autoconfigure`依赖。 我的版本是： `spring-boot-autoconfigure-2.2.7.RELEASE.jar` 。

自动配置类：`org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration`。

配置属性对应类：`org.springframework.boot.autoconfigure.jackson.JacksonProperties`。

```java
@ConfigurationProperties(
    prefix = "spring.jackson"
)
public class JacksonProperties {
    private String dateFormat;
    private String jodaDateTimeFormat;
    private String propertyNamingStrategy;
    private final Map<PropertyAccessor, Visibility> visibility = new EnumMap(PropertyAccessor.class);
    private final Map<SerializationFeature, Boolean> serialization = new EnumMap(SerializationFeature.class);
    private final Map<DeserializationFeature, Boolean> deserialization = new EnumMap(DeserializationFeature.class);
    private final Map<MapperFeature, Boolean> mapper = new EnumMap(MapperFeature.class);
    private final Map<Feature, Boolean> parser = new EnumMap(Feature.class);
    private final Map<com.fasterxml.jackson.core.JsonGenerator.Feature, Boolean> generator = new EnumMap(com.fasterxml.jackson.core.JsonGenerator.Feature.class);
    private Include defaultPropertyInclusion;
    private TimeZone timeZone = null;
    private Locale locale;

    public JacksonProperties() {
    }
}
```    

### 3、YML配置

**配置属性说明：**

- **`spring.jackson.date-format`** ：指定日期格式，比如yyyy-MM-dd HH:mm:ss，或者具体的格式化类的全限定名

- **`spring.jackson.deserialization`** ：是否开启Jackson的反序列化

- **`spring.jackson.generator`** ：是否开启json的generators.

- **`spring.jackson.joda-date-time-format`** ：指定Joda date/time的格式，比如yyyy-MM-ddHH:mm:ss). 如果没有配置的话，dateformat会作为backup

- **`spring.jackson.locale`** ：指定json使用的Locale.

- **`spring.jackson.mapper`** ：是否开启Jackson通用的特性.

- **`spring.jackson.parser`** ：是否开启jackson的parser特性.

- **`spring.jackson.property-naming-strategy`** ：指定PropertyNamingStrategy(CAMEL_CASE_TO_LOWER_CASE_WITH_UNDERSCORES)或者指定PropertyNamingStrategy子类的全限定类名.

- **`spring.jackson.serialization`** ：是否开启jackson的序列化.

- **`spring.jackson.serialization-inclusion`** ：指定序列化时属性的inclusion方式，具体查看JsonInclude.Include枚举.

- **`spring.jackson.time-zone`** ：指定日期格式化时区，比如America/Los_Angeles或者GMT+10.


YML配置样例：

```yml
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss #日期格式化
    serialization:
      indent_output: true #格式化输出 
      fail_on_empty_beans: false #忽略无法转换的对象
    defaultPropertyInclusion: NON_EMPTY   #设置空如何序列化
    deserialization:
      fail_on_unknown_properties: false  #允许对象忽略json中不存在的属性
    parser:
      allow_unquoted_control_chars: true #允许出现特殊字符和转义符
      allow_single_quotes: true  #允许出现单引号
```

properties 配置样例：

```properties
#日期格式化
spring.jackson.dateFormat='yyyy-MM-dd HH:mm:ss'
#格式化输出
spring.jackson.serialization.indent-output=true
#忽略无法转换的对象
spring.jackson.serialization.fail-on-empty-beans=false

#允许对象忽略json中不存在的属性
spring.jackson.deserialization.fail-on-unknown-properties=false
#允许出现特殊字符和转义符
spring.jackson.parser.allow-unquoted-control-chars=true
#允许出现单引号
spring.jackson.parser..allow-single-quotes=true
```

Bean 配置样例：

```java
@Configuration
public class JacksonConfig {

	@Bean
	@Primary
	@ConditionalOnMissingBean(ObjectMapper.class)
	public ObjectMapper jacksonObjectMapper(Jackson2ObjectMapperBuilder builder)
    {
        ObjectMapper objectMapper = builder.createXmlMapper(false).build();

        /* 通过该方法对mapper对象进行设置，所有序列化的对象都将按改规则进行系列化
        * Include.Include.ALWAYS 默认
        * Include.NON_DEFAULT 属性为默认值不序列化
        * Include.NON_EMPTY 属性为 空（""） 或者为 NULL 都不序列化，则返回的json是没有这个字段的。这样对移动端会更省流量
        * Include.NON_NULL 属性为NULL 不序列化
        */
        objectMapper.setSerializationInclusion(JsonInclude.Include.NON_EMPTY);
        objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        // 允许出现特殊字符和转义符
        objectMapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_CONTROL_CHARS, true);
        // 允许出现单引号
        objectMapper.configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true);
        // 字段保留，将null值转为""
        objectMapper.getSerializerProvider().setNullValueSerializer(new JsonSerializer<Object>()
        {
            @Override
            public void serialize(Object o, JsonGenerator jsonGenerator,
                                  SerializerProvider serializerProvider)
                    throws IOException
            {
                jsonGenerator.writeString("");
            }
        });
        return objectMapper;
    }

    public static final DateTimeFormatter LOCAL_DATE_TIME_FORMATTER = new DateTimeFormatterBuilder().appendPattern("yyyy-MM-dd HH:mm:ss").toFormatter();

    @Bean
    public ObjectMapper serializingObjectMapper() {
        JavaTimeModule module = new JavaTimeModule();
        LocalDateTimeDeserializer localDateTimeDeserializer = new LocalDateTimeDeserializer(LOCAL_DATE_TIME_FORMATTER);
        module.addDeserializer(LocalDateTime.class, localDateTimeDeserializer);
        ObjectMapper objectMapper = Jackson2ObjectMapperBuilder.json()
                .modules(module)
                .featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
                .build();
        return objectMapper;
    }
}
```
