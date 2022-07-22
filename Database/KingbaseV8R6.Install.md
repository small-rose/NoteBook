---
layout: default
title: KingBase V8R6 INSTALL
nav_order: 3
parent: Database
---

# KingBase
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


KingBase 在CentOS 7.2 环境安装与卸载


一些文档 https://www.showdoc.com.cn/wj00/8719535208130116


## 一、CentOS安装教程

### 1、从官网下载文件

KingbaseES_V008R006C005B0023_Lin64_single_install.iso

证书使用开发版 license_12350_0.dat



### 2、新建KingBase账户


```bash
cd /mnt/ && mkdir kingbase

groupadd kingbase

useradd kingbase -m -g kingbase

passwd kingbase

King@1234
 
```

### 3、挂载iso文件

上传安装文件

```bash
cd /home/kingbase/  && mkdir download
```

将文件和授权文件下载上传到download目录。授权文件使用开发版的证书即可。

确认文件

```bash
[root@localhost download]# ls
KingbaseES_V008R006C005B0023_Lin64_single_install.iso  license_12350_0.dat
```

执行挂载

这里是挂载到 /mnt/kingbase 目录

```bash
[root@localhost download]# mount -o loop /home/kingbase/download/KingbaseES_V008R006C005B0023_Lin64_single_install.iso /mnt/kingbase
```

查看文件挂载是否正常

```bash
[root@localhost download]# cd /mnt/kingbase/  && ls -lrt
total 6
dr-xr-xr-x. 2 root root 2048 Nov  5 18:09 setup
-r-xr-xr-x. 1 root root 3820 Nov  5 18:09 setup.sh
```

kingbase授权

把权限赋给kingbase用户，因为安装会在这个目录读写取文件

```bash
chown -R kingbase:kingbase /mnt/kingbase/ 
```

### 4、安装

#### 4.1  路径准备

(1)准备授权文件路径

```bash
 /home/kingbase/download/license_12350_0.dat
```

(2)准备安装文件路径。安装目录要授权读写给kingbase用户才行。

安装目录可以自己规划定义，这里示例为：`/u01/software/kingbase/kingbaseES/V8R6`

```bash
#创建目录
mkdir -p  /u01/software/kingbase/kingbaseES/V8R6
#授权
chown -R kingbase:kingbase /u01/software/kingbase
```

#### 4.2 安装

支持图形界面安装和命令行安装。

如果想使用中文请设置 Lang 变量

- export LANG=zh_CN.UTF8  # 中文
- export LANG=en_US.UTF8  # 英文


安装命令

```bash
su kingbase
./setup.sh
#或者使用图形界面
./setup.sh -i gui   
```

命令行安装命令。我使用的是命令安装

```bash
[kingbase@localhost kingbase]$ sh ./setup.sh -i console
```


英文安装日志如下：

