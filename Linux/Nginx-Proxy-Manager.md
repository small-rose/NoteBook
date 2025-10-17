---
layout: default
title: Nginx Proxy Manager
nav_order: 2
parent: Linux
---

# Nginx Proxy Manager
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Nginx 安全加固

>Nginx 的安全防护需从 信息隐藏、访问控制、加密传输、流量限速、日志审计、漏洞修复 多维度入手。
>
>定期更新 Nginx 和依赖组件；
>
>结合 WAF 和日志分析工具构建纵深防御体系；
>
>制定应急响应计划，快速应对突发攻击事件。
>
>通过以上策略，可显著提升 Nginx 服务的安全性，降低被攻击的风险。
 
### 1 信息泄露

**问题描述：** Nginx 默认会在错误页面和 HTTP 响应头中暴露版本号、服务器标识等敏感信息，攻击者可利用这些信息针对性地发起攻击。

**解决方案：**

（1）隐藏版本号

在 Nginx 配置文件中添加以下指令，关闭版本号显示：

server_tokens off; 会移除响应头中的 Server: nginx/x.x.x 信息。

```
http{
    server_tokens off; 
}
```

（2）隐藏目录列表功能

禁止 Nginx 自动列出目录内容，防止敏感文件泄露：

默认情况下 autoindex 已关闭，但需显式配置以确保安全性。

```
http{
    autoindex off; 
}
```

(3)精简 HTTP 响应头

移除不必要的响应头字段（如 X-Powered-By、Server），减少信息泄露风险：


### 2 访问控制限制

**问题描述：** 未设置 IP 白名单或黑名单时，敏感接口可能被恶意扫描或暴力破解。

**解决方案:**

(1)IP 白名单/黑名单

通过 allow 和 deny 指令限制访问权限：


```
http{
    server{
        
        location ^~ /bp-web/ {
            proxy_pass "http://192.168.1.100:8508/bp-web/";
            allow 192.168.1.0/20 ; # 放行网段
            deny all ;  # 禁止 allow 之外的IP 访问
        }
    }
}
```

> 注： 多个allow或多个 deny 会从上到下依次匹配，直到匹配到第一条规则。

(2)动态 IP 封禁（Fail2Ban）

集成 Fail2Ban 工具，自动封禁频繁攻击的 IP：        

a.安装fail2ban
```
yum install fail2ban
```
b.配置nginx过滤规则

c.启动fail2ban服务

d.封禁测试验证


（3）基于 Basic Auth 的身份验证

对敏感路径启用基础认证：

```aidl

```


### 3 加密传输

**问题描述：**

未启用 HTTPS 或证书配置不当可能导致数据被窃听或篡改。

**解决方案：**

（1）强制 HTTPS

配置 SSL/TLS 证书并强制跳转至 HTTPS：

```conf

server {

    listen  80;
    server_name xxx.zhangxiaocai.cn;
    rewrite ^(.*)$  https://$host$1 permanent;
}

server{
    listen       443 ssl;
    server_name  xxx.zhangxiaocai.cn;

    ssl_certificate      /home/nginx/ssl/xxx.zhangxiaocai.cn_bundle.crt;
    ssl_certificate_key  /home/nginx/ssl/xxx.zhangxiaocai.cn.key;
    
    ssl_session_cache  shared:SSL:10m;
    ssl_session_timeout  10m;   
    #ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_protocols TLSv1.2 TLSv1.3; # 只启用TLSv1.2和TLSv1.3

    # 配置强加密策略
    ssl_prefer_server_ciphers on;
    ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES256-GCM-SHA384'; # 使用强加密套件

    # 添加Strict-Transport-Security头部 强制浏览器仅通过 HTTPS 访问
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload";
}
```

（2）启用 HTTPS

强制浏览器仅通过 HTTPS 访问：

```
    # 添加Strict-Transport-Security头部
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload";
```

（3）禁用弱协议和加密套件

配置强加密策略：

```
    ssl_prefer_server_ciphers on;
    ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES256-GCM-SHA384'; # 使用强加密套件

```

### 4 拒绝服务攻击（DDoS）

**问题描述：**

高并发请求或恶意流量可能导致服务器资源耗尽。

**解决方案：**

（1）限制请求速率和连接数

使用 limit_req_zone 和 limit_conn_zone 控制流量：

**limit_req_zone**

```
# 定义一个以IP为限制请求的方式，名字为req_limit_zone，开辟10M的共享内存区域，每秒处理的速率为10个请求
limit_req_zone $binary_remote_addr zone=req_limit_zone:10m rate=10r/s;
```

说明 ：limit_req_zone指令通常在 HTTP 块中定义，使其可在多个上下文中使用，它需要以下三个参数：

-    key - 定义应用限制的请求特性。示例中使用的是 Nginx 嵌入变量binary_remote_addr（二进制客户端地址）

-   zone - 定义用于存储每个 IP 地址状态以及被限制请求 URL 访问频率的共享内存区域。保存在内存共享区域的信息，意味着可以在 Nginx 的 worker 进程之间共享。定义分为两个部分：通过zone=keyword标识区域的名字，以及冒号后面跟区域大小。16000 个 IP 地址的状态信息，大约需要 1MB，所以示例中区域可以存储 160000 个 IP 地址。

