# 实践2:体系结构

> **Practices for Lesson 2: Exploring Oracle Database Architecture**
>
> 2020.01.29 BoobooWei

[TOC]

## 实践2:概览

Practices for Lesson 2: Overview

## 实践2-1:探索Oracle数据库体系结构

Practice 2-1: Exploring the Oracle Database Architecture


## 实践2-2:关闭数据库实例transactional

### Overview

本实验的目的，了解关闭数据库实例的过程，实验中将使用`shutdown transactional`来关闭数据库。

### Task

### Practice

1. 会话1：登陆pdb，执行insert 不提交事务

```sql
[oracle@oracle01 ~]$ sqlplus / as sysdba
SYS@booboo>conn scott/tiger@booboopdb1
Connected.
SCOTT@booboopdb1>desc t02;
 Name						       Null?	Type
 ----------------------------------------------------- -------- ------------------------------------
 X						       NOT NULL NUMBER(38)
 Y								VARCHAR2(20)

SCOTT@booboopdb1>insert into t02 values (1,'superman');

1 row created.

SCOTT@booboopdb1>insert into t02 values (2,'batman');

1 row created.

```

2. 会话2：登陆cdb，执行create user c##test identified by oracle;

```sql
[oracle@oracle01 ~]$ sqlplus / as sysdba

SQL*Plus: Release 12.2.0.1.0 Production on Thu Feb 6 15:56:20 2020

Copyright (c) 1982, 2016, Oracle.  All rights reserved.


Connected to:
Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production

SYS@booboo>show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 2 PDB$SEED			  READ ONLY  NO
	 3 BOOBOOPDB1			  READ WRITE NO
	 4 BOOBOOPDB2			  MOUNTED
	 5 BOOBOOPDB3			  MOUNTED
	 6 BOOBOOPDB4			  MOUNTED
SYS@booboo>create user c##test identified by oracle;

User created.

```

3. 会话3：登陆cdb，执行 shutdown transactional 观察日志

```sql
[oracle@oracle01 ~]$ sqlplus / as sysdba

SQL*Plus: Release 12.2.0.1.0 Production on Thu Feb 6 16:17:12 2020

Copyright (c) 1982, 2016, Oracle.  All rights reserved.


Connected to:
Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production

SYS@booboo>shutdown transactional
Database closed.
Database dismounted.
ORACLE instance shut down.
SYS@booboo>!date
Thu Feb  6 16:19:20 CST 2020
```

观察日志
```bash
[oracle@oracle01 ~]$ adrci 

ADRCI: Release 12.2.0.1.0 - Production on Thu Feb 6 16:18:05 2020

Copyright (c) 1982, 2017, Oracle and/or its affiliates.  All rights reserved.

ADR base = "/u01/app/oracle"
adrci> help        

 HELP [topic]
   Available Topics:
        CREATE REPORT
        ECHO
        ESTIMATE
        EXIT
        HELP
        HOST
        IPS
        PURGE
        RUN
        SELECT
        SET BASE
        SET BROWSER
        SET CONTROL
        SET ECHO
        SET EDITOR
        SET HOMES | HOME | HOMEPATH
        SET TERMOUT
        SHOW ALERT
        SHOW BASE
        SHOW CONTROL
        SHOW HM_RUN
        SHOW HOMES | HOME | HOMEPATH
        SHOW INCDIR
        SHOW INCIDENT
        SHOW LOG
        SHOW PROBLEM
        SHOW REPORT
        SHOW TRACEFILE
        SPOOL

 There are other commands intended to be used directly by Oracle, type
 "HELP EXTENDED" to see the list

adrci> show alert

Choose the home from which to view the alert log:

1: diag/rdbms/booboo/booboo
2: diag/clients/user_oracle/host_2874269298_107
3: diag/tnslsnr/oracle01/listener
4: diag/tnslsnr/oracle01/listener2
Q: to quit

Please select option: 1
Output the results to file: /tmp/alert_43088_1406_booboo_1.ado

Please select option: q       
adrci> exit
```

日志分析