```text
Now launch installer...
tee: .installer.log: Read-only file system
Preparing to install
Extracting the JRE from the installer archive...
Unpacking the JRE...
Extracting the installation resources from the installer archive...
Configuring the installer for this system's environment...
          Verifying JVM........
Launching installer...

===============================================================================
KingbaseES V8                                    (created with InstallAnywhere)
-------------------------------------------------------------------------------

Preparing CONSOLE Mode Installation...




===============================================================================
Welcome
-------

This installer will guide you through the installation of KingbaseES V8.

It is strongly recommended that you quit all programs before continuing with 
this installation. You may cancel this installation by typing 'quit'.

KingbaseES Version: V8

Installer Version: V008R006C005B0023 Build

Kingbase Inc.
	http://www.kingbase.com.cn

PRESS <ENTER> TO CONTINUE: 



===============================================================================
License Agreement
-----------------

Installation and Use of KingbaseES V8 Requires Acceptance of the Following 
License Agreement:


Beijing Kingbase Information Technologies Inc.
"SOFTWARE END-USER LICENSE AGREEMENT"
IMPORTANT-READ CAREFULLY: This End-User License Agreement ("EULA") is a legal 
agreement between you (either an individual or a single entity) and Beijing 
Kingbase Information Technologies Inc.(abbreviated as Kingbase in the 
context). The "software product" includes computer software, and may include 
associated media, printed materials, and online documentation(Software 
product). This "software product" includes any upgrade and supplemental 
materials to the original "software product" provided by Kingbase. Any 
software product that are provided with the "software product", which are 
accompanied by their own license agreements or terms of use are governed by 
this EULA. You agree to be bound by the terms of this EULA by installing, 
copying, downloading, accessing or otherwise using the "software product". If 
you do not agree to the terms of this EULA, you may not install, copy or use 
the "software product".

SOFTWARE PRODUCT LICENSE

The "software product" is protected by copyright laws, international copyright
treaty and other intellectual property laws and treaties.

The "software product" is licensed to use, not sold.

PRESS <ENTER> TO CONTINUE: 


1. GRANT OF LICENSE. As long as you follow this EULA, Kingbase grants you the 
following non-exclusive, non-transitive rights:

APPLICATION SOFTWARE. You can use the software in locations as agreed in the 
related contract. The usage is restricted by the quantity of the purchased and
the type of the license (as agreed in the contract). If the license type has 
no special specification, you can only install, use, access, display, run or 
use other methods to interact(run) with one copy of this "software product" 
(or any previous versions on the same operating system) on a single computer, 
workstation, handheld computer, smart phone or other electronic device 
("computer").

RESERVATION OF RIGHTS. Except for the specific statements in this EULA, 
Kingbase reserves all rights not expressly granted to you. Kingbase reserves 
the right to interpret the content of the agreement.

2. LIMITATIONS AND OTHER RIGHT

LIMITATION ON TRANSFER. Under any condition, without the written permission of
Kingbase, you may not transfer this license and any usage permission under 
this license.

PRESS <ENTER> TO CONTINUE: 


NOT FOR RESALE SOFTWARE. If this "software product" is identified as "Not For 
Resale", it may not be sold or otherwise transferred for value, or used for 
any purpose other than demonstration, test or evaluation, no matter what else 
is stated in this EULA.

LIMITATIONS ON REVERSE ENGINEERING, DECOMPILATION, AND DISASSEMBLY. You may 
not reverse engineer, decompile, or disassemble the "software product", except
and only to the extent that such activity is expressly permitted by applicable
law not with standing this limitation.

TRADEMARKS. This EULA does not grant you any rights in connection with any 
trademarks or service marks of Kingbase.

NO RENTAL. You may not rent, lease or lend the "software product" to others.

EXPORT LIMITATION. You admit that the "software product" is governed by the 
export laws of People's Republic of China. You agree to obey all the 
international and domestic laws applicable to this "software product", 
including "Foreign Trade Law of the People's Republic of China", and other 
restrictions and regulations issued by the Chinese government related to 
software exportation.

PRESS <ENTER> TO CONTINUE: 


PRODUCT SUPPORT. Kingbase provides you the support service related to the 
software product ("support service"), but the specific content of the support 
service is limited by related contract. Kingbase reserves the right to charge 
the support service. The usage of support service is restricted by the 
policies and plans in the user's manual, online document and/or other Kingbase
provided materials. Any supplemental software code provided to you should be 
treated as a part of the "software product", and has to comply the terms and 
conditions in this EULA. As for the technical information you provided to 
Kingbase as a part of the support service, Kingbase may commercialize it, 
including product support and development. Kingbase will not mention you 
individually when using this technical information.

TERMINATION. Without prejudice to other rights, Kingbase may terminate this 
EULA if you fail to comply with the terms and conditions of this EULA. In such
event, you must destroy all copies of the "software product" and all of its 
component parts.

3. UPGRADES. If the "software product" is labeled as an upgrade, you must be 
properly licensed to use a product identified by Kingbase as being eligible 
for the upgrade in order to use the "software product". Kingbase reserves the 
right to charge the upgrade version or upgrade. "software product" labeled as 

PRESS <ENTER> TO CONTINUE: 

an upgrade replaces and/or supplements the Eligible Product which you are 
qualified to use. You may use the resulting upgraded product only in 
accordance with the terms of this EULA. If the "software product" is an 
upgrade of a component of a package of software programs that you licensed as 
a single product, the "software product" may be used and transferred only as 
part of that single product package and may not be separated for use on more 
than one computer.

4. INTELLECTUAL PROPERTY. The ownership, copyright and other intellectual 
property of the "software product" (including but not limited to the picture, 
photo, animation, video, recordings, music, text and supplemental programs 
contained in the "software product"), enclosed printed materials and any 
copies of the "software product", is owner by Kingbase and its suppliers. The 
ownership and intellectual property accessed by this "software product" 
belongs to the owner of the contents, and may be protected by copyright laws, 
and other intellectual property laws and treaties. This EULA does not grant 
you the right to use these contents. If this "software product" include 
documents only provided electronically, you can print one copy of the 
electronic documents. You may not remove the copyright declaration from the 
software, and ensures that the copyright declaration is copied for the replica
(whole or part) of the "software product". You agree to stop any forms of 
illegal copying this software and the documents. You cannot copy the enclosed 

PRESS <ENTER> TO CONTINUE: 

printed materials in this "software product".

5. MULTIMEDIA SOFTWARE. You may obtain the "software product" through multiple
medias. No matter the type and size of the media you receive, you can only use
the media which is applicable to your single computer. You cannot run a 
different media on another computer. Except for the part as in "permanent 
transfer" of the "software product (as stated above), you cannot transfer the 
other medias to another user via rent, lease or lend.

6. BACKUP COPY. After installing a copy of the "software product" according to
the EULA, you may keep the original media by which Kingbase provided you the 
"software product" solely for backup or archival purpose. If original media is
needed to use the "software product", you may make one back-up copy solely for
your backup or archival purposes. Except as expressly provided in this EULA, 
you may not otherwise make copies of the software, including the printed 
materials accompanying the software. Authorized copy should be kept in secured
environments.

7. CONFIDENTIALITY. "Confidential information" includes program(including the 
methods and concepts used in the program) and any information that Kingbase 
identifies as exclusive or confidential. Unless expressly granted by this 
EULA, you may not expose or provide the confidential information by other 

PRESS <ENTER> TO CONTINUE: 

methods to a third party and any employees to whom you do not grant the usage 
in their business. You should take any reasonable, necessary measures to 
ensure that the program or any component of the software is not exposed or 
provided by other methods to a third party.

If you acquired the Kingbase product in People's Republic of China, the 
following limited warranty applies to you.

1. LIMITED WARRANTY.
As long as you have valid license, Kingbase warrants that: (1) The "software 
product" will perform substantially in accordance with the accompanying 
materials for a period of ninety (90) days after the date of receipt. (2) The 
provided support will perform substantially in accordance with the 
accompanying materials, and Kingbase engineers will try their best to solve 
any problems permitted by commercial range. If the product is not compliant to
the warranty, Kingbase will fix, replace the product or refund you for the 
product, and you have to return the "software product" to Kingbase along with 
the invoice held by you. If the malfunction of the product is due to accident,
abuse or misuse, then the warranty is not effective. The replaced product has 
the residual of the original warranty period, or thirty days, whichever is 
longer. To the maximum extent permitted by applicable law, except for the 
above warranty, all expressed or implied warranties, conditions and other 

PRESS <ENTER> TO CONTINUE: 

terms are denied by Kingbase. All implicit warranties which cannot be excluded
are limited to 90 days or the minimum period allowed by the appropriate laws, 
whichever is longer.

2. LIMITATION OF LIABILITY
To the maximum extent permitted by applicable law, except for the above 
warranty, Kingbase and its suppliers shall not be liable for any damages 
whatsoever (including without limitation, damages for loss of business 
profits, business interruption, loss of business information or other 
pecuniary loss) arising out of the use or inability to use the "software 
product", even if Kingbase and its suppliers have been advised of the 
possibility of such damages. In any case Kingbase and any of its suppliers' 
entire liability under any provision of this EULA shall be limited to the 
amount actually paid by you for the "software product" or RMB 10.00 Yuan, 
which ever is higher. However, if you have signed support agreement with 
Kingbase, all Kingbase's liability of the support service will be defined by 
that agreement.

GOVERNING LAWS
This EULA is governed by the laws of the People's Republic of China (including
but not restricted to "Copyright Law of the People's Republic of China", 
"Regulations for the Protection of Computer Software of the People's Republic 

PRESS <ENTER> TO CONTINUE: 

of China", "Trademark Law of the People's Republic of China", "Patent Law of 
the People's Republic of China", "Anti-Unfair Competition Law of the People's 
Republic of China", etc.). In respect of any dispute or claim which may arise 
by this EULA or the violation of the EULA, you consent to the jurisdiction of 
the federal and provincial courts sitting in the location of Kingbase. If 
Kingbase's intellectual property is violated, the above terms do not restrict 
Kingbase to apply remedial measures from the legitimate court with governing 
rights.

Beijing Kingbase Information Technologies Inc.

Add: 3 layer, B block, Information Industrial Park, Rongda Road 7, Chaoyang 
District, Beijing, 100102 China
Tel: 86-10-5885 1118
Http: //www.kingbase.com.cn
National Hotline: 400-601-1188
Support E-mail: support@kingbase.com.cn


DO YOU ACCEPT THE TERMS OF THIS LICENSE AGREEMENT? (Y/N): Y



===============================================================================
Choose Install Set
------------------

Please choose the Install Set to be installed by this installer.

  ->1- Full
    2- Client

    3- Custom

ENTER THE NUMBER FOR THE INSTALL SET, OR PRESS <ENTER> TO ACCEPT THE DEFAULT
   : 1



===============================================================================
Choose License File
-------------------


File Path: /home/kingbase/download/license_12350_0.dat

License序列号 --- 启用 --- 74FE7946-4378-11EC-AE8E-000C29CBE49F
生产日期 --- 启用 --- 2021-11-12
产品名称 --- 启用 --- KingbaseES V8
细分版本模板名 --- 启用 --- SALES-开发版 V8R6
产品版本号 --- 启用 --- V008R006C
浮动基准日期 ------ 启用
有效期间 --- 启用 --- 0
用户名称 --- 启用 --- 官方网站试用授权
项目名称 --- 启用 --- 官方网站试用授权
CPU检查 --- 启用 --- 0
容器名称 --- 禁用 --- 0
MAC地址 --- 启用 --- 00:00:00:00:00:00
最大连接数 --- 启用 --- 10
分区 --- 启用 --- 0
物理同步 --- 启用 --- 0
读写分离模块 --- 启用 --- 0
恢复到指定时间点 --- 启用 --- 0
集群对网络故障的容错 --- 启用 --- 0
快速加载 --- 启用 --- 0
日志压缩 --- 启用 --- 0
全文检索 --- 启用 --- 0
性能优化包(性能诊断) --- 启用 --- 0
性能优化包(性能调优) --- 启用 --- 0
保密通讯协议 --- 启用 --- 0
审计 --- 启用 --- 0
三权分立 --- 启用 --- 0
透明加密 --- 启用 --- 0
强制访问控制 --- 启用 --- 0
列加密 --- 启用 --- 0
密码复杂度 --- 启用 --- 0
用户锁定 --- 启用 --- 0
集群管理软件 --- 启用 --- 0
集群配置工具 --- 启用 --- 0
集群高级管理包 --- 启用 --- 0
并行查询 --- 启用 --- 0
并行备份还原 --- 启用 --- 0
异构数据源 --- 启用 --- 0
日志解析 --- 启用 --- 0



===============================================================================
Choose Install Folder
---------------------

Where would you like to install?

  Default Install Folder: /opt/Kingbase/ES/V8

ENTER AN ABSOLUTE PATH, OR PRESS <ENTER> TO ACCEPT THE DEFAULT
      : /u01/software/kingbase/kingbaseES/V8R6

      【这里是安装路径，贴上完整路径即可】

INSTALL FOLDER IS: /u01/software/kingbase/kingbaseES/V8R6
   IS THIS CORRECT? (Y/N): Y  

You do not have write permissions to the chosen installation destination.
Please choose a different location for installation

ENTER AN ABSOLUTE PATH, OR PRESS <ENTER> TO ACCEPT THE DEFAULT
      : /home/kingbase/kingbaseE^Ccat: .installer.log: No such file or directory
Installation failed .
[kingbase@localhost kingbase]$ sh ./setup.sh -i console
Now launch installer...
tee: .installer.log: Read-only file system
Preparing to install
Extracting the JRE from the installer archive...
Unpacking the JRE...
Extracting the installation resources from the installer archive...
Configuring the installer for this system's environment...
          Verifying JVM........
Launching installer...

===============================================================================
KingbaseES V8                                    (created with InstallAnywhere)
-------------------------------------------------------------------------------

Preparing CONSOLE Mode Installation...




===============================================================================
Welcome
-------

This installer will guide you through the installation of KingbaseES V8.

It is strongly recommended that you quit all programs before continuing with 
this installation. You may cancel this installation by typing 'quit'.

KingbaseES Version: V8

Installer Version: V008R006C005B0023 Build

Kingbase Inc.
	http://www.kingbase.com.cn

PRESS <ENTER> TO CONTINUE: 



===============================================================================
License Agreement
-----------------

Installation and Use of KingbaseES V8 Requires Acceptance of the Following 
License Agreement:


Beijing Kingbase Information Technologies Inc.
"SOFTWARE END-USER LICENSE AGREEMENT"
IMPORTANT-READ CAREFULLY: This End-User License Agreement ("EULA") is a legal 
agreement between you (either an individual or a single entity) and Beijing 
Kingbase Information Technologies Inc.(abbreviated as Kingbase in the 
context). The "software product" includes computer software, and may include 
associated media, printed materials, and online documentation(Software 
product). This "software product" includes any upgrade and supplemental 
materials to the original "software product" provided by Kingbase. Any 
software product that are provided with the "software product", which are 
accompanied by their own license agreements or terms of use are governed by 
this EULA. You agree to be bound by the terms of this EULA by installing, 
copying, downloading, accessing or otherwise using the "software product". If 
you do not agree to the terms of this EULA, you may not install, copy or use 
the "software product".

SOFTWARE PRODUCT LICENSE

The "software product" is protected by copyright laws, international copyright
treaty and other intellectual property laws and treaties.

The "software product" is licensed to use, not sold.

PRESS <ENTER> TO CONTINUE: 


1. GRANT OF LICENSE. As long as you follow this EULA, Kingbase grants you the 
following non-exclusive, non-transitive rights:

APPLICATION SOFTWARE. You can use the software in locations as agreed in the 
related contract. The usage is restricted by the quantity of the purchased and
the type of the license (as agreed in the contract). If the license type has 
no special specification, you can only install, use, access, display, run or 
use other methods to interact(run) with one copy of this "software product" 
(or any previous versions on the same operating system) on a single computer, 
workstation, handheld computer, smart phone or other electronic device 
("computer").

RESERVATION OF RIGHTS. Except for the specific statements in this EULA, 
Kingbase reserves all rights not expressly granted to you. Kingbase reserves 
the right to interpret the content of the agreement.

2. LIMITATIONS AND OTHER RIGHT

LIMITATION ON TRANSFER. Under any condition, without the written permission of
Kingbase, you may not transfer this license and any usage permission under 
this license.

PRESS <ENTER> TO CONTINUE: 


NOT FOR RESALE SOFTWARE. If this "software product" is identified as "Not For 
Resale", it may not be sold or otherwise transferred for value, or used for 
any purpose other than demonstration, test or evaluation, no matter what else 
is stated in this EULA.

LIMITATIONS ON REVERSE ENGINEERING, DECOMPILATION, AND DISASSEMBLY. You may 
not reverse engineer, decompile, or disassemble the "software product", except
and only to the extent that such activity is expressly permitted by applicable
law not with standing this limitation.

TRADEMARKS. This EULA does not grant you any rights in connection with any 
trademarks or service marks of Kingbase.

NO RENTAL. You may not rent, lease or lend the "software product" to others.

EXPORT LIMITATION. You admit that the "software product" is governed by the 
export laws of People's Republic of China. You agree to obey all the 
international and domestic laws applicable to this "software product", 
including "Foreign Trade Law of the People's Republic of China", and other 
restrictions and regulations issued by the Chinese government related to 
software exportation.

PRESS <ENTER> TO CONTINUE: 


PRODUCT SUPPORT. Kingbase provides you the support service related to the 
software product ("support service"), but the specific content of the support 
service is limited by related contract. Kingbase reserves the right to charge 
the support service. The usage of support service is restricted by the 
policies and plans in the user's manual, online document and/or other Kingbase
provided materials. Any supplemental software code provided to you should be 
treated as a part of the "software product", and has to comply the terms and 
conditions in this EULA. As for the technical information you provided to 
Kingbase as a part of the support service, Kingbase may commercialize it, 
including product support and development. Kingbase will not mention you 
individually when using this technical information.

TERMINATION. Without prejudice to other rights, Kingbase may terminate this 
EULA if you fail to comply with the terms and conditions of this EULA. In such
event, you must destroy all copies of the "software product" and all of its 
component parts.

3. UPGRADES. If the "software product" is labeled as an upgrade, you must be 
properly licensed to use a product identified by Kingbase as being eligible 
for the upgrade in order to use the "software product". Kingbase reserves the 
right to charge the upgrade version or upgrade. "software product" labeled as 

PRESS <ENTER> TO CONTINUE: 

an upgrade replaces and/or supplements the Eligible Product which you are 
qualified to use. You may use the resulting upgraded product only in 
accordance with the terms of this EULA. If the "software product" is an 
upgrade of a component of a package of software programs that you licensed as 
a single product, the "software product" may be used and transferred only as 
part of that single product package and may not be separated for use on more 
than one computer.

4. INTELLECTUAL PROPERTY. The ownership, copyright and other intellectual 
property of the "software product" (including but not limited to the picture, 
photo, animation, video, recordings, music, text and supplemental programs 
contained in the "software product"), enclosed printed materials and any 
copies of the "software product", is owner by Kingbase and its suppliers. The 
ownership and intellectual property accessed by this "software product" 
belongs to the owner of the contents, and may be protected by copyright laws, 
and other intellectual property laws and treaties. This EULA does not grant 
you the right to use these contents. If this "software product" include 
documents only provided electronically, you can print one copy of the 
electronic documents. You may not remove the copyright declaration from the 
software, and ensures that the copyright declaration is copied for the replica
(whole or part) of the "software product". You agree to stop any forms of 
illegal copying this software and the documents. You cannot copy the enclosed 

PRESS <ENTER> TO CONTINUE: 

printed materials in this "software product".

5. MULTIMEDIA SOFTWARE. You may obtain the "software product" through multiple
medias. No matter the type and size of the media you receive, you can only use
the media which is applicable to your single computer. You cannot run a 
different media on another computer. Except for the part as in "permanent 
transfer" of the "software product (as stated above), you cannot transfer the 
other medias to another user via rent, lease or lend.

6. BACKUP COPY. After installing a copy of the "software product" according to
the EULA, you may keep the original media by which Kingbase provided you the 
"software product" solely for backup or archival purpose. If original media is
needed to use the "software product", you may make one back-up copy solely for
your backup or archival purposes. Except as expressly provided in this EULA, 
you may not otherwise make copies of the software, including the printed 
materials accompanying the software. Authorized copy should be kept in secured
environments.

7. CONFIDENTIALITY. "Confidential information" includes program(including the 
methods and concepts used in the program) and any information that Kingbase 
identifies as exclusive or confidential. Unless expressly granted by this 
EULA, you may not expose or provide the confidential information by other 

PRESS <ENTER> TO CONTINUE: 

methods to a third party and any employees to whom you do not grant the usage 
in their business. You should take any reasonable, necessary measures to 
ensure that the program or any component of the software is not exposed or 
provided by other methods to a third party.

If you acquired the Kingbase product in People's Republic of China, the 
following limited warranty applies to you.

1. LIMITED WARRANTY.
As long as you have valid license, Kingbase warrants that: (1) The "software 
product" will perform substantially in accordance with the accompanying 
materials for a period of ninety (90) days after the date of receipt. (2) The 
provided support will perform substantially in accordance with the 
accompanying materials, and Kingbase engineers will try their best to solve 
any problems permitted by commercial range. If the product is not compliant to
the warranty, Kingbase will fix, replace the product or refund you for the 
product, and you have to return the "software product" to Kingbase along with 
the invoice held by you. If the malfunction of the product is due to accident,
abuse or misuse, then the warranty is not effective. The replaced product has 
the residual of the original warranty period, or thirty days, whichever is 
longer. To the maximum extent permitted by applicable law, except for the 
above warranty, all expressed or implied warranties, conditions and other 

PRESS <ENTER> TO CONTINUE: 

terms are denied by Kingbase. All implicit warranties which cannot be excluded
are limited to 90 days or the minimum period allowed by the appropriate laws, 
whichever is longer.

2. LIMITATION OF LIABILITY
To the maximum extent permitted by applicable law, except for the above 
warranty, Kingbase and its suppliers shall not be liable for any damages 
whatsoever (including without limitation, damages for loss of business 
profits, business interruption, loss of business information or other 
pecuniary loss) arising out of the use or inability to use the "software 
product", even if Kingbase and its suppliers have been advised of the 
possibility of such damages. In any case Kingbase and any of its suppliers' 
entire liability under any provision of this EULA shall be limited to the 
amount actually paid by you for the "software product" or RMB 10.00 Yuan, 
which ever is higher. However, if you have signed support agreement with 
Kingbase, all Kingbase's liability of the support service will be defined by 
that agreement.

GOVERNING LAWS
This EULA is governed by the laws of the People's Republic of China (including
but not restricted to "Copyright Law of the People's Republic of China", 
"Regulations for the Protection of Computer Software of the People's Republic 

PRESS <ENTER> TO CONTINUE: 

of China", "Trademark Law of the People's Republic of China", "Patent Law of 
the People's Republic of China", "Anti-Unfair Competition Law of the People's 
Republic of China", etc.). In respect of any dispute or claim which may arise 
by this EULA or the violation of the EULA, you consent to the jurisdiction of 
the federal and provincial courts sitting in the location of Kingbase. If 
Kingbase's intellectual property is violated, the above terms do not restrict 
Kingbase to apply remedial measures from the legitimate court with governing 
rights.

Beijing Kingbase Information Technologies Inc.

Add: 3 layer, B block, Information Industrial Park, Rongda Road 7, Chaoyang 
District, Beijing, 100102 China
Tel: 86-10-5885 1118
Http: //www.kingbase.com.cn
National Hotline: 400-601-1188
Support E-mail: support@kingbase.com.cn


DO YOU ACCEPT THE TERMS OF THIS LICENSE AGREEMENT? (Y/N): Y



===============================================================================
Choose Install Set
------------------

Please choose the Install Set to be installed by this installer.

  ->1- Full
    2- Client

    3- Custom

ENTER THE NUMBER FOR THE INSTALL SET, OR PRESS <ENTER> TO ACCEPT THE DEFAULT
   : 1



===============================================================================
Choose License File
-------------------


File Path: /home/kingbase/download/license_12350_0.dat   

License序列号 --- 启用 --- 74FE7946-4378-11EC-AE8E-000C29CBE49F
生产日期 --- 启用 --- 2021-11-12
产品名称 --- 启用 --- KingbaseES V8
细分版本模板名 --- 启用 --- SALES-开发版 V8R6
产品版本号 --- 启用 --- V008R006C
浮动基准日期 ------ 启用
有效期间 --- 启用 --- 0
用户名称 --- 启用 --- 官方网站试用授权
项目名称 --- 启用 --- 官方网站试用授权
CPU检查 --- 启用 --- 0
容器名称 --- 禁用 --- 0
MAC地址 --- 启用 --- 00:00:00:00:00:00
最大连接数 --- 启用 --- 10
分区 --- 启用 --- 0
物理同步 --- 启用 --- 0
读写分离模块 --- 启用 --- 0
恢复到指定时间点 --- 启用 --- 0
集群对网络故障的容错 --- 启用 --- 0
快速加载 --- 启用 --- 0
日志压缩 --- 启用 --- 0
全文检索 --- 启用 --- 0
性能优化包(性能诊断) --- 启用 --- 0
性能优化包(性能调优) --- 启用 --- 0
保密通讯协议 --- 启用 --- 0
审计 --- 启用 --- 0
三权分立 --- 启用 --- 0
透明加密 --- 启用 --- 0
强制访问控制 --- 启用 --- 0
列加密 --- 启用 --- 0
密码复杂度 --- 启用 --- 0
用户锁定 --- 启用 --- 0
集群管理软件 --- 启用 --- 0
集群配置工具 --- 启用 --- 0
集群高级管理包 --- 启用 --- 0
并行查询 --- 启用 --- 0
并行备份还原 --- 启用 --- 0
异构数据源 --- 启用 --- 0
日志解析 --- 启用 --- 0



===============================================================================
Choose Install Folder
---------------------

Where would you like to install?

  Default Install Folder: /opt/Kingbase/ES/V8

ENTER AN ABSOLUTE PATH, OR PRESS <ENTER> TO ACCEPT THE DEFAULT
      : /u01/software/kingbase/kingbaseES/V8R6

INSTALL FOLDER IS: /u01/software/kingbase/kingbaseES/V8R6
   IS THIS CORRECT? (Y/N): Y



===============================================================================
Please Wait
-----------



===============================================================================
Please Wait
-----------



===============================================================================
Pre-Installation Summary
------------------------

Please Review the Following Before Continuing:

Product Name:
    KingbaseES V8

Install Folder:
    /u01/software/kingbase/kingbaseES/V8R6

Product Features:
    SERVER,
    HELP,
    MANAGER,
    DTS,
    INTERFACE,
    CLIENTTOOLS

Disk space information
    Required Space:820M                             Available Space:17278M



PRESS <ENTER> TO CONTINUE: 



===============================================================================
Ready To Install
----------------

InstallAnywhere is now ready to install KingbaseES V8 onto your system at the 
following location:

   /u01/software/kingbase/kingbaseES/V8R6

PRESS <ENTER> TO INSTALL: 



===============================================================================
Installing...
-------------

 [==================|==================|==================|==================]
 [------------------|------------------|------------------|------------------]



===============================================================================
Please Wait
-----------



===============================================================================
Please Wait
-----------



===============================================================================
Please Wait
-----------



===============================================================================
Please Wait
-----------



===============================================================================
Please Wait
-----------



===============================================================================
Please Wait
-----------



===============================================================================
Please Wait
-----------



===============================================================================
Please Wait
-----------



===============================================================================
Please Wait
-----------



===============================================================================
Please Wait
-----------



===============================================================================
Please Wait
-----------



===============================================================================
Please Wait
-----------



===============================================================================
Please Wait
-----------



===============================================================================
Please Wait
-----------



===============================================================================
Choose a Folder for data directory
----------------------------------

Please choose a folder. the folder must be empty

Data folder (Default:/u01/software/kingbase/kingbaseES/V8R6/data): 




===============================================================================
Please Wait
-----------



===============================================================================
Port
----


Port (Default: 54321): 
【默认端口】



===============================================================================
User
----


User: (Default: system): 
【默认系统用户system，可以不用改】



===============================================================================
Enter Password
--------------


Please Enter the Password: Please Enter the Password:*   
Password is required to continue.

Please Enter the Password: Please Enter the Password:**********

【这里要输入2次密码】

===============================================================================
Enter Password again
--------------------


Please Enter the Password Again: Please Enter the Password Again:**********



===============================================================================
Server-encoding
---------------
【服务端字符集】

  ->1- UTF8
    2- GBK
    3- GB18030

ENTER THE NUMBER FOR YOUR CHOICE, OR PRESS <ENTER> TO ACCEPT THE DEFAULT: 1




===============================================================================
Database_Mode
-------------
【安装模式，默认是ORACLE,我们选PG】

 -> 1- PG
 	2- ORACLE

ENTER THE NUMBER FOR YOUR CHOICE, OR PRESS <ENTER> TO ACCEPT THE DEFAULT: 




===============================================================================
Tips
----

The database will be initialized, which may take some time. Please be patient.

PRESS <ENTER> TO CONTINUE: 



===============================================================================
Please Wait
-----------



===============================================================================
Installation Complete
---------------------

Congratulations. KingbaseES V8 has been successfully installed to:

/u01/software/kingbase/kingbaseES/V8R6

If you want to register KingbaseES V8 as OS service, please run

    /u01/software/kingbase/kingbaseES/V8R6/Scripts/root.sh

PRESS <ENTER> TO EXIT THE INSTALLER: 
cat: .installer.log: No such file or directory
Complete.
```

