# 第11课：执行恢复的实践

> 2020-04-19 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [第11课：执行恢复的实践](#第11课：执行恢复的实践)   
   - [实践概述](#实践概述)   
   - [练习11-1：从介质故障中恢复（数据文件丢失）](#练习11-1：从介质故障中恢复（数据文件丢失）)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习11-2：从介质故障中恢复：不完全恢复](#练习11-2：从介质故障中恢复：不完全恢复)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   

<!-- /MDTOC -->

## 实践概述

在这些实践中，您将从许多不同的数据库故障中恢复。强烈建议您完成开始的步骤，因为这会影响以下做法。

确保已完成练习7-3“创建档案备份”，因为如果恢复失败，则可以将重复的数据库文件用于“救援”操作。

通过使用归档备份的可能的恢复步骤：

a.在环境变量指向ORCL实例的情况下，登录到RMAN并连接到RMAN目录：

```SQL
rman target "'/ as sysbackup'" catalog rcatowner@rcat
```

b.确认要使用的还原点的名称，在此示例中为KEEPDB：

```SQL
LIST RESTORE POINT ALL;
```

C.使用还原点还原和恢复数据库：

```SQL
RESTORE DATABASE UNTIL RESTORE POINT 'KEEPDB';
RECOVER DATABASE UNTIL RESTORE POINT 'KEEPDB';
```

d.由于您的数据库现在处于较早的时间，因此请使用RESETLOGS打开数据库

```SQL
ALTER DATABASE OPEN RESETLOGS;
SELECT DBID FROM V$DATABASE;
```

e.在大多数环境中，Oracle建议在恢复后执行新的备份：

```SQL
BACKUP DATABASE PLUS ARCHIVELOG DELETE INPUT;
```

## 练习11-1：从介质故障中恢复（数据文件丢失）

### 假设条件

您将打开一个终端窗口，其中$ LABS作为当前目录。为orcl实例设置了环境变量。
假设条件

您将打开一个终端窗口，其中$ LABS作为当前目录。为orcl实例设置了环境变量。
在这种情况下，您将首先通过删除USERS数据文件来创建问题。然后，由于丢失了重要的数据文件，您将对数据库进行完全恢复。在这种情况下

无法使用Data Recovery Advisor。

### 任务

```SQL
# 1.执行setup_11_01.sh脚本，并确保您以ARCHIVELOG模式备份了整个数据库。例如，使用RMAN客户端执行：
BACKUP DATABASE PLUS ARCHIVELOG DELETE INPUT;
# 或使用backup_orcl.sh脚本。输出在/tmp/backup.log中。
./setup_11_01.sh
./backup_orcl.sh

# 2.通过执行break_11_01.sh脚本来删除USERS表空间数据文件，从而导致数据库故障。脚本的输出重定向到/tmp/break.log
./break_11_01.sh

# 3.（可选）使用cat在其他终端窗口中查看输出文件/tmp/break.log Linux命令。您可以在 break_11_01.sh时查看输出
cat /tmp/break.log
# 4.以oracle用户身份在终端窗口中继续，环境变量指向orcl实例。在SQL * Plus中，尝试启动orcl数据库实例。注意错误消息。
sqlplus / as sysdba
startup
# 5.诊断故障。请遵循错误消息的说明（出于培训目的）。
# a.错误消息代码的含义是什么？
# 使用oerr实用程序确定错误代码的含义。
# − ORA-01110 ____________________________
# − ORA-01157 ____________________________
! oerr ORA 01110
! oerr ORA 1157
# b.查找并检查步骤5中STARTUP命令的输出中列出的DBWR跟踪文件。
# 该跟踪文件将位于DIAGNOSTIC_DEST参数中列出的目录中，
# 位于 diag/rdbms/orcl/orcl/trace子目录中。
 show parameter DIAG
# c.将 diag/rdbms/orcl/orcl/trace  子目录添加到路径名或显示Background_DUMP_DEST参数。然后退出SQL * Plus。
show parameter dump_dest
exit
# d.使用另一个终端窗口。该 ls -ltr 的时间按相反的顺序跟踪目录Linux命令列表中的文件。最新文件列在最后。
cd /u01/app/oracle/diag/rdbms/orcl/orcl/trace
ls -ltr *dbw*
# e.查看最后一个跟踪dbw跟踪文件。通常，最后一条错误消息列出了原因。
cat orcl_dbw0_30109.trc
# 问题：出什么问题了？
# 答： USERS表空间中的数据文件丢失。
# 6.使用RMAN命令行（指向orcl实例）并检查哪些数据
rman target "'/ as sysbackup'"
list failure;
# 7.使用RMAN命令行，收集建议并修复故障。还原，恢复和打开数据库。
# 收集建议
advise failure;
# （可选）预览潜在的修复：
repair failure preview;
# 要修复故障并打开数据库：
repair failure;
# 8.检查Data Recovery Advisor是否有其他故障。
list failure;
# 9.使用$LABS目录中的cleanup_11_01.sh脚本清理您的测试用例
./cleanup_11_01.sh
# 10.在RMAN会话中，使用BACKUP DATABASE PLUS ARCHIVELOG命令备份数据库。输出在/tmp/backup.log中。然后退出。
BACKUP DATABASE PLUS ARCHIVELOG;
exit
```

## 练习11-2：从介质故障中恢复：不完全恢复

### 总览

在这种实践中，您设置了需要不完全恢复的方案。

然后你执行上次备份后缺少归档日志时（和存在无法重新创建的事务）时需要执行的步骤；

因此，不可能完全恢复。

### 假设条件

存在完整备份，并且从备份时间到当前时间的归档日志文件可用。

### 任务

```sql
# 1.通过执行$LABS目录中的setup_11_02.sh脚本进行设置。
# 使用此脚本可以创建新的表空间和用户。
# 用户创建一个表并填充它。执行表空间的备份，然后更新表。
# 输出在/tmp/setup.log文件中。
./setup_11_02.sh
# 2.通过执行break_11_02.sh脚本在数据库中引起故障。
# 在失败之前，用户表被更新了几次。
# 模拟了延长的时间，并且发生了几个日志切换。
./break_11_02.sh
# 3.尝试启动数据库实例。注意错误消息。这些是与练习11-1中相同的错误消息。
sqlplus / as sysdba
startup
# 4.记下您的数据文件号。
# 5.（可选）在另一个终端窗口中，检查DBWR跟踪文件，然后返回到$LABS 目录。
cd /u01/app/oracle/diag/rdbms/orcl/orcl/trace
ls -ltr *dbw*
cat orcl_dbw0_29806.trc
cd $LABS
# 6.使用RMAN LIST FAILURE命令查找更多信息。您可能只会看到列出的一个失败。
rman target "'/ as sysbackup'"
list failure;
# 7.使用RMAN ADVISE FAILURE命令确定自动恢复是否可用。您可能只会看到列出的一个失败。
ADVISE FAILURE;
# 8.仅缺少一个数据文件，但没有自动恢复可用。这表明顾问发现还原或恢复有问题。
# 尝试还原和恢复由list failure命令指定的数据文件。系统上的数据文件号可能会有所不同。
restore datafile 2;
recover datafile 2;
# 注意：如果收到两条RMAN-06025错误消息，则在以下步骤中也要关注最新的一条，即数字最高的一条。
# 在生产系统中，您将确定该文件是否存在另一个副本，可能在RMAN未知的OS备份中。如果可以找到并还原存档日志文件，则可以进行完全恢复。对于这种做法，假定存档日志文件已丢失。
# 注意：找到的归档日志序列号可能与示例中显示的序列号不同。记下你缺少归档日志序列号：
# 9.在这种情况下，不可能完全恢复。使用您的SQL * Plus会话来确定将丢失多少数据。在此示例中，当前重做日志文件的序列号为73，而缺少日志号70。因此，包含在日志文件70 至73中的所有数据将迷路了。startup nomount;
restore controlfile from autobackup;

archive log list
SELECT NAME, DBID, CURRENT_SCN, LOG_MODE, OPEN_MODE FROM V$DATABASE;
# 10.确定缺少的日志的开始SCN和开始时间（在本示例中为日志70）。
# 记录FIRST_CHANGE＃和FIRST_TIME列中的值。
# FIRST_TIME中的值可用于通知用户他们必须走多远才能恢复丢失的任何事务。注销SQL * Plus。
select sequence#, first_change#, first_time, status from v$archived_log
where sequence# = 70 and name is not null;
# 注意： SCN已经显示在RMAN错误消息中，但是第一次使用此存档日志时，以前没有显示它。
# 还要注意，V$ARCHIVED_LOG视图包含先前数据库实例的历史信息。
# 活动数据库化身的NAME列包含归档日志的路径和名称；历史化身为空值。
# 状态A用于归档日志，状态D表示已删除的日志。
# 11.建议始终先恢复控制文件以进行不完全恢复，以使RMAN知道数据结构中的潜在更改。
shutdown immediate;
# 使数据库进入 NOMOUNT状态。
startup nomount;
# 从自动备份还原控制文件
restore controlfile from autobackup;
# 挂载数据库。
alter database mount;
# 12.使用RESTORE DATABASE UNTIL SEQUENCE nn命令从丢失的归档日志文件之前进行的备份还原整个数据库。
RESTORE DATABASE UNTIL SEQUENCE 70;
# 13.通过最后一个可用的日志文件恢复数据库。
# 注意：如果有增量备份可用，将首先应用它们，然后再应用归档日志。需要应用的日志文件的数量可能与所示示例有所不同。
recover database until sequence 70;
# 14.使用RESETLOGS选项打开数据库。查询V $DATABASE以显示CURRENT_SCN 和 DBID 。
alter database open resetlogs;
SELECT NAME, DBID, CURRENT_SCN, LOG_MODE, OPEN_MODE FROM V$DATABASE;
# 15.使用Data Recovery Advisor LIST FAILURE命令验证是否已修复故障。
# 然后，您必须退出，以便可以在下一步中连接到恢复目录。
list failure;
exit;
# 16.因为break_11_02.sh脚本删除了一个存档日志以创建一个出于您的学习目的的问题，所以请交叉检查所有连接到恢复目录的存档日志。
rman target "'/ as sysbackup'" catalog rcatowner@rcat
CROSSCHECK ARCHIVELOG ALL;
# 17.删除过时的备份，然后退出RMAN客户端。
delete noprompt obsolete;
exit
# 18.（可选）登录到SQL * Plus。从BAR.BARCOPY表的一行中选择SALARY列。薪水的最后一位数字指示BARCOPY表已更新的次数。
# 此结果与步骤2中的结果之间的差异说明了不完全恢复后可能缺少多个更新。退出
select salary from bar.barcopy where rownum < 2;
# 19. 从$ LABS目录执行cleanup_11_02.sh脚本，以删除本练习中创建的新用户和表空间。输出在/tmp/cleanup.log文件中。
./cleanup_11_02.sh
# 20.备份数据库。尽管在某些情况下可以使用较早的备份，但是您具有数据库的新形式并且较早的备份已过时。执行RESETLOGS命令时，创建了数据库的新形式。使用RMAN客户端来创建新的完整备份：
BACKUP DATABASE PLUS ARCHIVELOG DELETE INPUT;
# 或在终端窗口中使用backup_orcl.sh脚本。
./backup_orcl.sh
```
