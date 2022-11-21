---
layout: default
title: DevCommon
parent: DevNotes
nav_order: 1
---

## 开发常用
{: .no_toc }


## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


### centos 源

CentOS 官方下载地址：https://www.centos.org/download/

Centos国内下载源

- http://man.linuxde.net/download/CentOS
- http://mirrors.btte.net/centos/7/isos/x86_64/
- http://mirrors.cn99.com/centos/7/isos/x86_64/
- http://mirrors.sohu.com/centos/7/isos/x86_64/
- http://mirrors.aliyun.com/centos/7/isos/x86_64/
- http://centos.ustc.edu.cn/centos/7/isos/x86_64/
- http://mirrors.neusoft.edu.cn/centos/7/isos/x86_64/
- http://mirror.lzu.edu.cn/centos/7/isos/x86_64/
- http://mirrors.163.com/centos/7/isos/x86_64/
- http://ftp.sjtu.edu.cn/centos/7/isos/x86_64/ 



>帐号：1583039459@qq.com
>密码：www.lovecoder.cn123W
>请大家不要修改密码。注意：此帐号只用于下载JDK使用，请勿用于非法用途。


### 电脑护眼色

绿豆沙色能有效的减轻长时间用电脑的用眼疲劳！

色调：85，饱和度：123，亮度：205；

RGB颜色红：199，绿：237，蓝：204；

十六进制颜色：#C7EDCC 或用 #CCE8CF 
<table  width="200px">
<tr>
<td style="background-color: #C7EDCC" width="40px" height="40px">#C7EDCC</td>
<td style="background-color: #CCE8CF" width="40px" height="40px">#CCE8CF</td>
<td style="background-color: #FAF9DE" width="40px" height="40px">#FAF9DE</td>
<td style="background-color: #FFF2E2" width="40px" height="40px">#FFF2E2</td>
<td style="background-color: #FDE6E0" width="40px" height="40px">#FDE6E0</td>
</tr>
<tr>
<td style="background-color: #E3EDCD" width="40px" height="40px">#E3EDCD</td>
<td style="background-color: #DCE2F1" width="40px" height="40px">#DCE2F1</td>
<td style="background-color: #E9EBFE" width="40px" height="40px">#E9EBFE</td>
<td style="background-color: #EAEAEF" width="40px" height="40px">#EAEAEF</td>
<td style="background-color: #B7E8BD" width="40px" height="40px">#B7E8BD</td>
</tr>
<tr>
<td style="background-color: #19CAAD" width="40px" height="40px">#19CAAD</td>
<td style="background-color: #8CC7B5" width="40px" height="40px">#8CC7B5</td>
<td style="background-color: #A0EEE1" width="40px" height="40px">#A0EEE1</td>
<td style="background-color: #BEE7E9" width="40px" height="40px">#BEE7E9</td>
<td style="background-color: #BEEDC7" width="40px" height="40px">#BEEDC7</td>
</tr>
<tr>
<td style="background-color: #D6D5B7" width="40px" height="40px">#D6D5B7</td>
<td style="background-color: #D1BA74" width="40px" height="40px">#D1BA74</td>
<td style="background-color: #E6CEAC" width="40px" height="40px">#E6CEAC</td>
<td style="background-color: #ECAD9E" width="40px" height="40px">#ECAD9E</td>
<td style="background-color: #F4606C" width="40px" height="40px">#F4606C</td>

</tr>
</table>



其他几种电脑窗口视力保护色：
```
银河白    #FFFFFF    RGB(255, 255, 255)

杏仁黄    #FAF9DE    RGB(250, 249, 222)

秋叶褐    #FFF2E2    RGB(255, 242, 226)

胭脂红    #FDE6E0    RGB(253, 230, 224)

青草绿    #E3EDCD    RGB(227, 237, 205)

海天蓝    #DCE2F1    RGB(220, 226, 241)

葛巾紫    #E9EBFE    RGB(233, 235, 254)

极光灰    #EAEAEF    RGB(234, 234, 239)
```

### JAVA环境变量

1、新建系统环境变量

变量名：JAVA_HOME

变量值：C:\Program Files\Java\jdk1.8.0_191

变量值就是jdk安装目录。

2、配置Path，

（1）有就在变量值处追加，以“;”分隔，

放末尾：；%JAVA_HOME%\bin

