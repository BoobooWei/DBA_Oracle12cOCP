# 第2课：多租户容器数据库和可插拔数据库的基础

> 2010.04.21 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [第2课：多租户容器数据库和可插拔数据库的基础](#第2课：多租户容器数据库和可插拔数据库的基础)   
   - [练习2：概述](#练习2：概述)   
   - [练习2-1：探索CDB的体系结构](#练习2-1：探索cdb的体系结构)   
      - [总览](#总览)   
      - [任务](#任务)   
         - [1.探索cdb1实例后台进程和多租户容器数据库](#1探索cdb1实例后台进程和多租户容器数据库)   
         - [2.探索服务](#2探索服务)   
         - [3.显示可插拔数据库](#3显示可插拔数据库)   
         - [4.查看一些新的观点系列CDB_ xxx](#4查看一些新的观点系列cdb_-xxx)   
         - [5.检查CDB的所有文件](#5检查cdb的所有文件)   
         - [6.列出所有创建的用户](#6列出所有创建的用户)   
         - [7.列出CDB的所有角色和特权。](#7列出cdb的所有角色和特权。)   

<!-- /MDTOC -->

## 练习2：概述

在这种实践中，您将探索并熟悉CDB和PDB的体系结构。

## 练习2-1：探索CDB的体系结构

### 总览

在本练习中，您将探索cdb1及其可插入数据库的体系结构和结构。

### 任务

#### 1.探索cdb1实例后台进程和多租户容器数据库

```bash
# a.使用ps –ef | grep Unix命令
ps -ef | grep cdb1
# b.连接到多租户容器数据库cdb1
. oraenv
sqlplus / as sysdba
# c.检查数据库是否为多租户容器数据库。
select name, cdb, con_id from v$database;
# d.检查实例名称。
select INSTANCE_NAME, STATUS, CON_ID from v$instance;
```

#### 2.探索服务

```bash
# a.检查侦听器是否已启动。
lsnrctl status
# 如果尚未启动，请使用以下命令启动侦听器：
lsnrctl start
# b.检查服务
lsnrctl services
# c.列出为每个容器自动创建的服务。
sqlplus / as sysdba
col name format A20
select name, con_id from v$services;
```

#### 3.显示可插拔数据库

```sql
# a.使用新视图V$PDBS。
select CON_ID, NAME, OPEN_MODE from v$pdbs;
# b.使用新命令SHOW CON_NAME和CON_ID知道您连接到哪个容器
show con_name
show CON_ID
# 您还可以使用SYS_CONTEXT函数查看CON_NAME和CON_ID会话上下文的属性。
SELECT sys_context('userenv','CON_NAME') from dual;
SELECT sys_context('userenv','CON_ID') from dual;
```

#### 4.查看一些新的观点系列CDB_ xxx

```sql
col PDB_NAME format a8
col CON_ID format 99
select PDB_ID, PDB_NAME, DBID, GUID, CON_ID from cdb_pdbs;
```

#### 5.检查CDB的所有文件

```SQL
# a.查看CDB的重做日志文件
col MEMBER format A40
select GROUP#, CON_ID, MEMBER from v$logfile;
# b.查看CDB的控制文件
col NAME format A60
select NAME , CON_ID from v$controlfile;
# c.查看CDB的所有数据文件，包括根目录和所有PDB的数据文件。
## 1）使用CDB_DATA_FILES视图：
col file_name format A50
col tablespace_name format A8
col file_id format 9999
col con_id format 999
select FILE_NAME, TABLESPACE_NAME, FILE_ID, con_id 2 from cdb_data_files order by con_id ;
## 使用ls Unix命令：
!ls -l $ORACLE_BASE/oradata/cdb1
!ls -l $ORACLE_BASE/oradata/cdb1/pdbseed
# d.确保您已连接到根目录；然后使用DBA_DATA_FILES视图。
col file_name format A42
select FILE_NAME, TABLESPACE_NAME, FILE_ID from dba_data_files;
# e.现在使用V$TABLESPACE和V$DATAFILE视图。
col NAME format A12
select FILE#, ts.name, ts.ts#, ts.con_id
from v$datafile d, v$tablespace  ts
where d.ts#=ts.ts#
and   d.con_id=ts.con_id
order by 4,3;
# 列出CDB的临时文件
col file_name format A57
select FILE_NAME, TABLESPACE_NAME, FILE_ID from cdb_temp_files;
```

#### 6.列出所有创建的用户

```SQL
# a.验证是否创建了SYSTEM用户。
col username format A22
select username, common, con_id from cdb_users where username ='SYSTEM';
# 请注意，用户SYSTEM作为普通用户存在于所有容器中。
# b.列出CDB的所有普通用户。
select distinct username from cdb_users where common ='YES' ORDER BY 1;
# c.列出CDB的所有本地用户List all the local users of the CDB.
select distinct username, con_id from cdb_users where common ='NO';
# d.列出所有本地用户List the local users in the root.
# 请注意，根容器中没有本地用户，因为不可能在根中创建任何本地用户。
select username, con_id from cdb_users where common ='NO';
```

#### 7.列出CDB的所有角色和特权。

```SQL
# a.列出CDB的所有角色。
# 注意，根容器中没有本地角色，因为不可能在根中创建任何本地角色。
col role format A30
select role, common, con_id from cdb_roles order by 3;
# b.确保特权本质上既不是普通的也不是本地的。
# 请注意，没有COMMON列。
desc sys.system_privilege_map
desc sys.table_privilege_map
# c.验证授予后的特权是否变为普通特权或本地特权。
desc CDB_SYS_PRIVS
desc CDB_TAB_PRIVS
# d.请注意，角色（尽管取决于角色的创建方式是公共角色还是本地角色），也像特权一样，也可以是公共角色或本地角色。
col grantee format A10
col granted_role format A28
select grantee, granted_role, common, con_id from cdb_role_privs where grantee='SYSTEM';
