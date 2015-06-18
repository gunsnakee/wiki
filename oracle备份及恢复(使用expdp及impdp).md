# oracle备份及恢复（使用expdp及impdp）

## oracle备份

cronExpdp.sh 

```javascript

#!/bin/bash
. ~/.bash_profile

# 要跟 DATA_PUMP_DIR 一致
export EXPORT_FOLDER=/oracle/product/11R2/rdbms/log

DATE=$(date +"%Y%m%d")
FNAME=$DATE-db11g_export

cd $EXPORT_FOLDER

$ORACLE_HOME/bin/expdp \"/ as sysdba\" schemas=hpp directory=DATA_PUMP_DIR \
dumpfile=$FNAME.dmp logfile=$FNAME.log \
flashback_time=SYSTIMESTAMP

tar -cjf $FNAME.tar.bz2 $FNAME.dmp $FNAME.log

scp $FNAME.tar.bz2 oracle@10.255.1.10:/home/oracle/backup/192.168.100.102

sleep 3

ssh oracle@10.255.1.10 "/home/oracle/scripts/recreateSchema.sh";

sleep 2

rm $FNAME.dmp $FNAME.log

find $EXPORT_FOLDER/*db11g_export.tar.bz2 -mtime +3 -delete

```

## oracle 恢复

recreateSchema.sh

```javascript

#!/bin/bash

. ~/.bash_profile

cd /home/oracle/scripts

DATE=$(date +"%Y%m%d")
FNAME=$DATE-db11g_export

DIR_BACKUP=/home/oracle/backup/192.168.100.102
# 要跟 DATA_PUMP_DIR 一致
DIR_DATA_PUMP=/u01/app/oracle/admin/DB11G/dpdump

if [ -f $DIR_BACKUP/$FNAME.tar.bz2 ] 
  then
    cd $DIR_BACKUP
    tar -jxvf $DIR_BACKUP/$FNAME.tar.bz2
    sleep 1
    cp $DIR_BACKUP/$FNAME.dmp $DIR_DATA_PUMP
    sleep 1
    /home/oracle/scripts/recreateHPP.sh
    impdp \"/ as sysdba\" schemas=HPP directory=DATA_PUMP_DIR dumpfile=$FNAME.dmp logfile=$FNAME_impdp.log
    sqlplus /nolog @/home/oracle/scripts/setTablespaceReadOnly.sql
    rm $DIR_BACKUP/$FNAME.dmp
    rm $DIR_BACKUP/$FNAME.log
    rm $DIR_DATA_PUMP/$FNAME.dmp
    
    find $DIR_BACKUP/*db11g_export.tar.bz2 -mtime +15 -delete
fi

```

setTablespaceReadOnly.sql

```javascript

conn / as sysdba
ALTER TABLESPACE TBS_DATA read only;
exit;

``` 