至此安装过程完毕。可以看到安装过程中有默认的初始化数据

安装路径：/u01/software/kingbase/kingbaseES/V8R6
数据库文件夹： /u01/software/kingbase/kingbaseES/V8R6/data
默认端口：54321
默认账户名称：system
默认字符集编码：UTF8
默认的数据库模式：ORACLE，本次安装PG



### 5、启动与配置

#### 5.1 注册为系统服务

执行官方的脚本，注册成系统服务
```bash
su - root
./u01/software/kingbase/kingbaseES/V8R6/Scripts/root.sh
```

确认是否注册

```bash
chkconfig --list |grep --color e8d

systemctl list-dependencies |grep 8d
```

#### 5.2 配置环境变量

编辑环境变量

```bash
su - kingbase
vim ~/.bash_profile
```

添加一下信息

```bash
export KINGBASE_DATA=/u01/software/kingbase/kingbaseES/V8R6/data
export KINGBASE_HOME=/u01/software/kingbase/kingbaseES/V8R6
export PATH=$PATH:$KINGBASE_HOME/Server/bin:：
```

配置生效
```bash
source ~/.bash_profile
```

#### 5.3 服务方式启停

```bash
# 方法一
service kingbase8d start
service kingbase8d stop
service kingbase8d status
service kingbase8d restart

# 方法二
./ect/init.d/kingbase8d status
./ect/init.d/kingbase8d restart
```



