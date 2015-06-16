# wiki

知识经验的总结和记录

## Oracle 11g Release 2 在CentOS 6.6 32bit 的安装过程（静默方式 silent）

1、准备

Oracle 11g安装文件： linux_11gR2_database_1of2.zip,linux_11gR2_database_2of2.zip (存放到/tmp目录)

1.1 修改Linux主机名和/etc/hosts

```javascript

vi /etc/sysconfig/network

增加或者修改主机名称： HOSTNAME=centos101-oracle

ifconig -a 查看本机ip，并记住这个ip,然后 vi /etc/hosts

添加： <ip> <hostname> ， 例如我的是：  192.168.137.101 centos101-oracle

用 hostname 检查主机名是否正确，用 ping <主机名> 查看是否ping通，并解析到自己的ip。

```

1.2 编辑 /etc/sysctl.conf

```javascript
fs.suid_dumpable = 1
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmall = 2097152
kernel.shmmax = 536870912
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default=262144
net.core.rmem_max=4194304
net.core.wmem_default=262144
net.core.wmem_max=1048586
```
运行 /sbin/sysctl -p 使修改生效。

1.3 编辑 /etc/security/limits.conf

```javascript
oracle              soft    nproc   16384
oracle              hard    nproc   16384
oracle              soft    nofile  4096
oracle              hard    nofile  65536
oracle              soft    stack   10240
```

vi /etc/pam.d/login (在文件最后增加或修改以下参数)  

session required /lib/security/pam_limits.so    <--- 注意是32位还是64位的系统

vi /etc/profile (在文件最后增加或修改以下脚本)  

if [ $USER = "oracle" ]; then  
if [ $SHELL = "/bin/ksh" ]; then  
ulimit -p 16384  
ulimit -n 65536  
else  
ulimit -u 16384 -n 65536  
fi  
fi 

1.4 安装软件包

```javascript
yum update

yum groupinstall 'Development Tools'

yum install binutils.i686 binutils-devel.i686
yum install glibc.i686 glibc-devel.i686 glibc-common.i686 glibc-headers.i686
yum install nss-softokn-freebl.i686 nss-softokn-freebl-devel.i686
yum install compat-libstdc++-33.i686
yum install elfutils-libelf.i686 elfutils-libelf-devel.i686
yum install gcc.i686
yum install gcc-c++.i686
yum install ksh.i686
yum install libaio.i686 libaio-devel.i686
yum install libgcc.i686
yum install libstdc++.i686 libstdc++-devel.i686
yum install make.i686
yum install numactl-devel.i686
yum install sysstat.i686
yum install compat-libcap1.i686
```

1.5 创建用户和用户组

```javascript
groupadd -g 501 oinstall
groupadd -g 502 dba
groupadd -g 503 oper
groupadd -g 504 asmadmin
groupadd -g 506 asmdba
groupadd -g 505 asmoper

useradd -u 502 -g oinstall -G dba,asmdba,oper oracle

passwd oracle <--- 修改oracle用户密码
```
1.6 修改 /etc/security/limits.d/90-nproc.conf

```javascript

# 把这个
 *          soft    nproc    1024

# 修改为
* - nproc 16384

```

1.7 修改 /etc/selinux/config

```javascript
SELINUX=permissive 或者 disabled
```

1.8 创建Oracle必要的文件目录

```javascript
mkdir -p /u01/app/oracle/product/11.2.0/db_1
chown -R oracle:oinstall /u01
chmod -R 775 /u01
```

1.9 设置oracle用户的环境变量

su oracle

创建清单文件目录

mkdir -p /home/oracle/inventory

编辑 ~/.bash_profile

```javascript
# Oracle Settings
umask 022
TMP=/tmp; export TMP
TMPDIR=$TMP; export TMPDIR

ORACLE_HOSTNAME=centos101-oracle; export ORACLE_HOSTNAME   <--- 一定要跟你的主机名一样
ORACLE_UNQNAME=DB11G; export ORACLE_UNQNAME
ORACLE_BASE=/u01/app/oracle; export ORACLE_BASE
ORACLE_HOME=$ORACLE_BASE/product/11.2.0/db_1; export ORACLE_HOME
ORACLE_SID=DB11G; export ORACLE_SID

PATH=/usr/sbin:$PATH; export PATH
PATH=$ORACLE_HOME/bin:$PATH; export PATH

LC_ALL="en_US"; export  LC_ALL
LANG="en_US"; export LANG 
NLS_LANG="AMERICAN_AMERICA.ZHS16GBK"; export  NLS_LANG
NLS_DATE_FORMAT="YYYY-MM-DD HH24:MI:SS"; export NLS_DATE_FORMAT 

LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib; export LD_LIBRARY_PATH
CLASSPATH=$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib; export CLASSPATH
```
编辑完后执行：  . ~/.bash_profile  

最好reboot一下系统。


2、开始安装

2.1 修改 linux_11gR2_database_1of2.zip,linux_11gR2_database_2of2.zip 的文件宿主为oracle

切回root用户，然后

chown oracle:oinstall linux_11gR2_database_*of2.zip

2.2 切换到oracle 用户，解压 linux_11gR2_database_1of2.zip,linux_11gR2_database_2of2.zip

unzip linux_11gR2_database_1of2.zip

unzip linux_11gR2_database_2of2.zip

2.3 编辑db_install.rsp 

cd /tmp/database

cp response/db_install.rsp ./

