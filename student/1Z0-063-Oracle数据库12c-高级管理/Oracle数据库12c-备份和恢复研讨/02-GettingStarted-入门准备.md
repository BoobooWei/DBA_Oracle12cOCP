# 第2课的练习：入门

> 2020-03-02 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [第2课的练习：入门](#第2课的练习：入门)   
   - [第2课的练习：概述](#第2课的练习：概述)   
   - [练习2-1：以NOARCHIVELOG模式备份](#练习2-1：以noarchivelog模式备份)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习2-2：创建恢复用的测试用例](#练习2-2：创建恢复用的测试用例)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习2-3：以NOARCHIVELOG模式进行恢复](#练习2-3：以noarchivelog模式进行恢复)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   

<!-- /MDTOC -->

## 第2课的练习：概述

在这些实践中，您将执行数据库备份，创建要在恢复操作中使用的测试用例，并执行数据库恢复。

## 练习2-1：以NOARCHIVELOG模式备份

### 总览

在这种强制性做法中，您将调用RMAN客户端并使用默认设置执行数据库备份。备份将在以后的实践中用于执行恢复（在有意引入的“灾难”之后）。

### 假设条件

您从终端窗口开始，其中环境变量指向ORCL实例。（如果不确定如何使用oraenv设置环境变量，请重新使用以前的做法。）

### 任务

1.SYS用户登陆后解锁SYSBACKUP用户

```sql
--解锁备份用户sysbackup
select username,account_status from dba_users where username = 'SYSBACKUP';
alter user sysbackup identified by oracle container=all;
alter user sysbackup account unlock;
```

2.SYSBACKUP用户登陆RMAN尝试备份数据库

```sql
rman target "'/ as sysbackup'"
select name,dbid,log_mode from v$database;
select user from dual;
show all;
backup database;
--得到一个报错“RMAN-06149：无法以NOARCHIVELOG模式备份数据库”
```

**注意：**以SYSBACKUP特权登录（将您连接为SYSBACKUP用户），这与SYSDBA特权非常相似，不同之处在于它不包括用户内容表的 SELECT 特权。默认情况下， SYSDBA 可以查看用户表的内容，但 SYSBACKUP 无法看到。（两者都可以查询数据字典和动态视图。）

 3.MOUNT模式下RMAN备份数据库

```sql
shutdown immediate;
startup mount;
backup database;
ALTER DATABASE OPEN;
list backup;
delete obsolete;
```

问题：备份集中有哪些对象？

答案：第一个备份集包含数据文件备份。第二个备份集包含控制文件和SPFILE备份。

使用DELETE OBSOLETE命令确定是否可以通过删除重复项来节省空间

## 练习2-2：创建恢复用的测试用例

### 总览

在实践中，您将创建第一个测试用例，它是一个新的表空间，用户和一个表。

### 假设条件

您已完成练习2-1（并已在NOARCHIVELOG中备份了已关闭的数据库模式）。

`export LABS=/home/oracle/labs`除非另有说明，否则请在此目录中开始所有实践。

脚本路径：[实验脚本路径](labs)

### 任务

1.根据自己的实际情况修改脚本中用户密码和服务名等信息后，运行。

```bash
bash setup_02_02.sh
```

此脚本创建BAR22用户，BAR22TBS表空间 和BARCOPY表。

2.查看日志`/tmp/setup.log`文件

3.以SYSDBA身份登录到SQL * Plus，模拟误操作删除表空间文件。

```sql
--查看新表,总数应为428
conn sys/oracle@booboopdb1 as sysba;
select * from bar22.barcopy where rownum <2;
select count(*) from bar22.barcopy;

--查看归档模式
archive log list

--强制关闭实例
conn / as sysba;
shutdown abort
--查看实例进程
!pgrep -lf pmon
--删除数据文件
!rm /u01/backup/booboo/bar22tbs01.dbf

--启动实例
startup
```

 报错如下：

```sql
SYS@booboo>startup
ORACLE instance started.

Total System Global Area  855638016 bytes
Fixed Size		    8798504 bytes
Variable Size		  327159512 bytes
Database Buffers	  511705088 bytes
Redo Buffers		    7974912 bytes
Database mounted.
ORA-01157: cannot identify/lock data file 34 - see DBWR trace file
ORA-01110: data file 34: '/u01/backup/booboo/bar22tbs01.dbf'
```



## 练习2-3：以NOARCHIVELOG模式进行恢复

### 总览

在这种情况下，您将使用RMAN客户端恢复数据库。

### 假设条件

此练习具有用于学习目的的可选步骤。

### 任务

由于数据库处于NOARCHIVELOG模式，因此恢复数据库有两种方法：

1）删除缺少的表空间，保证其他表空间完整

2）恢复整个数据库到上一次备份的时间

以下步骤选择选项1)：

```sql
--cdb中查看当前实例的status为 mount
select status from v$instance;

--切换到pdb中，因为损坏的表空间文件是在pdb中创建。查看数据文件和状态
alter session set container=booboopdb1;
select name,status from v$datafile where name like '%bar22tbs01%';
--删除丢失的数据文件
ALTER DATABASE DATAFILE '/u01/backup/booboo/bar22tbs01.dbf' OFFLINE FOR DROP;

--切换到根，打开数据库
alter session set container=cdb$root;
ALTER DATABASE OPEN;

--切换到pdb中，删除丢失的数据文件所在的表空间
alter session set container=booboopdb1;
DROP TABLESPACE BAR22TBS INCLUDING CONTENTS AND DATAFILES;
```