放开头：%JAVA_HOME%\bin；

（2）没有就新建系统环境变量

变量名：Path

变量值： %JAVA_HOME%\bin

3、配置CLASSPATH

变量名：CLASSPATH

变量值：.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\rt.jar%JAVA_HOME%\lib\tools.jar

    rt.jar：Java基础类库，也就是Java doc里面看到的所有的类的class文件。

    tools.jar：是系统用来编译一个类的时候用到的，即执行javac的时候用到。


常用的环境变量

CLASSPATH

```
.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\rt.jar%JAVA_HOME%\lib\tools.jar;%ANT_HOME%\lib;%JMETER_HOME\lib\ext\ApacheJMeter_core.jar;%JMETER_HOME%\lib\jorphan.jar;
```

PATH:

```
%JAVA_HOME%\bin;

%MAVEN_HOME%\bin;

%ANT_HOME%\bin;

d:\subversion\bin;
```

4、配置验证

打开cmd命令窗口执行java -version， javac ,java -verbose等皆可。

### MAVEN环境变量

1、新建系统环境变量

变量名：MAVEN_HOME

变量值：E:\apache-maven-3.3.9

变量值就是MAVEN安装目录。

可以参考 MAVEN官网安装指南

2、配置setting.xml

配置文件一般在...\apache-maven-3.3.9\conf目录下。

（1）本地仓库路径修改：

直接放<settings>根节点下即可。
```xml
 <localRepository>E:/server/MavenRepository/maven_jar</localRepository>
```
如果不配置，默认路径是：C:/Users/${user.home}/.m2/repository

（2）修改源镜像仓库地址。默认去maven官方源仓库下载太慢了，修改为国内的源仓库。

在mirrors节点下：

```xml

 <!-- 阿里云中央仓库 -->
 <mirror>
       <id>nexus-aliyun</id>
       <mirrorOf>central</mirrorOf>
       <name>Nexus aliyun</name>
       <url>http://maven.aliyun.com/nexus/content/groups/public</url>
 </mirror>
```

3、在开发工具中配置。

一般是：window—Preferences—maven—User Settings 更换maven的setting.xml配置。

有些开发工具已经内嵌了maven，通常默认的setting.xml文件位置在：
```
C:/Users/${user.home}/.m2/setting.xml
```
其他



idea 自带maven

```
D:\dev-tools\JetBrains\IntelliJ IDEA 2020.1\plugins\maven\lib\maven3\conf
```


### Gradle 环境变量

官网：https://gradle.org/releases/

配置环境变量

1、新建系统环境变量


变量名：GRADLE_HOME

变量值：D:\apache\gradle-7.0
     

变量名：GRADLE_USER_HOME  （自定义仓库，可不配置（可以为Maven的仓库目录)）

变量值：D:\apache\maven-repository
 
 
环境变量 ：Path (追加)

变量值：%GRADLE_HOME%\bin;

2、验证

CMD窗口输入命令行： 

```bash
gradle -v
```



3、配置Gradle仓库源

配置Gradle仓库源（可不配置）：

在Gradle安装目录下的 init.d 文件夹下，新建一个 init.gradle 文件，里面填写以下配置。

```
allprojects
    repositories{
        maven { url 'file:///D:\apache\maven-repository'}
        mavenLocal()
        maven { name "Alibaba" ; url "https://maven.aliyun.com/repository/public"}
        maven { name "Bstek";    url "http://nexus.bsdn.org/content/groups/public/"}
        mavenCentral()
    }

    buildscript { 
        repositories{
            maven { name "Alibaba" ; url 'https://maven.aliyun.com/repository/public'}
            maven { name "Bstek" ; url 'http://nexus.bsdn.org/content/groups/public/'}
            maven { name "M2" ; url  'https://plugins.gradle.org/m2/'}
        }
    }
}
```
repositories 中写的是获取 jar 包的顺序。先是本地的 Maven 仓库路径；接着的 mavenLocal() 是获取 Maven 本地仓库的路径，应该是和第一条一样，但是不冲突；第三条和第四条是从国内和国外的网络上仓库获取；最后的 mavenCentral() 是从Apache提供的中央仓库获取 jar 包。

**Gradle全局配置**

（类似maven默认的 .m2文件。）

在 ${USER_HOME}/.gradle/ 目录下创建 init.gradle 文件，添加以下内容：