### 6、初始化数据库实例

#### 6.1 大小写敏感问题

这里说下这个步骤，我第一次安装的时候是没有这个步骤的，使用V8R6版本在安装时已经不支持大小写敏感的设置。所以只能在数据库初始化的时候设置大小写敏感的问题。

数据库默认为大小写敏感。如果你使用的跟我一个版本，不要show case_sensitive; 没有这个参数，需要使用

 ```sql
show enable_ci;
 ```

on 是大小写不敏感，大小写通用;

off 是大小写敏感，需要严格区分大小写的。

#### 6.2 模式问题

默认的数据库模式：ORACLE。如果安装的时候选错了，在初始化数据库实例的时候还可以修改。

参数是` -m  ORACLE` 或者 ` -m  PG`



这里的初始化为空库。如果数据库有数据，就先执行一下备份命令，也可以适应官方的客户端界面操作备份。

- V8R6C5023 以上版本初始化：`./initdb -Usystem -x 密码  -D  /u01/software/kingbase/kingbaseES/V8R6/data  --enable-ci`

- V8R6C5023 以下版本初始化：`./initdb -Usystem -x 密码  -D  /u01/software/kingbase/kingbaseES/V8R6/data  --case-insensitive`
- V8R3 版本初始化：`./initdb -Usystem -W 密码  -D  /u01/software/kingbase/kingbaseES/V8R6/data  --case-insensitive`

