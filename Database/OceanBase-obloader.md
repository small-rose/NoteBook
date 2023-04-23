---
layout: default
title: OceanBase Exp/Imp
nav_order: 61
parent: Database
---
# OceanBase export/import
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## ob环境导入导出使用

### obclient和 obloader安装

#### 1、上传安装包

两个obclient的包

- libobclient-2.1.2-20211201104607.el7.alios7.x86_64_rpm
- obclient-2.1.2-20220507120550.el7.alios7.x86_64_rpm

1个obloader-obdumper的包

- loader-obdumper-4.0.1-SNAPSHOT.zip


如上传到 /home/dcos/data 目录。

#### 2、安装obclient

**安装：**

> 安装需要root权限。且依赖jdk1.8或以上版本。

```bash
cd /home/dcos/data
# 先安装依赖
rpm -ivh libobclient-2.1.2-20211201104607.el7.alios7.x86_64.rpm
rpm -ivh obclient-2.1.2-20220507120550.el7.alios7.x86_64.rpm


rpm -ivh --prefix=/data/obclient  obclient-2.1.2-20220507120550.el7.alios7.x86_64.rpm

rpm -ivh --prefix=/data/jdk1.8  /home/dcos/data/jdk-8u341-linux-x64.rpm
cd /data/jdk1.8
mv *  jdk1.8.0_341
/data/jdk1.8/jdk1.8.0_341/bin/java -version
```

>应用不是1.8，也需要按照jdk1.8，本身是jdk1.8的环境，jdk可省。

如果安装错了，也可以卸载

```bash
rpm -e libobclient-2.1.2-20211201104607.el7.alios7.x86_64_rpm
rpm -e obclient-2.1.2-20220507120550.el7.alios7.x86_64_rpm
```

也可以安装到自己指定的目录。记得配置环境变量。

**验证：**

执行连接串命令进入控制台。

```bash
# 有环境变量
obclient -h192.168.88.123  -P2883 -uAMSDB01@amsdb01#cx_xc_obcluster -p123456

# 无环境变量
cd /data/obclient
./obclient -h192.168.88.123  -P2883 -uAMSDB01@amsdb01#cx_xc_obcluster -p123456
```

> 命令串格式：oclient -h连接IP地址 -P连接端口 -u

参数含义如下：
- -h：提供 OceanBase 数据库连接的 IP，通常是一个 OBProxy 地址。
- -u：提供租户的连接帐号，格式包含两种： 用户名@租户名#集群名 或者 集群名:租户名:用户名。Oracle 租户的管理员用户名默认是 sys。
- -P：提供 OceanBase 数据库连接端口，也是 OBProxy 的监听端口，默认是 2883，可以自定义。
- -p：提供帐号密码。为了安全可以不提供，改为在后面提示符下输入，密码文本不可见。
- -A：表示在连接数据库时不去获取全部表信息，可以使登录数据库速度最快。

登录成功之后可以执行查询

```sql
select sysdate from dual ;
```

#### 3、安装ob-loader-obdumper

```bash
cd /home/dcos/data
# 解压到当前文件夹
unzip ob-loader-obdumper-4.0.1-SNAPSHOT.zip

# 解压到当前文件夹
unzip -d /data  ob-loader-obdumper-4.0.1-SNAPSHOT.zip
# 写完整路径，就可以在任意目录执行
unzip -d /data  /home/dcos/data/ob-loader-obdumper-4.0.1-SNAPSHOT.zip
```

解压即可。

> 注意使用obloader/obdumper 需要jdk1.8的环境。
> 
> 如果是**容器化部署**。可以将 obclient的安装路径和 ob-loader-dumper路径映射到容器中使用。
>
> 如果容器中的应用低于jdk1.8，也需要将jdk的安装环境映射到容器中。导入导出可以在shell脚本中使用这边路径作为环境变量。
>
> 示例如下
> > -v /data/obclient:/app/obclient
> >
> > -v /data/ob-loader-obdumper-4.0.1-SNAPSHOT:/app/ob-loader-obdumper-4.0.1-SNAPSHOT
> > 
> > -v /data/jdk1.8/jdk1.8.0_341:/app/jdk1.8.0_341

### 导入导出使用

