---
layout: default
title: Windows
nav_order: 2
parent: Tools
---

# Windows
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}




windows 端口转发

windows 添加端口转发
```text
netsh interface portproxy add v4tov4 listenaddress=127.0.0.1 listenport=9000 connectaddress=37.220.51.222 connectport=1521
```

windows 移除端口转发

```text
netsh interface portproxy delete v4tov4 listenaddress=127.0.0.1 listenport=9000
```