注意，如果这里不指定模式，如果安装的时候选PG，这里没有指定会重置成ORACLE模式。

`./initdb -Usystem -x 密码  -D  /u01/software/kingbase/kingbaseES/V8R6/data  --enable-ci -m PG`

下面是执行日志:

```txt
The files belonging to this database system will be owned by user "kingbase".
This user must also own the server process.

The database cluster will be initialized with locale "en_US.UTF-8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

The comparision of strings is case-insensitive.
Data page checksums are disabled.

creating directory /u01/software/kingbase/kingbaseES/V8R6/data ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... PRC
creating configuration files ... ok
Begin setup encrypt device
initializing the encrypt device ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
create security database ... ok
load security database ... ok
syncing data to disk ... ok

initdb: warning: enabling "trust" authentication for local connections
You can change this by editing sys_hba.conf or using the option -A, or
--auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

    ./sys_ctl -D /u01/software/kingbase/kingbaseES/V8R6/data -l logfile start
```



初始化成功之后就可以启动实例了。

使用专用命令指定目录启动

和服务启停类似。这里是kingbase的方式。

```bash			
#带日志文件的
./sys_ctl -D /u01/software/kingbase/kingbaseES/V8R6/data -l kingbase.log start
./sys_ctl start -D /home/kingbase/kingbaseES/V8R6/data
./sys_ctl stop -D /home/kingbase/kingbaseES/V8R6/data
./sys_ctl restart -D /home/kingbase/kingbaseES/V8R6/data
```