### 导出

方式一：直接命令

```bash
cd /data/ob-loader-obdumper-4.0.1-SNAPSHOT

./obdumper-debug -h29.30.130.208 -P2883 -u AMSDB01 -t amsdb01 -c cx_xc_obcluster -p '@5EDcft%7' -D AMSDB01 -f /home/admin/data --cut --column-splitter '||' --query-sql "SELECT ID||'||'||VARCHAR_COL||'||'||CHAR_COL||'||'||CLOB_COL||'||'||RAW_COL||'||'||DATE_COL||'||'||TIMESTAMP_COL||'||'||FLOAT_COL FROM ALL_DATA_TYPES" --table ALL_DATA_TYPES --no-sys
```
> 此种方式要求安装jdk1.8环境且配置好环境变量，obclient环境变量。否则会提示找不到命令。

方式二： shell脚本配合使用

一般通过脚本文件操作，`vi dumper_test.sh`

```bash
#!/bin/bash

echo "参数1" $1
echo "参数2" $2
echo "参数3" $4

# 路径可以通过参数传递
JAV_HOME=/data/jdk1.8/jdk1.8.0_341
OBC_HOME=/data/obclient
OBLD_HOME=/data/ob-loader-obdumper-4.0.1-SNAPSHOT

export JAVA_HOME=$JAV_HOME;
export JRE_HOME=${JAVA_HOME}/jre;
export CLASS_PATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib;
export OBCLIENT_HOME=$OBC_HOME;
export PATH=${JAVA_HOME}/bin:${OBCLIENT_HOME}/bin:${PATH};

# 定义OB_DUMPER路径,
OB_DUMPER=${OBLD_HOME}/bin/obdumper

#检查 OBCLIENT_HOME 是否存在
if [! -d ${OBCLIENT_HOME} ]; then
    echo "OBCLIENT_HOME目录未找到！退出脚本！"
    exit 1 ;
fi

#检查 OB_DUMPER 是否存在
if [! -d ${OB_DUMPER} ]; then
    echo "OB_DUMPER 目录未找到！退出脚本！"
    exit 1 ;
fi

HOST=192.168.130.208
PORT=2883
USER=AMSDB01
PASSWORD=123456Pwd
DATABASE=AMSDB01
TENANT=amsdb01
OBCLUSTER=cx_xc_obcluster


client_param=-h${HOST} -P${PORT} -u${USER} -p ${PASSWORD} -c ${OBCLUSTER} -t ${TENANT} -D ${DATABASE}
DUMPER_PARAM=

v_data_log=$4/log
v_data_file=$4/export_fen

notime='date +%Y%m%d`;
v_filesuffix=.txt;
v_file_flag=_;

v_count=0；

v_subcompay=$(obclient ${client_param} -N -e 'select t.subcompany from ams_subcompany_tc where 1=1 and exists (select 1 from ams_department_tc p where p.subcompany = t.subcompany)')

echo company: ${v_subcompany}

for i in ${v_subcompany}
do
echo sub $i

v_count=$(obclient ${client_param} -N -e 'select count(1) from ams_department_tc p where p.subcompany = to_char('$i')')

if [ ${v_count} -ne 0 ] 
then
    echo "分公司"$i"的数据条数":
    echo count:'${v_count}'
    
v_now = 'date +%Y%m%d%H%M';
v_name=$v_data_file/$iv_file_flag$v_nowv_filesuffix

#执行导出命令
$OB_DUMPER ${DUMPER} --non-sys --trail-delimiter -f ${v_data_file} --file-name ${v_name} --skip-check-dir --log-path ${v_data_log} --query-sql "select 
departmentcode|| '||' ||
subcompany|| '||' ||
realldept|| '||' ||
departmentname|| '||' ||
timestamp|| '||' ||
orgcode|| '||' ||
deptgroupcode|| '||' ||
deptgroupname|| '||' ||
taxpayercode|| '||' ||
taxmanagementcode|| '||' ||
status|| '||' ||
ifvalid
from ams_department_tc t where t.subcompany = to_char($i)" --cut --column-splitter '\\||'

fi

okfile=${v_data_file}/$i$v_now.ok
echo okfile ${okfile}

