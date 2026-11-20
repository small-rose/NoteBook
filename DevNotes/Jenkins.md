---
layout: default
title: Jenkins 
parent: DevNotes
nav_order: 33
---


# Idea
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
   {:toc}

---

> 之前公司的 Jenkins 一直都是我维护的，使用上没有任何难度。但是自己也试着从零搭建试试。

## Jenkins 到底是什么

Jenkins 是一款开源、免费、插件化的持续集成 / 持续交付（CI/CD）自动化工具，基于 Java 开发，是目前 DevOps 领域最主流的自动化流水线工具之一。

Jenkins 就是一个 24 小时待命的自动化运维机器人，你只需要提前给它定义好一套执行规则，它就能在指定时机（比如代码提交后, 推送特定分支后）执行回调任务，自动完成从代码拉取到服务上线的全流程操作，全程无需人工干预，极大提升开发效率，降低人为操作失误。

## Jenkins 核心能力

- 持续集成（CI）：代码提交后自动拉取、编译、单元测试、代码扫描，提前暴露问题，保障主干代码稳定性，防止本地没问题，更换环境就报错问题。
- 持续部署（CD）：自动完成制品打包、镜像构建、多环境发布，实现代码提交到服务上线的全流程自动化，告别重复的手动部署。
- 插件化生态：官方提供数千款插件，完美兼容 Git、Docker、K8s、各类代码仓库、云平台，适配所有主流开发语言和技术栈
- 流水线即代码：通过 Jenkinsfile 定义流水线，和业务代码一起存入 Git 仓库，可追溯、可复用、可团队协作。

## Jenkins 部署

1、直接在Linux上部署。：适合练手。

