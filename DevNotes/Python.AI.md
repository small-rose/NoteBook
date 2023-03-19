---
layout: default
title: Python AI Painting
parent: DevNotes
nav_order: 99
---


# Python AI Painting
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Python 环境安装


### 1. 安装python环境

1、下载

官网：[https://www.python.org/](https://www.python.org/)

2、安装

没什么好说的，双击。

3、配置环境变量

如果在安装过程中勾选添加到path环境变量，可省略此步。

4、验证

```shell script
python
```

### 2. 安装conda或 miniconda 

conda 为了方便的创建多个python虚拟环境，可以进行多个python项目同时开发的时候，每个项目都有自己独立的python开发环境。

**pip、conda、anaconda和miniconda的区别**

- conda是一个包和环境管理工具，它不仅能管理包，还能隔离和管理不同python版本的环境。类似管理nodejs环境的nvm工具。

- anaconda和miniconda都是conda的一种发行版。只是包含的包不同。

- anaconda包含了conda、python等180多个科学包及其依赖项，体格比较大。但很多东西你未必用到，所以才有mini版。

- miniconda是最小的conda安装环境，只有conda+python+pip+zlib和一些其他常用的包，体格非常迷你。

- pip也叫包管理器，和conda的区别是，pip只管理python的包，而conda可以安装所有语言的包。而且conda可以管理python环境，pip不行。

注意：miniconda所有的操作命令皆在命令行中完成，没有GUI界面。而anaconda是有界面的。

官网：Miniconda — Conda documentation

>安装需要使用和Python版本一致的环境。养成安装目录不能有空格的好习惯。

安装完成后配置环境变量。找到 “系统变量” 里的path，点编辑。

```
# 添加三个路径:
d:\dev-tools-python\Miniconda3 
d:\dev-tools-python\Miniconda3\Scripts
d:\dev-tools-python\Miniconda3\Library\bin 
```

打开cmd命令窗口，输入conda命令，正常出现参数提示说明安装正常。

## 常用命令

创建python虚拟环境，默认创建到miniconda安装目录下的envs里

```bash
conda create -n test01 python=3.10     # -n是name的缩写，python不指定版本就默认最新版
```

查看已经安装的虚拟环境列表

```shell script
conda env list
```


进入虚拟环境并安装其他三方库

输入需要进入的环境的名称即可。

```shell script
activate test01 
```


当命令前面出现括号加上我们环境的名称，如 (Test1) C:\Users\hp> ，的时候即说明我们已经进入我们创建的虚拟环境了。

退出当前虚拟环境

```shell script
deactivate
```

删除虚拟环境

```shell script
conda env remove -n test01
```

>镜像网站
>阿里云镜像： https://mirrors.aliyun.com/pypi/simple/
>清华大学镜像：https://pypi.tuna.tsinghua.edu.cn/simple/
>豆瓣镜像：https://pypi.doubanio.com/simple/
>中科大镜像：https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple/


```
1. conda --version #查看conda版本，验证是否安装
2. conda update conda #更新至最新版本，也会更新其它相关包
3. conda update --all #更新所有包
4. conda update package_name #更新指定的包
5. conda create -n env_name package_name #创建名为env_name的新环境，并在该环境下安装名为package_name 的包，可以指定新环境的版本号，例如：conda create -n python2 python=python2.7 numpy pandas，创建了python2环境，python版本为2.7，同时还安装了numpy pandas包
6. source activate env_name #切换至env_name环境
7. source deactivate #退出环境
8. conda info -e #显示所有已经创建的环境
9. conda create --name new_env_name --clone old_env_name #复制old_env_name为new_env_name
10. conda remove --name env_name –all #删除环境
11. conda list #查看所有已经安装的包
12. conda install package_name #在当前环境中安装包
13. conda install --name env_name package_name #在指定环境中安装包
14. conda remove -- name env_name package #删除指定环境中的包
15. conda remove package #删除当前环境中的包
16. conda env remove -n env_name #采用第10条的方法删除环境失败时，可采用这种方法
转自：https://www.bilibili.com/video/BV1PT4y1i7qt?spm_id_from=333.337.search-card.all.click
```


## AI Painting

### 1. 安装python环境

### 2. 安装git环境

### 3. 安装conda或miniconda 

安装conda

### 4、安装准备

（1）创建虚拟环境，终端中输入下面命令，等待安装。

```shell script
conda create -n sd-webui python=3.10
```


（2）激活创建的虚拟环境：

```shell script
conda activate sd-webui
```

### 5、下载并启动安装依赖

```shell script
git clone  https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
```

安装各种依赖并运行: 执行 `webui-user.bat`
【安装过程可能会报错，建议开启科学上网，多试几次】

针对自动安装多次还是报错可以手动安装：

>从github将open_clip的源文件下载到本地，这一步可以使用git clone也可以直接下载zip文件。下载后，解压（如果用git clone就不需要）到d:\\stable-diffusion-webui\venv\Scripts目录下（stable-diffusion-webui是你stable diffusion webui的根目录，这个地址只是我电脑中的，请根据自己放的位置调整）。
打开cmd，cd到d:\\stable-diffusion-webui\\venv\\Scripts\\open_clip下（或在该目录下打开cmd，效果相同）。
卸载clip:d:\\stable-diffusion-webui\\venv\\Scripts\\python.exe -m pip uninstall clip
使用d:\\dev-tools-python\\stable-diffusion-webui\\venv\\Scripts\\python.exe setup.py build install安装open_clip。

打开浏览器输入 `http://127.0.0.1:7860`,即可进入webui页面。

> 对显卡配置有点要求。


### 5.1 中文配置

安装中文
 
https://github.com/dtlnor/stable-diffusion-webui-localization-zh_CN
 
 

### 6、模型下载与使用

网站：[https://www.civitai.com](https://www.civitai.com)

Stable diffusion模型下载好放入 `models/Stable-diffusion/`文件夹中

lora模型下载好放入models/lora/文件夹中