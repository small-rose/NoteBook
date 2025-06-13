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

- 为token赋予权限。如果从命令行操作仓库，至少选中repo, gist, read:org  。


6.填写好后点击 Generate Token。

生成之后务必先复制保存一份，后面会用到。在离开token展示页面之后你将再也看不见这个token的明文了。

## github 镜像站

- https://cdn.githubjs.cf
- https://hub.おうか.tw
- https://hub.連接.台灣
- https://gitclone.com/ （仅支持git）
- https://hub.fastgit.xyz （支持git）



## github push 问题

> 确认网络无问题，代理无问题。 tracert github.com
> 确认配置无问题（用户名，邮箱） git config list

1.尝试切换 Git 的 SSL 后端为 OpenSSL​​ 
Windows 默认的 Schannel 库对 TLS 关闭握手要求严格，而 OpenSSL 兼容性更好：

```
# 设置 Git 使用 OpenSSL 替代 Schannel
git config --global http.sslBackend openssl
# 验证配置
git config --global http.sslBackend
```

2.尝试增大 POST 缓冲区​​ 
推送大文件时默认缓冲区（1MB）不足会触发连接重置：

```
git config --global http.postBuffer 524288000  # 500MB
# 验证配置
git config --global http.postBuffer
```

3.尝试降级 HTTP 协议版本​​ （不建议）
HTTP/2 在某些网络环境下不稳定，切换为 HTTP/1.1 可提升可靠性：

```
git config --global http.version HTTP/1.1
​​验证​​：

git config --global http.version
```