```
allprojects {
    repositories {
        def ALIYUN_REPOSITORY_URL = 'https://maven.aliyun.com/repository/public'
        all { ArtifactRepository repo ->
            if(repo instanceof MavenArtifactRepository){
                def url = repo.url.toString()
                if (url.startsWith('https://repo1.maven.org/maven2')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_REPOSITORY_URL."
                    remove repo
                }
            }
        }
        maven { url ALIYUN_REPOSITORY_URL }
    }
}
```

**单项目配置**

修改项目的 build.gradle 文件，添加以下内容：

```
buildscript {
    repositories {
        maven { url 'https://maven.aliyun.com/repository/public' }
    }
}

allprojects {
    repositories {
        maven { url 'https://maven.aliyun.com/repository/public' }
    }
}
```

也可以直接添加在 repositories 内：

```
repositories {
    maven { url 'https://maven.aliyun.com/repository/public' }
    mavenCentral()
}

```

IDEA配置Gradle

全局配置Gradle仓库：File --> New Projects Settings --> Settings for New Projects




hosts
----------------------------------
C:\Windows\System32\drivers\etc


联想壁纸位置
----------------------------------

**联想电脑管家**


```text
C:\ProgramData\Lenovo\devicecenter\LockScreen\cache

C:\Program Files (x86)\Lenovo\LockScreen\Modules\LockScreen
```

**联想锁屏**
```
C:\ProgramData\lLockscreen
```



Github
----------------------------------

```text
https://github.com.ipaddress.com/
```


https://ftools.cc/github




IDEA 插件官网
----------------------------------
https://plugins.jetbrains.com/idea
常用插件

- Jrebel --热部署

- GenerateAllSetter --对象赋值插件

- generateO2O -- 对象快速赋值

- Free-idea-mybatis --mapper-xml

- intellij-mybaitslog --开源

- Json Parser -- json解析

- Mybatis-log-plugin --mybatis 日志

- FindBugs -- 找bug

- CodeGlance -- 编辑区缩略图插件

- GrepConsole -- 控制台速查

- Thief-Book -- 看书

- TranslationPlugin -- 翻译

- IdeaJad --反编译

- SVNLable -- SVN提交人和时间显示

- background Image Plus + 壁纸

- cota AI


JRebel
--------------------------------------------------

生成 GUID 的网址
https://www.guidgen.com/

用这个网址 + 生成的 GUID 激活
https://jrebel.qekang.com/

例如:
https://jrebel.qekang.com/cb2546bb-9d43-4115-bf4b-10539349efed


设置离线模式 来防止失效
File -> Settings -> JRebel -> [Work offline]按钮


redis 下载

http://download.redis.io/releases/

redis 6  要求gcc的版本高于5

JPS
------------------
命令用法: 
```bash
jps [options] [hostid]
```
参数说明：

- options:命令选项，用来对输出格式进行控制

- hostid:指定特定主机，可以是ip地址和域名, 也可以指定具体协议，端口。

```
[protocol:][[//]hostname][:port][/servername]
```

功能描述: jps是用于查看有权访问的hotspot虚拟机的进程. 当未指定hostid时，默认查看本机jvm进程，否者查看指定的hostid机器上的jvm进程，此时hostid所指机器必须开启jstatd服务。 jps可以列出jvm进程lvmid，主类类名，main函数参数, jvm参数，jar名称等信息。

options 命令选项及功能:

没添加option的时候，默认列出VM标示符号和简单的class或jar名称.如下:

- `-p` : 仅仅显示VM 标示，不显示jar,class, main参数等信息.

- `-m` : 输出主函数传入的参数. 下的hello 就是在执行程序时从命令行输入的参数

- `-l` : 输出应用程序主类完整package名称或jar完整名称.

- `-v` : 列出jvm参数, -Xms20m -Xmx50m是启动程序指定的jvm参数

- `-V` : 输出通过.hotsportrc或-XX:Flags=<filename>指定的jvm参数

- `-Joption` : 传递参数到javac 调用的java lancher.


Springboot 停止
-------------------
Spring Boot提供了2种优雅关闭进程的方式：
    - 基于管理端口关闭进程
    - 基于系统服务方式关闭进程


### 基于端口

基于管理端口方式实现进程关闭实际上是模块spring-boot-actuator提供的功能。