```bash
2020-02-06 16:17:14.624000 +08:00
Shutting down instance (transactional) (OS id: 42991)
## 开始：down instance (transactional)


2020-02-06 16:17:15.816000 +08:00
Stopping background process SMCO # 停止后台进程 SMCO
2020-02-06 16:17:17.028000 +08:00
Shutting down instance: further logons disabled #禁用进一步的登录
2020-02-06 16:17:18.147000 +08:00
Stopping background process CJQ0  # 停止后台进程 CJQ0
Stopping background process MMNL  # 停止后台进程 MMNL
2020-02-06 16:17:19.219000 +08:00
Stopping background process MMON  # 停止后台进程 MMON
2020-02-06 16:17:20.900000 +08:00


alter pluggable database all close immediate # 开始关闭pdb【immediate】：只产生检查点，不等待事务结果
JIT: pid 42991 requesting stop
KILL SESSION for sid=(67, 40381):
  Reason = PDB close immediate
  Mode = KILL HARD FORCE -/-/-
  Requestor = USER (orapid = 33, ospid = 42991, inst = 1)
  Owner = Process: USER (orapid = 43, ospid = 41840)
  Result = ORA-0
2020-02-06 16:17:22.565000 +08:00
Pluggable database BOOBOOPDB1 closed # booboopdb1 该PDB已成功关闭。
Completed: alter pluggable database all close immediate # 完成关闭pdb【immediate】
## 完成：关闭pdb all close immediate


JIT: pid 42991 requesting stop
All transactions complete. Performing immediate shutdown # 所有事务完成，开始执行【immediate】 shutdown
License high water mark = 17
Dispatchers and shared servers shutdown # 调度程序和共享服务器关闭
2020-02-06 16:17:24.676000 +08:00
ALTER DATABASE CLOSE NORMAL
Stopping Emon pool # 停止Emon池
alter pluggable database all close immediate # ALL PDB已成功关闭。
Completed: alter pluggable database all close immediate # 完成关闭pdb【immediate】
## 完成：关闭pdb all close immediate


Stopping Emon pool # 停止Emon池
Shutting down archive processes # 【开始】关闭归档过程
TT00: Gap Manager exiting (PID:100073)
OS process OFSD (ospid 100019) idle for 30 seconds, exiting
2020-02-06 16:17:25.809000 +08:00
Archiving is disabled # 【完成】关闭归档过程
Thread 1 closed at log sequence 14
Successful close of redo thread 1
Completed: ALTER DATABASE CLOSE NORMAL
## 完成：关闭 DATABASE NORMAL


ALTER DATABASE DISMOUNT # dismount 数据库
Shutting down archive processes # 关闭归档进程
Archiving is disabled
Completed: ALTER DATABASE DISMOUNT

## 完成：关闭 DATABASE DISMOUNT


2020-02-06 16:17:27.001000 +08:00
ARCH: Archival disabled due to shutdown: 1089
Shutting down archive processes
Archiving is disabled
JIT: pid 42991 requesting stop
2020-02-06 16:17:28.008000 +08:00
ARCH: Archival disabled due to shutdown: 1089
Shutting down archive processes
Archiving is disabled
Stopping background process VKTM
JIT: pid 42991 requesting stop
2020-02-06 16:17:37.473000 +08:00
Instance shutdown complete (OS id: 42991)
## 完成：Instance shutdown
```

1. 开始：down instance (transactional)
2. 完成：关闭pdb all close immediate(booboopdb1)
3. 完成：关闭pdb all close immediate(all)
4. 完成：关闭 DATABASE NORMAL
5. 完成：关闭 DATABASE DISMOUNT
6. 完成：Instance shutdown



重新启动数据库后，发现事务并没有提交

```sql

SYS@booboo>conn scott/tiger@booboopdb1
Connected.
SCOTT@booboopdb1>select * from t02;

no rows selected
```

### KnowledgePoint

![](pic/0201.png)

注意：虽然发起的关闭是 transactional ，但是在Oracle12c中，关闭PDB时使用的却是`alter pluggable database all close immediate`。因此不会等待PDB的事务提交即立刻关闭数据库。

## 实践2-3:关闭数据库实例abort

### Overview

本实验的目的，了解关闭数据库实例的过程，实验中将使用`shutdown abort`来关闭数据库。

### Task

### Practice

1. 关闭实例

   ```sql
   SYS@booboo>shutdown abort
   ORACLE instance shut down.
   SYS@booboo>!date
   Thu Feb  6 18:13:24 CST 2020
   ```

   

2. 查看日志

   ```bash
   Shutting down instance (abort) (OS id: 44076)
   License high water mark = 6
   USER (ospid: 44076): terminating the instance
   Instance terminated by USER, pid = 44076
   2020-02-06 18:13:18.716000 +08:00
   Instance shutdown complete (OS id: 44076)
   ```

### KnowledgePoint

关闭速度非常快。

## 实践2-4:隐藏列 invisible

oracle 12c 新特性之不可见字段

在Oracle 11g R1中，Oracle以不可见索引和虚拟字段的形式引入了一些不错的增强特性。继承前者并发扬光大，Oracle 12c 中引入了不可见字段思想。在之前的版本中，为了隐藏重要的数据字段以避免在通用查询中显示，我们往往会创建一个视图来隐藏所需信息或应用某些安全条件。
在12c中，你可以在表中创建不可见字段。当一个字段定义为不可见时，这一字段就默认不会出现在通用查询中，除非在SQL语句或条件中有显式的提及这一字段，或是在表定义中有DESCRIBED。要添加或是修改一个不可见字段是非常容易的，反之亦然。

```sql
CREATE TABLE test_p (prod_id number(4),
Prod_name varchar2 (20),
Category_id number(30),
Quantity_on_hand number (3) INVISIBLE);

desc test_p
alter table test_p add constraint test_01  unique (Quantity_on_hand);
alter table test_p modify(Quantity_on_hand visible);
alter table test_p modify(Quantity_on_hand sible);
```

## 实践2-5:存储过程DR和IR

如何防止对CREATE_TEST存储过程具有EXECUTE特权的用户将值插入他们没有任何特权的表中？

Create the CREATE_TEST procedure with invoker rights.

