# 第9课的练习：其他(审计和导出导入)

> 2010.04.26 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [第9课的练习：其他(审计和导出导入)](#第9课的练习：其他审计和导出导入)   
   - [第9课：概述的练习](#第9课：概述的练习)   
      - [总览](#总览)   
   - [练习9-1：使用统一审核进行审核](#练习9-1：使用统一审核进行审核)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习9-2：从非CDB导出并导入PDB](#练习9-2：从非cdb导出并导入pdb)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习9-3：在PDB之间进行导出和导入](#练习9-3：在pdb之间进行导出和导入)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   

<!-- /MDTOC -->

## 第9课：概述的练习

### 总览

在这种实践中，您将审核在PDB中执行的操作，例如在一个PDB中执行的用户创建和用户删除操作，并使用Unified Auditing在另一个PDB中创建表空间。

然后，您将在非CDB和PDB之间以及PDB之间执行Oracle Data Pump导出和导入操作。



## 练习9-1：使用统一审核进行审核

### 总览

在本练习中，您将配置审核策略在新pdb_orcl的cdb2审核任何未来的CREATE TABLESPACE语句。

您还可以在cdb2的另一个PDB中创建审核策略，以审核本地用户LU_PDB2在pdb2中执行的任何CREATE USER或DROP USER操作。

### 假设条件

从练习3-1成功创建了cdb2。从实践3-3成功创建了pdb2_1。

pdb2_1已从练习4-4成功重命名为pdb2。

如果无法成功创建触发器，请执行以下追赶脚本：

```bash
cd /home/oracle/solutions/catchup_04_03
./cr_trig.sh
```

### 任务

```SQL
# 1.创建新的pdb_orcl在cdb2，这将是用于从非CDB导出的数据的容器或接收ORCL。
# a.为cdb2的 pdb_orcl的新数据文件创建目录。
ORACLE_SID = cdb2
cd $ORACLE_BASE/oradata/cdb2
mkdir pdb_orcl
# b.使用具有CREATE PLUGGABLE DATABASE特权的用户连接到根。
sqlplus / as sysdba
CREATE PLUGGABLE DATABASE pdb_orcl ADMIN USER orcl_admin
IDENTIFIED BY oracle_4U ROLES=(CONNECT)
FILE_NAME_CONVERT=('/u01/app/oracle/oradata/cdb2/pdbseed'
,'/u01/app/oracle/oradata/cdb2/pdb_orcl');
# c.检查pdb_orcl的打开模式。
col con_id format 999
col name format A10
select con_id, NAME, OPEN_MODE, DBID, CON_UID from V$PDBS;
# d.打开pdb_orcl
netca
sqlplus sys/oracle_4U@pdb_orcl AS SYSDBA
alter pluggable database pdb_orcl open;
EXIT

# 2.启用统一审核。
lsnrctl stop
ps -ef | grep pmon
sqlplus / as sysdba
shutdown immediate
EXIT
cd /u01/app/oracle/product/middleware/oms
export OMS_HOME=/u01/app/oracle/product/middleware/oms

[ORACLE_SID = [cdb2] ? em12rep
sqlplus / as sysdba
shutdown immediate
EXIT
ps -ef | grep pmon
cd $ORACLE_HOME/rdbms/lib
make -f ins_rdbms.mk uniaud_on ioracle
ORACLE_HOME=$ORACLE_HOME

[ORACLE_SID = [em12rep] ? cdb2
sqlplus / as sysdba
startup mount
ALTER DATABASE ARCHIVELOG;
ALTER DATABASE OPEN;
lsnrctl start

# 3. 为中的任何CREATE TABLESPACE操作创建审核策略AUDIT_TABLESPACE。
CREATE AUDIT POLICY audit_tablespace ACTIONS create tablespace;

# 4.启用审核策略。
audit policy AUDIT_TABLESPACE;

# 5.检查审核策略是否存在。
col user_name format A10
 col policy_name format A20
SELECT * FROM AUDIT_UNIFIED_ENABLED_POLICIES
where POLICY_NAME like '%TABLESPACE%';

# 6.在pdb_orcl中创建一个新表空间，并验证该操作已被审核。一个。创建一个新表空间TBS_ORCL。
CREATE TABLESPACE tbs_orcl DATAFILE
'/u01/app/oracle/oradata/cdb2/pdb_orcl/tbs_orcl01.dbf'
SIZE 100M;
COL dbusername FORMAT a12
COL action_name FORMAT a20
COL object_name FORMAT a20
SELECT dbusername, action_name, object_name
FROM unified_audit_trail WHERE action_name like '%TABLESPACE%';
CONNECT / as sysdba
CREATE TABLESPACE tbs_root DATAFILE
'/u01/app/oracle/oradata/cdb2/ tbs_root01.dbf' SIZE 10M;
COL dbusername FORMAT a12
COL action_name FORMAT a20
COL object_name FORMAT a20
SELECT dbusername, action_name, object_name
FROM unified_audit_trail WHERE action_name like '%TABLESPACE%';
DROP TABLESPACE tbs_root INCLUDING CONTENTS AND DATAFILES;
CONNECT sys/oracle_4U@pdb_orcl AS SYSDBA
DROP TABLESPACE tbs_orcl INCLUDING CONTENTS AND DATAFILES;

# 7.为中的任何CREATE USER或DROP USER操作创建审核策略AUDIT_USER。
CONNECT system/oracle_4U@pdb2
SELECT policy_name, user_name FROM audit_unified_enabled_policies;
set pages 100
COL audit_option FORMAT A40
SELECT audit_option FROM audit_unified_policies
WHERE policy_name ='ORA_SECURECONFIG' ORDER BY 1;

# 8.在pdb2中以 lu_pdb2身份连接，并创建一个新用户并将其删除。
CREATE USER lu_pdb2 IDENTIFIED BY oracle_4U;
GRANT dba TO lu_pdb2;
CONNECT lu_pdb2/oracle_4U@pdb2
CREATE USER test IDENTIFIED BY test;
DROP USER test;

# 9.验证审核策略已审核这两项操作。使用UNIFIED_AUDIT_TRAIL视图。如果内存中的审计记录审计信息尚未刷新到表，请执行DBMS_AUDIT_MGMT.FLUSH_UNIFIED_AUDIT_TRAIL过程。
CONNECT system/oracle_4U@pdb2
COL dbusername FORMAT a12
COL action_name FORMAT a20
COL object_name FORMAT a20
SELECT dbusername, action_name, object_name
FROM unified_audit_trail
WHERE dbusername='LU_PDB2';

# 10.请注意，如果您连接到root用户并尝试读取为pdb2收集的审核记录，则不会找到任何信息。根目录下的UNIFIED_AUDIT_TRAIL视图仅显示根目录的审核记录（如果有）。从CDB_UNIFIED_AUDIT_TRAIL视图读取，即所有PDB的合并视图。
CONNECT / AS SYSDBA
COL action_name FORMAT a20
COL object_name FORMAT a20
SELECT dbusername, action_name, object_name
FROM unified_audit_trail
WHERE dbusername='LU_PDB2';

SELECT dbusername, action_name, object_name
FROM cdb_unified_audit_trail
WHERE dbusername='LU_PDB2';

EXIT
```

## 练习9-2：从非CDB导出并导入PDB

### 总览

在这种情况下，您将使用FULL TRANSPORTABLE模式从非CDB导出

orcl 并导入到新的PDB pdb_orcl中。

### 假设条件

在练习9-1之后，成功创建了 pdb_orcl 。

### 任务

```sql
# 1. 使用FULL TRANSPORTABLE模式导出非CDB Orcl。
ORACLE_SID = [cdb2] ? orcl
STARTUP
SELECT tablespace_name FROM dba_tablespaces;
SELECT count(*)FROM hr.employees;
ALTER TABLESPACE example READ ONLY;
ALTER TABLESPACE users READ ONLY;
rm /u01/app/oracle/admin/orcl/dpdump/expfull.dmp
expdp system/oracle_4U DUMPFILE=expfull.dmp FULL=Y
TRANSPORTABLE=ALWAYS LOGFILE=exp.log
# 2.将数据文件复制到目标位置
ORACLE_SID = [orcl] ? cdb2
sqlplus system/oracle_4U@pdb_orcl
SELECT tablespace_name FROM dba_tablespaces;
create directory dp_orcl as '/u01/app/oracle/admin/cdb2/dpdump';
EXIT
cp /u01/app/oracle/oradata/orcl/example01.dbf /u01/app/oracle/oradata/orcl/users01.dbf /u01/app/oracle/oradata/cdb2/pdb_orcl
cp /u01/app/oracle/admin/orcl/dpdump/expfull.dmp /u01/app/oracle/admin/cdb2/dpdump/expfull.dmp
# 3. 以FULL TRANSPORTABLE模式将orcl数据库导入pdb_orcl。由于APEX选项，存在许多错误，需要处理。但是，出于本练习的目的，可以忽略这些错误。请注意，impdp命令在userid子句中包含net service_name。
rm /u01/app/oracle/admin/cdb2/dpdump/import.log
impdp system/oracle_4U@pdb_orcl FULL=Y dumpfile=expfull.dmp directory=dp_orcl TRANSPORT_DATAFILES='/u01/app/oracle/oradata/cdb2/pdb_orcl/users01.dbf','/u01/app/oracle/oradata/cdb2/pdb_orcl/example01.dbf' logfile=import.log
# 4.检查表空间EXAMPLE和USERS到位，以及HR.EMPLOYEES
sqlplus sys/oracle_4U@pdb_orcl as sysdba
SELECT tablespace_name from DBA_TABLESPACES;
SELECT count(*) FROM hr.employees;
EXIT
# 5.将orcl的表空间设置回读写模式。
ORACLE_SID = [cdb2] ? orcl
sqlplus / as sysdba
ALTER TABLESPACE example READ WRITE;
ALTER TABLESPACE users READ WRITE;
EXIT
```

## 练习9-3：在PDB之间进行导出和导入

### 总览

在实践中，您将整个模式从一个PDB pdb_orcl导出到另一个PDB pdb2。

在同一CDB中。模式HR将从pdb_orcl导出并导入到pdb2中。

### 假设条件

该pdb_orcl一直在练习9-1和商店成功创建HR模式的表9-2练习之后。

如果无法成功创建pdb_orcl并使用ORCL HR模式导入pdb_orcl，请执行以下追赶脚本：

```SQL
cd /home/oracle/solutions/catchup_09_02
./cr_imp_PDB_ORCL.sh
```

来自非CDB Orcl的应用程序已成功导出，然后导入到在练习9-2中为 pdb_orcl 。

### 任务

```SQL
# 1.如果必须使用catchup_09_02脚本，则DP_ORCL目录不再存在。按照惯例9-2 3.b）重新创建目录。
sqlplus system/oracle_4U@pdb_orcl
create directory dp_orcl as
'/u01/app/oracle/admin/cdb2/dpdump';
EXIT
# 2. 从pdb_orcl导出HR模式。
expdp system/oracle_4U@pdb_orcl DUMPFILE=exppdb_orcl
DIRECTORY=dp_orcl SCHEMAS=hr

# 3.将模式HR导入到pdb2中。
# a.在pdb2中创建一个数据泵目录。
sqlplus system/oracle_4U@pdb2
CREATE DIRECTORY dp_pdb2 AS '/u01/app/oracle/admin/cdb2/dpdump';
# b.为HR模式创建表空间USERS和EXAMPLE。
CREATE TABLESPACE users DATAFILE
'/u01/app/oracle/oradata/cdb2/pdb2_1/users01.dbf'
SIZE 100M;

CREATE TABLESPACE example DATAFILE
'/u01/app/oracle/oradata/cdb2/pdb2_1/example01.dbf'
SIZE 100M;
EXIT

# c.将模式HR导入到pdb2中。
rm /u01/app/oracle/admin/cdb2/dpdump/import.log
impdp system/oracle_4U@pdb2 DIRECTORY=dp_pdb2 SCHEMAS=hr

# d.检查cdb2中有两个不同的HR本地用户，一个在pdb_orcl中，另一个在pdb2中。
sqlplus sys/oracle_4U@cdb2 as sysdba
COL username FORMAT A20
SELECT username, con_id, common FROM cdb_users WHERE username= 'HR';
EXIT
```