或者在变量里需要配置data目录`/u01/software/kingbase/kingbaseES/V8R6/data`

```bash
vi ~/.bash_profile 
# 需要配置下面这个
export KINGBASE_DATA=/u01/software/kingbase/kingbaseES/V8R6/data
```

后面启动就可以不带目录，这种没有测试。

启动 


```bash
sys_ctl start 启动KINGBASE_DATA目录下的金仓服务
```
停止

```bash
sys_ctl stop -m smart    #相当于Oracle的shutdown normal
sys_ctl stop -m fast     #相当于Oracle的shutdown immediate，是默认方式
sys_ctl stop -m imediate #相当于Oracle的shutdown abo
```

### 7、链接验证

#### 7.1 命令目录

```bash
su - kingbase
ksql -U system  -W TEST  # 登录
ksql -U system  -W TEST -l  # 登录 列出数据库
输入密码 King@1234


TEST=# select version();    
                    version                                                        
--------------------------------------------------
 KingbaseES V008R006C005B0023 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 4.1.2 20080704 (Red Hat 4.1.2-46), 64-bit
(1 row)

TEST=# exit
```

#### 7.2 客户端连接

可以使用官方的客户端-数据库对象管理工具连接。

或者其他数据库连接客户端，比如Datagrip，自定义指一下驱动包就可以。