oracle存储过程分两种，DR(Definer's Rights ) Procedure和IR(Invoker's Rights ) Procedure。
定义存储过程时，通过指定AUTHID 属性，定义DR Procedure 和IR Procedure

1. 定义者权限：定义者权限PL/SQL程序单元是以这个程序单元拥有者的特权来执行它的，也就是说，任何具有这个PL/SQL程序单元执行权的用户都可以访问程序中的对象。所有具有执行权的用户都有相同的访问权限，在定义者权限下，执行的用户操作的schema为定义者，所操作的对象是定义者在编译时指定的对象。在定义者(definer)权限下，当前用户的权限为角色无效情况下所拥有的权限。

   ```sql
   CREATE OR REPLACE procedure DEMO(ID in NUMBER) 
   AUTHID DEFINER as
   ...
   BEGIN
   ...
   ND DEMO
   ```

   

2. 调用者权限：调用者权限是指当前用户（而不是程序的创建者）执行PL/SQL程序体的权限。这意味着不同的用户对于某个对象具有的权限很可能是不同的，这个思想的提出，解决了不同用户更新不同表的方法。在调用者权限下，执行的用户操作的schema为当前用户，所操作的对象是当前模式下的对象。在调用者(invoker)权限下，当前用户的权限为当前所拥有的权限(含角色)。

   ```sql
   CREATE OR REPLACE procedure DEMO(ID in NUMBER) 
   AUTHID CURRENT_USER  as
   ...
   BEGIN
   ...
   ND DEMO
   ```

ORACLE默认为定义者权限，定义者权限在存储过程中ROLE无效，需要显示授权，例如在存储过程中调用其他用户的表，但是定义存储过程的当前用户没有显示访问该表的权限，即使当前用户具有dba角色，编译过程中也会出现权限不足的问题，因为role无效。

## 实践2-6:参数DB_8K_CACHE_SIZE

查看参数

```sql
select * from V$SYSTEM_PARAMETER where name='db_8k_cache_size';

exec print_table(q'[select * from V$SYSTEM_PARAMETER WHERE name='db_8k_cache_size']')
```

执行结果

```sql
NUM			      : 1447
NAME			      : db_8k_cache_size
TYPE			      : 6
VALUE			      : 0
DISPLAY_VALUE		      : 0
DEFAULT_VALUE		      : 0
ISDEFAULT		      : TRUE
ISSES_MODIFIABLE	      : FALSE
ISSYS_MODIFIABLE	      : IMMEDIATE
ISPDB_MODIFIABLE	      : FALSE
ISINSTANCE_MODIFIABLE	      : TRUE
ISMODIFIED		      : FALSE
ISADJUSTED		      : FALSE
ISDEPRECATED		      : FALSE
ISBASIC 		      : FALSE
DESCRIPTION		      : Size of cache for 8K buffers
UPDATE_COMMENT		      :
HASH			      : 1564706367
CON_ID			      : 0
-----------------

PL/SQL procedure successfully completed.
```

* ISSYS_MODIFIABLE	      : IMMEDIATE
* 指示参数是否可以更改`ALTER SYSTEM`以及更改何时生效：`IMMEDIATE`- `ALTER SYSTEM`无论用于启动实例的参数文件的类型如何，都可以更改参数。更改将立即生效。
	 DESCRIPTION		      : Size of cache for 8K buffers 8K缓冲区的缓存大小 

## 实践2-7:参数ENABLE_DDL_LOGGING

当需要确定某些ddl操作什么时候发生，确定是表啥时间被动了，可以设置此参数。

```sql
--此参数支持动态调整
alter system set enable_ddl_logging=true scope=both;
show parameter enable_ddl_logging;
drop procedure print_table;
CREATE OR REPLACE PROCEDURE print_table(p_query IN VARCHAR2)
AUTHID CURRENT_USER
IS
 l_thecursor INTEGER DEFAULT dbms_sql.open_cursor;
 l_columnvalue VARCHAR2(4000);
 l_status  INTEGER;
 l_desctbl  dbms_sql.desc_tab;
 l_colcnt  NUMBER;
BEGIN
 EXECUTE IMMEDIATE 'alter session set nls_date_format=''dd-mon-yyyy hh24:mi:ss'' ';

 dbms_sql.parse(l_thecursor, p_query, dbms_sql.native);

 dbms_sql.describe_columns (l_thecursor, l_colcnt, l_desctbl);

 FOR i IN 1 .. l_colcnt LOOP
  dbms_sql.define_column (l_thecursor, i, l_columnvalue, 4000);
 END LOOP;

 l_status := dbms_sql.EXECUTE(l_thecursor);

 WHILE ( dbms_sql.Fetch_rows(l_thecursor) > 0 ) LOOP
  FOR i IN 1 .. l_colcnt LOOP
   dbms_sql.column_value (l_thecursor, i, l_columnvalue);

   dbms_output.Put_line (RPAD(L_desctbl(i).col_name, 30)
         || ': '
         || l_columnvalue);
  END LOOP;

  dbms_output.put_line('-----------------');
 END LOOP;

 EXECUTE IMMEDIATE 'alter session set nls_date_format=''dd-MON-rr'' ';
EXCEPTION
 WHEN OTHERS THEN
    EXECUTE IMMEDIATE
    'alter session set nls_date_format=''dd-MON-rr'' ';

    RAISE;
END;
/
exec print_table('select 1 from dual')
```

