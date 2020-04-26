# 第5课：在CDB和PDB中管理表空间的实践

> 2010.04.26 BoobooWei

## 第5课：概述的练习

### 实践概述

在这种实践中，您将管理根目录和PDB中的表空间。

### 假设条件

* 练习3-1成功创建了cdb2。
* 练习3-3成功创建了pdb2_1。
* 练习4-4已成功将pdb2_1重命名为pdb2。


## 练习5-1：管理永久和临时表空间

### 总览

在这种实践中，您将在根目录和PDB中管理永久和临时表空间。

### 任务

```sql
# 1.查看cdb2中的永久和临时表空间属性。
ORACLE_SID = cdb2
sqlplus / as sysdba

col PROPERTY_NAME format a30
col PROPERTY_VALUE format a25
SELECT property_name, property_value
FROM database_properties
WHERE property_name LIKE 'DEFAULT_%TABLE%';

SELECT tablespace_name, CON_ID from CDB_TABLESPACES;

SELECT tablespace_name, CON_ID from CDB_TABLESPACES
WHERE TABLESPACE_NAME LIKE 'TEMP%';
# 2.在根容器中创建一个永久表空间CDATA。
CREATE TABLESPACE CDATA
DATAFILE '/u01/app/oracle/oradata/cdb2/cdata_01.dbf'
SIZE 10M ;

SELECT tablespace_name, CON_ID from CDB_TABLESPACES
WHERE TABLESPACE_NAME = 'CDATA';

# 3.将CDATA表空间设置为根容器中的默认表空间。
ALTER DATABASE DEFAULT TABLESPACE CDATA ;
SELECT property_name, property_value
FROM database_properties
WHERE property_name LIKE 'DEFAULT_%TABLE%';

# 4.创建一个永久表，LDATA，在PDB2。
connect system/oracle_4U@PDB2

CREATE TABLESPACE ldata DATAFILE
'/u01/app/oracle/oradata/cdb2/pdb2_1/ldata_01.dbf'
SIZE 10M ;

# 5.将LDATA表空间设置为PDB2容器中的默认表空间。
ALTER PLUGGABLE DATABASE DEFAULT TABLESPACE LDATA ;
SELECT property_name, property_value
FROM database_properties
WHERE property_name LIKE 'DEFAULT_%TABLE%';

# 6.在根容器中创建一个临时表空间。
connect system/oracle_4U
CREATE TEMPORARY TABLESPACE TEMP_ROOT
TEMPFILE '/u01/app/oracle/oradata/cdb2/temproot_01.dbf'
SIZE 500M ;

# 7.将TEMP_ROOT设为根容器中的默认临时表空间。
ALTER DATABASE DEFAULT TEMPORARY TABLESPACE TEMP_ROOT ;
SELECT property_name, property_value
FROM database_properties
WHERE property_name LIKE 'DEFAULT_%TABLE%';

# 8.创建临时表空间TEMP_PDB2在PDB2。
connect system/oracle_4U@PDB2
CREATE TEMPORARY TABLESPACE TEMP_PDB2 TEMPFILE
'/u01/app/oracle/oradata/cdb2/pdb2_1/temppdb2_01.dbf'
SIZE 100M ;

# 9.进行TEMP_PDB2在默认的临时表空间PDB2。
ALTER DATABASE DEFAULT TEMPORARY TABLESPACE TEMP_PDB2 ;
SELECT property_name, property_value
FROM database_properties
WHERE property_name LIKE 'DEFAULT_%TABLE%';

# 10.创建一个临时表空间MY_TEMP在PDB2。
CREATE TEMPORARY TABLESPACE MY_TEMP TEMPFILE
'/u01/app/oracle/oradata/cdb2/pdb2_1/my_temp_pdb2_01.dbf'
SIZE 10M;

# 11.在cdb2中显示另一个PDB的默认表空间。
connect system/oracle_4U@PDB_ORCL2
SELECT property_name, property_value
FROM database_properties
WHERE property_name LIKE 'DEFAULT_%TABLE%';

# 12.管理用户的默认永久和临时表空间。
# a.创建一个普通用户C##U.
connect system/oracle_4U
CREATE USER c##u IDENTIFIED BY x;

# b.查看所有容器中用户c##u的默认表空间和临时表空间分配。
COLUMN username format A12
COLUMN default_tablespace format A18
COLUMN temporary_tablespace format A20
COLUMN con_id format 999
SELECT username, default_tablespace,
temporary_tablespace, con_id
FROM CDB_USERS
WHERE username = 'C##U';
# c.在PDB2中创建本地用户LU。
connect system/oracle_4U@PDB2
CREATE USER lu IDENTIFIED BY x;
# d.查看用户LU的默认表空间和临时表空间分配。
SELECT username, default_tablespace, temporary_tablespace
FROM DBA_USERS
WHERE username = 'LU';
# e.将用户LU的临时表空间分配更改为PDB2中的MY_TEMP。
ALTER USER lu TEMPORARY TABLESPACE MY_TEMP;
# f.查看用户LU的默认表空间和临时表空间分配。
SELECT username, default_tablespace, temporary_tablespace
FROM DBA_USERS
WHERE username = 'LU';
```

## 练习5-2：管理UNDO表空间
### 总览

在实践中，您将管理UNDO表空间。

### 任务

```SQL
# 1.显示CDB中使用的UNDO表空间。
connect system/oracle_4U
col NAME format A12
select FILE#, ts.name, ts.ts#, ts.con_id
from v$datafile d, v$tablespace ts
where d.ts#=ts.ts#
and d.con_id=ts.con_id
and ts.name like 'UNDO%';
# 2.在PDB中创建一个UNDO表空间，并将其设置为CDB 的UNDO_TABLESPACE。
connect system/oracle_4U@PDB2
CREATE UNDO TABLESPACE UNDO_PDB2 DATAFILE
'/u01/app/oracle/oradata/cdb2/pdb2/undo_pdb2_01.dbf'
SIZE 10M;
alter system set undo_tablespace='UNDO_PDB2' scope=both;
#alter system set undo_tablespace='UNDO_PDB2' scope=both
#*
#ERROR at line 1:
#ORA-65040: operation not allowed from within a pluggable database
EXIT
```
