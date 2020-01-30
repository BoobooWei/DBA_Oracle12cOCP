# 实践4:管理实例

> **Practices for Lesson 4: Managing the Database Instance**
>
> 2020.01.29 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:0 updateOnSave:1 -->

[实践4:管理实例](#实践4管理实例)   
&emsp;[实践4:概览](#实践4概览)   
&emsp;[实践4-1:使用Oracle Enterprise Manager Cloud Control管理Oracle实例](#实践4-1使用oracle-enterprise-manager-cloud-control管理oracle实例)   
&emsp;[实践4-2:使用Oracle Enterprise Manger Database Express管理Oracle实例](#实践4-2使用oracle-enterprise-manger-database-express管理oracle实例)   
&emsp;[实践4-3:使用SQL*Plus管理Oracle实例](#实践4-3使用sqlplus管理oracle实例)   
&emsp;&emsp;[Overview](#overview)   
&emsp;&emsp;[Tasks](#tasks)   
&emsp;&emsp;[Practice](#practice)   
&emsp;&emsp;[KnowledgePoint](#knowledgepoint)   
&emsp;&emsp;&emsp;[12c 参数文件 与 11g 参数文件的不同之处](#12c-参数文件-与-11g-参数文件的不同之处)   
&emsp;&emsp;&emsp;[动态参数修改](#动态参数修改)   
&emsp;&emsp;&emsp;[静态参数修改](#静态参数修改)   
&emsp;&emsp;&emsp;[查看参数](#查看参数)   
&emsp;[实践4-4:通过使用自动诊断存储库命令查看警报日志接口(ADRCI)](#实践4-4通过使用自动诊断存储库命令查看警报日志接口adrci)   
&emsp;&emsp;[Overview](#overview)   
&emsp;&emsp;[Tasks](#tasks)   
&emsp;&emsp;[Practice](#practice)   
&emsp;&emsp;[KnowledgePoint](#knowledgepoint)   

<!-- /MDTOC -->

## 实践4:概览

Practices for Lesson 4: Overview

**Background:** The Oracle software has been installed and a database has been created. You want to ensure that you can start and stop the database instance and see the application data.

背景:已安装Oracle软件，并创建了数据库。您希望确保能够启动和停止数据库实例并查看应用程序数据。

## 实践4-1:使用Oracle Enterprise Manager Cloud Control管理Oracle实例

Practice 4-1: Managing the Oracle Instance by Using Oracle Enterprise Manager Cloud Control

## 实践4-2:使用Oracle Enterprise Manger Database Express管理Oracle实例

Practice 4-2: Managing the Oracle Instance by Using Oracle Enterprise Manger Database Express

## 实践4-3:使用SQL*Plus管理Oracle实例

Practice 4-3: Managing the Oracle Instance by Using SQL*Plus

### Overview

In this practice, you use SQL*Plus to view and change instance parameters.

### Tasks

1. Optimize SQL*Plus：configuring `$ORACLE_HOME/sqlplus/admin/glogin.sql`
2. Set the JOB_QUEUE_PROCESSES initialization parameter to 1000 by using SQL*Plus.
3. Using SQL*Plus, shut down and restart the orcl database instance.
4. Use the **SHOW PARAMETER** command to verify the settings for **SGA_MAX_SIZE**, **DB_CACHE_SIZE**, and **SHARED_POOL_SIZE**.
5. Check the value of **JOB_QUEUE_PROCESSES**.
6. Exit from SQL*Plus.

### Practice

1. 优化 SQL*Plus：配置 `$ORACLE_HOME/sqlplus/admin/glogin.sql`

	```bash
	cat > $ORACLE_HOME/sqlplus/admin/glogin.sql << ENDF
	define _editor=vi
	set serveroutput on size 1000000
	set trimspool on
	set long 5000
	set linesize 100
	set pagesize 9999
	column plan_plus_exp format a80
	set sqlprompt '&_user.@&_connect_identifier.>'
	ENDF
	```

2. 设置 JOB_QUEUE_PROCESSES 的初始化参数值为1000

   ```sql
   sqlplus / as sysdba
   SHOW PARAMETER job
   ALTER SYSTEM SET job_queue_processes=1000 SCOPE=BOTH;
   SHOW PARAMETER job
   ```

3. 使用 SQL*Plus 关闭并重启orcl实例

   ```sql
   shutdown immediate
   startup
   ```

4. 使用 **SHOW PARAMETER** 命令查看参数 **SGA_MAX_SIZE**, **DB_CACHE_SIZE**, **SHARED_POOL_SIZE** 的值

   ```sql
   show parameter sga_max_size
   show parameter db_cache_size
   show parameter shared_pool_size
   ```

5. 检查 **JOB_QUEUE_PROCESSES** 参数的值

   ```sql
   show parameter job_queue_processes
   ```

6. 退出 SQL*Plus

   ```sql
   exit
   ```

拓展

1. 查看参数的属性

   ```sql
   SYS@booboo>exec print_table(q'[select * from V$SYSTEM_PARAMETER WHERE name='job_queue_processes']')
NUM			      		    : 3106
NAME			      	    : job_queue_processes
TYPE			      	    : 3
VALUE			      	    : 1000
DISPLAY_VALUE		      : 1000
DEFAULT_VALUE		      : 4000
ISDEFAULT		      	  : FALSE
ISSES_MODIFIABLE	    : FALSE
ISSYS_MODIFIABLE	    : IMMEDIATE
ISPDB_MODIFIABLE	    : TRUE
ISINSTANCE_MODIFIABLE	: TRUE
ISMODIFIED		      	: FALSE
ISADJUSTED		      	: FALSE
ISDEPRECATED		      : FALSE
ISBASIC 		      	  : FALSE
DESCRIPTION		      	: maximum number of job queue slave processes
UPDATE_COMMENT		    :
HASH			     	      : 1663833312
CON_ID			        	: 0
   ```

   

2. 说出该参数的重要属性

   ISSES_MODIFIABLE：FALSE —— 不能执行 `alter session`

   ISSYS_MODIFIABLE：IMMEDIATE —— 可动态修改且立刻生效 `alter system`

   ISPDB_MODIFIABLE：TRUE —— pdb级别可以覆盖cdb

   ISINSTANCE_MODIFIABLE：TRUE —— 实例级别可以修改


### KnowledgePoint

[11g 参数文件](https://github.com/BoobooWei/booboo_oracle/blob/master/D-体系结构和存储引擎-03-物理结构_参数文件parameter_files.md)

[12c 参数文件]( https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/creating-and-configuring-an-oracle-database.html#GUID-7302C60F-E96E-4202-AC81-25A6C93EEFA3  )

- [*《 Oracle数据库SQL语言参考》*](https://www.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/12.2/admin&id=SQLRF00902)中有关该`ALTER` `SYSTEM`命令的 信息
-  [在CDB中使用ALTER SYSTEM SET语句]( https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/administering-a-cdb-with-sql-plus.html#GUID-E47348D6-5350-4890-ACD6-7BA1C1DD4E95 )

#### 视图解释V$SYSTEM_PARAMETER

`V$SYSTEM_PARAMETER`显示有关实例当前有效的初始化参数的信息。新会话将从实例范围的值继承参数值。

| 柱                      | 数据类型         | 描述                                                         |
| :---------------------- | :--------------- | :----------------------------------------------------------- |
| `NUM`                   | `NUMBER`         | 参数编号                                                     |
| `NAME`                  | `VARCHAR2(80)`   | 参数名称                                                     |
| `TYPE`                  | `NUMBER`         | 参数类型：`1` -布尔值`2` -弦`3` - 整数`4` -参数文件`5` -保留`6` -大整数 |
| `VALUE`                 | `VARCHAR2(4000)` | 实例范围的参数值                                             |
| `DISPLAY_VALUE`         | `VARCHAR2(4000)` | 用户友好格式的参数值。例如，如果该`VALUE`列显示`262144`大整数参数的值，则该`DISPLAY_VALUE`列将显示value `256K`。 |
| `ISDEFAULT`             | `VARCHAR2(9)`    | 指示参数是设置为默认值（`TRUE`）还是在参数文件中指定了参数值（`FALSE`） |
| `ISSES_MODIFIABLE`      | `VARCHAR2(5)`    | 指示是否参数可以与被改变`ALTER SESSION`（`TRUE`）否（`FALSE`） |
| `ISSYS_MODIFIABLE`      | `VARCHAR2(9)`    | 指示参数是否可以更改`ALTER SYSTEM`以及更改何时生效：`IMMEDIATE`- `ALTER SYSTEM`无论用于启动实例的参数文件的类型如何，都可以更改参数。更改将立即生效。`DEFERRED`- `ALTER SYSTEM`无论用于启动实例的参数文件的类型如何，都可以更改参数。该更改在后续会话中生效。`FALSE`- `ALTER SYSTEM`除非使用服务器参数文件启动实例，否则无法使用参数进行更改。该更改在后续实例中生效。 |
| `ISINSTANCE_MODIFIABLE` | `VARCHAR2(5)`    | 对于可以使用更改的参数`ALTER SYSTEm`，指示参数值对于每个实例（`TRUE`）可以不同，还是对于所有Real Application Clusters实例（`FALSE`）参数必须具有相同的值。如果该`ISSYS_MODIFIABLE`列为`FALSE`，则此列始终为`FALSE`。 |
| `ISMODIFIED`            | `VARCHAR2(8)`    | 指示如何修改参数。如果`ALTER SYSTEM`执行，则值为`MODIFIED`。 |
| `ISADJUSTED`            | `VARCHAR2(5)`    | 指示Oracle是否将输入值调整为更合适的值（例如，参数值应为质数，但用户输入了非质数，因此Oracle将值调整为下一个质数） |
| `ISDEPRECATED`          | `VARCHAR2(5)`    | 指示是否已弃用该参数`TRUE`（`FALSE`）                        |
| `ISBASIC`               | `VARCHAR2(5)`    | 指示参数是否是基本参数（`TRUE`），或者（`FALSE`）            |
| `DESCRIPTION`           | `VARCHAR2(255)`  | 参数说明                                                     |
| `UPDATE_COMMENT`        | `VARCHAR2(255)`  | 与最新更新相关的评论                                         |
| `HASH`                  | `NUMBER`         | 参数名称的哈希值                                             |

```sql
SYS@booboo>select distinct ISINSTANCE_MODIFIABLE,ISSYS_MODIFIABLE,ISSES_MODIFIABLE,ISPDB_MODIFIABLE from v$system_parameter;

ISINS ISSYS_MOD ISSES ISPDB
----- --------- ----- -----
FALSE FALSE	FALSE TRUE
TRUE  IMMEDIATE TRUE  FALSE
FALSE IMMEDIATE FALSE TRUE
TRUE  IMMEDIATE FALSE FALSE
TRUE  DEFERRED	FALSE FALSE
TRUE  DEFERRED	TRUE  TRUE
TRUE  IMMEDIATE FALSE TRUE
FALSE FALSE	TRUE  TRUE
FALSE FALSE	TRUE  FALSE
TRUE  IMMEDIATE TRUE  TRUE
FALSE IMMEDIATE FALSE FALSE
FALSE FALSE	FALSE FALSE

12 rows selected.

SYS@booboo>select count(*) from v$system_parameter;

  COUNT(*)
----------
       417
```



#### 12c 参数文件 与 11g 参数文件的不同之处

```bash
1. pdb 继承 cdb 的参数
2. pdb 参数可以变更，保存在CDB级别的 pdb_spfile$ 系统表中，对参数文件没有影响
3. pdb 参数哪些可以变更，当 ISPDB_MODIFIABLE 列TRUE用于V$SYSTEM_PARAMETER视图中的参数时，可以修改PDB的初始化参数。
4. 文本初始化参数文件（PFILE）不能包含特定于PDB的参数值。

-- 查询可被PDB修改的所有初始化参数：
SELECT NAME FROM V$SYSTEM_PARAMETER WHERE ISPDB_MODIFIABLE='TRUE' ORDER BY NAME;

-- 可被动态修改 且 pdb 中可被覆盖的 参数
SELECT NAME,ISPDB_MODIFIABLE,ISSYS_MODIFIABLE FROM V$SYSTEM_PARAMETER WHERE ISPDB_MODIFIABLE='TRUE' and ISSYS_MODIFIABLE<>'FALSE' ORDER BY NAME;

-- 不可动态修改 且 pdb 中可被覆盖的 参数
SELECT NAME,ISPDB_MODIFIABLE,ISSYS_MODIFIABLE FROM V$SYSTEM_PARAMETER WHERE ISPDB_MODIFIABLE='TRUE' and ISSYS_MODIFIABLE='FALSE' ORDER BY NAME;
```

#### 动态参数修改

```sql
-- pdb中查看当前动态参数open_cursors
SYS@booboo>exec print_table(q'[select * from V$SYSTEM_PARAMETER WHERE name='open_cursors']')
NUM			      				: 3328
NAME			     				: open_cursors
TYPE			      			: 3
VALUE			      			: 300
DISPLAY_VALUE		      : 300
DEFAULT_VALUE		      : 50
ISDEFAULT		      		: FALSE
ISSES_MODIFIABLE	    : FALSE
ISSYS_MODIFIABLE	    : IMMEDIATE
ISPDB_MODIFIABLE	    : TRUE
ISINSTANCE_MODIFIABLE	: TRUE
ISMODIFIED		      	: FALSE
ISADJUSTED		      	: FALSE
ISDEPRECATED		      : FALSE
ISBASIC 		      		: TRUE
DESCRIPTION		      	: max # cursors per session
UPDATE_COMMENT		    :
HASH			      			: 4033294835
CON_ID			      		: 0
-- 可以在pdb中直接修改
SYS@booboo>show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 3 BOOBOOPDB1			  READ WRITE NO
SYS@booboo>show parameter open_cursors;

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
open_cursors			     integer	 300
SYS@booboo>alter system set open_cursors=500 scope=both;

System altered.

SYS@booboo>show parameter open_cursors;

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
open_cursors			     integer	 500

SYS@booboo>exec print_table(q'[select * from V$SYSTEM_PARAMETER WHERE name='open_cursors']')
NUM			      				: 3328
NAME			     				: open_cursors
TYPE			      			: 3
VALUE			      			: 500
DISPLAY_VALUE		      : 500
DEFAULT_VALUE		      : 50
ISDEFAULT		      		: FALSE
ISSES_MODIFIABLE	    : FALSE
ISSYS_MODIFIABLE	    : IMMEDIATE
ISPDB_MODIFIABLE	    : TRUE
ISINSTANCE_MODIFIABLE	: TRUE
ISMODIFIED		      	: MODIFIED
ISADJUSTED		      	: FALSE
ISDEPRECATED		      : FALSE
ISBASIC 		      		: TRUE
DESCRIPTION		      	: max # cursors per session
UPDATE_COMMENT		    :
HASH			      			: 4033294835
CON_ID			      		: 3

--切换到cdb，查看参数值还是300
SYS@booboo>conn / as sysdba
Connected.
SYS@booboo>show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 2 PDB$SEED			  READ ONLY  NO
	 3 BOOBOOPDB1			  READ WRITE NO
	 4 BOOBOOPDB2			  MOUNTED
	 5 BOOBOOPDB3			  MOUNTED
	 6 BOOBOOPDB4			  MOUNTED
	 
SYS@booboo>show parameter open_cursors;

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
open_cursors			     integer	 300	 

SYS@booboo>exec print_table(q'[select * from V$PARAMETER WHERE name='open_cursors']')
NUM			      : 3328
NAME			      : open_cursors
TYPE			      : 3
VALUE			      : 300
DISPLAY_VALUE		      : 300
DEFAULT_VALUE		      : 50
ISDEFAULT		      : FALSE
ISSES_MODIFIABLE	      : FALSE
ISSYS_MODIFIABLE	      : IMMEDIATE
ISPDB_MODIFIABLE	      : TRUE
ISINSTANCE_MODIFIABLE	      : TRUE
ISMODIFIED		      : FALSE
ISADJUSTED		      : FALSE
ISDEPRECATED		      : FALSE
ISBASIC 		      : TRUE
DESCRIPTION		      : max # cursors per session
UPDATE_COMMENT		      :
HASH			      : 4033294835
CON_ID			      : 1

SYS@booboo>exec print_table(q'[select * from V$SYSTEM_PARAMETER WHERE name='open_cursors']')
NUM			      : 3328
NAME			      : open_cursors
TYPE			      : 3
VALUE			      : 300
DISPLAY_VALUE		      : 300
DEFAULT_VALUE		      : 50
ISDEFAULT		      : FALSE
ISSES_MODIFIABLE	      : FALSE
ISSYS_MODIFIABLE	      : IMMEDIATE
ISPDB_MODIFIABLE	      : TRUE
ISINSTANCE_MODIFIABLE	      : TRUE
ISMODIFIED		      : FALSE
ISADJUSTED		      : FALSE
ISDEPRECATED		      : FALSE
ISBASIC 		      : TRUE
DESCRIPTION		      : max # cursors per session
UPDATE_COMMENT		      :
HASH			      : 4033294835
CON_ID			      : 0
-----------------
NUM			      : 3328
NAME			      : open_cursors
TYPE			      : 3
VALUE			      : 500
DISPLAY_VALUE		      : 500
DEFAULT_VALUE		      : 50
ISDEFAULT		      : FALSE
ISSES_MODIFIABLE	      : FALSE
ISSYS_MODIFIABLE	      : IMMEDIATE
ISPDB_MODIFIABLE	      : TRUE
ISINSTANCE_MODIFIABLE	      : TRUE
ISMODIFIED		      : MODIFIED
ISADJUSTED		      : FALSE
ISDEPRECATED		      : FALSE
ISBASIC 		      : TRUE
DESCRIPTION		      : max # cursors per session
UPDATE_COMMENT		      :
HASH			      : 4033294835
CON_ID			      : 3
-----------------
```

#### 静态参数修改

```sql
-- pdb中查看当前静态参数memory_max_target
SQL> show pdbs

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 3 BOOBOOPDB1			  READ WRITE NO

SQL> column name format a20
SQL> column value format a20
SQL> select NAME,VALUE,ISSYS_MODIFIABLE from v$parameter where name='memory_max_target';

NAME		     VALUE		  ISSYS_MOD
-------------------- -------------------- ---------
memory_max_target    0			  FALSE

-- 查看memory_max_target的ISPDB_MODIFIABLE的值为FALSE，代表pdb中不可以修改
SQL> select num,name,type,value,isdefault,ISBASIC,ISPDB_MODIFIABLE from v$system_parameter where name='memory_max_target';

       NUM NAME 		      TYPE VALUE		ISDEFAULT ISBAS
---------- -------------------- ---------- -------------------- --------- -----
ISPDB
-----
      1376 memory_max_target		 6 0			TRUE	  FALSE
FALSE

-- pdb中修改该参数值
SQL> alter system set memory_max_target=1g scope=spfile;
alter system set memory_max_target=1g scope=spfile
*
ERROR at line 1:
ORA-65040: operation not allowed from within a pluggable database


-- pdb中修改open_links
SQL> select num,name,type,value,isdefault,ISBASIC,ISPDB_MODIFIABLE from v$system_parameter where name='open_links';

       NUM NAME 						    TYPE VALUE		      ISDEFAULT ISBAS ISPDB
---------- -------------------------------------------------- ---------- -------------------- --------- ----- -----
      3288 open_links						       3 4		      TRUE	FALSE TRUE

SQL> alter system set open_links=10 scope=spfile;

System altered.

SYS@booboo>show parameter open_lin;

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
open_links			     integer	 4
open_links_per_instance 	     integer	 4

-- 通过cdb，重启pdb后生效
SQL> STARTUP PLUGGABLE DATABASE booboopdb1 FORCE
Pluggable Database opened.

-- pdb中查看参数值
SYS@booboo>show parameter open_lin;

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
open_links			     integer	 10
open_links_per_instance 	     integer	 4


--文本初始化参数文件（PFILE）不能包含特定于PDB的参数值。
--PDB 的参数存储在 CDB 的 PDB_SPFILE$ 字典表中以 con_id 区别，所以， PDB 的 PDB_SPFILE$ 表是空的。
SYS@booboo> exec print_table('select PDB_UID, NAME, VALUE$ from pdb_spfile$')
PDB_UID 		      : 2350619425
NAME			      : open_cursors
VALUE$			      : 500
-----------------
PDB_UID 		      : 2350619425
NAME			      : open_links
VALUE$			      : 10
-----------------

PL/SQL procedure successfully completed.
```

#### 查看参数

```sql
SELECT NAME,ISPDB_MODIFIABLE,ISSYS_MODIFIABLE 
FROM V$SYSTEM_PARAMETER 
WHERE 
	ISPDB_MODIFIABLE='TRUE' and 
	ISSYS_MODIFIABLE<>'FALSE' 
ORDER BY NAME;

SELECT NAME,ISPDB_MODIFIABLE,ISSYS_MODIFIABLE 
FROM V$SYSTEM_PARAMETER 
WHERE 
	ISPDB_MODIFIABLE='TRUE' and 
	ISSYS_MODIFIABLE='FALSE' 
ORDER BY NAME;
```

## 实践4-4:通过使用自动诊断存储库命令查看警报日志接口(ADRCI)
Practice 4-4: Viewing the Alert Log by Using the Automatic Diagnostic Repository Command Interface (ADRCI)

### Overview

In this practice, you use command-line tools to view the orcl instance alert log and find the startup phases.

### Tasks

1. In the alert log, view the phases that the database went through during startup. What are they?
2. Scroll through the log and review the phases of the database during startup. Use the vi search commands to find the appropriate lines. Your alert log may differ from what is shown in this practice.

### Practice

1. 在警报日志中，查看数据库在启动期间经历的阶段。

   ```bash
   [oracle@oracle01 ~]$ adrci

   ADRCI: Release 12.2.0.1.0 - Production on Thu Jan 30 02:27:35 2020

   Copyright (c) 1982, 2017, Oracle and/or its affiliates.  All rights reserved.

   ADR base = "/u01/app/oracle"
   adrci> show alert

   Choose the home from which to view the alert log:

   1: diag/rdbms/booboo/booboo
   2: diag/clients/user_oracle/host_2874269298_107
   3: diag/tnslsnr/oracle01/listener
   Q: to quit

   Please select option: 1
   Output the results to file: /tmp/alert_88137_1406_booboo_1.ado
   ```

   **Note: 默认会使用vi命令打开alert文件。**

2. 滚动日志并在启动期间检查数据库的各个阶段。使用vi搜索命令查找适当的行。您的警报日志可能与此实践中显示的有所不同。

   ```bash
   2019-09-22 02:03:08.120000 -07:00
   Starting ORACLE instance (normal) (OS id: 71748)
   ......
   alter database "booboo" open resetlogs
   ......
   2020-01-29 18:00:00.047000 +08:00
   Closing scheduler window
   Closing Resource Manager plan via scheduler window
   Clearing Resource Manager CDB plan via parameter
   2020-01-29 23:08:35.821000 +08:00
   Resize operation completed for file# 3, old size 563200K, new size 573440K

   --输入:q 退出
   --退出vi后继续进入adrci的交互界面
   --输入Q
   Please select option: Q
   adrci> exit
   ```

### KnowledgePoint

[自动诊断存储库命令ADRCI](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/diagnosing-and-resolving-problems.html#GUID-EA97EEF6-9207-4536-B808-3B91DACA7AD6)