运行结果

* DDL日志明细xml格式文件：`less /u01/app/oracle/diag/rdbms/emcdb/emcdb/log/ddl/log.xml`
* DDL日志明细TXT格式文件：`less /u01/app/oracle/diag/rdbms/emcdb/emcdb/log/ddl_emcdb.log`

```sql
[oracle@ocm log]$ pwd
/u01/app/oracle/diag/rdbms/emcdb/emcdb/log
[oracle@ocm log]$ ll
total 8
drwxr-x--- 2 oracle oinstall   21 Feb  7 19:31 ddl
-rw-r----- 1 oracle oinstall 1217 Feb  7 19:32 ddl_emcdb.log
drwxr-x--- 2 oracle oinstall   21 Feb  2 19:57 debug
-rw-r----- 1 oracle oinstall  888 Feb  5 02:57 debug.log
drwxr-x--- 2 oracle oinstall    6 Dec  8 16:30 hcs
drwxr-x--- 2 oracle oinstall    6 Dec  8 16:30 imdb
drwxr-x--- 2 oracle oinstall    6 Dec  8 16:30 test
[oracle@ocm log]$ head ddl_emcdb.log 
2020-02-07T19:31:14.248644+08:00
diag_adl:drop procedure print_table
2020-02-07T19:32:01.698631+08:00
diag_adl:CREATE OR REPLACE PROCEDURE print_table(p_query IN VARCHAR2)
AUTHID CURRENT_USER
IS
 l_thecursor INTEGER DEFAULT dbms_sql.open_cursor;
 l_columnvalue VARCHAR2(4000);
 l_status  INTEGER;
 l_desctbl  dbms_sql.desc_tab;
[oracle@ocm log]$ head ddl/log.xml 
<msg time='2020-02-07T19:31:14.248+08:00' org_id='oracle' comp_id='rdbms'
 msg_id='opiexe:4695:2946163730' type='UNKNOWN' group='diag_adl'
 level='16' host_id='ocm' host_addr='192.168.14.154'
 pid='20083' version='1' con_uid='1'
 con_id='1' con_name='CDB$ROOT'>
 <txt>drop procedure print_table
 </txt>
</msg>
<msg time='2020-02-07T19:32:01.698+08:00' org_id='oracle' comp_id='rdbms'
 msg_id='opiexe:4695:2946163730' type='UNKNOWN' group='diag_adl'
```



### KnowledgePoint

> ENABLE_DDL_LOGGING

https://docs.oracle.com/en/database/oracle/oracle-database/12.2/refrn/ENABLE_DDL_LOGGING.html

| Property            | Description                     |
| ------------------- | ------------------------------- |
| Parameter type      | Boolean                         |
| Default value       | `false`                         |
| Modifiable          | `ALTER SESSION`, `ALTER SYSTEM` |
| Modifiable in a PDB | Yes                             |
| Range of values     | `true | false`                  |
| Basic               | No                              |

* `ENABLE_DDL_LOGGING` enables or disables the writing of a subset of data definition language (DDL) statements to a DDL log. 
* DDL日志是与警报日志具有相同格式和基本行为的文件，但它仅包含数据库发出的DDL语句。仅当RDBMS组件创建DDL日志，并且`ENABLE_DDL_LOGGING`初始化参数设置为`true`。当此参数设置`false`为时，DDL语句不包含在任何日志中。
* 对于数据库发出的每个DDL语句，DDL日志均包含一个日志记录。DDL日志包含在IPS事件包中。
* 有两个DDL日志包含相同的信息。一个是XML文件，另一个是文本文件。DDL日志存储在`log/ddl`ADR主目录的子目录中。

当`ENABLE_DDL_LOGGING`设置`true`为时，将以下DDL语句写入日志：

- `ALTER/CREATE/DROP/TRUNCATE CLUSTER`
- `ALTER/CREATE/DROP FUNCTION`
- `ALTER/CREATE/DROP INDEX`
- `ALTER/CREATE/DROP OUTLINE`
- `ALTER/CREATE/DROP PACKAGE`
- `ALTER/CREATE/DROP PACKAGE BODY`
- `ALTER/CREATE/DROP PROCEDURE`
- `ALTER/CREATE/DROP PROFILE`
- `ALTER/CREATE/DROP SEQUENCE`
- `CREATE/DROP SYNONYM`
- `ALTER/CREATE/DROP/RENAME/TRUNCATE TABLE`
- `ALTER/CREATE/DROP TRIGGER`
- `ALTER/CREATE/DROP TYPE`
- `ALTER/CREATE/DROP TYPE BODY`
- `DROP USER`
- `ALTER/CREATE/DROP VIEW`

## 实践2-8:Oracle Data Redaction

### Overview

