---
layout: default
title: SpringBoot YAML
parent: SpringBoot
has_children: true
nav_order: 10
---



## YAML语法学习

其实 `YAML` 并不是springboot 特有语法，比如 Hexo 的配置文件都是这种格式，只是一时又不好归类，就先放到springboot 里了。



### 一、基本语法

 `YAML` 是以键值对的方式书写代码块的，支持多层级，对象数组等集合元素。

基本格式：

```yml
key: value
key:
  key2: value2
  key2：
    key3: value3
```

`key:  value`：表示一对键值对（冒号后面必须有空格）；属性键和对应的值是大小写敏感的。

上述写法等价于 `properties` 格式：

```properties
key=value
key.key2=value2
key.key2.key3=value3
```

 

`YAML` 以空格的缩进来控制层级关系；只要是左对齐的一列数据，都是同一个层级的，比如

```yml
server:
  port: 8080
  servlet:
    context-path: /dbcm
    session:
      timeout: 1800s
```



### 二、特殊的值

#### 1、单引号和双引号

一般的键和值都是字面量和普通的值（数字，字符串，布尔）组成。字符串默认不带单引号`' '`或双引号 `" "`。

- `''` : 单引号，会转义特殊字符，特殊字符会解析成普通的字符串数据。
- `" "` ：双引号，不会转义字符串里面的特殊字符。特殊字符表达解析成原本的含义。

如单引号示例：

```yml
xiaocai:
  name: 'small \n rose' 
```

解析之后得到的 name 值就是：

````txt
 small \n rose 
````

双引号示例：

```yml
xiaocai:
  name: "small \n rose" 
```

解析之后得到的 name 值就是：

```txt
small
rose
```

#### 2、数组与集合

（1）对象

对象依然是键值对的方式:

```yml
user:
  name: smale-rose
  email: small-rose@qq.com
```

也可以使用类似 map 的行间方式书写：

```yaml
user: { name: smale-rose, email: small-rose@qq.com}
```

注意冒号后面的空格。



（2）数组或集合

用 `-` 值表示数组中的一个元素。

如：

```yaml
fruits:
  ‐ apple
  ‐ pear
  ‐ banana
  ‐ watermelon
```

也可以使用类似 list 的行间方式书写：

```yaml
fruits: [apple, pear, banana, watermelon]
```

### 三、文档块特征

YAML 的文档块的特征，可以很好的配合 SpringBoot 的 `Profile` 特征。

YAML 文档使用  `---` 进行分割文档块。

比如：

```yaml
server:
  port: 80
spring:
  profiles:
    active: dev  
---

server:
  port: 8081
spring:
  profiles:
    active: dev
---
server:
  port: 9090
spring:
  profiles:
    active: prod
```

示例中在第一个文档块激活了 `dev` 环境，应用启动之后使用端口是 8081。




### 四：占位符

#### 1、随机数

```yaml
${random.value}、
${random.uuid}、
${random.int}、
${random.long}
${random.int(10)}、
${random.int[1024,65536]}
```

#### 2、占位符取值

占位符获取之前配置的值，如果没有可以是用 `:` 指定默认值。

```yaml
user:
  name: smale-rose
  email: ${random.value}_small-rose@qq.com
  age: ${random.int[10,50]}
  addr: Address_${user.name: no_user}
  last_name: LastName_${user.lastName: no_user}
```



### 五、配置与JavaBean 绑定

#### 1、绑定示例

（1）可以引入配置文件处理

```xml
<!‐‐导入配置文件处理器，配置文件进行绑定就会有提示‐‐>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring‐boot‐configuration‐processor</artifactId>
  <optional>true</optional>
</dependency> 
```

（2）对应 `application.yaml`  的 配置

```yaml
initvalue:
  evn: prod
  port: 8080
  flag: false
  dbTypeIdList:
      - 1
      - 2
      - 3
```

（3）使用注解 `@ConfigurationProperties` 进行绑定

比如我这个示例是实际项目有一些初始化配置值，字段特别多，作为示例，选几个：

```java
@Component
@ConfigurationProperties(prefix = "initvalue")
public class InitValueConfig {
  
  private String evn ;
  
  private String port ;
 
  private List<Long> dbTypeIdList = new ArrayList<Long>();  
     private boolean flag;
   // setter getter ...
}
```

注意： **@ConfigurationProperties(prefix = "initvalue")** 表示默认从全局配置文件中获取值；如果 initvalue 前缀的配置放在另外的 `initvalue.yml` 文件中，则需要添加  **`@PropertySource("classpath:initvalue.yml")` ** 表示从指定的配置中查找对应的前缀，可以指定多个配置文件  **`@PropertySource({"classpath:initvalue.yml","classpath:initvalue2.yml"})`** 

如果是properties文件，同理：

```properties
initvalue.evn=prod
initvalue.port: 8080
initvalue.dbTypeIdList[0]=1
initvalue.dbTypeIdList[1]=2
initvalue.dbTypeIdList[2]=3
initvalue.flag=false
```

也可以进行绑定：

```java
@Configuration
@PropertySource("classpath:initvalue.properties")
@ConfigurationProperties(prefix = "initvalue")
public class InitValueConfig {
  
  private String evn ;
  
  private String port ;
 
  private List<Long> dbTypeIdList = new ArrayList<Long>();  
  private boolean flag;
   // setter getter ...
}
```