添加依赖

```XML
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

添加配置

```yaml
management:
  server:
    address: 127.0.0.1
    port: 8000
  endpoint:
    shutdown:
      enabled: true
  endpoints:
    web:
      base-path: /ops
      path-mapping:
        - shutdown: shutdown
      exposure:
        include: info,health,shutdown
```

启动Spring Boot应用，通过调用`POST http://localhost:8000/ops/shutdown`即可关闭进程。

```bash
curl -X POST http://127.0.0.1:8000/ops/shutdown --connect-timeout 3 --max-time 5
```

一般整合shell 脚本, pname 指进程名称。

```bash
#!/bin/bash
# 先通过管理端口关闭进程
curl -X POST http://127.0.0.1:8000/ops/shutdown --connect-timeout 3 --max-time 5

# 再次通过名称检查进程是否被成功停止
count=`ps -ef |grep pname |grep -v "grep" |wc -l`
if [ $count -gt 0 ]; then
    if [ -f "$pid_file" ]; then
        # 如果存在进程ID文件，则读取进程ID使用信号量通知方式关闭进程
        pid=`cat $pid_file`
        kill -15 $pid
    else
        # 通过名称方式查找到进程ID，使用信号量通知方式关闭进程
        pid=`ps -ef |grep pname |grep -v "grep"| awk '{print $2}'`
        kill -15 $pid
    fi
fi
```

完整版：

```bash
#!/bin/bash
#这里可替换为你自己的执行程序，其他代码无需更改
APP_NAME=springboot-demo-1.0.0.jar
  
#使用说明，用来提示输入参数
usage() {
    echo "Usage: sh 脚本名.sh [start|stop|restart|status]"
    exit 1
}
  
#检查程序是否在运行
is_exist(){
  pid=`ps -ef|grep $APP_NAME|grep -v grep|awk '{print $2}' `
  #如果不存在返回1，存在返回0    
  if [ -z "${pid}" ]; then
   return 1
  else
    return 0
  fi
}
  
#启动方法
start(){
  is_exist
  if [ $? -eq "0" ]; then
    echo "${APP_NAME} is already running. pid=${pid} ."
  else
    nohup java -jar /mnt/ssd1/project/websocket/$APP_NAME > /mnt/ssd1/project/websocket/websocketserverlog.file 2>&1 &
    echo "${APP_NAME} start success"
  fi
}
  
#停止方法
stop(){
  is_exist
  if [ $? -eq "0" ]; then
    kill -9 $pid
  else
    echo "${APP_NAME} is not running"
  fi 
}
  
#输出运行状态
status(){
  is_exist
  if [ $? -eq "0" ]; then
    echo "${APP_NAME} is running. Pid is ${pid}"
  else
    echo "${APP_NAME} is NOT running."
  fi
}
  
#重启
restart(){
  stop
  start
}
  
#根据输入参数，选择执行对应方法，不输入则执行使用说明
case "$1" in
  "start")
    start
    ;;
  "stop")
    stop
    ;;
  "status")
    status
    ;;
  "restart")
    restart
    ;;
  *)
    usage
    ;;
esac
```


### 通过系统服务方式停止进程

Spring Boot支持直接将打包好的可执行jar包以系统服务方式运行，具体实现方式如下所述。

首先，将应用打包为完全可执行的jar包。

Maven打包配配置

```XML
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <!-- 这个配置非常重要，使打包好的jar包具备可执行权限-->
        <executable>true</executable>
    </configuration>
</plugin>
```
Gradle打包配置
```
bootJar {
    launchScript()
}
```

其次，将打包好的应用jar包添加为系统服务（在ubuntu18.04 LTS上实现，基于systemd）

1.假设将Spring Boot应用安装到 `/var/companyApp` 目录下：将上述打包好的jar包拷贝到`/var/companyApp`（目录不存在，手动创建）

2.在 `/etc/systemd/system` 下添加指定名称的系统服务： `myapp.service`，内容如下：

```
[Unit]
Description=myapp
After=syslog.target

[Service]
User=root  ## 注意：这里配置的是将来启动该服务的Linux系统用户名，影响权限
ExecStart=/var/companyApp/myapp.jar
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```

3.启动服务

```bash
$ sudo systemctl enable myapp.service
$ sudo systemctl start myapp.service
```
如果需要查看应用启动日志，请执行：$ journalctl -f。