Oracle Data Redaction 提供了一种简单方法来快速编写应用程序中显示的敏感信息, 而无需变更磁盘或高速缓存中的基础数据库块。数据根据灵活的多因素策略实时编写。作为 Oracle Advanced Security 的一部分授权数据编写。

本实验将学习如何配置一个数据编写策略并验证。

### Task

1. 使用DBA1用户，创建一个表regions
2. 登陆到企业云管理平台，点击 **安全 > 数据编辑**，新建一个数据编辑策略。
3. 使用scott用户访问dba1的regions表，查看是否生效
4. 使用hr用户访问dba1的regions表，查看是否生效

### Practice

1. 使用DBA1用户，创建一个表regions

   ```sql
   SQL> conn dba1/oracle@emrep
   Connected.
   SQL> desc regions;
    Name					   Null?    Type
    ----------------------------------------- -------- ----------------------------
    REGION_ID				   NOT NULL NUMBER
    REGION_NAME				   NOT NULL VARCHAR2(25)
   
   SQL> select * from regions;
   
    REGION_ID REGION_NAME
   ---------- -------------------------
   	 1 Europe
   	 2 Americas
   	 3 Asia
   	 4 Middle East and Africa
   	 5 192.168.1.1
   	 
   SQL> grant select on regions to hr;
   SQL> grant select on regions to scott;
   ```

   

2. 登陆到企业云管理平台，点击 **安全 > 数据编辑 > 创建策略**

   要求，当scott用户访问`dba1.regions`表时策略才会生效，具体策略如下表，含义为：如果region_name列匹配到ip地址，则变更更最后一位为999；否则不显示。

   | col        | value                                            |
   | ---------- | ------------------------------------------------ |
   | schema     | DBA1                                             |
   | 表/视图    | REGIONS                                          |
   | 策略名     | dba1_regions_exp                                 |
   | 策略表达式 | SYS_CONTEXT('USERENV', 'SESSION_USER') = 'SCOTT' |

   | 对象列      | 列数据类型 | 编写函数 | 函数属性                                            |
   | ----------- | ---------- | -------- | --------------------------------------------------- |
   | REGION_NAME | VARCHAR2   | REGEX    | `(\d{1,3}\.\d{1,3}\.\d{1,3})\.\d{1,3},\1.999,1,1,i` |

   ![](pic/0202.png)

   ![](pic/0203.png)

   ```sql
   BEGIN
   
   	 BEGIN  DBMS_REDACT.ADD_POLICY  (OBJECT_SCHEMA => 'DBA1',  object_name => 'REGIONS',  policy_name => 'dba1_regions_exp',  expression => 'SYS_CONTEXT(''USERENV'', ''SESSION_USER'') = ''SCOTT'''); END; 
   
   	BEGIN  DBMS_REDACT.ALTER_POLICY  (OBJECT_SCHEMA => 'DBA1',  object_name => 'REGIONS',  policy_name => 'dba1_regions_exp',  action => DBMS_REDACT.ADD_COLUMN,  column_name => '"REGION_NAME"',  function_type => DBMS_REDACT.REGEXP , regexp_pattern         => (\d{1,3}\.\d{1,3}\.\d{1,3})\.\d{1,3},regexp_replace_string  => \1.999,regexp_position        => 1,regexp_occurrence      => 1,regexp_match_parameter => i);  END; 
   
   END;
   /
   ```

   

3. 使用scott用户访问dba1的regions表，查看是否生效

   ```sql
   SQL> conn scott/tiger@emrep
   Connected.
   SQL> column region_name format a25
   SQL> select * from dba1.regions;
   
    REGION_ID REGION_NAME
   ---------- -------------------------
   	 1
   	 2
   	 3
   	 4
   	 5 192.168.1.999
   ```

   id为5的行 region_name只显示了为ip的行，且对最后一个地址做了替换。说明成功了。

4. 使用hr用户访问dba1的regions表，查看是否生效

    ```sql
       SQL> conn hr/hr@emrep;
       Connected.
       SQL> column region_name format a25
       SQL> select * from dba1.regions;
       
    REGION_ID REGION_NAME
    ```
