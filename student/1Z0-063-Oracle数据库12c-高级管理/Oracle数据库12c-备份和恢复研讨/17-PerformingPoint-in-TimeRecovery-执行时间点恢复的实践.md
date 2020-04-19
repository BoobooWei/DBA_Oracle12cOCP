# 第17课：执行时间点恢复的实践

> 2020.04.19 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [第17课：执行时间点恢复的实践](#第17课：执行时间点恢复的实践)   
   - [实践概述](#实践概述)   
   - [练习17-1：从备份中恢复表](#练习17-1：从备份中恢复表)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   

<!-- /MDTOC -->

## 实践概述

在这些实践中，您将从备份集中恢复表，而不会影响表空间或架构中的其他对象。


## 练习17-1：从备份中恢复表

### 总览

在这种实践中，您将从备份集中恢复表（不影响表空间或架构中的其他对象）。这些任务包括以下内容：

* 设置测试环境并确认配置，这通常是一项一次性的任务。
* 在RMAN中，执行0级备份以及归档日志，并删除过时的备份。
* 在SQL * Plus中，创建并填充一个新的TEST_TABLE。提交后记下SCN。
* 在RMAN中，执行1级备份。
* 在SQL * Plus中，通过清除TEST_TABLE来创建恢复表的需求。
* 在RMAN中，将测试表恢复到SCN。
* 在SQL * Plus中，确认恢复成功。
* 清理练习环境。

### 假设条件

打开两个终端窗口，您以oracle OS用户身份登录。$LABS是当前目录。为orcl数据库实例设置了环境变量。

### 任务

```SQL
# 1.通过执行$ LABS目录中的setup_17_01.sh脚本进行设置。
# 在此脚本中：创建了新的表空间和用户。用户创建一个表并填充它。输出在/tmp/setup.log文件中。
./setup_17_01.sh
# 2.启动一个SQL * Plus会话并验证您的测试配置。一种。
# a.以SYS用户身份登录。
sqlplus / as sysdba
# b.确认数据库处于存档日志模式。
SELECT NAME, LOG_MODE, OPEN_MODE FROM V$DATABASE;
# c.确认兼容性设置为12.0或更高。
show parameter compatible
# d.确认您的备份位置和大小。
show parameter recovery_f
# e.通过执行$LABS目录中的lab_17_02.sql脚本来确认设置。
# 该BAR用户应该拥有BARCOPY表。
@lab_17_02.sql
COL TABLE_NAME FORMAT A30
COL TABLESPACE_NAME FORMAT A15
COL OWNER FORMAT A10
SELECT TABLE_NAME, TABLESPACE_NAME, STATUS
FROM DBA_TABLES
WHERE OWNER = 'BAR';
# 3.在另一个终端窗口中，启动RMAN会话并连接到您的ORCL数据库作为目标实例。
# 注意：将RMAN输出发送到日志文件和标准输出的最简单方法是使用Linux tee命令或其等效命令。如果您的标准输出允许您随意滚动，则无需执行此操作。您可以在
rman target "'/ as sysbackup'"| tee /home/oracle/rman_17.log
# 4.确认或配置控制文件的自动备份，然后执行0级备份。
show CONTROLFILE AUTOBACKUP;
backup incremental level 0 database plus archivelog;
# 5.在SQL * Plus中，创建并填充一个名为BAR.TEST_TABLE的新表。注意SCN
@lab_17_05.sql
# 6.在RMAN会话中，执行1级备份。如果使用tee命令启动RMAN会话，那么您的输出将重定向到/home/oracle/rman_17.log文件。
backup incremental level 1 database plus archivelog;
# 7.在SQL * Plus中，创建需要通过清除表来恢复表的需求。（可选）查看您的SCN
# 注意：假定您没有此表重复或在闪回日志中或其他任何位置，因此需要从备份中恢复它。
SELECT NAME, CURRENT_SCN FROM V$DATABASE;
# 8.（可选）查看BAR用户拥有的当前表。该TEST_TABLE不应显示。
# 9.在RMAN中，将测试表恢复到YOUR SCN。在以下输入中提供RECOVER 命令
# * 要恢复的表或表分区的名称
# * 需要将表或表分区恢复到的SCN（或时间点）
# * 是否必须将恢复的表或表分区导入目标数据库（默认为Yes）。
# * 辅助目标“ / u01 / backup / test”。
# 首先，确认辅助目标目录为空，然后执行RECOVER 命令。
HOST "ls /u01/backup/test/*";
# 注意：RECOVER命令之前的此肯定错误确认辅助目标为空。
RECOVER TABLE BAR.TEST_TABLE UNTIL SCN 4379440 <<Your SCN 2> AUXILIARY DESTINATION '/u01/backup/test';
# 注意：RMAN使用您的输入来自动执行恢复指定表的过程。它执行以下任务：
# a.RMAN根据您的SCN确定备份。
# b.RMAN创建一个辅助实例。
# c.RMAN会在指定的时间点之前将表或表分区恢复到该辅助实例中。
# d.RMAN创建一个包含已恢复对象的数据泵导出转储文件。
# e.RMAN将恢复的对象导入目标数据库。
# f.RMAN删除辅助实例。
# 10.删除过时的存档日志，然后退出RMAN。
delete noprompt obsolete;
exit
# 11.在SQL * Plus中，查询测试表的所有行以确认恢复成功。然后退出。
SELECT * FROM BAR.TEST_TABLE;
exit
# 12.通过执行cleanup_17_01.sh脚本来清理练习环境。该脚本将删除原始和传输的表空间以及备份和转储文件。输出在/tmp/cleanup.log文件中。
./cleanup_17_01.sh
```
