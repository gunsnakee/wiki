# wiki

知识经验的总结和记录

## Oracle 11g Release 2 在CentOS 6.6 32bit 的安装过程（静默方式 silent）

1、准备

Oracle 11g安装文件： linux_11gR2_database_1of2.zip,linux_11gR2_database_2of2.zip (存放到/tmp目录)

2、开始安装

2.1 修改Linux主机名和/etc/hosts

```javascript

vi /etc/sysconfig/network

增加或者修改主机名称： HOSTNAME=centos101-oracle

ifconig -a 查看本机ip，并记住这个ip,然后 vi /etc/hosts

添加： <ip> <hostname> ， 例如我的是：  192.168.137.101 centos101-oracle

用 hostname 检查主机名是否正确，用 ping <主机名> 查看是否ping通，并解析到自己的ip。