### 8、简易优化

1、调整连接数

使用官方数据库对象管理工具连接数据库。

在"管理"---"系统参数"---"连接认证"---调整最大连接数的值。重启一下数据库。

2、简单授权

（1）新建用户。

（2）建表，或者数据迁移表。

（3）将表授权给用户

```sql
grant all on all tables  in schema PUBLIC to bpdev01 ;
```



## 二、卸载

有时候因为一些缘故，死活解决不了。就卸载重装吧，比如安装完机器磁盘快满了，需要迁移到另外的盘。

```shell
systemctl stop kingbase8d.service
su - kingbase
cd /u01/software/kingbase/kingbaseES/V8R6/Uninstall/
./Uninstaller
```

其他如果有目录还有多余的就是要 `rm` 命令删除一下就可以。



## 三、数据库常用命令

```bash
ksql -U用户       -W密码              需要登录数据库名称
ksql -USYSTEM -W12345678ab  TEST
```

登录数据库后，如果需要执行sql脚本
```bash
\i  /opt/Kingbase/XX/XX.sql(这是sql脚本绝对路径)
```

```txt
列出数据库:   \l
列出索引:      \di
列出表：       \dt
列出表结构：\d 表
查询数据大小写敏感： show case_sensitive
切换数据库： \c dbname
显示字符集：\encoding
退出：\q
查看所有存储过程（函数）: \df
切换用户：\c 库名
```


