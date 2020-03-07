# 第4课：使用RMAN恢复目录的实践

> 2020.03.08 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [第4课：使用RMAN恢复目录的实践](#第4课：使用rman恢复目录的实践)   
   - [第4课：概述的练习](#第4课：概述的练习)   
   - [练习4-1：创建恢复目录所有者](#练习4-1：创建恢复目录所有者)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习4-2：创建恢复目录](#练习4-2：创建恢复目录)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习4-3：在恢复目录中注册数据库](#练习4-3：在恢复目录中注册数据库)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习4-4：为RMAN目录配置企业管理器](#练习4-4：为rman目录配置企业管理器)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习4-5：配置恢复目录以进行恢复](#练习4-5：配置恢复目录以进行恢复)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   

<!-- /MDTOC -->

## 第4课：概述的练习

在这些实践中，您将执行一次性设置任务：

- 创建恢复目录所有者。

- 创建恢复目录。

- 在恢复目录中注册数据库。

- 执行EM设置任务。

然后，您将通过创建备份来准备您的培训环境，如果您无法完成所描述的练习，则该备份将使您能够还原数据库。

## 练习4-1：创建恢复目录所有者

### 总览

在该实践中，您将创建用户并授予适当的特权。

### 假设条件

准备一个新实例`ORACLE_SID`为 `rcat`

初始化参数设置如下：

DB_RECOVERY_FILE_DEST = /u01/app/oracle/fast_recovery_area
DB_RECOVERY_FILE_DEST_SIZE = 10000M

### 任务

```sql
--以具有SYSDBA角色的SYS用户身份登录到SQL * Plus
sqlplus / as sysdba
--创建一个名为 rcatbs 来保存存储库数据,大小为15 MB
create tablespace rcatbs datafile '/u01/backup/booboo/rcat01.dbf' SIZE 15M REUSE;
--创建一个名为 rcatownerde 的用户，默认表空间rcatbs，且在rcatbs上无限制的配额；
CREATE USER rcatowner IDENTIFIED BY "oracle_4U" DEFAULT
TABLESPACE rcatbs
QUOTA unlimited on rcatbs;
--将RECOVERY_CATALOG_OWNER角色授予RCATOWNER用户
GRANT recovery_catalog_owner to rcatowner;
--从SQL * Plus退出
exit
```



## 练习4-2：创建恢复目录

### 总览

在该实践中，您可以使用RMAN在恢复目录数据库中创建恢复目录。

### 假设条件

您完成了之前的练习。

### 任务

```sql
--使用RMAN连接到恢复目录数据库。以您刚刚创建的恢复目录所有者的身份登录。
rman catalog rcatowner@rcat
--创建恢复目录。此命令可能需要几分钟才能完成。
create catalog;
```

## 练习4-3：在恢复目录中注册数据库

### 总览

在该实践中，您可以使用RMAN 在刚创建的恢复目录中注册数据库`booboo`。

### 假设条件

您完成了之前的练习。

### 任务

```sql
--修改数据库环境变量ORACLE_SID为待备份恢复的数据库实例
export ORACLE_SID=booboo
--使用RMAN连接到目标数据库（要注册）和恢复目录数据库。
rman target "'/ as sysbackup'" catalog rcatowner@rcat
--在目录中注册数据库。
register database;
--验证注册是否成功
REPORT SCHEMA;
```

注意：

- 数据文件的大小可能会因您的设置而异。
- 如果未连接到恢复目录，则RB segs列中将包含`***`作为值。当您连接到恢复目录时，它包含Yes和N。



## 练习4-4：为RMAN目录配置企业管理器

### 总览

在该实践中，您也可以使用企业云管理器来配置RMAN目录

### 假设条件

您完成了之前的练习。

企业管理云控制12C安装。

### 任务

图形化界面操作忽略。

## 练习4-5：配置恢复目录以进行恢复

### 总览

公司要求，如果恢复目录丢失或损坏，则需要快速，完整地恢复它。

在该实践中，您将为恢复目录配置保留策略（保留两个备份），启用存档日志模式，并备份RCAT数据库。

您备份恢复目录以实施应用于映像副本的增量备份的备份策略。通过切换到映像副本，而不是将备份复制回原始位置，这提供了一种快速还原的方法。

该练习在命令行中执行任务，但是SYSDBA也可以在Cloud中执行它们控制。（在**一个**选定的界面中执行任务。）

导航提示：

- **配置保留策略**：RCAT主页>可用性>备份和恢复>备份设置>策略选项卡>备份存档的重做日志文件，并备份指定的次数后>备份：2
- **要启用存档日志模式**： RCAT主页>可用性>备份和恢复>恢复设置> ARCHIVELOG模式* *必须重新启动数据库*。（向导将指导您完成这些步骤。）
- **备份数据库**：RCAT主页>可用性>备份和恢复>计划备份>计划自定义备份>完整>下一步>磁盘>下一步>一次（立即）>下一步>提交作业。

### 假设条件

您完成了之前的练习。

企业管理云控制12C安装。

### 任务

图形化界面操作忽略。

命令行如下：

```sql
--配置至少具有冗余性的保留策略2
rman target sys@rcat
show retention policy;
configure retention policy to redundancy 2;
--开启归档模式
sqlplus / as sysdba
SHUTDOWN IMMEDIATE
ALTER DATABASE ARCHIVELOG;
ALTER DATABASE OPEN;
archive log list
exit
--将恢复目录数据库备份为映像副本和增量备份的基础。
--通过切换到映像副本，而不是将备份复制回原始位置，这提供了一种快速还原的方法。
rman target sys@rcat
BACKUP AS COPY INCREMENTAL LEVEL 0 DATABASE;
```