# 输出ok文件
if [ ${v_count} -ne 0 ]; then
    echo $v_now\;${v_count}>${okfile};
fi

done
```


### 导入

控制文件: `vi AMS_DEPARTMENT_TC.ctrl`

```text
lang=java
(
DEPARTMENTCODE,
SUBCOMPANY,
REALLDEPT,
DEPARTMENTNAME,
TIMESTAMP,
ORGCODE,
DEPTGROUPCODE,
DEPTGROUPNAME,
TAXPAYERCODE,
TAXMANAGEMENTCODE,
STATUS,
IFVALID
);
```

>几个要注意的点：
>
>1、表名大写；2、控制文件名大写,小写的控制文件名不起作用。

```bash
#!/bin/bash

echo 'importdepartment shell start '
fileprerfix=$(cd `dirname $0`; pwd)
cd fileprerfix
echo fileprerfix : ${fileprerfix}

echo "参数1" $1
echo "参数2" $2
echo "参数3" $4

# 路径可以通过参数传递
JAV_HOME=/data/jdk1.8/jdk1.8.0_341
OBC_HOME=/data/obclient
OBLD_HOME=/data/ob-loader-obdumper-4.0.1-SNAPSHOT

export JAVA_HOME=$JAV_HOME;
export JRE_HOME=${JAVA_HOME}/jre;
export CLASS_PATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib;
export OBCLIENT_HOME=$OBC_HOME;
export PATH=${JAVA_HOME}/bin:${OBCLIENT_HOME}/bin:${PATH};

# 定义OB_DUMPER路径,
OB_DUMPER=${OBLD_HOME}/bin/obdumper

#检查 OBCLIENT_HOME 是否存在
if [! -d ${OBCLIENT_HOME} ]; then
    echo "OBCLIENT_HOME目录未找到！退出脚本！"
    exit 1 ;
fi

#检查 OB_DUMPER 是否存在
if [! -d ${OB_DUMPER} ]; then
    echo "OB_DUMPER 目录未找到！退出脚本！"
    exit 1 ;
fi

HOST=192.168.130.208
PORT=2883
USER=AMSDB01
PASSWORD=123456Pwd
DATABASE=AMSDB01
TENANT=amsdb01
OBCLUSTER=cx_xc_obcluster

TABLE=AMS_DEPARTMENT_TC
OUTPUT_FIEL=/home/dcos/data/file



client_param=-h${HOST} -P${PORT} -u${USER} -p ${PASSWORD} -c ${OBCLUSTER} -t ${TENANT} -D ${DATABASE}
DUMPER_PARAM=

PRIDIR=$4/export
LOGDIR=$4/log
BADDIR=$4/bad
BACKDIR=$4/backup

nowtime=`date +%Y%m%d`;
echo 'file:importing...'

echo ${PRIDIR}

echo 'get subcompany:'%5

filecount=$(ls $PRIDIR/$5${nowtime}*.ok 2> /dev/null | wc -l)
if [ ${filecount} -ne 0 ]; then

echo 'start insert AMS_DEPARTMENT_TC ...'
echo '数据库连接是:'
echo database:${DBALIAS}

subcompany_value=$5
ret_delete=$(obclient ${client_param} -N -e 'delete from AMS_DEPARTMENT_TC where subcompany = to_char('$subcompany_value');commit;')

echo '输入删除的数据'
echo ret+delete:${ret_delete}
CONTROL_FILE=$fileprerfix/ctrl/AMS_DEPARTMENT_TC.ctrl

for FILE in $PRIDIR/$5_${nowtime}.txt
do
#执行导入
echo LOADER : ${LOADER}
echo FILE : ${FILE}

$OB_LOADER ${LOADER} -f ${FILE} --no-sys --log-path ${LOGDIR} --column-splitter '||' --file-suffix '.txt' --table  AMS_DEPARTMENT_TC --ctl-path ${CONTROL_FILE} --external-data --cut

mv $PRIDIR/$5_${nowtime}.txt  /${BACKDIR}
mv $PRIDIR/$5${nowtime}.ok  /${BACKDIR}
echo 'move the file back dir end!'

done

echo 'insert AMS_DEPARTMENT_TC end '
fi

echo 'file:imported'
```