二进制格式:

```bash
备份全库示例：
	sys_dump -h ip -p 端口 -U 用户 -W 密码 –F c -f 备份路径/xxx.dmp 库名
还原全库：
	sys_restore -h ip -p 端口 -U 用户 -W 密码 -d 库名 备份路径/xxx.dmp

备份全库示例：
	sys_dump -h ip -p 端口 -U 用户 -W 密码  -f 备份路径/xxx.sql 库名
还原全库：
	ksql  -h ip -U用户名 -W密码 -d 库名 -f  备份路径/xxx.sql


备份指定某张表加上-t参数，这个可以重复加，就可以实现备份多张表
	sys_dump -h ip -p 端口 -U 用户 -W 密码 –t 表名  –F c -f 备份路径/xxx.dmp 库名
```





全文检索

创建扩展
```sql
create extension zhparser;
CREATE TEXT SEARCH CONFIGURATION parser_zw (PARSER = zhparser); // 添加配置 
ALTER TEXT SEARCH CONFIGURATION parser_zw ADD MAPPING FOR n,v,a,i,e,l,j WITH simple; // 设置分词规则 
```
建表并插入数据：

```sql
create table test_js(id serial ,col2 clob);
```

通过应用程序将doc文件插入表中。

创建索引

```sql
create index idx_col2 on table using gin(to_tsvector('parser_zw', col2)); 
```
进行查询：

```sql
#
SELECT * from test_js where to_tsvector('parser_zw',col2) @@ to_tsquery('测试 &航天 ');
# 
SELECT * from test_js where to_tsvector('parser_zw',col2) @@ to_tsquery('测试 | 航天');
#
SELECT * from test_js where to_tsvector('parser_zw',col2) @@ to_tsquery('测试 &!航天');
```

批量清空表

```SQL
truncate table xxx;
```

试用版license过期了怎么办？

查看版本号：

    在kingbase用户下执行：kingbase -V查看产品版本号 。

下载license：

    登录kingbase官网找到微信公众号扫码，点击“试用下载”，找到相应的产品版本的license.dat。

替换license步骤：

- 第一步，登录服务器，执行：find / -name license.dat，看license.dat在哪些路径。 
    
- 第二步：把上一步找到的路径记录下来。
    
- 第三步，根据上一步的路径把原有license.dat重命名为license.dat_old或者https://bbs.kingbase.com.cn名字也行 
    
- 第四步：把下载的license.dat传到服务器，重命名为license.dat，并且执行chown -R kingabse:kingbase license.dat
    
- 第五步，执行su kingbase切换到kingbase用户，把最新的license拷贝到第二步记录下来的路径。
    
- 第六步：重启数据库：sys_ctl restart –D /home/kingbase/KingbaseES/V8/data (备注：每个地方data路径可能不一样，通过ps -ef查找出带-D的进程，确定准确的data路径)