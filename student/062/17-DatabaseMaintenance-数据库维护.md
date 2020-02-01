# 实践17:数据库维护

> **Practices for Lesson 17: Database Maintenance**
>
> 2020.01.29 BoobooWei

[TOC]

## 实践17:概览

Practices for Lesson 17: Overview

## 实践17-1:数据库维护

Practice 17-1: Database Maintenance

### Overview

### Task

### Practice

### KnowledgePoint

## 实践17-2:修改PDB数据库名称

1. 将pdb 从 booboopdb1修改为 emrep

[更改PDB的全局数据库名称](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/administering-pdbs-with-sql-plus.html#GUID-2003FF3F-C403-45B7-B198-4CAF7172A4F4)

```sql
sqlplus / as sysdba
startup
show pdbs
alter session set container=booboopdb1;
show pdbs
select name,open_mode,db_unique_name from v$database;
ALTER SYSTEM ENABLE RESTRICTED SESSION;
ALTER PLUGGABLE DATABASE RENAME GLOBAL_NAME TO emrep;
ALTER SYSTEM DISABLE RESTRICTED SESSION;
SHUTDOWN IMMEDIATE
STARTUP OPEN
show pdbs
```



运行结果

```sql
[oracle@ocm ~]$ sqlplus / as sysdba

SQL*Plus: Release 12.2.0.1.0 Production on Sat Feb 1 19:16:16 2020

Copyright (c) 1982, 2016, Oracle.  All rights reserved.

Connected to an idle instance.
--启动cdb
SQL> startup
ORACLE instance started.

Total System Global Area  855638016 bytes
Fixed Size		    8798504 bytes
Variable Size		  327159512 bytes
Database Buffers	  511705088 bytes
Redo Buffers		    7974912 bytes
Database mounted.
Database opened.

--查看pdb
SQL> show pdbs

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 2 PDB$SEED			  READ ONLY  NO
	 3 BOOBOOPDB1			  READ WRITE NO
	 4 BOOBOOPDB2			  MOUNTED
	 5 BOOBOOPDB3			  MOUNTED
	 6 BOOBOOPDB4			  MOUNTED

--切换到容器booboopdb1	 
SQL> alter session set container=booboopdb1;

Session altered.

SQL> show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 3 BOOBOOPDB1			  READ WRITE NO

--查看pdb中的db_name参数
SQL> show parameter db_name;

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
db_name 			     string	 booboo

--查看pdb中的数据库名
SQL> select name,open_mode,db_unique_name from v$database;

NAME	  OPEN_MODE	       DB_UNIQUE_NAME
--------- -------------------- ------------------------------
BOOBOO	  READ WRITE	       booboo
--尝试直接修改pdb的数据库名，发现报错
SQL> ALTER PLUGGABLE DATABASE RENAME GLOBAL_NAME TO emrep;
ALTER PLUGGABLE DATABASE RENAME GLOBAL_NAME TO emrep
*
ERROR at line 1:
ORA-65045: pluggable database not in a restricted mode

--修改pdb的数据库模式为RESTRICTED SESSION
SQL> ALTER SYSTEM ENABLE RESTRICTED SESSION;

System altered.
--修改pdb的数据库名成功
SQL> ALTER PLUGGABLE DATABASE RENAME GLOBAL_NAME TO emrep;

Pluggable database altered.
--关闭RESTRICTED SESSION
SQL> ALTER SYSTEM DISABLE RESTRICTED SESSION;

System altered.
--关闭pdb
SQL> SHUTDOWN IMMEDIATE
Pluggable Database closed.
--启动pdb
SQL> STARTUP
Pluggable Database opened.
--查看pdb
SQL> show pdbs

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 3 EMREP			  READ WRITE NO
--切换到cdb
SQL> alter session set container=cdb$root;

Session altered.
--查看pdbs
SQL> show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 2 PDB$SEED			  READ ONLY  NO
	 3 EMREP			  READ WRITE NO
	 4 BOOBOOPDB2			  MOUNTED
	 5 BOOBOOPDB3			  MOUNTED
	 6 BOOBOOPDB4			  MOUNTED
```

## 实践17-3:修改CDB的数据库名称

将cdb 从 booboo 修改为 emcdb

```sql
--查看当前dbname和dbid；并将数据库启动到mount状态
sqlplus / as sysdba
select name,dbid from v$database;
shutdown immediate
startup mount
--nid命令同时修改dbname 和 dbid
nid target=sys/oracle dbname=emcdb
--将数据库启动到nomount状态；修改db_name;强制启动到mount状态
sqlplus / as sysdba
startup nomount
alter system set db_name='emcdb' scope=spfile;
startup force mount
--关闭数据库实例修改环境变量
shutdown immediate

mv $ORACLE_HOME/dbs/orapwbooboo $ORACLE_HOME/dbs/orapwemcdb
mv $ORACLE_HOME/dbs/spfilebooboo.ora $ORACLE_HOME/dbs/spfileemcdb.ora
--启动数据库实例
startup
alter database open resetlogs;
```

执行结果

```sql
SQL> alter session set container=cdb$root;

Session altered.

SQL> show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 2 PDB$SEED			  READ ONLY  NO
	 3 EMREP			  READ WRITE NO
	 4 BOOBOOPDB2			  MOUNTED
	 5 BOOBOOPDB3			  MOUNTED
	 6 BOOBOOPDB4			  MOUNTED
SQL> select name,dbid from v$database;

NAME		DBID
--------- ----------
BOOBOO	  3416647573

SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.
SQL> startup mount
ORACLE instance started.

Total System Global Area  855638016 bytes
Fixed Size		    8798504 bytes
Variable Size		  327159512 bytes
Database Buffers	  511705088 bytes
Redo Buffers		    7974912 bytes
Database mounted.
SQL> exit
Disconnected from Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
[oracle@ocm ~]$ which nid
/u01/app/oracle/product/12.2.0/db_1/bin/nid
[oracle@ocm ~]$ nid tartget=sys/oracle dbname=emcdb

DBNEWID: Release 12.2.0.1.0 - Production on Sat Feb 1 19:53:07 2020

Copyright (c) 1982, 2017, Oracle and/or its affiliates.  All rights reserved.


NID-00002: Parse error: LRM-00101: unknown parameter name 'tartget'


Change of database ID failed during validation - database is intact.
DBNEWID - Completed with validation errors.

[oracle@ocm ~]$ nid target=sys/oracle dbname=emcdb

DBNEWID: Release 12.2.0.1.0 - Production on Sat Feb 1 19:53:21 2020

Copyright (c) 1982, 2017, Oracle and/or its affiliates.  All rights reserved.

Connected to database BOOBOO (DBID=3416647573)

Connected to server version 12.2.0

Control Files in database:
    /u01/app/oracle/oradata/booboo/control01.ctl
    /u01/app/oracle/oradata/booboo/control02.ctl

Change database ID and database name BOOBOO to EMCDB? (Y/[N]) => y

Proceeding with operation
Changing database ID from 3416647573 to 785286577
Changing database name from BOOBOO to EMCDB
    Control File /u01/app/oracle/oradata/booboo/control01.ctl - modified
    Control File /u01/app/oracle/oradata/booboo/control02.ctl - modified
    Datafile /u01/app/oracle/oradata/booboo/system01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/sysaux01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/undotbs01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/pdbseed/system01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/pdbseed/sysaux01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/users01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/pdbseed/undotbs01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/booboopdb1/system01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/booboopdb1/sysaux01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/booboopdb1/undotbs01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/booboopdb1/users01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/booboopdb2/system01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/booboopdb2/sysaux01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/booboopdb2/undotbs01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/booboopdb2/users01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/booboopdb3/system01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/booboopdb3/sysaux01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/booboopdb3/undotbs01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/booboopdb3/users01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/booboopdb4/system01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/booboopdb4/sysaux01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/booboopdb4/undotbs01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/booboopdb4/users01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/temp01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/pdbseed/temp012019-09-22_02-06-21-856-AM.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/booboopdb1/temp01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/booboopdb2/temp01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/booboopdb3/temp01.db - dbid changed, wrote new name
    Datafile /u01/app/oracle/oradata/booboo/booboopdb4/temp01.db - dbid changed, wrote new name
    Control File /u01/app/oracle/oradata/booboo/control01.ctl - dbid changed, wrote new name
    Control File /u01/app/oracle/oradata/booboo/control02.ctl - dbid changed, wrote new name
    Instance shut down

Database name changed to EMCDB.
Modify parameter file and generate a new password file before restarting.
Database ID for database EMCDB changed to 785286577.
All previous backups and archived redo logs for this database are unusable.
Database has been shutdown, open database with RESETLOGS option.
Succesfully changed database name and ID.
DBNEWID - Completed succesfully.

[oracle@ocm ~]$ sqlplus / as sysdba

SQL*Plus: Release 12.2.0.1.0 Production on Sat Feb 1 19:54:38 2020

Copyright (c) 1982, 2016, Oracle.  All rights reserved.

Connected to an idle instance.

SQL> startup nomount
ORACLE instance started.

Total System Global Area  855638016 bytes
Fixed Size		    8798504 bytes
Variable Size		  327159512 bytes
Database Buffers	  511705088 bytes
Redo Buffers		    7974912 bytes
SQL> alter system set db_name='emcdb' scope=spfile;

System altered.

SQL> startup force mount
ORACLE instance started.

Total System Global Area  855638016 bytes
Fixed Size		    8798504 bytes
Variable Size		  327159512 bytes
Database Buffers	  511705088 bytes
Redo Buffers		    7974912 bytes
Database mounted.
SQL> select name,dbid from v$database;

NAME		DBID
--------- ----------
EMCDB	   785286577
SQL> shutdown immediate
ORA-01109: database not open


Database dismounted.
ORACLE instance shut down.
SQL> exit
Disconnected from Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
[oracle@ocm ~]$ vim ~/scripts/setEnv.sh 
[oracle@ocm ~]$ source ~/scripts/setEnv.sh 
[oracle@ocm ~]$ echo $ORACLE_SID
emcdb
[oracle@ocm ~]$ cd $ORACLE_HOME/dbs
[oracle@ocm dbs]$ ll
total 36
-rw-rw----. 1 oracle oinstall 1544 Feb  1 19:59 hc_booboo.dat
-rw-rw----  1 oracle oinstall 1544 Feb  1 19:15 hc_emcdb.dat
-rw-r--r--  1 oracle oinstall   14 Feb  1 18:55 initemcdb.ora
-rw-r--r--. 1 oracle oinstall 3079 May 15  2015 init.ora
-rw-r-----. 1 oracle oinstall   24 Sep 22 17:03 lkBOOBOO
-rw-r-----  1 oracle oinstall   24 Dec  8 16:31 lkEMCDB
-rw-r-----. 1 oracle oinstall 3584 Nov 17 14:03 orapwbooboo
-rw-r-----  1 oracle oinstall 2048 Dec  8 16:30 orapwemcdb
-rw-r-----. 1 oracle oinstall 3584 Feb  1 19:57 spfilebooboo.ora
[oracle@ocm dbs]$ mv $ORACLE_HOME/dbs/orapwbooboo $ORACLE_HOME/dbs/orapwemcdb
[oracle@ocm dbs]$ mv $ORACLE_HOME/dbs/spfilebooboo.ora $ORACLE_HOME/dbs/spfileemcdb.ora 
[oracle@ocm dbs]$ ll
total 32
-rw-rw----. 1 oracle oinstall 1544 Feb  1 19:59 hc_booboo.dat
-rw-rw----  1 oracle oinstall 1544 Feb  1 19:15 hc_emcdb.dat
-rw-r--r--  1 oracle oinstall   14 Feb  1 18:55 initemcdb.ora
-rw-r--r--. 1 oracle oinstall 3079 May 15  2015 init.ora
-rw-r-----. 1 oracle oinstall   24 Sep 22 17:03 lkBOOBOO
-rw-r-----  1 oracle oinstall   24 Dec  8 16:31 lkEMCDB
-rw-r-----. 1 oracle oinstall 3584 Nov 17 14:03 orapwemcdb
-rw-r-----. 1 oracle oinstall 3584 Feb  1 19:57 spfileemcdb.ora
[oracle@ocm ~]$ sqlplus / as sysdba

SQL*Plus: Release 12.2.0.1.0 Production on Sat Feb 1 20:03:32 2020

Copyright (c) 1982, 2016, Oracle.  All rights reserved.

Connected to an idle instance.

SQL> startup
ORACLE instance started.

Total System Global Area  855638016 bytes
Fixed Size		    8798504 bytes
Variable Size		  327159512 bytes
Database Buffers	  511705088 bytes
Redo Buffers		    7974912 bytes
Database mounted.
ORA-01589: must use RESETLOGS or NORESETLOGS option for database open

SQL> alter database open resetlogs;

Database altered.

SQL> select name,dbid,open_mode from v$database;

NAME		DBID OPEN_MODE
--------- ---------- --------------------
EMCDB	   785286577 READ WRITE

SQL> show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 2 PDB$SEED			  READ ONLY  NO
	 3 EMREP			  MOUNTED
	 4 BOOBOOPDB2			  MOUNTED
	 5 BOOBOOPDB3			  MOUNTED
	 6 BOOBOOPDB4			  MOUNTED

```