- 安装基础环境：jdk、git/svn、maven/gradle
- 官网下载中间件：tomcat或类似
- 官网下载 [Jenkins.war](https://www.jenkins.io/download/) LTS长期支持稳定版本。

2、使用docker部署 Jenkins： 开箱即用、环境隔离、快速迁移、资源可控、维护简单。

docker 安装详见 [docker相关文章](https://docs.zhangxiaocai.cn/Container/Docker.html)

Jenkins 端口与防火墙配置:

Jenkins 默认使用 8080 端口提供 Web 访问，50000 端口用于 Agent 节点通信，我们需要开放这两个端口：

```bash
# CentOS系统（firewalld防火墙）
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --zone=public --add-port=50000/tcp --permanent
firewall-cmd --reload
 
# Ubuntu系统（ufw防火墙）
ufw allow 8080/tcp
ufw allow 50000/tcp
ufw reload
```

### Docker 部署 Jenkins 全流程

> 前置条件： docker 相关环境、源都准备完成，如果有web界面可以安装web界面。

1、拉取 Jenkins 官方镜像

```bash
# 拉取Jenkins LTS JDK11稳定版镜像
docker pull jenkins/jenkins:lts-jdk11
 
# 验证镜像是否拉取成功
docker images | grep jenkins
```

>  我们选择LTS 长期支持版，这个版本是官方维护的稳定版，bug 少、兼容性好，搭配 JDK11（Jenkins 2.361 + 版本强制要求 JDK11 以上）：
> 
> 避坑：不要使用旧版本的 jenkinsci/blueocean 镜像，这个镜像已经停止维护，官方推荐使用 jenkins/jenkins:lts-jdk11 镜像。

2、数据持久化存储

Jenkins 的所有配置、插件、构建记录、任务数据都存在 `/var/jenkins_home`目录下，如果不做持久化，容器删除后所有数据都会丢失，必须创建数据卷做持久化。

A/B两种方式选择其一即可。

A、使用宿主机目录挂载: （目录可自行规划）

```bash
# 创建宿主机挂载目录
mkdir -p /opt/jenkins_home
 
# 给目录赋权，避免容器内权限不足无法写入
chmod 777 /opt/jenkins_home
```

B、使用docker 官方数据卷
```bash
# 创建Jenkins专用数据卷
docker volume create jenkins_data
 
# 查看数据卷信息，确认创建成功
docker volume inspect jenkins_data
```

3、启动 Jenkins 容器

> A和B两种方式挂着目录，启动命令稍有差异。

A方式挂着目录

```bash
    docker run -d \
      --name jenkins \
      --restart=unless-stopped \
      --memory=4g \
      --cpus=2 \
      -p 8080:8080 \
      -p 50000:50000 \
      -v /opt/jenkins_home:/var/jenkins_home \
      -v /var/run/docker.sock:/var/run/docker.sock \
      -v /usr/bin/docker:/usr/bin/docker \
      -v /etc/localtime:/etc/localtime:ro \
      -e TZ="Asia/Shanghai" \
      -e JAVA_OPTS="-Xms1g -Xmx2g -Duser.timezone=Asia/Shanghai" \
      --user root \
      jenkins/jenkins:lts-jdk11
```

B方式创建数据卷：

```bash
docker run -d \
  --name jenkins \
  --restart=unless-stopped \
  --memory=4g \
  --cpus=2 \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /usr/bin/docker:/usr/bin/docker \
  -v /etc/localtime:/etc/localtime:ro \
  -e TZ="Asia/Shanghai" \
  -e JAVA_OPTS="-Xms1g -Xmx2g -Duser.timezone=Asia/Shanghai" \
  --user root \
  jenkins/jenkins:lts-jdk11
```

- --memory=4g ： 限制容器最大使用 4G 内存，避免 Jenkins 占用过多宿主机内存，导致宿主机卡顿崩溃；
- --cpus=2 ：限制容器最多使用 2 核 CPU，控制资源占用，保障宿主机其他服务正常运行；
- -p 8080:8080 ：宿主机 8080 端口映射到容器 8080 端口，用于浏览器访问 Jenkins Web 管理界面；
- -p 50000:50000 ：宿主机 50000 端口映射到容器 50000 端口，用于 Jenkins 主节点和 Agent 构建节点通信；
- -v /var/run/docker.sock:/var/run/docker.sock：挂载宿主机 Docker 套接字，核心配合，可以让 Jenkins 容器内可以直接调用宿主机的 Docker 服务，实现镜像构建、推送等核心操作；
-  -v /usr/bin/docker:/usr/bin/docker ：挂载宿主机 Docker 命令	让容器内可以直接使用 docker 命令，和宿主机 Docker 版本完全一致，避免版本兼容问题；
-  -v /etc/localtime:/etc/localtime:ro	同步宿主机时区	只读挂载，避免容器修改宿主机时区，解决 Jenkins 日志时间和本地时间不一致的问题；
-  -e TZ="Asia/Shanghai"	设置容器时区为上海	同上，双重保障时区正确，避免日志时间错乱；
-  -e JAVA_OPTS="-Xms1g -Xmx2g"	设置 Java 虚拟机参数	优化 Jenkins 运行内存，初始堆内存 1G，最大堆内存 2G，避免内存溢出，大幅提升运行稳定性；
-  --user root	以 root 用户运行容器，避免容器内权限不足，无法操作挂载的 Docker 套接字和数据卷，新手推荐使用，生产环境可进一步优化权限配置。


4、验证容器启动状态

执行完启动命令后，我们需要确认 Jenkins 容器是否正常运行：

```bash
# 查看运行中的容器，能看到jenkins容器说明启动成功
docker ps | grep jenkins

# 查看容器启动日志，实时排查启动问题
docker logs jenkins -f
```

如果安装docker 管理控制台，也可以从页面点点日志控制台查看。

日志中出现Jenkins is fully up and running这句话时，说明 Jenkins 已经完全启动成功

## Jenkins 初始化全流程

1、访问 Jenkins Web 界面

如我的地址是：http://192.168.1.72:8080。

正常访问的话，会进入解锁 Jenkins的页面，这是 Jenkins 的安全机制，防止未经授权的访问。

2、获取初始管理员密码

读取容器密码文件
```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

日志读取
```basj
docker logs jenkins | grep "Initial Admin Password"
```
出的结果中，Initial Admin Password后面的字符串就是初始密码。

3、安装推荐插件

使用密码登录后，进入安装插件页面。

- 安装推荐的插件：Jenkins 会自动安装最常用的基础插件，适合新手，强烈推荐选择这个
- 选择要安装的插件：自定义安装插件，适合有经验的用户

> 如果出现个别插件安装失败，不用慌，点击【重试】即可，或者直接点击【继续】，后续可以在插件管理中手动安装。

4、创建管理员账号

插件安装完成后，会进入创建管理员用户的页面，这里需要填写：

- 用户名：比如 admin（建议简单好记） 
- 密码：设置一个强密码，务必妥善保存
- 全名：你的昵称 / 姓名
- 邮箱地址：用于接收 Jenkins 的通知告警

填写完成后，点击【保存并继续】。

5 配置 Jenkins 实例地址

接下来会进入实例配置页面，这里需要填写 Jenkins 的访问地址，默认会自动填充你当前访问的地址，比如 `http://192.168.1.72:8080`

直接保持默认，点击【保存并完成】。


最后会出现【Jenkins 已就绪！】的页面，点击【开始使用 Jenkins】，就会进入 Jenkins 的主界面了。

恭喜你！到这里，你已经成功完成了 Jenkins 的 Docker 部署和初始化，接下来我们就进行核心配置，让 Jenkins 真正能用起来。


## Jenkins 核心配置：打造可用的自动化环境

初始化完成后，我们还需要进行几个核心配置，才能让 Jenkins 支持代码拉取、镜像构建、自动化部署等操作。

1、安装必备插件

Jenkins 的核心能力都来自插件，推荐的插件安装完成后，我们还需要补充安装几个 CI/CD 流水线必备的插件，步骤如下：

- 进入 Jenkins 主界面，点击左侧菜单栏的【系统管理】
- 找到【插件管理】，点击进入
- 切换到【可选插件】标签页，在搜索框中搜索以下插件，勾选后点击【直接安装】
- 安装完成后，勾选【安装完成后重启 Jenkins】，等待重启生效

必备插件清单：

|插件名称	|核心作用	|必装原因|
|-----------|---------|---------|
|Git Plugin	|拉取 Git 仓库的代码	|实现从 GitLab/GitHub/Gitee 拉取业务代码，是 CI 的基础|
|Docker Pipeline	|提供 Docker 流水线支持	|让 Jenkins 流水线可以执行 Docker 构建、推送等命令|
|Kubernetes CLI Plugin	|K8s 集群操作支持	|要部署到 K8s 集群，必须装这个插件|
|Generic Webhook Trigger	|通用 Webhook 触发器	|实现代码提交后，Git 仓库自动触发 Jenkins 流水线|
|Credentials Binding Plugin	凭|证绑定	|加密管理 Git 账号、镜像仓库密码等敏感信息，避免明文泄露|
|Maven Integration	|Maven 项目构建支持	|Java 开发必备，用于编译打包 Java 项目|
|NodeJS Plugin	|NodeJS 项目构建支持	|前端开发必备，用于打包前端项目|

2、全局工具配置

配置 Jenkins 构建 项目 时用到的工具，比如 JDK、Git、Maven、NodeJS 等，步骤如下：

 - 进入 Jenkins 主界面，点击【系统管理】
 - 找到【全局工具配置】，点击进入

**配置 Git**

Git 是拉取代码的必备工具，Jenkins 容器内已经自带了 Git，我们只需要配置即可：

- 找到【Git】配置项，点击【新增 Git】
- 名称填写Git，Path to Git executable 填写git（容器内自带的 Git 命令路径）
- 点击【测试配置】，能正常输出版本号说明配置成功

**配置 JDK**

Java 项目编译需要 JDK，我们可以让 Jenkins 自动安装：

- 找到【JDK】配置项，点击【新增 JDK】 
- 名称填写JDK11，勾选【自动安装】
- 版本选择jdk-11.0.xx（最新的 JDK11 版本）
- 勾选接受许可协议

配置 Maven/NodeJS

根据你的开发语言配置对应的构建工具，和 JDK 配置逻辑一致，勾选自动安装，选择对应的版本即可。

所有工具配置完成后，点击页面底部的【保存】，配置就生效。

3、全局凭证管理

核心原则：所有敏感信息绝对不能明文写在流水线中，必须统一存入 Jenkins 的凭证系统中，加密存储。

比如 Git 仓库的账号密码、镜像仓库的登录信息、K8s 集群的 kubeconfig 等，都要存在凭证里，步骤如下：

- （1）进入 Jenkins 主界面，点击【系统管理】
- （2）找到【凭证】，点击进入
- （3）点击【系统】→【全局凭据】→【添加凭据】
- （4）选择对应的凭据类型，填写信息，设置一个好记的凭据 ID（后续流水线会用到），点击【确定】

常用凭据类型与配置说明

|凭据类型|适用场景|配置要点|
|-----|-----|--------|
|用户名密码	|Git 仓库拉取、镜像仓库登录	|用户名填写你的 Git / 镜像仓库账号，密码填写对应密码，凭据 ID 建议设置为git-auth/registry-auth，方便后续使用|
|Secret text |Webhook 触发令牌、API 密钥	|填写自定义的随机字符串，用于 Webhook 触发验证，凭据 ID 设置为webhook-token|
|Kubeconfig	|K8s 集群访问	|填写你的 K8s 集群的 kubeconfig 文件内容，凭据 ID 设置为k8s-kubeconfig|


## 自动化 CI/CD 流水线

### 目标规划

> 开发提交代码到 Git 仓库 → 自动触发 Jenkins 流水线 → 拉取代码 → 编译打包 → 单元测试 → 构建 Docker 镜像 → 推送镜像仓库 → 自动部署到 K8s 集群

流水线整体架构：

```text
开发提交或合并代码 → Git仓库触发Webhook → Jenkins接收请求 → 启动流水线 →

流水线拉取代码 → 编译&单元测试 → 构建Docker镜像 → 推送镜像仓库 → 部署到K8s集群 →

执行完成，发送通知（成功/失败）
```

### 项目准备

业务项目根目录必须包含以下 3 个核心文件，和代码一起提交到 Git 仓库：

1、Dockerfile：镜像构建描述文件

```bash
# 基础镜像，使用JRE11，体积更小
FROM openjdk:11-jre-slim
     
# 设置工作目录
WORKDIR /app
     
# 复制Maven编译后的jar包到容器内
COPY target/*.jar app.jar
     
# 暴露服务端口
EXPOSE 8080
     
# 容器启动命令
ENTRYPOINT ["java", "-jar", "app.jar"]
```

2、K8s 部署清单：k8s/deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-app
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
    spec:
      containers:
      - name: demo-app
        # 镜像地址变量，流水线动态替换
        image: ${REGISTRY_URL}/demo/demo-app:${IMAGE_TAG}
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: 100m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
---
apiVersion: v1
kind: Service
metadata:
  name: demo-app
  namespace: default
spec:
  selector:
    app: demo-app
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

3、Jenkinsfile：流水线核心定义文件

```bash
pipeline {
    // 运行环境，这里用any，直接在Jenkins主节点运行
    agent any
    
    // 全局环境变量，替换为你自己的实际地址
    environment {
        // Git仓库地址
        GIT_URL = "http://gitlab.zhangxiaocai.cn/my-group/demo-app.git"
        // 镜像仓库地址（Harbor/Docker Hub）
        REGISTRY_URL = "harbor.example.com"
        // 镜像名称
        IMAGE_NAME = "demo/demo-app"
        // 镜像标签，用Git提交短哈希，唯一可追溯，避免latest标签的坑
        IMAGE_TAG = "${sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()}"
        // 完整镜像地址
        FULL_IMAGE = "${REGISTRY_URL}/${IMAGE_NAME}:${IMAGE_TAG}"
    }
    
    // Webhook触发器配置，实现代码提交自动触发
    triggers {
        GenericTrigger(
            token: 'webhook-token', // 和之前凭证里的token一致
            whitelist: ['192.168.1.0/24'], // 可选，限制Git仓库IP，提升安全性
            causeString: '代码提交触发自动构建',
            printContributedVariables: true,
            printPostContent: true
        )
    }
    
    // 流水线核心阶段，按顺序执行
    stages {
        stage('1. 拉取代码') {
            steps {
                echo "===== 开始拉取代码，分支：${env.BRANCH_NAME} ====="
                // 拉取Git代码，使用之前配置的git-auth凭证
                git url: "${GIT_URL}", 
                    branch: "${env.BRANCH_NAME}", 
                    credentialsId: 'git-auth'
                // 打印最新提交记录，方便追溯
                sh 'git log --oneline -1'
            }
        }
        
        stage('2. 代码编译与单元测试') {
            steps {
                echo "===== 开始编译代码，执行单元测试 ====="
                // Java项目用Maven编译，前端项目替换为 npm install && npm run build
                sh 'mvn clean package -Dmaven.test.skip=false'
            }
            post {
                // 单元测试失败，直接终止流水线
                failure {
                    echo "❌ 单元测试不通过，流水线终止"
                    currentBuild.result = 'FAILURE'
                }
            }
        }
        
        stage('3. 构建Docker镜像') {
            steps {
                echo "===== 开始构建Docker镜像：${FULL_IMAGE} ====="
                sh 'docker build -t ${FULL_IMAGE} .'
            }
        }
        
        stage('4. 推送镜像到仓库') {
            steps {
                echo "===== 登录镜像仓库：${REGISTRY_URL} ====="
                // 使用凭证里的镜像仓库账号密码，避免明文
                withCredentials([usernamePassword(credentialsId: 'registry-auth', usernameVariable: 'REGISTRY_USER', passwordVariable: 'REGISTRY_PWD')]) {
                    sh 'docker login ${REGISTRY_URL} -u ${REGISTRY_USER} -p ${REGISTRY_PWD}'
                    echo "===== 推送镜像：${FULL_IMAGE} ====="
                    sh 'docker push ${FULL_IMAGE}'
                    // 登出镜像仓库，清理凭证
                    sh 'docker logout ${REGISTRY_URL}'
                }
            }
        }
        
        stage('5. 部署到K8s集群') {
            steps {
                echo "===== 开始部署到K8s集群，镜像版本：${IMAGE_TAG} ====="
                // 替换部署文件中的镜像变量
                sh 'sed -i "s#\\${REGISTRY_URL}#${REGISTRY_URL}#g" k8s/deployment.yaml'
                sh 'sed -i "s#\\${IMAGE_TAG}#${IMAGE_TAG}#g" k8s/deployment.yaml'
                // 使用kubeconfig凭证，执行K8s部署命令
                withCredentials([kubeconfigFile(credentialsId: 'k8s-kubeconfig', variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f k8s/deployment.yaml'
                    // 等待滚动更新完成，超时5分钟
                    sh 'kubectl rollout status deployment/demo-app -n default --timeout=300s'
                }
                echo "✅ 部署完成，服务已成功更新！"
            }
        }
    }
    
    // 流水线执行后的收尾操作
    post {
        // 执行成功后的操作
        success {
            echo "✅ 全流程自动化流水线执行成功！"
            // 可选：配置钉钉/企业微信/邮件成功通知
        }
        // 执行失败后的操作
        failure {
            echo "❌ 流水线执行失败，请检查构建日志！"
            // 可选：配置失败告警通知
        }
        // 无论成功失败，都执行的操作
        always {
            // 清理本地镜像和构建产物，释放磁盘空间
            sh 'docker rmi ${FULL_IMAGE} || true'
            sh 'mvn clean || true'
        }
    }
}
```

### 创建 Jenkins 流水线任务

- (1)进入 Jenkins 主界面，点击左侧【新建任务】
- (2)输入任务名称，比如demo-app-cicd，选择【流水线】，点击【确定】
- (3)进入任务配置页面，找到【构建触发器】，勾选Generic Webhook Trigger，Token 填写之前设置的webhook-token
- (4)找到【流水线】配置项，【定义】选择Pipeline script from SCM（流水线代码从 Git 仓库获取）
- (5)SCM 选择Git，填入你的代码仓库地址，选择之前配置的git-auth凭证
- (6)分支指定，填写你要触发的分支，比如*/main或者*/master
- (7)脚本路径，填写Jenkinsfile（和项目根目录的文件名完全一致）
- (8)其他保持默认，点击页面底部的【保存】

### 配置 Git 仓库 Webhook

配置 Git 仓库的 Webhook，实现代码提交后自动触发 Jenkins 流水线，以 GitLab 为例，GitHub/Gitee 配置逻辑完全一致：

- (1)进入你的 GitLab 项目页面，点击左侧【设置】→【Webhooks】
- (2)URL 填写：http://<Jenkins服务器IP>:8080/generic-webhook-trigger/invoke?token=webhook-token

  >注意：这里的 token 必须和 Jenkins 流水线里配置的 token 完全一致，GitLab 服务器必须能访问到这个 Jenkins 地址，内网环境需要打通网络

- (3)触发事件，勾选【Push events】，选择要触发的分支（比如 main 分支）
- (4)如果是自签名证书，取消勾选【启用 SSL 验证】
- (5)点击【添加 Webhook】
- (6)测试：点击添加好的 Webhook 右侧的【测试】→【Push events】，如果返回 200 状态码，说明 Webhook 配置成功，Jenkins 会自动触发流水线。


### 流水线测试与验证

1、手动触发测试

进入 Jenkins 流水线任务页面，点击左侧【立即构建】，Jenkins 会启动流水线，你可以在页面上看到每个阶段的执行状态，点击阶段可以查看详细日志，排查问题。

如果所有阶段都显示绿色对勾，说明流水线执行成功，镜像已经推送到仓库，服务已经部署到 K8s 集群。

2、自动触发测试

本地修改业务代码，提交并推送到 Git 仓库的对应分支，此时 GitLab 会触发 Webhook，Jenkins 会自动启动流水线，你可以在 Jenkins 页面看到自动创建的构建任务，全程无需人工干预。

到这里，已经可以成功落地了一套完整的 Jenkins 自动化 CI/CD 流水线，真正实现了代码提交后自动构建部署。

## 常见问题和排查

1、Jenkins 容器启动后很快退出

常见原因：端口被占用、权限不足、内存不足

解决方案：
   - 执行docker logs jenkins查看报错信息
   - 检查 8080 端口是否被占用：netstat -tulpn | grep 8080，如果被占用，修改启动命令的端口映射
   - 检查挂载的目录 / 数据卷权限，确保有写入权限
   - 检查服务器内存是否充足，至少预留 2G 可用内存

2、插件安装失败，速度极慢

常见原因：Jenkins 默认插件源是国外的，网络不通

解决方案：替换为国内清华大学镜像源 

```bash
# 进入Jenkins容器
docker exec -it jenkins bash
 
# 修改插件源配置
sed -i 's#https://updates.jenkins.io/download#https://mirrors.tuna.tsinghua.edu.cn/jenkins#g' /var/jenkins_home/hudson.model.UpdateCenter.xml
sed -i 's#http://www.google.com#https://www.baidu.com#g' /var/jenkins_home/hudson.model.UpdateCenter.xml
 
# 重启Jenkins容器
docker restart jenkins
```

3、流水线中执行 docker 命令报错：permission denied

常见原因：容器内没有权限访问宿主机的 Docker 套接字

解决方案：
- 启动命令中必须添加--user root
- 给宿主机 Docker 套接字赋权：`chmod 666 /var/run/docker.sock`
- 重启 Jenkins 容器：`docker restart jenkins`

4、Webhook 触发不了，Jenkins 没有收到请求

常见原因：网络不通、token 不一致、防火墙 / 安全组未开放端口

解决方案：
- 在 GitLab 服务器上执行curl http://<JenkinsIP>:8080，看是否能正常访问，不通的话检查防火墙和安全组
- 检查 Webhook URL 中的 token 和 Jenkins 中配置的 token 是否完全一致
- 查看 GitLab Webhook 的请求日志，看返回的状态码和报错信息

5、镜像推送失败，提示 denied: requested access to the resource is denied

常见原因：镜像仓库账号密码错误、账号没有对应仓库的读写权限、镜像名称不符合仓库规范

解决方案：
- 检查 Jenkins 凭证中的镜像仓库账号密码是否正确
- 手动在服务器上用该账号登录镜像仓库，看是否能正常推送镜像
- 检查镜像名称是否符合仓库规范，比如 Harbor 必须是仓库地址/项目名/镜像名:标签的格式


## Jenkins 使用最佳实践

1、数据备份与恢复

- 定期备份 Jenkins 数据卷：
  > docker run --rm -v jenkins_data:/data -v /opt/backup:/backup alpine tar -zcvf /backup/jenkins_backup_$(date +%Y%m%d).tar.gz /data
- 恢复备份：解压备份文件到数据卷目录即可

2. 安全加固

- 生产环境尽量不要用--user root运行容器，创建专用用户，分配最小必要权限
- 不要在流水线中明文写任何敏感信息，全部使用凭证管理
- 配置 Jenkins 用户权限，基于 RBAC 给不同用户分配不同权限，开发人员只能看日志，不能修改流水线配置
- 生产环境使用 HTTPS 访问 Jenkins，配置域名和 SSL 证书

3. 流水线优化

- 坚持「流水线即代码」，所有流水线都用 Jenkinsfile 定义，和代码一起存入 Git 仓库
- 镜像标签不要用 latest，用 Git 提交哈希，唯一可追溯，避免滚动更新的坑
- 增加质量门禁，比如集成 SonarQube 代码扫描，单元测试覆盖率不达标直接终止流水线
- 生产环境部署增加人工审批环节，降低发布风险

4. 性能优化

- 限制 Jenkins 容器的资源使用，避免占用过多宿主机资源
- 定期清理构建历史和无用镜像，释放磁盘空间
- 大规模使用时，配置主从架构，用 K8s 动态创建构建 Agent，主节点只做调度，不执行构建任务