-   rate - 定义最大请求速率。在示例中，速率不能超过每秒 10 个请求。Nginx 实际上以毫秒的粒度来跟踪请求，所以速率限制相当于每 100 毫秒 1 个请求。因为不允许”突发情况”，这意味着在距离前一个请求 100 毫秒内到达的请求将被拒绝。

指令limit_req的使用
```
limit_req zone=req_limit_zone burst=10 nodelay;
```
说明：

 - limit_req zone=req_limit_zone; 每个 IP 地址被限制为每秒只能请求 10 次 URL，更准确地说，在距离前一个请求的 100 毫秒内不能请求该 URL。
- limit_req zone=req_limit_zone burst=10; burst 参数定义了超出 req_limit_zone指定速率的情况下(示例中的 req_limit_zone区域，速率限制在每秒 10 个请求，或每 100 毫秒一个请求)，客户端还能发起多少请求。距离上一个请求 100 毫秒内到达的请求将会被放入队列，我们将队列大小设置为 10。

这说明如果从一个给定 IP 地址发送 11 个请求，Nginx 会立即将第一个请求发送到上游服务器群，然后将余下 10 个请求放在队列中。然后每 100 毫秒转发一个排队的请求，只有当传入请求使队列中排队的请求数超过 10 时，Nginx 才会向客户端返回 503。

配置 burst 参数将会使通讯更流畅，但是可能会不太实用，因为该配置会使站点看起来很慢。在上面的示例中，队列中的第 10 个包需要等待 1 秒才能被转发，此时返回给客户端的响应可能不再有用。

```
limit_req zone=req_limit_zone burst=10 nodelay;  
#使用 nodelay 参数，可以实现无延迟的排队；Nginx 仍将根据 burst 参数分配队列中的位置，当一个请求到达时，只要在队列中能分配位置，Nginx 将立即转发这个请求。将队列中的该位置标记为”taken”(占据)，并且不会被释放以供另一个请求使用，直到一段时间后才会被释放(在这个示例中是，100 毫秒后)。
```

假设如前所述，队列中有 10 个空位，从给定的 IP 地址发出的 11 个请求同时到达。Nginx会立即转发这个 11 个请求，并且标记队列中占据的 10 个位置，然后每 100 毫秒释放一个位置。如果是15个请求同时到达，Nginx 将会立即转发其中的 11 个请求，标记队列中占据的 10 个位置，并且返回 503 状态码来拒绝剩下的 4 个请求。

**limit_conn_zone**

指令limit_conn_zone定义一个以IP为限制连接的方式，名字为conn_limit_zone，并开辟10m的共享内存区
```
limit_conn_zone $binary_remote_addr zone=conn_limit_zone:10m;
```

指令limit_conn的使用
```
limit_conn conn_limit_zone 2;
```
说明： 使用定义conn_limit_zone的连接限流，限制每个IP地址只允许同时创建2个连接。

```
http{
    #limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
    limit_req_zone $binary_remote_addr zone=req_limit_zone_myname:10m rate=1r/s;
    limit_conn_zone $binary_remote_addr zone=conn_limit_zone_myname:10m;

    server{
        location /server1 {
            limit_req zone=cust_req_limit_zone_myname burst=10 nodelay;
            limit_conn addr 10;  # 每个IP地址最多10个并发连接
        }

    }
}
```

burst=10 允许突发流量，nodelay 不延迟丢弃。

（2）超时设置

减小空闲连接超时时间：

```
client_body_timeout 10s
client_header_timeout 10s
keepalive_timeout 10s
```

（3）防御 CC 攻击

限制单个 IP 的请求数：拦截非标准 HTTP 方法和包含 HTML 标签的查询参数。

```
if($request_method !~^(GET|POST)$){
    return 444 ;
}
if(args ~*"(<|%3C).*script.*(>|%3E)"){
    return 444 ;
}
```

### 5 跨站脚本攻击（XSS）与点击劫持

**问题描述：**

恶意脚本注入或网页被嵌入框架可能导致用户数据泄露。

**解决方案：**

(1)启用安全响应头

配置 HTTP 响应头增强浏览器安全机制：

```
add_header X-Frame-options SAMEORIGIN ;
add_header X-XSS-Protection "1;mode=block";
add_header X-Content-Type-Options nosniff;
add_header Content-Security-Policy "default-src 'self'";
```
- X-Frame-Options 防止点击劫持，
- Content-Security-Policy 限制资源加载来源。

(2)输入过滤

使用正则表达式拦截可疑请求：


```
if(query_string ~* "<script>|eval|expression"){
    return 444 ;
}
```

### 6 日志管理与监控

**问题描述：**

日志文件可能包含敏感信息，且异常访问行为难以及时发现。

**解决方案：**

（1）日志轮换与压缩

使用 logrotate 定期归档日志：

>   logrotate是Linux系统下一个非常实用的日志管理工具，它主要用于切割日志文件、删除旧的日志文件，并创建新的日志文件，以节省磁盘空间。

```
/etc/nginx/logs/*.log {
        monthly       # 每月切割一次
        rotate 6      # 保留6个备份
        copytruncate  # 备份日志并截断
        delaycompress # 转储的日志到下一次转储时才压缩
        compress      # 转储后使用gzip压缩
        notifempty    # 日志为空时不处理
        missingok     # 切割中遇到日志错误忽略
        dateext       # 在转储后的日志加上日期做后缀
        dateformat -%Y%m%d # 指定日期的格式
}
```