## druid 加密

bpjydata 是数据库密码

```bash
cd d:\apache\maven-repository

cd com\alibaba\druid\1.0.31

java -cp druid-1.0.31.jar com.alibaba.druid.filter.config.ConfigTools  bpjydata

privateKey:MIIBVQIBADANBgkqhkiG9w0BAQEFAASCAT8wggE7AgEAAkEAlXK6uwv0SHObw6saUGZDT93uOCL9MJ+ZQerBIFC3yeJfeCaCXxxR28Im4jwzJtSeWBCTtbqOMkyuAYyJVzFsIwIDAQABAkAot9monNkx5E3MQhIpVbOBTzZYlS/mz
5UyIIP+CgAJQPmM1Ns+woUc8+qeQiYlOjTc+suNBTDPmWy5E+xzjAIBAiEAzS3lTTdHxKcDy8iW4A8SFBmBY08WuzmzzqDT3o4+VFUCIQC6dwI9IeXuBDZysjyox3fX56AKqzeVMkHBwCN74dT2lwIhAMwFWiBY2r1Z0bV+JUBw2+ounnEwgIr1S
q0pUOPZj3LtAiB6GG013FlzhfylE8KWfa4iiL+J3N0Ta4oVNRvHBXPuVwIhAIi1RTfqJMIVkvlaE8JWmxDXfQmvYN2iNQhptpyqxE+K

publicKey:MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAJVyursL9Ehzm8OrGlBmQ0/d7jgi/TCfmUHqwSBQt8niX3gmgl8cUdvCJuI8MybUnlgQk7W6jjJMrgGMiVcxbCMCAwEAAQ==

password:QO3SkdCiTfS263n92Zf8s+DLsmrx7TTCSb/7ZVIphOOoEhFwPfIFvhv0UHdlGq196PBJsooPDjygvbPz9imowQ==

```

对应的配置：

```
spring:
  main:
    allow-bean-definition-overriding: true
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: oracle.jdbc.driver.OracleDriver
    #开发环境
    url: jdbc:oracle:thin:@192.168.10.118:1521/orcl
    username: bpjydata
    password: bpjydata
    druid: # 数据源专有配置
      initialSize: 5
      minIdle: 5
      maxActive: 20
      maxWait: 60000
      timeBetweenEvictionRunsMillis: 60000
      minEvictableIdleTimeMillis: 300000
      validationQuery: SELECT 1 FROM DUAL
      testWhileIdle: true
      testOnBorrow: false
      testOnReturn: false
      poolPreparedStatements: true

      #配置监控统计拦截的filters，stat:监控统计、log4j：日志记录、wall：防御sql注入
      #如果允许时报错  java.lang.ClassNotFoundException: org.apache.log4j.Priority
      #则导入 log4j 依赖即可，Maven 地址：https://mvnrepository.com/artifact/log4j/log4j
      filters: stat,wall,log4j
      maxPoolPreparedStatementPerConnectionSize: 20
      decrypt: #自定义的配置文件 druid 解密密码使用的公钥
        publickey: "MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAOH4lZpPQRZQMCdb2V618LI6vfTa0tuve1nbiEJEZA6B2kHISbGd8SQvKP9BhSFaaYBMngRTFHVx+ugr2vhhnS8CAwEAAQ=="
        password: "FMibYKX1bTy7GHyYBNpAWVuV+udlXGvGLtWZnsL4ngHuH5cEpg+abZ1JX/Az7gy/rLmsyWC+Ua3L1GWIxrC/0g=="
      connectionProperties: druid.log.stmt.executableSql=true;druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000;config.decrypt=false;config.decrypt.key=${spring.datasource.druid.decrypt.publickey} # 通过connectProperties属性来打开mergeSql功能；慢SQL记录
      useGlobalDataSourceStat: false # 合并多个DruidDataSource的监控数据
```



MAVEN
-------------------

### 常用命令

#### 1、查看依赖关系

```bash
# 查看依赖包关系树结构
mvn  dependency:tree

# 查看依赖包关系树结构 输出到文件
mvn  dependency:tree > tree.txt
```

#### 2、添加本地jar包依赖以及打包方法

**将jar包安装到本地仓库**

1）安装到本地仓库，执行以下命令（其中的-Dfile/-DgroupId/-DartifactId/-Dversion项可以根据pom文件内容填写）：

