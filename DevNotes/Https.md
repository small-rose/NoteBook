---
layout: default
title: Https
parent: DevNotes
nav_order: 8
---

# Https
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## https

https 流程

![HTTPS流程图](https-flow.png)

来源： [https://segmentfault.com/a/1190000021494676](https://segmentfault.com/a/1190000021494676)

[HTTP 与 HTTPS 的区别](https://www.runoob.com/w3cnote/http-vs-https.html)

## Boot Tomcat Https

首先使用 jdk 自带的 keytool 命令生成证书复制到项目的 resources 目录下（生成的证书一般在用户目录下 C:\Users\Administrator\server.keystore）

（1）配置

```yaml
server:
  ssl:
    # 证书路径
    key-store: classpath:server.keystore
    key-alias: tomcat
    enabled: true
    key-store-type: JKS
    #与申请时输入一致
    key-store-password: 123456
    # 浏览器默认端口 和 80 类似
  port: 443
  servlet:
    context-path: /
```

（2）配置类

```java
/**
 * <p>
 * HTTPS 配置类
 * </p>
 *
 * @author yangkai.shen
 * @date Created in 2020-01-19 10:31
 * 来源 https://github.com/xkcoding/spring-boot-demo/tree/master/demo-https
 */
@Configuration
public class HttpsConfig {
    /**
     * 配置 http(80) -> 强制跳转到 https(443)
     */
    @Bean
    public Connector connector() {
        Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
        connector.setScheme("http");
        connector.setPort(80);
        connector.setSecure(false);
        connector.setRedirectPort(443);
        return connector;
    }

    @Bean
    public TomcatServletWebServerFactory tomcatServletWebServerFactory(Connector connector) {
        TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory() {
            @Override
            protected void postProcessContext(Context context) {
                SecurityConstraint securityConstraint = new SecurityConstraint();
                securityConstraint.setUserConstraint("CONFIDENTIAL");
                SecurityCollection collection = new SecurityCollection();
                collection.addPattern("/*");
                securityConstraint.addCollection(collection);
                context.addConstraint(securityConstraint);
            }
        };
        tomcat.addAdditionalTomcatConnectors(connector);
        return tomcat;
    }
}
```

## JDK certificate

### keytool 命令:

```
 -certreq            生成证书请求
 -changealias        更改条目的别名
 -delete             删除条目
 -exportcert         导出证书
 -genkeypair         生成密钥对
 -genseckey          生成密钥
 -gencert            根据证书请求生成证书
 -importcert         导入证书或证书链
 -importpass         导入口令
 -importkeystore     从其他密钥库导入一个或所有条目
 -keypasswd          更改条目的密钥口令
 -list               列出密钥库中的条目
 -printcert          打印证书内容
 -printcertreq       打印证书请求的内容
 -printcrl           打印 CRL 文件的内容
 -storepasswd        更改密钥库的存储口令

使用 "keytool -command_name -help" 获取 command_name 的用法
```
### 证书查看

查看证书命令

```bash
keytool -list -keystore d:/tomcat.keystore -storepass 123456
keytool -list -v -keystore d:/tomcat.keystore -storepass 123456
keytool -list -rfc -keystore d:/tomcat.keystore -storepass 123456
```

参数：

-list 列出秘钥库中的条目

-v 详细输出

-rfc 以RFC样式输出


### 证书删除

```bash
keytool -delete -alias emailcert -keystore "D:/Java/jdk1.8/jre/lib/security/cacerts"  -storepass changeit
```

-alias  指定别名

### 证书导入

Windowns 环境

```
# 切换到命令目录
C:\Program Files\Java\jdk1.8.0_212\bin

## 导入证书
keytool -import -alias ssocas -keystore C:/Program Files/Java/jdk1.8.0_212/jre/lib/security/cacerts -file D:/casserver.cer -trustcacerts
```

默认密码是 changeit

Linux 环境

```bash
cd /usr/local/java/jdk1.8.0_212/jre/lib/security/cacerts

## 上传 casserver.cer  casserver.keystore 文件后执行
keytool -import -alias ssocas -keystore  /usr/local/java/jdk1.8.0_212/jre/lib/security/cacerts  -file  casserver.cer -trustcacerts
```