---------- -------------------------
   	 1 Europe
   	 2 Americas
   	 3 Asia
   	 4 Middle East and Africa
   	 5 192.168.1.1
    ```

   可以看到规则对hr用户没有生效。

### KnowledgePoint

- [Oracle Data Redaction简介](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/asoag/introduction-to-oracle-data-redaction.html#GUID-82EA9712-387C-4D3A-BB72-F64A707C67CA)
  Oracle Data Redaction是实时编辑敏感数据的功能。
- [Oracle Data Redaction特性和功能](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/asoag/oracle-data-redaction-features-and-capabilities.html#GUID-98B40084-EA10-4265-A6E5-D2EAC934E005)
  Oracle Data Redaction提供了多种编辑不同类型数据的方法。
- [配置Oracle数据修订策略](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/asoag/configuring-oracle-data-redaction-policies.html#GUID-78C0ECD1-388C-4E12-87C3-32FBC05F294D)
  Oracle数据修订策略根据表列类型和要使用的修订类型定义如何对列中的数据进行修订。
- [在Oracle Enterprise Manager中管理Oracle Data Redaction策略](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/asoag/using-oracle-data-redaction-in-oracle-enterprise-manager.html#GUID-8267183A-1778-428E-9B6E-AB06D8CC7EAB)
  Oracle Enterprise Manager Cloud Control（云控制）可以管理Oracle Data Redaction策略和格式。
- [将Oracle Data Redaction与Oracle数据库功能](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/asoag/oracle-data-redaction-use-with-oracle-database-features.html#GUID-D0C97997-0F35-42B7-98B1-1DA4197001F0)
  一起使用Oracle Data Redaction可以与其他Oracle功能一起使用，但是某些Oracle功能可能对Oracle Data Redaction有限制。
- [Oracle Data Redaction的安全注意事项](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/asoag/security-considerations-for-using-oracle-data-redaction.html#GUID-1D2850E8-2485-4C3D-BBF2-75E2CFAC3F37)
  Oracle提供了使用Oracle Data Redaction的准则。

## 实践2-9:REDACTION_VALUES_FOR_TYPE_FULL

### Overview

通过DBMS_REDACT.REDACTION_VALUES_FOR_TYPE_FULL 此过程修改“数据修订”策略的数值默认显示值以进行完整修订。

### Task

1. 修改redaction_values_for_type_full的NUMBER_VALUE为-1（pdb中操作）
2. 登陆云管理控制台添加策略，scott用户访问dba1.employees表时，看不到真实的salary。看到的都是-1。
3. 使用scott用户测试是否策略生效。
4. 重启服务
5. 使用scott用户测试是否策略生效。

### Practice

1. 修改redaction_values_for_type_full的NUMBER_VALUE为-1

   ```sql
   SQL> conn / as sysdba
   Connected.
   SQL>SQL> conn / as sysdba
   Connected.
   SQL> alter session set container=emrep;
   
   Session altered.
   
   SQL> select number_value from redaction_values_for_type_full;
   
   NUMBER_VALUE
   ------------
   	   0
   
   SQL> exec dbms_redact.update_full_redaction_values(-1)
   
   PL/SQL procedure successfully completed.
   
   SQL> select number_value from redaction_values_for_type_full;
   
   NUMBER_VALUE
   ------------
   	  -1
   
    select number_value from redaction_values_for_type_full;
   
   NUMBER_VALUE
   ------------
   	   0
   
   SQL> exec dbms_redact.update_full_redaction_values(-1)
   
   PL/SQL procedure successfully completed.
   
   SQL> select number_value from redaction_values_for_type_full;
   
   NUMBER_VALUE
   ------------
   	  -1
   ```

   

2. 登陆云管理控制台添加策略

   ![](pic/0204.png)

   ![](pic/0205.png)

   ```sql
   BEGIN
   
   	 BEGIN  DBMS_REDACT.ADD_POLICY  (OBJECT_SCHEMA => 'DBA1',  object_name => 'EMPLOYEES',  policy_name => 'dba1_employees_exp',  expression => 'SYS_CONTEXT(''USERENV'', ''SESSION_USER'') = ''SCOTT'''); END; 
   
   	BEGIN  DBMS_REDACT.ALTER_POLICY  (OBJECT_SCHEMA => 'DBA1',  object_name => 'EMPLOYEES',  policy_name => 'dba1_employees_exp',  action => DBMS_REDACT.ADD_COLUMN,  column_name => '"SALARY"',  function_type => DBMS_REDACT.FULL );  END; 
   
   END;
   ```

   

3. 使用scott用户测试是否策略生效。

   ```sql
   SQL> conn scott/tiger@emrep;
   Connected.
   SQL> desc dba1.employees;
    Name					   Null?    Type
    ----------------------------------------- -------- ----------------------------
    EMPLOYEE_ID				   NOT NULL NUMBER(6)
    FIRST_NAME					    VARCHAR2(20)
    LAST_NAME				   NOT NULL VARCHAR2(25)
    EMAIL					   NOT NULL VARCHAR2(25)
    PHONE_NUMBER					    VARCHAR2(20)
    HIRE_DATE				   NOT NULL DATE
    JOB_ID 				   NOT NULL VARCHAR2(10)
    SALARY 					    NUMBER(8,2)
    COMMISSION_PCT 				    NUMBER(2,2)
    MANAGER_ID					    NUMBER(6)
    DEPARTMENT_ID					    NUMBER(4)
   
   SQL> select employee_id,first_name,salary from dba1.employees where rownum < 5;
   
   EMPLOYEE_ID FIRST_NAME		     SALARY
   ----------- -------------------- ----------
   	198 Donald			  0
   	199 Douglas			  0
   	200 Jennifer			  0
   	201 Michael			  0
   ```

   发现salary显示的时0而不是-1。

4. 重启服务

   ```sql
   SQL> shutdown immediate
   Database closed.
   Database dismounted.
   ORACLE instance shut down.
   SQL> startup
   ORACLE instance started.
   
   Total System Global Area  838860800 bytes
   Fixed Size		    8798312 bytes
   Variable Size		  490737560 bytes
   Database Buffers	  331350016 bytes
   Redo Buffers		    7974912 bytes
   Database mounted.
   Database opened.
   
   ```

   

5. 使用scott用户测试是否策略生效。

   ```sql
   SQL> conn scott/tiger@emrep;
   Connected.
   SQL> select employee_id,first_name,salary from dba1.employees where rownum < 5;
   
   EMPLOYEE_ID FIRST_NAME		     SALARY
   ----------- -------------------- ----------
   	198 Donald			 -1
   	199 Douglas			 -1
   	200 Jennifer			 -1
   	201 Michael			 -1
   
   SQL> conn hr/hr@emrep;
   Connected.
   SQL> select employee_id,first_name,salary from dba1.employees where rownum < 5;
   
   EMPLOYEE_ID FIRST_NAME		     SALARY
   ----------- -------------------- ----------
   	198 Donald		       2600
   	199 Douglas		       2600
   	200 Jennifer		       4400
   	201 Michael		      13000
   ```

   重启后已生效，数字类型的salary列全部显示为 -1。

### KnowledgePoint

[提示与技巧：Oracle Database 12*c* 中的数据编辑](https://www.oracle.com/technetwork/cn/articles/database/data-redaction-odb12c-2331480-zhs.html)

#### 简介

Oracle Data Redaction 是 Oracle Database 12c 引入的一个新特性。这个新特性是 Advanced Security 选件的一部分，可以实时保护显示给用户的数据，无需更改应用。

Oracle Database 12c 在执行查询时应用保护。存储的数据保持不变，要显示的数据则在离开数据库前动态转换。

不要将此特性与自版本 11g 起提供的 Oracle Data Masking 混淆。采用 Oracle Data Masking 时，将以屏蔽形式处理数据，更新后的数据存储在新数据块中。因此，Data Masking 更适合非生产环境。

下面是已有的其他一些有助于提高数据安全性的特性：

- 虚拟专用数据库 (VPD) — 通过向针对数据库执行的 SQL 语句动态添加谓词，控制行级和列级访问。
- Oracle Label Security — 允许向表记录添加用户定义的值。与 VPD 相结合，可以更精细地控制访问用户和访问内容。
- Database Vault — 数据编辑不能防止特权用户（如 DBA）访问受保护的数据。为解决此问题，您可以利用 Database Vault。

 

从许可角度来说，Oracle Data Masking 仅适用于企业版数据库，且需要 Advanced Security 许可。

#### 工作原理

我们可以创建编辑策略，指定必须满足哪些条件后才能对数据进行编辑并将其返回给用户。定义此类策略期间，DBA 可以指定必须对哪些列应用何种类型的保护。

用于创建保护规则的软件包名为 **DBMS_REDACT**。它包括五个用于管理规则的过程，以及一个用于更改完全编辑策略默认值的过程。

- **DBMS_REDACT.ALTER_POLICY** — 允许更改现有策略。
- **DBMS_REDACT.DISABLE_POLICY** — 禁用现有策略。
- **DBMS_REDACT.DROP_POLICY** — 删除现有策略。
- **DBMS_REDACT.ENABLE_POLICY** — 启用现有策略。
- **DBMS_REDACT.UPDATE_FULL_REDACTION_VALUES** — 更改完全编辑的默认返回值。必须重启数据库才能生效。

#### DBMS_REDACT.REDACTION_VALUES_FOR_TYPE_FULL

* [DBMS_REDACT](https://docs.oracle.com/database/121/ARPLS/d_redact.htm#ARPLS73800)
* [REDACTION_VALUES_FOR_TYPE_FULL](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/refrn/REDACTION_VALUES_FOR_TYPE_FULL.html#GUID-7C9711A8-C3FA-413E-90A4-5E875FFAB870) 

DBMS_REDACT.REDACTION_VALUES_FOR_TYPE_FULL 此过程将修改“数据修订”策略的默认显示值以进行完整修订。

句法

```
DBMS_REDACT.UPDATE_FULL_REDACTION_VALUES（
   number_val IN NUMBER：= NULL，
   binfloat_val IN BINARY_FLOAT：= NULL，
   bindouble_val IN BINARY_DOUBLE：= NULL，
   char_val IN CHAR：= NULL，
   varchar_val IN VARCHAR2：= NULL，
   nchar_val IN NCHAR：= NULL，
   nvarchar_val IN NVARCHAR2：= NULL，
   date_val IN DATE：= NULL，
   ts_val IN TIMESTAMP：= NULL，
   tswtz_val在带有时区的TIMESTAMP中：= NULL，
   blob_val IN BLOB：= NULL，
   clob_val IN CLOB：= NULL，
   nclob_val IN NCLOB NULL）;
