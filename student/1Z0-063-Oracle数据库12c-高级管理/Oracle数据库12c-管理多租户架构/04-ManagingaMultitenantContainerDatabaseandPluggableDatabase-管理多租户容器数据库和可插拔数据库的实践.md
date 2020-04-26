# 第4课：管理多租户容器数据库和可插拔数据库的实践

> 2010.04.26 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [第4课：管理多租户容器数据库和可插拔数据库的实践](#第4课：管理多租户容器数据库和可插拔数据库的实践)   
   - [第4课：概述的练习](#第4课：概述的练习)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
   - [练习4-1：CDB的关机和启动](#练习4-1：cdb的关机和启动)   
      - [总览](#总览)   
      - [任务](#任务)   
   - [练习4-2：关闭和打开PDB](#练习4-2：关闭和打开pdb)   
      - [总览](#总览)   
      - [任务](#任务)   
   - [练习4-3：在启动触发器后创建以打开所有PDB](#练习4-3：在启动触发器后创建以打开所有pdb)   
      - [总览](#总览)   
      - [任务](#任务)   
   - [练习4-4：更改PDB的GLOBAL_NAME](#练习4-4：更改pdb的global_name)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习4-5：更改实例参数](#练习4-5：更改实例参数)   
      - [总览](#总览)   
      - [任务](#任务)   
   - [练习4-6：更改PDB中的操作行为](#练习4-6：更改pdb中的操作行为)   
      - [总览](#总览)   
      - [任务](#任务)   

<!-- /MDTOC -->

## 第4课：概述的练习

### 总览

在这种实践中，您将在CDB上执行启动和关闭操作，在PDB上执行打开和关闭操作，以及与PDB的连接以显示当前上下文。

### 假设条件

* 完成第三节课的所有练习。
* 在此步骤中，不必成功创建pdb1_1和pdb_orcl2。


## 练习4-1：CDB的关机和启动

### 总览

在这种情况下，您将关闭cdb2并启动cdb2。

### 任务

```SQL
# 1.连接到容器数据库cdb2以将其关闭。一个。以具有SYSDBA特权的用户身份连接到CDB 。
# a.设置环境变量并连接数据库实例
ORACLE_SID = cdb2
sqlplus / as sysdba
select name, cdb, con_id from v$database;
# 立即关闭数据库
shutdown immediate
EXIT
# 探索后台流程
ps -ef|grep cdb2
# 2.连接到容器数据库cdb2并启动它。
sqlplus / as sysdba
select name, cdb, con_id from v$database;
# 3. 探索后台进程
ps -ef|grep cdb2
# 4.探索PDB。
sqlplus / as sysdba
select CON_ID, NAME, OPEN_MODE from v$pdbs;
# 5.打开所有的pdb
alter pluggable database all open;
select CON_ID, NAME, OPEN_MODE from v$pdbs;
# 6.连接到cdb2中的PDB2_1
connect sys/oracle_4U@PDB2_1 AS SYSDBA
select CON_ID, NAME, OPEN_MODE from v$pdbs;
show con_name
# 7.连接到cdb2中的PDB2_2
connect sys/oracle_4U@PDB2_2 AS SYSDBA
select CON_ID, NAME, OPEN_MODE from v$pdbs;
show con_name
```

## 练习4-2：关闭和打开PDB

### 总览

在这种情况下，您将关闭PDB并打开PDB。

### 任务

```SQL
sqlplus / as sysdba
select name, cdb, con_id from v$database;
shutdown immediate
startup
select CON_ID, NAME, OPEN_MODE from v$pdbs;
alter pluggable database all open;

# Close PDB2_1.
sqlplus sys/oracle_4U@pdb2_1 as sysdba
create table system.mytab (c number);
insert into system.mytab values (1);
COMMIT;
exit

alter pluggable database pdb2_1 close immediate;
select CON_ID, NAME, OPEN_MODE from v$pdbs;

connect system/oracle_4U@pdb2_1

# Open pdb2_1.
connect / as sysdba
alter pluggable database PDB2_1 open;
connect system/oracle_4U@PDB2_1
select * from system.mytab;


CONNECT / AS SYSDBA
select name, cdb, con_id from v$database;
shutdown immediate
startup nomount
select CON_ID, NAME, OPEN_MODE from v$pdbs;
# Mount cdb2.
alter database mount;
select CON_ID, NAME, OPEN_MODE from v$pdbs;

# Open cdb2.
alter database open;
select CON_ID, NAME, OPEN_MODE from v$pdbs;

# Open all PDBs except PDB2_2.
alter pluggable database all except pdb2_2 open;
select CON_ID, NAME, OPEN_MODE from v$pdbs;

# Close all pluggable databases except pdb2_1 and pdb1_1.
alter pluggable database all except pdb2_1, pdb1_1 close;
select CON_ID, NAME, OPEN_MODE from v$pdbs;
```


## 练习4-3：在启动触发器后创建以打开所有PDB

### 总览

在实践中，您将创建`AFTER STARTUP`触发器以打开CDB的所有PDB。

### 任务

```SQL
# 1.在cdb2中创建一个触发器，以在启动cdb2之后自动打开所有PDB 。
# a.创建触发器。
CREATE TRIGGER open_all_PDBs
         AFTER STARTUP ON DATABASE
begin
execute immediate 'alter pluggable database all open';
        end open_all_PDBs;
       /
# b. 关闭cdb2。
shutdown immediate
# c. 启动cdb2
startup
# d.请注意此时PDB都处于READ WRITE打开模式。
select CON_ID, NAME, OPEN_MODE from v$pdbs;
```


## 练习4-4：更改PDB的GLOBAL_NAME

### 总览

在这种实践中，您将更改PDB的打开模式以进行特定操作。

### 假设条件

如果无法成功创建触发器，请执行以下追赶脚本：

```bash
cd / home / oracle / solutions / catchup_04_03
./cr_trig.sh
```

### 任务

```SQL
# 1. 修改pdb的名字
CONNECT sys/oracle_4U@pdb2_1 as sysdba
alter pluggable database close immediate;
alter pluggable database open restricted;
select CON_ID, NAME, OPEN_MODE, RESTRICTED from v$pdbs;
alter pluggable database RENAME GLOBAL_NAME TO pdb2;
select CON_ID, NAME, OPEN_MODE, RESTRICTED from v$pdbs;
alter pluggable database close immediate;
alter pluggable database open;
select CON_ID, NAME, OPEN_MODE, RESTRICTED from v$pdbs;
```

## 练习4-5：更改实例参数

### 总览

在实践中，您将发现实例参数更改对PDB的影响。

### 任务

```SQL
# 1.在此示例中，您将在cdb2中使用实例参数`OPTIMIZER_USE_SQL_PLAN_BASELINES`，因为它在`V$PARAMETER`中是`ISPDB_MODIFIABLE` 。
CONNECT / AS SYSDBA
select ISPDB_MODIFIABLE from v$parameter where name='optimizer_use_sql_plan_baselines';
show parameter optimizer_use_sql_plan_baselines;

# 2. 配置监听服务
# 使用netca 为pdb2的可插入数据库添加PDB2网络服务名称
netca
lsnrctl reload


# 3.在pdb2中将实例参数值更改为FALSE。
sqlplus sys/oracle_4U@pdb2 AS SYSDBA
show parameter optimizer_use_sql_plan_baselines
ALTER SYSTEM SET optimizer_use_sql_plan_baselines=FALSE SCOPE=BOTH;
show parameter optimizer_use_sql_plan_baselines

# 4. pdb2_2中查看参数值还是 True（检查同一CDB的其他PDB中的实例参数值。）
CONNECT sys/oracle_4U@pdb2_2 AS SYSDBA
show parameter optimizer_use_sql_plan_baselines

# 5.pdb2中关闭pdb再开启查看参数值 FALSE
CONNECT sys/oracle_4U@pdb2 AS SYSDBA
ALTER PLUGGABLE DATABASE CLOSE IMMEDIATE;
ALTER PLUGGABLE DATABASE OPEN;
show parameter optimizer_use_sql_plan_baselines

# 6. 在CDB关闭/启动后检查实例参数值。
connect / as sysdba
shutdown immediate
STARTUP
col VALUE format a20
select CON_ID, VALUE from V$SYSTEM_PARAMETER where name ='optimizer_use_sql_plan_baselines';
```

<img src='https://i.loli.net/2020/04/26/kEwtnlYHoZ6cLN5.jpg' alt='kEwtnlYHoZ6cLN5'/>


## 练习4-6：更改PDB中的操作行为

### 总览

在实践中，您将限制特定PDB中的会话。

### 任务

```SQL
# 1. 仅限制pdb2_2中的会话
CONNECT sys/oracle_4U@pdb2_2 AS SYSDBA
# 在pdb2_2中启用受限会话
ALTER SYSTEM ENABLE RESTRICTED SESSION;
# 在pdb2_2中创建一个用户
CREATE USER u1_pdb2_2 IDENTIFIED BY oracle_4U;
GRANT create session TO u1_pdb2_2;
# 尝试以用户u1_pdb2_2在pdb2_2中进行连接。
CONNECT u1_pdb2_2/oracle_4U@pdb2_2 ERROR:
# ORA-01035: ORACLE only available to users with RESTRICTED
# SESSION privilege
# Warning: You are no longer connected to ORACLE.

# 2. 在另一个PDB pdb2中创建一个用户。
CONNECT sys/oracle_4U@pdb2 AS SYSDBA Connected.
CREATE USER u1_pdb2 IDENTIFIED BY oracle_4U;
GRANT create session TO u1_pdb2;
CONNECT u1_pdb2/oracle_4U@pdb2
# 连接成功

# 3. 取消对pdb2_2的会话限制
CONNECT sys/oracle_4U@pdb2_2 AS SYSDBA
alter pluggable database close;
alter pluggable database open;
CONNECT u1_pdb2_2/oracle_4U@pdb2_2
# 连接成功

# 4. 删除用户
CONNECT sys/oracle_4U@pdb2_2 AS SYSDBA
DROP USER u1_pdb2_2;
CONNECT sys/oracle_4U@pdb2 AS SYSDBA
DROP USER u1_pdb2;
EXIT
```
