---
layout: default
title: Node Env
parent: Node
grand_parent: Front-end Dev
nav_order: 700
---

Here are commonly used JavaScript examples .
{: .fs-6 .fw-300 }


1. TOC
{:toc}


## Node Env

### 1、直接安装

直接Node 官网 [https://nodejs.cn/en/download](https://nodejs.cn/en/download)

根据环境下载对应版本安装

> [node-v22.20.0-win-x64.zip](https://cdn.npmmirror.com/binaries/node/v22.20.0/node-v22.20.0-win-x64.zip)


### 2、使用 nvm 安装

> 可以让多个node版本共存，在多个项目使用不同node版本切换便捷。

#### 安装nvm


> nvm 在github下载 [https://github.com/coreybutler/nvm-windows/releases](https://github.com/coreybutler/nvm-windows/releases)

- nvm-setup.zip  约4.8M,安装版,解压后自动配置环境变量
- nvm-noinstall.zip 约6.1M, 解压到安装目录, 然后需要手动配置环境变量

{: .tips }
> 1. 如果手工安装了node 需要先卸载，删除已有的node还有nvm相关的环境变量
> 2. 安装或解压, 请确保目录路径**无中文**和**无空格**
> 3. 解压解压后,需要配置环境变量,否则无法使用

验证安装

打开 cmd 检查版本执行 `nvm -v` 出现版本号即可, 或者 `nvm`有显示版本或语法提示亦可。
```shell
nvm -v
1.2.2
```

nvm是node.js的版本管理工具，使用nvm安装node，可以实现node版本的快速切换

|常用命令 | 说明 |
|-|-|
|nvm list available	| 显示可以安装的所有node.js的版本 |
|nvm list	| 显示所有已安装的node.js版本 |
|nvm use	| 切换到指定的nodejs版本 |
|nvm install	| 安装指定版本的node.js，例如：nvm install 8.12.0 |
|nvm uninstall	| 卸载指定版本的node.js，例如：nvm uninstall 8.12.0 |
|nvm on	| 启用node.js版本管理 |
|nvm off	| 禁用node.js版本管理(不卸载任何东西) |

#### 使用nvm安装node



查看 node 版本：

```shell
nvm list

nvm ls
```

有的话就列举，版本前带*则表示当前使用版本。 如果没有的话会提示尚未安装node。


#### 安装node

修改国内镜像配置：

```shell
nvm node_mirror https://npmmirror.com/mirrors/node/
nvm npm_mirror https://npmmirror.com/mirrors/npm/
```
或者找到 `D:\Node\nvm\settings.txt` 配置后面的两行镜像地址:

```txt
root: D:\Node\nvm
path: d:\Node\nvm4w\nodejs
node_mirror: https://npmmirror.com/mirrors/node/
npm_mirror: https://npmmirror.com/mirrors/npm/
```

安装对应的node版本

```shell
nvm install 22.20.0
```

等待下载安装完成。

> 目录 D:\Node\nodejs\ 下会生成对应版本的nodejs目录


当 NVM 安装 Node.js 后没有 npm ?
> 通常是因为 NVM 默认安装 Node.js 时并未包含 npm，需要手动补齐，方法是下载对应版本的 npm 包解压到 Node.js 目录下的 node_modules 文件夹中
>
> 或者修改 settings.txt 使用淘宝镜像源，让安装过程能自动下载 npm。


切换版本

```shell
nvm use 22.20.0
```

再次检查可用

```shell
nvm ls
```
结果
```txt
* 22.20.0 (Currently using 64-bit executable)
```

检查一下 npm

```shell
npm -v
```

{: .tips }
> 如果是原始镜像安装 npm 很可能没有 npm, 可以手动选择合和node搭配的版本手工安装，也可以切换过内镜像之后重新执行安装

### 3 NVM：切换node版本后无法使用npm全局包


方法一 查看全局安装路径,并在切换后设置

查看全局安装路径

```shell
npm root -g
```
结果

```txt
d:\Node\nvm4w\nodejs\node_modules
```

即当前nvm全局变量路径，可以通过以下方式设置

```shell
npm config set prefix "d:\Node\nvm4w\nodejs\node_modules\node_global"
npm config set cache "d:\Node\nvm4w\nodejs\node_modules\node_cache"
```
方式二 找当 .npmrc 文件路径，打开 .npmrc 文件进行设置

```shell
npm config get userconfig 
```
C:\Users\YOUR_NAME\.npmrc

追加配置
```text
prefix = d:\Node\nvm4w\nodejs\node_modules\node_global
cache = d:\Node\nvm4w\nodejs\node_modules\node_cache
```

```shell

npm install -g yarn 全局安装yarn，可以正常使用。
```
环境变量-用户变量，NODE_PATH和PATH都已添加，设置成1中返回的路径
（d:\Node\nvm4w\nodejs\node_modules）


### nrm 安装

NRM （Node Registry Manager）Node 镜像源 管理工具

```shell
npm install -g nrm
```

然后查看可用源
```shell
nrm ls
```

测试响应
```shell
nrm test
```
切换源

```shell
nrm use npm
nrm use taobao
nrm use yarn
```



### 4 npm 安装

npm（ Node Package Manager） 简称为Node包管理工具

>npm默认下载的镜像源是国外的官方网站，这导致国内的下载速度过慢，为了解决下载速度过慢的问题，淘宝搭建了淘宝npm国内镜像服务器，每隔一段时间就会同步国外官网的包


npm包安装

  -  npm 本地安装： 执行 `npm install `命令时，例如：`npm install lodash` 软件包会被安装到当前文件树中的 node_modules 子文件夹下。
  -  npm 全局安装： `npm install -g lodash ` npm不会将软件包安装到本地文件夹下，而是使用全局的位置。

查看 npm -g 全局安装路径

 - npm config get prefix 查看 npm 安装路径
 - npm prefix -g  npm config get prefix
 - npm config set prefix 可设置 npm 安装路径
 - npm config ls -l 查看配置列表的全部信息
 - npm root -g  node_modules 的全局安装路径
 - npm config get cache 查看当前npm包的全局cache路径
 - npm config set cache 设置当前npm包的全局cache路径

{: .tips }
> 每次使用nvm切换node版本，最好都查看一下npm全局配置路径是否失效

npm 如何查看一个包的版本信息？

 - npm view jquery versions  查看npm服务器上所有的jquery版本信息—npm查看指定包的所有版本
 - npm view jquery versions  查看的最新的版本是哪一个
 - npm info jquery  查看jquery所有的版本以及相关信息

如查看本地下载的jquery版本信息

  -  npm ls jquery       ：即可（查看本地安装的jQuery）
  -  npm ls jquery -g    ：(查看全局安装的jquery)