```bash
mvn install:install-file -Dfile=xxxxx.jar -DgroupId=xxx.xxx.xxx -DartifactId=xxxxx -Dversion=1.0.0 -Dpackaging=jar
```

示例：

```bash
mvn install:install-file -Dfile=yop-java-sdk-3.3.1-jdk18.jar -DgroupId=com.yeepay.yop.sdk -DartifactId=yop-java-sdk -Dversion=3.3.1-jdk18 -Dpackaging=jar
```

（2）安装之后可以在本地仓库中找到对应的jar包。然后将对应的依赖信息插入到工程的pom文件即可：

```XML
<dependency>
    <groupId>xxx.xxx.xxx</groupId>
    <artifactId>xxxxx</artifactId>
    <version>1.0.0</version>
</dependency>
```

（3）利用 maven-install-plugin 配置install-file 进行安装

版本可以根据实际情况自行适配更改。

```xml
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-install-plugin</artifactId>
                <version>2.5.2</version>
                <executions>
                    <execution>
                        <id>jms-1.0</id>
                        <!-- 标识在clean阶段安装到本地仓库 -->
                        <phase>clean</phase>
                        <configuration>
                            <file>${project.basedir}/lib/yop-java-sdk-3.3.1-jdk18.jar</file>
                            <repositoryLayout>default</repositoryLayout>
                            <groupId>com.yeepay.yop.sdk</groupId>
                            <artifactId>yop-java-sdk</artifactId>
                            <version>3.3.1-jdk18</version>
                            <packaging>jar</packaging>
                            <generatePom>true</generatePom>
                        </configuration>
                        <goals>
                            <goal>install-file</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```

执行 ** mvn clean ** 安装到本地仓库。


#### 3、dependency中指定scope="system"和本地jar包路径

这种方法适用于其他方式导出的jar包，jar包中不含有pom信息，从而无法安装进本地仓库的情况。做法是：先配置本地jar包依赖，然后在build时将设置将jar包导出，同时配置manifest。

（1）配置本地jar包依赖（systemPath指向本地jar包路径）：

```xml
<dependency>
     <groupId>com.yeepay.yop.sdk</groupId>
     <artifactId>yop-java-sdk</artifactId>
     <version>3.3.1-jdk18</version>
     <scope>system</scope>
     <systemPath>${project.basedir}/lib/yop-java-sdk-3.3.1-jdk18.jar</systemPath>
</dependency>
```

（2）在<build>的spring-boot-maven-plugin中设置将本地jar包导出到项目最终的依赖库中：

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <includeSystemScope>true</includeSystemScope>
    </configuration>
</plugin>
```
（3）如果项目使用maven-jar-plugin插件打包的话，还需要在manifectEntries中添加对应的jar包信息；否则虽然jar包导出了，但是项目生成的MANIFEST.MF文件中没有对应的依赖信息，也会导致运行时找不到对应的class。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <configuration>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <classpathPrefix>lib/</classpathPrefix>
                <mainClass>XXXX</mainClass>
            </manifest>
            <manifestEntries>
                <Class-Path>./lib/xxxxx.jar</Class-Path>
            </manifestEntries>
        </archive>
        <outputDirectory>
            ${project.build.directory}/XXXXX
        </outputDirectory>
    </configuration>
</plugin>
```
(4)项目完整的<build>配置（该配置可以将最终生成的jar包和依赖库、配置文件分开）。

```xml
<build>
    <finalName>XXXXX</finalName>
    <sourceDirectory>src/main/java</sourceDirectory>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <targetPath>${project.build.directory}/XXXXX</targetPath>
            <excludes>
                <exclude>**/*.java</exclude>
            </excludes>
        </resource>
    </resources>

    <testSourceDirectory>src/test/java</testSourceDirectory>
    <testResources>
        <testResource>
            <directory>src/test/resources</directory>
            <filtering>true</filtering>
            <excludes>
                <exclude>**/*.java</exclude>
            </excludes>
        </testResource>
    </testResources>

    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <includeSystemScope>true</includeSystemScope>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <configuration>
                <skipTests>true</skipTests>
            </configuration>
        </plugin>

        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-dependency-plugin</artifactId>
            <executions>
                <execution>
                    <id>copy-dependencies</id>
                    <phase>package</phase>
                    <goals>
                        <goal>copy-dependencies</goal>
                    </goals>
                    <configuration>
                        <outputDirectory>
                            ${project.build.directory}XXXXX/lib
                        </outputDirectory>
                    </configuration>
                </execution>
            </executions>
        </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <classpathPrefix>lib/</classpathPrefix>
                            <mainClass>xxx.xxx.XXXXX</mainClass>
                        </manifest>
                        <manifestEntries>
                            <Class-Path>./ lib/xxxxx.jar</Class-Path>
                        </manifestEntries>
                    </archive>
                    <outputDirectory>
                        ${project.build.directory}/XXXXX
                    </outputDirectory>
                </configuration>
            </plugin>
    </plugins>
</build>
```