```

参量

表121-11 UPDATE_FULL_REDACTION_VALUES过程参数

| 参数            | 描述                                             |
| --------------- | ------------------------------------------------ |
| `number_val`    | 修改`NUMBER`数据类型列的默认值                   |
| `binfloat_val`  | 修改`BINARY_FLOAT`数据类型列的默认值             |
| `bindouble_val` | 修改`BINARY_DOUBLE`数据类型列的默认值            |
| `char_val`      | 修改`CHAR`数据类型列的默认值                     |
| `varchar_val`   | 修改`VARCHAR2`数据类型列的默认值                 |
| `nchar_val`     | 修改`NCHAR`数据类型列的默认值                    |
| `nvarchar_val`  | 修改`NVARCHAR2`数据类型列的默认值                |
| `date`          | 修改`DATE`数据类型列的默认值                     |
| `ts_val`        | 修改`TIMESTAMP`数据类型列的默认值                |
| `tswtz_val`     | 修改`TIMESTAMP WITH TIME ZONE`数据类型列的默认值 |
| `blob_val`      | 修改`BLOB`数据类型列的默认值                     |
| `clob_val`      | 修改`CLOB`数据类型列的默认值                     |
| `nclob_val`     | 修改`NCLOB`数据类型列的默认值                    |

 

例外情况

`ORA-28082`-参数参数是无效的（其中，可能的值是`char_val`，`nchar_val`，`varchar_val`和`nvarchar_val`）

#### BINARY_DOUBLE_VALUE

例如，如果将修订策略应用于类型为的列，`BINARY_DOUBLE`并且修订类型为完全修订，则该列将使用`BINARY_DOUBLE_VALUE`此视图的列中显示的值进行编辑。

| 列                               | 数据类型                      | 空值       | 描述                                               |
| -------------------------------- | ----------------------------- | ---------- | -------------------------------------------------- |
| `NUMBER_VALUE`                   | `NUMBER`                      | `NOT NULL` | 对NUMBER列进行完全修订的修订结果                   |
| `BINARY_FLOAT_VALUE`             | `BINARY_FLOAT`                | `NOT NULL` | 对BINARY_FLOAT列进行完全修订的修订结果             |
| `BINARY_DOUBLE_VALUE`            | `BINARY_DOUBLE`               | `NOT NULL` | 对BINARY_DOUBLE列进行完全修订的修订结果            |
| `CHAR_VALUE`                     | `VARCHAR2(1)`                 |            | 对CHAR列进行完全修订的修订结果                     |
| `VARCHAR_VALUE`                  | `VARCHAR2(1)`                 |            | 对VARCHAR2列进行完全修订的修订结果                 |
| `NCHAR_VALUE`                    | `NCHAR(1)`                    |            | 对NCHAR列进行完全修订的修订结果                    |
| `NVARCHAR_VALUE`                 | `NVARCHAR2(1)`                |            | 用于NVARCHAR2列的完整修订的修订结果                |
| `DATE_VALUE`                     | `DATE`                        | `NOT NULL` | 用于DATE列的完整修订的修订结果                     |
| `TIMESTAMP_VALUE`                | `TIMESTAMP(6)`                | `NOT NULL` | 对TIMESTAMP列进行完全修订的修订结果                |
| `TIMESTAMP_WITH_TIME_ZONE_VALUE` | `TIMESTAMP(6) WITH TIME ZONE` | `NOT NULL` | 对TIMESTAMP WITH TIME ZONE列进行完全修订的修订结果 |
| `BLOB_VALUE`                     | `BLOB`                        |            | 对BLOB列进行完全修订的修订结果                     |
| `CLOB_VALUE`                     | `CLOB`                        |            | 对CLOB列进行完全修订的修订结果                     |
| `NCLOB_VALUE`                    | `NCLOB`                       |            | 对NCLOB列进行完全修订的修订结果                    |

## 实践2-10:RMAN VALIDATE



### KnowledgePoint

[VALIDATE](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/rcmrf/VALIDATE.html#GUID-ECE57997-40CE-4029-A434-320441D2AD1E)

#### 目的

使用该`VALIDATE`命令检查损坏的块和丢失的文件，或确定是否可以还原备份集。

如果`VALIDATE`在验证期间检测到问题，则RMAN会显示该问题并触发执行故障评估。如果检测到故障，则RMAN将其记录到自动化诊断系统信息库。您可以`LIST` `FAILURE`用来查看失败。

#### 先决条件

目标数据库必须已安装或打开。

#### 使用说明

`VALIDATE`命令中的选项在语义上等效于命令中的选项`BACKUP` `VALIDATE`。`BACKUP VALIDATE`但是，与`VALIDATE`可以不同的是，它可以检查单个备份集和数据块。

该`VALIDATE`命令在验证期间不会跳过任何块。如果RMAN由于未使用的块压缩而未读取块，并且如果该块已损坏，则RMAN不会检测到损坏。损坏的未使用块无害。

在物理损坏中，数据库根本无法识别该块。在逻辑损坏中，块的内容在逻辑上不一致。默认情况下，该`VALIDATE`命令仅检查物理损坏。您也可以指定`CHECK LOGICAL`检查逻辑损坏。RMAN填充`V$DATABASE_BLOCK_CORRUPTION` 查看其发现。

块损坏可分为块间损坏和块内损坏。在块内损坏中，损坏发生在块本身内，并且可以是物理或逻辑损坏。在块间损坏中，损坏发生在块之间，并且只能是逻辑损坏。该`VALIDATE`命令仅检查块内损坏。