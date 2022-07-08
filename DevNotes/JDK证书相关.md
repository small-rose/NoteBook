---
layout: default
title: JDK cacerts
parent: DevNotes
nav_order: 8
---

# JDK cacerts
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
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

```bash
# 切换到命令目录
C:\Program Files\Java\jdk1.8.0_212\bin

## 导入证书
keytool -import -alias ssocas -keystore D:/Java/jdk1.8/jre/lib/security/cacerts -file D:/casserver.cer -trustcacerts
```

默认密码是 changeit

Linux 环境

```bash
cd /usr/local/java/jdk1.8.0_212/jre/lib/security/cacerts

## 上传 casserver.cer  casserver.keystore 文件后执行
keytool -import -alias ssocas -keystore  /usr/local/java/jdk1.8.0_212/jre/lib/security/cacerts  -file  casserver.cer -trustcacerts
```

