---
layout: default
title: Git & GitHub
parent: Git
nav_order: 1
---

# Git & GitHub
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Git
-----------------------------


### 生成公私钥

（1）生成默认

```bash
ssh-keygen -t rsa -C 'xxxxx@youremail.com' 
```

（2）生成多个，可以指定名称

```bash
ssh-keygen -t rsa -C 'xxxxx@youremail.com' -f ~/.ssh/git_tencent_id_rsa
```

然后将公钥 `id_rsa.pub` 或 `gitee_id_rsa.pub ` 导入要使用的地方。


Github
-----------------------------

## github使用Token登录

现在github提交代码需要配置token登录。

简要步骤：

1.登录github账户，点击头像下拉的setting

2.点击左侧底的Developer settings

3.选择Personal access tokens

4.点击 Generate new token

5.为你创建的token添加描述

注意：

- Note描述千万不要重复，如果重复会提示`insufficient scopes granted to the token` 。重新生成一个就可以。

- 选择token有效期时间。可以选择永不过期，但我个人不建议，因为一旦你的token丢失就会不安全，再一个自己忘记token放哪里了，更换机器登录还是要重新生成。
     我是一年一换，既有过期时限，过期了就要逼着自己来折腾。

- 为token赋予权限。如果从命令行操作仓库，至少选中repo 。


6.填写好后点击 Generate Token。

生成之后务必先复制保存一份，后面会用到。在离开token展示页面之后你将再也看不见这个token的明文了。

## github 镜像站

- https://cdn.githubjs.cf
- https://hub.おうか.tw
- https://hub.連接.台灣
- https://gitclone.com/ （仅支持git）
- https://hub.fastgit.xyz （支持git）


## github 项目

- [x] [https://github.com/syhyz1990/baiduyun](https://github.com/syhyz1990/baiduyun)
- [x] [https://github.com/wuxs231/romantic](https://github.com/wuxs231/romantic)