```javascript

oracle.install.responseFileVersion=/oracle/install/rspfmt_dbinstall_response_schema_v11_2_0
oracle.install.option=INSTALL_DB_SWONLY
ORACLE_HOSTNAME=hpp-backup
UNIX_GROUP_NAME=oinstall
INVENTORY_LOCATION=/home/oracle/inventory
SELECTED_LANGUAGES=en
ORACLE_HOME=/u01/app/oracle/product/11.2.0/db_1
ORACLE_BASE=/u01/app/oracle
oracle.install.db.InstallEdition=EE
oracle.install.db.isCustomInstall=true
oracle.install.db.customComponents=oracle.server:11.2.0.1.0,oracle.sysman.ccr:10.2.7.0.0,oracle.xdk:11.2.0.1.0,oracle.rdbms.oci:11.2.0.1.0,oracle.network:11.2.0.1.0,oracle.network.listener:11.2.0.1.0,oracle.rdbms:11.2.0.1.0,oracle.options:11.2.0.1.0,oracle.rdbms.partitioning:11.2.0.1.0,oracle.oraolap:11.2.0.1.0,oracle.rdbms.dm:11.2.0.1.0,oracle.rdbms.dv:11.2.0.1.0,orcle.rdbms.lbac:11.2.0.1.0,oracle.rdbms.rat:11.2.0.1.0
oracle.install.db.DBA_GROUP=dba
oracle.install.db.OPER_GROUP=oper
oracle.install.db.config.starterdb.type=GENERAL_PURPOSE
oracle.install.db.config.starterdb.globalDBName=DB11G
oracle.install.db.config.starterdb.SID=DB11G
oracle.install.db.config.starterdb.characterSet=AL32UTF8
oracle.install.db.config.starterdb.memoryOption=true
oracle.install.db.config.starterdb.installExampleSchemas=false
oracle.install.db.config.starterdb.enableSecuritySettings=true
oracle.install.db.config.starterdb.password.ALL=abcd1234
oracle.install.db.config.starterdb.control=DB_CONTROL
oracle.install.db.config.starterdb.automatedBackup.enable=false
oracle.install.db.config.starterdb.storageType=ASM_STORAGE
DECLINE_SECURITY_UPDATES=true

```

各参数具体含义可参考 http://loofeer.blog.51cto.com/707932/1119713

 ./runInstaller -silent -responseFile /tmp/database/db_install.rsp 
 
2.6 安装数据库实例

cd /tmp/database

cp response/dbca.rsp ./

vi dbca.rsp

```javascript

RESPONSEFILE_VERSION = "11.2.0"
OPERATION_TYPE = "createDatabase"
GDBNAME = "DB11G"
SID = "DB11G"
TEMPLATENAME = "General_Purpose.dbc"
SYSPASSWORD = "abcd1234"
SYSTEMPASSWORD = "abcd1234"


```

dbca -silent -cloneTemplate -responseFile /tmp/database/dbca.rsp

2.6 创建监听器

cd /tmp/database

cp response/netca.rsp ./

vi netca.rsp

```javascript

RESPONSEFILE_VERSION="11.2"
CREATE_TYPE="CUSTOM"
INSTALLED_COMPONENTS={"server","net8","javavm"}
INSTALL_TYPE=""custom""
LISTENER_NUMBER=1
LISTENER_NAMES={"LISTENER"}
LISTENER_PROTOCOLS={"TCP;1521"}
LISTENER_START=""LISTENER""
NAMING_METHODS={"TNSNAMES","ONAMES","HOSTNAME"}
NSN_NUMBER=1
NSN_NAMES={"EXTPROC_CONNECTION_DATA"}
NSN_SERVICE={"PLSExtProc"}
NSN_PROTOCOLS={"TCP;HOSTNAME;1521"}

```

netca /silent /responseFile /tmp/database/netca.rsp

2.7 修改 /etc/oratab

DB11G:/u01/app/oracle/product/11.2.0/db_1:Y

2.8 启动监听，启动实例

lsnrctl start
dbstart $ORACLE_HOME

2.9 为系统添加服务

touch /etc/init.d/oracle

chmod 755 /etc/init.d/oracle

vi /etc/init.d/oracle

```javascript

#!/bin/bash
# chkconfig: 35 90 10
# description: Oracle Database Service Daemon.
ORCL_BASE="/u01/app/oracle"
ORACLE_HOME=/u01/app/oracle/product/11.2.0/db_1
ORACLE_OWNER=oracle
case "$1" in
 start)
    su - $ORACLE_OWNER -c "$ORACLE_HOME/bin/lsnrctl start"             
    su - $ORACLE_OWNER -c "$ORACLE_HOME/bin/dbstart $ORACLE_HOME"      
    su - $ORACLE_OWNER -c "$ORACLE_HOME/bin/emctl start dbconsole"     
    ;;
 stop)
    su - $ORACLE_OWNER -c "$ORACLE_HOME/bin/emctl stop dbconsole"      
    su - $ORACLE_OWNER -c "$ORACLE_HOME/bin/dbshut $ORACLE_HOME"       
    su - $ORACLE_OWNER -c "$ORACLE_HOME/bin/lsnrctl stop"              
    ;;
 status)
    if(pgrep "tnslsnr" && netstat -anpt | grep ":1521") &> /dev/null
    then
        echo "Oracle 11g Net Listener is running."
    else
        echo "Oracle 11g Net Listener is not running."
    fi
    if(netstat -anpt | grep ":1158" && netstat -anpt | grep ":5520") &> /dev/null
    then
        echo "Oracle 11g Enterprise Manager is running."
    else
        echo "Oracle 11g Enterprise Manager is not running."
    fi
    ;;
 restart)
    $0 stop
    $0 start
    ;;
 *)
    echo "Usage: $0 {start|stop|restart|status}"
    exit 1
    ;;
esac
exit 0

```

service oracle restart

chkconfig --add oracle