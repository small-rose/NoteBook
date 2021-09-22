
## JSON 相关


### fastjson 

依赖包

```xml
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.4</version>
        </dependency>
```

#### @JSONField

使用 @JSONField

可以让JSON字符串里的key和Bean的属性不一致时，映射赋值到对应的属性上。

```java
@Data
public class MyBean implements Serializable {

    @JSONField(name = "Status")
    private String status ;

    @JSONField(name = "RETURN_MSG")
    private String ReturnMsg ;
}
```

```java
String data = "{Status":"0","RETURN_MSG":success"}";
MyBean request = JSON.parseObject(data, MyBean.class);
```