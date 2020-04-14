# 第10课：还原和恢复概念的实践

> 2020-04-14 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [第10课：还原和恢复概念的实践](#第10课：还原和恢复概念的实践)   
   - [实践概述](#实践概述)   
   - [练习10-1：案例研究：确定恢复程序](#练习10-1：案例研究：确定恢复程序)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [Case 1](#case-1)   
      - [Case 2](#case-2)   
      - [Case 3](#case-3)   

<!-- /MDTOC -->

## 实践概述

在这种做法中，您将考虑故障情况和备份设置，以确定恢复和恢复的策略。

## 练习10-1：案例研究：确定恢复程序

### 总览

在这种做法中，您将考虑故障情况和备份设置，以确定恢复和恢复的策略。

### 假设条件

您熟悉Oracle备份和恢复。

### Case 1

* 备份是在夜间关机期间采用增量备份策略进行的。
* 每晚将1级备份应用于以前的0级备份。

`ARCHIVE LOG LIST`命令显示以下内容

```SQL
SQL> archive log list
Database log mode               No Archive Mode
Automatic archival              Disabled
Archive destination             USE_DB_RECOVERY_FILE_DEST
Oldest online log sequence      61
Next log sequence to archive    63
Current log sequence            63
```

包含SYSAUX表空间数据文件的磁盘已崩溃。

**a.是否可以完全恢复？步骤是什么？**

答：如果从最新备份到当前日志文件的所有重做日志文件均可用，则可以完全恢复。

步骤：
1. 恢复SYSAUX数据文件
2. 恢复数据文件
3. 打开数据库

**b.如果无法完全恢复，则应采取什么步骤来尽可能多地恢复？哪些数据（交易）将丢失？**

答：如果缺少任何重做日志文件，则在进行备份的时间与当前时间（数据库关闭时）之间不可能进行完全恢复。

在这种情况下，步骤：
1. 还原所有数据库文件
2. 应用最新的增量备份（RMAN RECOVER命令）
3. 使用 RESETLOGS 选项打开数据库。

在备份时间和当前时间之间的任何事务都将丢失。

### Case 2

* 数据库备份每晚进行联机备份，并采用增量备份策略。
* 每晚将级别1备份应用于之前的级别0备份

`ARCHIVE LOG LIST`命令显示以下内容：

```SQL
SQL> archive log list
Database log mode               Archive Mode
Automatic archival              Enabled
Archive destination             USE_DB_RECOVERY_FILE_DEST
Oldest online log sequence      61
Next log sequence to archive    63
Current log sequence            63
```

**属于关键数据的属于应用程序表空间的一部分数据文件已丢失。描述执行完整恢复的步骤。**

答：使用数据恢复顾问`Data Recovery Advisor`来确定丢失了哪些文件，还原数据文件，还原数据文件并打开数据库。

```SQL
rman target "'/ as sysbackup'"
LIST FAILURE;
ADVISE FAILURE;
REPAIR FAILURE;
```

### Case 3

* 昨晚 `8:00 pm` 在数据库上执行批处理作业，导致了数据错误。
* 通过不完全恢复将数据库恢复到昨天的 `6:00 pm`。
* 不完全恢复之后，将重新打开数据库。
* 恢复后执行的检查发现在晚上`7:15`之前执行的某些关键事务丢失了。

**a.恢复这些事务的有效选择是什么？**

答：不完全恢复步骤：

1. 恢复晚上7:15的控制文件
2. 恢复晚上6:00的数据库备份
3. 使用归档日志到晚上7:15

**b.描述恢复这些事务的要求**

答：
1. 备份必须可用。
2. 归档日志文件和redo必须可用。
3. 使用RMAN时，设置incarnation编号以获取正确的重做日志和归档日志集。
