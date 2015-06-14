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

编辑 ~/.bash_profile

```javascript
# Oracle Settings
TMP=/tmp; export TMP
TMPDIR=$TMP; export TMPDIR

ORACLE_HOSTNAME=centos101-oracle; export ORACLE_HOSTNAME   <--- 一定要跟你的主机名一样
ORACLE_UNQNAME=DB11G; export ORACLE_UNQNAME
ORACLE_BASE=/u01/app/oracle; export ORACLE_BASE
ORACLE_HOME=$ORACLE_BASE/product/11.2.0/db_1; export ORACLE_HOME
ORACLE_SID=DB11G; export ORACLE_SID

PATH=/usr/sbin:$PATH; export PATH
PATH=$ORACLE_HOME/bin:$PATH; export PATH

LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib; export LD_LIBRARY_PATH
CLASSPATH=$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib; export CLASSPATH
```
编辑完后执行：  . ~/.bash_profile  <--环境变量马上生效，不需要重新登录

2、开始安装

2.1 修改 linux_11gR2_database_1of2.zip,linux_11gR2_database_2of2.zip 的文件宿主为oracle

切回root用户，然后

chown oracle:oinstall linux_11gR2_database_*of2.zip

2.2 切换到oracle 用户，解压 linux_11gR2_database_1of2.zip,linux_11gR2_database_2of2.zip

unzip linux_11gR2_database_1of2.zip
unzip linux_11gR2_database_2of2.zip