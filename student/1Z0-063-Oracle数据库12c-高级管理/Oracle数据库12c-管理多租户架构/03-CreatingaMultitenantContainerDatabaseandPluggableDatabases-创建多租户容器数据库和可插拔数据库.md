# 第3课：创建多租户容器数据库和可插拔数据库

> 2010.04.21 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [第3课：创建多租户容器数据库和可插拔数据库](#第3课：创建多租户容器数据库和可插拔数据库)   
   - [练习3：概述](#练习3：概述)   
   - [练习3-1：创建新的CDB](#练习3-1：创建新的cdb)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习3-2：探索CDB和PDB结构](#练习3-2：探索cdb和pdb结构)   
      - [总览](#总览)   
      - [任务](#任务)   
   - [练习3-3：从种子创建PDB](#练习3-3：从种子创建pdb)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
         - [命令行创建](#命令行创建)   
         - [SQL Developer图形化界面创建](#sql-developer图形化界面创建)   
   - [练习3-4：在同一CDB中克隆PDB](#练习3-4：在同一cdb中克隆pdb)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习3-5：将非CDB插入CDB](#练习3-5：将非cdb插入cdb)   
      - [总览](#总览)   
      - [任务](#任务)   
   - [练习3-6：将CDB的所有PDB合并为一个CDB](#练习3-6：将cdb的所有pdb合并为一个cdb)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   

<!-- /MDTOC -->

##  练习3：概述

在实践中，您将创建一个名为cdb2的新CDB 。

CDB创建完成后，请检查新CDB的物理和逻辑结构。然后，您将使用不同的方法创建几个PDB：

* 在 `cdb2` 中创建 `pdb2_1`  （使用SQL * Plus，然后再使用 SQL Developer）
* 在 `cdb2` 中，从 `pdb2_1` 克隆 `pdb2_2` （首先使用SQL * Plus或SQL Developer）
* 将非CDB的 `ORCL2` 插入到CDB cdb2作为 `pdb_orcl2`（使用SQL * Plus）
* 将两个CDB `cdb1` 和 `cdb2` 合并到 `cdb2` 中，并可以选择删除数据库 `cdb1` （可选练习）

在这些实践中，您将练习使用DBCA，SQL Developer或SQL*Plus删除 PDB。

## 练习3-1：创建新的CDB

### 总览

在这种实践中，您将使用DBCA 创建一个名为`cdb2`的新CDB。

### 假设条件

创建的CDB `cdb1` 已存在。

### 任务

```SQL
# 1. 使用DBCA 创建一个名为`cdb2`的CDB 。首先释放其他实例拥有的资源，从而关闭orcl，orcl2和cdb1实例。
sqlplus / as sysdba
shutdown immediate
```

图形化界面操作：

| 步骤 | 窗口/页面说明     | 选择或价值观                                                 |
| ------ | ----------------- | ------------------------------------------------------------ |
| a | 步骤1：数据库操作 | 选择“创建数据库”。点击下一步。                               |
| b      | 步骤2：建立模式   | 选择“高级模式”。点击下一步。                                 |
| C      | 步骤3：数据库模板 | 选择“通用或交易处理中。” 点击下一步。                      |
| d      | 步骤4：数据库识别 | 输入全局数据库名称：cdb2 <br/>SID：cdb2<br/>选择**“创建为容器数据库”。<br/>**选择**“创建一个空容器数据库”。**点击下一步。 |
| e             | 步骤5：管理选项          | 取消选择“配置企业管理器（EM）Database Express”。点击下一步。 |
| f          | 步骤6：数据库凭证        | 选择“使用相同的管理密码...”输入：密码：oracle_4U确认密码：oracle_4U点击下一步。 |
| G           | 步骤7：网络配置          | 点击下一步。                                                 |
| h          | 步骤8：存放位置          | 确认存储类型为“文件系统”。选择“对所有数据库使用公用位置文件。”点击下一步。 |
| i      | 步骤9：数据库选项        | 点击下一步。                                                 |
| j         | 步骤10：初始化参数       | 选择“字符集”。选择“使用Unicode（**AL32UTF8**）”。点击下一步。 |
| k           | 步骤11：创建选项         | 选择“创建数据库”。点击下一步。                               |
| l           | 步骤12：必要条件前的检查 | 点击下一步。                                                 |
| m           | 步骤13：摘要             | 单击完成。                                                   |
| n           | 步骤14：进度页           | 在“数据库配置助手”页面上（用于密码管理），单击“退出”。单击关闭。 |


<img src='https://i.loli.net/2020/04/26/gilFxBtrEaswNyv.jpg' alt='gilFxBtrEaswNyv'/>

<img src='https://i.loli.net/2020/04/26/snpKgukhGXf8lT6.jpg' alt='snpKgukhGXf8lT6'/>

## 练习3-2：探索CDB和PDB结构

### 总览

在实践中，您将检查新CDB cdb2及其种子PDB 的物理和逻辑结构。

### 任务

```SQL
# 1.连接到多租户容器数据库cdb2。
sqlplus / as sysdba
# a.检查数据库是否为多租户容器数据库。
SELECT name, cdb, con_id from v$database;
# b.检查实例名称
SELECT INSTANCE_NAME, STATUS, CON_ID from v$instance;
# 2.探索服务。
# a.检查服务。
lsnrctl status
# b.列出为每个容器自动创建的服务。
sqlplus / as sysdba
col name format A20
SELECT name, con_id from v$services;
# 请注意，未列出PDB$SEED服务。没有用户应连接到该服务，因为在此容器上不应执行任何操作。它保留作为创建其他PDB的模板。
# 3.使用新视图V$PDBS显示可插拔数据库。
SELECT CON_ID, NAME, OPEN_MODE from v$pdbs;
# 4.查看新视图CDB_xxx：
col PDB_NAME format a8
col CON_ID format 999999
SELECT PDB_ID, PDB_NAME, DBID, GUID, CON_ID from cdb_pdbs order by 1;
# 5.检查CDB的所有文件。
# a.查看CDB的重做日志文件。
col MEMBER format A42
SELECT GROUP#, MEMBER, CON_ID from v$logfile;
# b.查看CDB的控制文件。
col name format A55
SELECT name, con_id from v$controlfile;
# c.使用以下命令查看CDB的所有数据文件，包括根目录和所有PDB的数据文件：
col file_name format A65
SELECT FILE_NAME, TABLESPACE_NAME, FILE_ID, con_id
from cdb_data_files order by con_id ;
# d.确保您仍连接到根目录；然后使用DBA_DATA_FILES视图。
col file_name format A42
col tablespace_name format A10
SELECT FILE_NAME, TABLESPACE_NAME, FILE_ID
from dba_data_files;

# 请注意，仅列出了根数据文件。
# e.启动cdb1数据库。
sqlplus / as sysdba
STARTUP
EXIT
# 1)使用netca为pdb1_1可插拔数据库添加 PDB1_1 网络服务名
netca
# 2） 在“欢迎”页面上，选择“本地网络服务名称配置”，然后单击“下一步”。
# 3）在“网络服务名称配置”页面上，接受“ 添加”，然后单击“下一步”。
# 4） 在“网络服务名称配置，服务名称”页面上，输入pdb1_1作为服务名称，然后单击下一步。
# 5）在“网络服务名称配置”的“选择协议”页面上，选择“ TCP”，然后单击“下一步”。
# 6）在“网络服务名称配置，TCP / IP协议”页面上，输入完整的主机名，例如< yourservername>或localhost，接受“使用标准端口号1521”，然后单击“下一步”。
# 7）在“网络服务名称配置”的“测试”页面上，选择“否，不测试”（尚未打开可插拔数据库），然后单击“下一步”。
# 8）在“网络服务名称配置，网络服务名称”页面上，接受pdb1_1作为网络服务名称，然后单击下一步。
# 9）在“网络服务名称配置，另一个网络服务”页面上，选择“否”，然后下一个。
# 10）在“网络服务名称配置完成”页面上，单击“下一步”。

# 11）回到“欢迎”页面时，单击“完成”。F。打开pdb1_1可插拔数据库CDB1。
# f.打开pdb1_1可插拔数据库CDB1。
sqlplus / as sysdba
ALTER PLUGGABLE DATABASE pdb1_1 OPEN;
EXIT

# g.连接到pdb1_1的CDB1和使用DBA_DATA_FILES查看。
sqlplus system/oracle_4U@pdb1_1
col file_name format A65
SELECT FILE_NAME, TABLESPACE_NAME, FILE_ID
from dba_data_files;

# h.现在使用V$TABLESPACE和V$DATAFILE视图。
col NAME format A12
SELECT FILE#, ts.name, ts.ts#, ts.con_id
from v$datafile d, v$tablespace  ts
where d.ts#=ts.ts#
and   d.con_id=ts.con_id
order by 4;

# i.列出cdb1和cdb2的密码文件和SPFILE 。
SELECT FILE_NAME, TABLESPACE_NAME from dba_temp_files;

# j.列出cdb1和cdb2的密码文件和SPFILE 。
cd $ORACLE_HOME/dbs
ls -l orapw* spfile*

# k.在alert.log中检查ADR文件，目录，新的DDL语句。
cd $ORACLE_BASE/diag/rdbms/
ls
cd cdb2/cdb2/trace
less alert_cdb2.log

# 6.列出在新CDB cdb2中创建的所有用户。
# a.连接到cdb2实例。
sqlplus / as sysdba
# b.验证是否创建了SYSTEM用户。
col username format A30
select username, common, con_id from cdb_users where username ='SYSTEM';
# 请注意，用户SYSTEM作为普通用户存在于所有容器中。
# c.列出CDB中的所有普通用户。
select distinct username from cdb_users where common ='YES';
# d.列出CDB中的所有本地用户
select distinct username, CON_ID from cdb_users where common ='NO';
# e.以root身份列出本地用户。
select distinct username from dba_users where common ='NO';
# 请注意，根容器中没有本地用户，因为不可能在根中创建任何本地用户。
# 7.查看不同容器对单个SGA的不同访问。
select distinct status, con_id from v_$bh order by 2 ;
```

## 练习3-3：从种子创建PDB

### 总览

在实践中，您将首先使用SQL * Plus，然后使用SQL Developer从种子在cdb2中创建一个新的PDB pdb2_1。

这意味着在这两个操作之间，您将必须删除PDB pdb2_1。

### 假设条件

CDB cdb2的创建成功。

### 任务

#### 命令行创建

```sql
# 1.为cdb2的 pdb2_1的新数据文件创建目录。
cd $ORACLE_BASE/oradata/cdb2
mkdir pdb2_1

# 2.运行SQL * Plus，并使用CREATE PLUGGABLE DATABASE与用户连接到根目录
sqlplus / as sysdba
CREATE PLUGGABLE DATABASE pdb2_1 ADMIN USER pdb2_1_admin
IDENTIFIED BY oracle_4U ROLES=(CONNECT)
FILE_NAME_CONVERT=('/u01/app/oracle/oradata/cdb2/pdbseed',
  '/u01/app/oracle/oradata/cdb2/pdb2_1');

# 3.检查pdb2_1的打开模式。
col con_id format 999
col name format A10
select con_id, NAME, OPEN_MODE,DBID, CON_UID from V$PDBS;

# 4.打开pdb2_1。
alter pluggable database pdb2_1 open;
EXIT
netca
sqlplus sys/oracle_4U@pdb2_1 AS SYSDBA
# 5.该服务现在可用并已在侦听器中注册。
!lsnrctl status

# 6. 使用EasyConnect 以sys用户身份然后以pdb2_1_admin用户身份连接到 pdb2_1。
CONNECT sys/oracle_4U@localhost:1521/pdb2_1 AS SYSDBA
connect pdb2_1_admin/oracle_4U@PDB2_1
show con_name

# 7.列出创建的数据文件。
!ls -l $ORACLE_BASE/oradata/cdb2/pdb2_1/*

# 8.使用视图检查服务，数据文件和表空间。
connect system/oracle_4U@pdb2_1
col name format A30
select name from v$services;

col file_name format A50
col tablespace_name format A8
col file_id format 99
col con_id format 9
select FILE_NAME, TABLESPACE_NAME, FILE_ID, con_id from cdb_data_files order by con_id ;
select FILE_NAME, TABLESPACE_NAME, FILE_ID from dba_data_files;

col file_name format A60
select FILE_NAME, TABLESPACE_NAME, FILE_ID from cdb_temp_files;
select FILE_NAME, TABLESPACE_NAME, FILE_ID from dba_temp_files;

# 9.为了能够查看CDB中所有容器的所有对象，请连接到根目录并使用CDB_XXX
connect / as sysdba
show con_name
select name from v$services;
select FILE_NAME, TABLESPACE_NAME, FILE_ID, con_id from cdb_data_files order by con_id, file_id ;
select FILE_NAME, TABLESPACE_NAME, FILE_ID from dba_data_files;
select FILE_NAME, TABLESPACE_NAME, FILE_ID from cdb_temp_files;
select FILE_NAME, TABLESPACE_NAME, FILE_ID from dba_temp_files;
EXIT
```

删除pdb2_1

```SQL
sqlplus / AS SYSDBA
ALTER PLUGGABLE DATABASE pdb2_1 CLOSE IMMEDIATE;
DROP PLUGGABLE DATABASE pdb2_1 INCLUDING DATAFILES;
EXIT
rm -r $ORACLE_BASE/oradata/cdb2/pdb2_1
```

#### SQL Developer图形化界面创建

<img src='https://i.loli.net/2020/04/26/LdwPpISc8KYteh1.jpg' alt='LdwPpISc8KYteh1'/>

<img src='https://i.loli.net/2020/04/26/GfiVuYsCKXO2nET.jpg' alt='GfiVuYsCKXO2nET'/>

<img src='https://i.loli.net/2020/04/26/DGJbEtISNcV6A31.jpg' alt='DGJbEtISNcV6A31'/>

<img src='https://i.loli.net/2020/04/26/pz8xPjYvE2CUifw.jpg' alt='pz8xPjYvE2CUifw'/>

<img src='https://i.loli.net/2020/04/26/eazOoYkN9udUhS6.jpg' alt='eazOoYkN9udUhS6'/>

<img src='https://i.loli.net/2020/04/26/eATF61SztgJXWwG.jpg' alt='eATF61SztgJXWwG'/>

## 练习3-4：在同一CDB中克隆PDB

### 总览

在本练习中，您将创建一个克隆方法一个新的PDB，克隆pdb2_2从pdb2_1 相同CDB内 cdb2 。

使用最简单的工具SQL * Plus 或 SQL Developer。

### 假设条件

该pdb2_1创建以来，在实践中3-3成功完成。

### 任务

```sql
cd $ORACLE_BASE/oradata/cdb2
mkdir pdb2_2
sqlplus / as sysdba
alter pluggable database pdb2_1 close;
alter pluggable database pdb2_1 open read only;
alter system set db_create_file_dest = '/u01/app/oracle/oradata/cdb2/pdb2_2';
CREATE PLUGGABLE DATABASE pdb2_2 FROM pdb2_1;
select name, open_mode from v$pdbs;
alter pluggable database PDB2_1 close;
alter pluggable database PDB2_1 open;
alter pluggable database PDB2_2 open;
EXIT
netca
sqlplus sys/oracle_4U@pdb2_2 AS SYSDBA
CONNECT / AS SYSDBA Connected.
select name, open_mode from v$pdbs;
connect system/oracle_4U@PDB2_2
show con_name
EXIT

cd $ORACLE_BASE/oradata/cdb2/pdb2_2;ls -l
cd CDB2;ls -l
cd CD2DDD0A5BF67AB8E0436B23B98B987D;ls -l
cd datafile;ls -l
```

## 练习3-5：将非CDB插入CDB

### 总览

在实践中，您将把非CDB orcl2插入CDB cdb2中。

您将不会使用Export / Import DataPump，这可能是一种可能的方法，并且在“其他”课程的另一实践中将介绍该方法。

当前将使用DBMS_PDB软件包中的方法：

* 在非CDB orcl2中执行的该程序包生成一个XML文件，该文件描述了非CDB orcl2的表空间和数据文件。
* 在cdb2中创建pdb_orcl2时，将使用XML文件。

### 任务

```sql
# 1.使用DBMS_PDB.DESCRIBE来“拔出”非CDB orcl2。
sqlplus / as sysdba
startup mount
alter database open read only;
exec dbms_pdb.describe('/u01/app/oracle/oradata/orcl2/xmlorcl2.xml')
shutdown immediate
EXIT

# 2.创建一个新的PDB pdb_orcl2，以使用生成的XML文件将非CDB orcl2插入cdb2。
# 您将必须删除临时文件，因为创建直到完成才可能完成
sqlplus / as sysdba
create pluggable database PDB_ORCL2 using '/u01/app/oracle/oradata/orcl2/xmlorcl2.xml' NOCOPY;
!rm /u01/app/oracle/oradata/orcl2/temp01.dbf
create pluggable database PDB_ORCL2 using '/u01/app/oracle/oradata/orcl2/xmlorcl2.xml' NOCOPY;
EXIT

# 3.要完成操作，您必须通过从PDB SYSTEM表空间删除不必要的元数据来将插入的非CDB转换为正确的PDB 。
# 使用NETCA添加PDB_ORCL2的网络服务名称pdb_orcl2的可插拔数据库cdb2中的tnsnames.ora文件。
netca
# 使用网络服务名称连接到pdb_orcl2。
sqlplus sys/oracle_4U@pdb_orcl2 as sysdba
# 执行 $ORACLE_HOME/rdbms/admin/noncdb_to_pdb.sql脚本。
# 预计需要30分钟以上才能完成。在继续执行的同时，您可以在CDB_PDBs视图中查看NEW状态更改为新插入的PDB的CONVERTING状态。
# 在以SYS身份连接到root 的并行会话中，准备运行以下语句：

@$ORACLE_HOME/rdbms/admin/noncdb_to_pdb.sql

SET SERVEROUTPUT ON
SET FEEDBACK 1
SET NUMWIDTH 10
SET LINESIZE 80
SET TRIMSPOOL ON
SET TAB OFF
SET PAGESIZE 100

SELECT pdb_name, status FROM cdb_pdbs;
# 转换完成后，打开PDB并退出会话。
alter pluggable database pdb_orcl2 open;
EXIT

# 4.连接到PDB_ORCL2。
sqlplus sys/oracle_4U@localhost:1521/PDB_ORCL2 as SYSDBA

# 5.验证应用程序数据是否在PDB pdb_orcl2中
select count(empno) from scott.emp;
EXIT
```


## 练习3-6：将CDB的所有PDB合并为一个CDB

### 总览

在实践中，您将cdb1的所有PDB合并到单个CDB cdb2中。

1. 将cdb1的所有PDB合并到cdb2中。
2. 删除cdb1。

### 假设条件

CDB cdb2存在。该cdb2创建以来，在实践中3-1成功完成。

### 任务

```SQL
# 1. 连接到多租户容器数据库cdb1，以拔出所有PDB。
# a.以具有ALTER PLUGGABLE DATABASE特权的普通用户身份连接到cdb1 root，以拔出pdb1_1。如果pdb1_1仍处于READ WRITE模式，请关闭PDB。在CDB_PDBS视图中拔出PDB时，请验证其状态。
sqlplus / as sysdba
select name, open_mode from v$pdbs;
alter pluggable database PDB1_1 close immediate;
alter pluggable database PDB1_1 unplug into 'xmlfilePDB1_1.xml';
col PDB_NAME format A20
select PDB_NAME, STATUS from CDB_PDBS where PDB_NAME='PDB1_1';
drop pluggable database PDB1_1 KEEP DATAFILES;
EXIT

# b.DBMS_PDB.CHECK_PLUG_COMPATIBILITY用于检查pdb1_1与兼容cdb2的兼容性。
# 以具有CREATE PLUGGABLE DATABASE特权的普通用户身份连接到cdb2 root，以插入pdb1_1。
SET SERVEROUTPUT ON
DECLARE
 compat BOOLEAN := FALSE;
 BEGIN
 compat := DBMS_PDB.CHECK_PLUG_COMPATIBILITY(
pdb_descr_file => '/u01/app/oracle/product/12.1.0/dbhome_1/dbs/xmlfilePDB1_1.xml', pdb_name => 'pdb1_1');
if compat then
DBMS_OUTPUT.PUT_LINE('Is pluggable compatible? YES'); else DBMS_OUTPUT.PUT_LINE('Is pluggable compatible? NO'); end if;
end;
/

# c.如果返回的值为“是”，则可以立即进入步骤d。
# 如果返回的值为NO，请检查PDB_PLUG_IN_VIOLATIONS视图以查看为什么它不兼容。
select message, action from pdb_plug_in_violations where name='PDB1_1';

# d.将pdb1_1插入cdb2。
create pluggable database pdb1_1 using 'xmlfilePDB1_1.xml' NOCOPY;
# 请注意，您使用子句NOCOPY，因为cdb2 pdb1_1文件位于正确的位置。否则，您应该已经描述了将文件从源移动到新目标的目标位置。

# e.在cdb2打开pdb1_1。
alter pluggable database pdb1_1 open;

# f.检查pdb1_1是在PDBS列表cdb2。
select name, open_mode from v$pdbs;

# 2.合并后可以将cdb1删除
# 修改环境变量 ORACLE_SID = [cdb2] ? cdb1
sqlplus / as sysdba
shutdown immediate
startup mount restrict
DROP DATABASE;
EXIT
```
