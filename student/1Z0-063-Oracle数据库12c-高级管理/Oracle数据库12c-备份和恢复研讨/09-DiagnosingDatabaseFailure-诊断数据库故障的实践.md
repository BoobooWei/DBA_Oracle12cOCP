# 第9课：诊断数据库故障的实践

> 2020-04-14 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [第9课：诊断数据库故障的实践](#第9课：诊断数据库故障的实践)   
   - [实践概述](#实践概述)   
   - [练习9-1：诊断和修复数据库故障](#练习9-1：诊断和修复数据库故障)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习9-2：执行和分析实例恢复](#练习9-2：执行和分析实例恢复)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习9-3：修复程序块损坏](#练习9-3：修复程序块损坏)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   

<!-- /MDTOC -->

## 实践概述

在这些实践中，您将使用`Data Recovery Advisor`诊断数据库故障。

## 练习9-1：诊断和修复数据库故障

### 总览

在这种实践中，您将通过RMAN客户端界面使用`Data Recovery Advisor`来诊断和修复数据文件的丢失。

### 假设条件

您将打开一个终端窗口，其中`$LABS`作为当前目录。为orcl实例设置了环境变量。

### 任务

```sql
# 1.执行setup_09_01.sh脚本。
# 该脚本创建BAR91TBS在文件系统表空间中，BAR91用户的BARCOPY表，填充表。
# 该脚本将继续并备份表空间并更新表。脚本的输出可以在/tmp/setup.log文件中查看。
./setup_09_01.sh
# 2.执行break_09_01.sh脚本。
# 该脚本删除数据文件并导致数据库失败。脚本的输出可以在/tmp/break.log文件中查看。
./break_09_01.sh
# 3.以oracle用户身份在另一个终端窗口中继续，环境变量指向orcl。
# 尝试启动实例并打开数据库。观察错误消息。退出SQL * Plus。
sqlplus / as sysdba
startup
# 4.使用Data Recovery Advisor列出数据库故障。
rman target "'/ as sysbackup'"
LIST FAILURE;
# 5.使用Data Recovery Advisor获得有关如何修复故障的建议。
ADVISE FAILURE;
# 6.使用Data Recovery Advisor修复故障。在执行之前，请检查为此修复生成的脚本。
# 当提示您执行脚本并打开数据库时，输入Y或YES。
repair failure;
# 7.按照此练习通过执行cleanup_09_01.sh清理环境脚本。
# 输出在/tmp/cleanup.log文件中。
./cleanup_09_01.sh
```

## 练习9-2：执行和分析实例恢复

### 总览

在这种实践中，您将强制实例失败，并检查实例在实例恢复期间采取的步骤。

### 假设条件

您将打开一个终端窗口，其中`$LABS`作为当前目录。为orcl实例设置了环境变量。

### 任务

```SQL
# 1.中止实例并重新启动它。您可以通过多种方式中止实例：
sqlplus / as sysdba
shutdown abort

# 2.检查警报日志中的orcl实例。
# 从文件底部的最新条目开始，找到实例的重新启动。
# 实例执行哪些步骤来恢复实例并确保数据库一致性？
# 使用ADRCI show alert命令，查看orcl实例的警报日志。
# 将编辑器设置为gedit可以更轻松地查看警报日志。默认编辑器是vi。
# 该gedit中选择一个警报日志后，将打开的窗口。根据先前的活动，
# 您可能具有不同的警报日志。选择您的orcl警报日志。
adrci
set editor gedit
show alert
```

## 练习9-3：修复程序块损坏

### 总览

在这种实践中，您将使用Data Recovery Advisor设置，发现和修复数据文件中的损坏块。

### 假设条件

您将打开一个终端窗口，其中`$LABS`作为当前目录。为orcl实例设置了环境变量。

### 任务

```sql
# 1.通过执行setup_09_04.sh设置此实践。
# 该脚本创建BC用户，BCTBS表空间和BCCOPY表。将填充该表，进行备份，并更新该表以为此作准备。
# 可以使用cat /tmp/setup.log Linux命令查看输出。
$ ./setup_09_04.sh
# 2.通过执行break_09_04.sql破坏上一步中创建的数据文件脚本。
# 注意：预计会出现损坏的块错误。该脚本对BCCOPY 执行查询表以强制发现损坏的块。
sqlplus / nolog @break_09_04.sql
# 3.使用RMAN作为SYSBACKUP，连接到orcl实例，并使用LIST FAILURE 命令。
rman target "'/ as sysbackup'"
LIST FAILURE;
# 4.使用RMAN ADVISE FAILURE命令并查看建议的维修策略。
ADVISE FAILURE;
# 5.使用RMAN REPAIR FAILURE命令恢复损坏的块。
REPAIR FAILURE;
# 6.（可选）确认没有其他故障。
list failure;
exit
```