**@ConfigurationProperties(prefix = "initvalue")** ：默认从全局配置文件中获取值；

**@PropertySource** ：表示加载指定的配置文件来映射绑定；

**@ImportResource**：导入Spring的配置文件，让配置文件里面的内容生效；

```java
@ImportResource(locations = {"classpath:beans.xml"})
```

当然 springboot  推荐使用注解` @Configuration` 和 `@Bean` ：

```java
@Configuration
public class BeanConfig {

    @Bean
    public Filter validatorFilter() throws FileNotFoundException{ 
      System.out.println("-------------load validatorFilter---------------");
        return new ValidatorFilter();
    }
}
```





#### 2、取值方式

获取值除了`@ConﬁgurationProperties` 还可以使用`@Value` 获取。

比如：

```java
public class InitValueConfig {
    
  @Value("${initvalue.evn}")
  private String evn ;
    
  @Value("${initvalue.port}")
  private String port ;
    
  @Value("#{'${initvalue.list}'.split(',')}")
  private List<Long> dbTypeIdList = new ArrayList<Long>();  
    
  @Value("#{${initvalue.maps}}")  
  private Map<String,String> maps;
    
  @Value("${initvalue.flag}")
  private boolean flag;
    
  @Value("#{${webServiceMap}}")
  private Map<String,String> webServiceMap;
   // setter getter ...
}
```

对应的 `YAML` 为

```yaml
initvalue:
  evn: prod
  port: 8080
  flag: false
  list: 1,2,3
    maps: "{key1: 'value1', key2: 'value2'}"
    webServiceMap: "{webServiceURL: 'http://web.com.server/xxx', success: '0000',filePath: '/home/file/'}"
```

取值方式比较

|                      | @value                   | @ConfigurationProperties |
| -------------------- | ------------------------ | ----------------------- |
| 功能                 | 批量注入配置文件中的属性 | 一个个指定              |
| 松散绑定（松散语法） | 支持                     | 不支持                  |
| SpEL                 | 不支持                   | 支持                    |
| JSR303数据校验       | 支持                     | 不支持                  |
| 复杂类型封装         | 支持                     | 不支持                  |

配置文件`yml` 或者 `properties` 二者都能获取到值；只获取一下配置文件中的某项值，使用@Value；参数较多时使用JavaBean来和配置文件进行绑定映射，可以直接使用`@ConﬁgurationProperties` 省去了一个个配置的麻烦。


{: .tips }
> (1)优先级为： properties > xml > yml > yaml
> (2)注意  @PropertySource 注解默认是只支持 properties 格式配置文件的读取的。不支持解析 yaml/yml文件. ：

支持yml解决方法
                                         
@PropertySource 的注解中，有一个factory属性，可指定一个自定义的PropertySourceFactory接口实现，用于解析指定的文件。默认的实现是DefaultPropertySourceFactory，使用了PropertiesLoaderUtils.loadProperties进行文件解析，所以默认就是使用Properties进行解析的。

1、可以写一个类继承DefaultPropertySourceFactory，然后重写createPropertySource()方法

```java
public class YamlAndPropertySourceFactory extends DefaultPropertySourceFactory {
    @Override
    public PropertySource<?> createPropertySource(String name, EncodedResource resource) throws IOException {
        if (resource == null) {
            return super.createPropertySource(name, resource);
        }
        Resource resourceResource = resource.getResource();
        if (!resourceResource.exists()) {
            return new PropertiesPropertySource(null, new Properties());
        } else if (resourceResource.getFilename().endsWith(".yml") || resourceResource.getFilename().endsWith(".yaml")) {
            List<PropertySource<?>> sources = new YamlPropertySourceLoader().load(resourceResource.getFilename(), resourceResource);
            return sources.get(0);
        }
        return super.createPropertySource(name, resource);
    }
}
```

2、使用的时候加上factory属性，属性值为YamlAndPropertySourceFactory.class；如代码：

```java
@Data
@Component
@ConfigurationProperties(prefix = "init")
@PropertySource(value = "classpath:application.yml", factory = YamlAndPropertySourceFactory.class)
public class InitDataSource {

    private List<DataSourceInfo> dataSourceList = new ArrayList<>();

}
```

yml配置如下:

```yaml
init:
  dataSourceList:
    - name: ORACLE_DEV
      driverName: oracle.jdbc.OracleDriver
      url: jdbc:oracle:thin:@192.168.10.118:1521:orcl
      username: BVIS
      password: bvis123
    - name: ORACLE_SIT
      driverName: oracle.jdbc.OracleDriver
      url: jdbc:oracle:thin:@192.168.10.118:1521:orcl
      username: BVIS
      password: bvis123
    - name: MYSQL_57
      driverName: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://192.168.10.184:3306/bp_dev
      username: sf_dev_01
      password: Sf@123321
```

3、如果这个绑定数据因环境改不同，也可以使用这种动态化的使用方式

```java
@Data
@Component
@ConfigurationProperties(prefix = "init")
/* 此处可以动态化 如  value = "classpath:application-${spring.profiles.active}.yml"
*/
@PropertySource(value = "classpath:application-${spring.profiles.active}.yml", factory = YamlAndPropertySourceFactory.class)
public class InitDataSource {

    private List<DataSourceInfo> dataSourceList = new ArrayList<>();

}
```


