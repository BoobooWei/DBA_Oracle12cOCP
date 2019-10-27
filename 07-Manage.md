# 07-Manage

> 2019.10.27 BoobooWei

[toc]

## 复习上一节课内容

1. 启动数据库实例
2. 查看数据库实例明细
3. 启动默认监听
4. 查看监听
5. 连接数据库服务

 ```bash
[oracle@oracle01 ~]$ sqlplus / as sysdba

SQL*Plus: Release 12.2.0.1.0 Production on Sat Oct 26 18:15:13 2019

Copyright (c) 1982, 2016, Oracle.  All rights reserved.

Connected to an idle instance.

SQL> startup
ORACLE instance started.

Total System Global Area  855638016 bytes
Fixed Size		    8798504 bytes
Variable Size		  327159512 bytes
Database Buffers	  515899392 bytes
Redo Buffers		    3780608 bytes
Database mounted.
Database opened.
SQL> show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 2 PDB$SEED			  READ ONLY  NO
	 3 BOOBOOPDB1			  READ WRITE NO
	 4 BOOBOOPDB2			  MOUNTED
	 5 BOOBOOPDB3			  MOUNTED
	 6 BOOBOOPDB4			  MOUNTED

[oracle@oracle01 ~]$ lsnrctl start

LSNRCTL for Linux: Version 12.2.0.1.0 - Production on 26-OCT-2019 18:45:28

Copyright (c) 1991, 2016, Oracle.  All rights reserved.

Starting /u01/app/oracle/product/12.2.0/db_1/bin/tnslsnr: please wait...

TNSLSNR for Linux: Version 12.2.0.1.0 - Production
System parameter file is /u01/app/oracle/product/12.2.0/db_1/network/admin/listener.ora
Log messages written to /u01/app/oracle/diag/tnslsnr/oracle01/listener/alert/log.xml
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=oracle01)(PORT=1521)))
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=ol7-122.localdomain)(PORT=1521)))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 12.2.0.1.0 - Production
Start Date                26-OCT-2019 18:46:05
Uptime                    0 days 0 hr. 0 min. 55 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Listener Parameter File   /u01/app/oracle/product/12.2.0/db_1/network/admin/listener.ora
Listener Log File         /u01/app/oracle/diag/tnslsnr/oracle01/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=oracle01)(PORT=1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
The listener supports no services
The command completed successfully
[oracle@oracle01 ~]$ lsnrctl status

LSNRCTL for Linux: Version 12.2.0.1.0 - Production on 26-OCT-2019 18:48:33

Copyright (c) 1991, 2016, Oracle.  All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=ol7-122.localdomain)(PORT=1521)))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 12.2.0.1.0 - Production
Start Date                26-OCT-2019 18:46:05
Uptime                    0 days 0 hr. 3 min. 5 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Listener Parameter File   /u01/app/oracle/product/12.2.0/db_1/network/admin/listener.ora
Listener Log File         /u01/app/oracle/diag/tnslsnr/oracle01/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=oracle01)(PORT=1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcps)(HOST=oracle01)(PORT=5500))(Security=(my_wallet_directory=/u01/app/oracle/admin/booboo/xdb_wallet))(Presentation=HTTP)(Session=RAW))
Services Summary...
Service "93227d66515623b9e0553ce49b3eaf97" has 1 instance(s).
  Instance "booboo", status READY, has 1 handler(s) for this service...
Service "932283ef357f23fbe0553ce49b3eaf97" has 1 instance(s).
  Instance "booboo", status READY, has 1 handler(s) for this service...
Service "9322894c27292419e0553ce49b3eaf97" has 1 instance(s).
  Instance "booboo", status READY, has 1 handler(s) for this service...
Service "93228eb75d13244ce0553ce49b3eaf97" has 1 instance(s).
  Instance "booboo", status READY, has 1 handler(s) for this service...
Service "booboo" has 1 instance(s).
  Instance "booboo", status READY, has 1 handler(s) for this service...
Service "boobooXDB" has 1 instance(s).
  Instance "booboo", status READY, has 1 handler(s) for this service...
Service "booboopdb1" has 1 instance(s).
  Instance "booboo", status READY, has 1 handler(s) for this service...
Service "booboopdb2" has 1 instance(s).
  Instance "booboo", status READY, has 1 handler(s) for this service...
Service "booboopdb3" has 1 instance(s).
  Instance "booboo", status READY, has 1 handler(s) for this service...
Service "booboopdb4" has 1 instance(s).
  Instance "booboo", status READY, has 1 handler(s) for this service...
The command completed successfully


=======
[oracle@oracle01 admin]$ sqlplus hr/Oracle123@oracle01:1521/booboopdb1

SQL> column  tname format a20
SQL> select * from tab;

TNAME		     TABTYPE  CLUSTERID
-------------------- ------- ----------
COUNTRIES	     TABLE
DEPARTMENTS	     TABLE
EMPLOYEES	     TABLE
EMP_DETAILS_VIEW     VIEW
JOBS		     TABLE
JOB_HISTORY	     TABLE
LOCATIONS	     TABLE
REGIONS 	     TABLE

8 rows selected.

SQL> show user;
USER is "HR"
 ```



## 管理网络

老师初步介绍网络，比较浅，只需掌握：

1. 配置监听
2. 查看监听
3. 通过监听服务连接数据库

```bash
[oracle@oracle01 ~]$ cd $ORACLE_HOME/network/admin
[oracle@oracle01 admin]$ pwd
/u01/app/oracle/product/12.2.0/db_1/network/admin
[oracle@oracle01 admin]$ ll
total 16
-rw-r-----. 1 oracle oinstall  343 Sep 22 02:02 listener.ora
drwxr-xr-x. 2 oracle oinstall   64 Sep 21 23:58 samples
-rw-r--r--. 1 oracle oinstall 1441 Aug 28  2015 shrept.lst
-rw-r-----. 1 oracle oinstall  198 Sep 22 02:02 sqlnet.ora
-rw-r-----. 1 oracle oinstall  430 Sep 22 02:34 tnsnames.ora
[oracle@oracle01 admin]$ cat listener.ora 
# listener.ora Network Configuration File: /u01/app/oracle/product/12.2.0/db_1/network/admin/listener.ora
# Generated by Oracle configuration tools.

LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = ol7-122.localdomain)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )
```

Tips-监听名字`LISTENER`为何不建议修改？

```
查看监听的命令 lsnrctl status 默认查看的就是`LISTENER`；
如果改变名称，则需要使用 lsnrctl status yourname
```





##  系统视图

### 性能视图`v$fixed_table`

### 字典视图`dict`

## 表视图 `tab`