### maven骨架原型

maven提供的41个骨架原型分别是：

1: appfuse-basic-jsf (创建一个基于Hibernate，Spring和JSF的Web应用程序的原型) 

2: appfuse-basic-spring(创建一个基于Hibernate，Spring和Spring MVC的Web应用程序的原型) 

3: appfuse-basic-struts(创建一个基于Hibernate，Spring和Struts 2的Web应用程序的原型) 

4: appfuse-basic-tapestry(创建一个基于Hibernate，Spring 和 Tapestry 4的Web应用程序的原型) 

5: appfuse-core(创建一个基于Hibernate，Spring 和 XFire的jar应用程序的原型) 

6: appfuse-modular-jsf(创建一个基于Hibernate，Spring和JSF的模块化应用原型) 

7: appfuse-modular-spring(创建一个基于Hibernate, Spring 和 Spring MVC 的模块化应用原型) 

8: appfuse-modular-struts(创建一个基于Hibernate, Spring 和 Struts 2 的模块化应用原型) 

9: appfuse-modular-tapestry (创建一个基于 Hibernate, Spring 和 Tapestry 4 的模块化应用原型) 

10: maven-archetype-j2ee-simple(一个简单的J2EE的Java应用程序) 

11: maven-archetype-marmalade-mojo(一个Maven的 插件开发项目 using marmalade) 

12: maven-archetype-mojo(一个Maven的Java插件开发项目) 

13: maven-archetype-portlet(一个简单的portlet应用程序) 

14: maven-archetype-profiles() 

15:maven-archetype-quickstart() 

16: maven-archetype-site-simple(简单的网站生成项目) 

17: maven-archetype-site(更复杂的网站项目) 

18:maven-archetype-webapp(一个简单的Java Web应用程序) 

19: jini-service-archetype(Archetype for Jini service project creation) 

20: softeu-archetype-seam(JSF+Facelets+Seam Archetype) 

21: softeu-archetype-seam-simple(JSF+Facelets+Seam (无残留) 原型) 

22: softeu-archetype-jsf(JSF+Facelets 原型) 

23: jpa-maven-archetype(JPA 应用程序) 

24: spring-osgi-bundle-archetype(Spring-OSGi 原型) 

25: confluence-plugin-archetype(Atlassian 聚合插件原型) 

26: jira-plugin-archetype(Atlassian JIRA 插件原型) 

27: maven-archetype-har(Hibernate 存档) 

28: maven-archetype-sar(JBoss 服务存档) 

29: wicket-archetype-quickstart(一个简单的Apache Wicket的项目) 

30: scala-archetype-simple(一个简单的scala的项目) 

31: lift-archetype-blank(一个 blank/empty liftweb 项目) 

32: lift-archetype-basic(基本（liftweb）项目) 

33: cocoon-22-archetype-block-plain([http://cocoapacorg2/maven-plugins/]) 

34: cocoon-22-archetype-block([http://cocoapacorg2/maven-plugins/]) 

35:cocoon-22-archetype-webapp([http://cocoapacorg2/maven-plugins/]) 

36: myfaces-archetype-helloworld(使用MyFaces的一个简单的原型) 

37: myfaces-archetype-helloworld-facelets(一个使用MyFaces和Facelets的简单原型) 

38: myfaces-archetype-trinidad(一个使用MyFaces和Trinidad的简单原型) 

39: myfaces-archetype-jsfcomponents(一种使用MyFaces创建定制JSF组件的简单的原型) 

40: gmaven-archetype-basic(Groovy的基本原型) 

41: gmaven-archetype-mojo(Groovy mojo 原型)


