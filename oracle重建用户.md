# oracle 重建用户

recreateHPP.sh

```javascript

#!/bin/bash

. ~/.bash_profile

cd /home/oracle/scripts

sqlplus /nolog @ /home/oracle/scripts/setTablespaceReadWrite.sql
sqlplus /nolog @ /home/oracle/scripts/recreateHPP.sql

```

setTablespaceReadWrite.sql

```javascript

conn / as sysdba
ALTER TABLESPACE TBS_DATA read write;
exit;

```

recreateHPP.sql

```javascript

conn / as sysdba


set serveroutput on;

spool test.log

declare

  hppcount number;

begin


  for s in ( select sid, serial# from v$session where username = 'HPP') loop
 
 
    dbms_output.put_line(s.sid||','||s.serial#);

    execute immediate 'alter system kill session '''||s.sid||','||s.serial#||''' immediate';
     
  end loop;

  select count(*) into hppcount  FROM dba_users WHERE username = 'HPP';

  dbms_output.put_line('hppcount='||hppcount);

  if ( hppcount>0 ) then

     DBMS_LOCK.sleep(3);
     execute immediate 'drop user HPP cascade';    

  end if;

end;
/

CREATE USER HPP
  IDENTIFIED BY "您的密码"
  DEFAULT TABLESPACE TBS_DATA
  TEMPORARY TABLESPACE TEMP
  PROFILE DEFAULT
  ACCOUNT UNLOCK;
  -- 3 Roles for HPP 
  GRANT CONNECT TO HPP WITH ADMIN OPTION;
  GRANT DBA TO HPP WITH ADMIN OPTION;
  GRANT EXP_FULL_DATABASE TO HPP;
  ALTER USER HPP DEFAULT ROLE ALL;
  -- 6 System Privileges for HPP 
  GRANT ADMINISTER ANY SQL TUNING SET TO HPP WITH ADMIN OPTION;
  GRANT ADMINISTER DATABASE TRIGGER TO HPP WITH ADMIN OPTION;
  BEGIN
SYS.DBMS_RESOURCE_MANAGER_PRIVS.GRANT_SYSTEM_PRIVILEGE
  (GRANTEE_NAME   => 'HPP', 
   PRIVILEGE_NAME => 'ADMINISTER_RESOURCE_MANAGER',
   ADMIN_OPTION   => TRUE);
END;
/
  GRANT ADMINISTER SQL MANAGEMENT OBJECT TO HPP WITH ADMIN OPTION;
  GRANT ADMINISTER SQL TUNING SET TO HPP WITH ADMIN OPTION;
  GRANT UNLIMITED TABLESPACE TO HPP WITH ADMIN OPTION;
  -- 1 Tablespace Quota for HPP 
  ALTER USER HPP QUOTA UNLIMITED ON TBS_DATA;
  -- 1 Object Privilege for HPP 
    GRANT READ, WRITE ON DIRECTORY DATA_PUMP_DIR TO HPP;

spool off;

exit;

```