# 第14课：使用闪回技术的实践

> 2020-04-19 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [第14课：使用闪回技术的实践](#第14课：使用闪回技术的实践)   
   - [实践概述](#实践概述)   
   - [练习14-1：准备使用闪回技术](#练习14-1：准备使用闪回技术)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习14-2：恢复已删除的表](#练习14-2：恢复已删除的表)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习14-3：使用闪回表](#练习14-3：使用闪回表)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   

<!-- /MDTOC -->

## 实践概述

在这些实践中，您将配置数据库以使用闪回技术。然后，您将使用闪回技术来还原已删除的表并撤消事务的操作。

## 练习14-1：准备使用闪回技术

### 总览

在这种实践中，您将配置数据库以使用闪回事务和闪回表功能。

### 假设条件

您打开了两个终端窗口，并以oracle OS用户身份登录。$LABS是当前目录。为orcl实例设置了环境变量。

### 任务

```SQL
# 1.确定撤消数据将允许您闪回当前数据库中的事务的距离。的V $ UNDOSTAT视图含有至多统计4天后，在每个10分钟的间隔。第一行包含当前（部分）时间段统计信息。（您的价值可能有所不同。）
sqlplus / as sysdba
select tuned_undoretention from v$undostat where rownum =
1;
# 问题： TUNED_UNDORETENTION时间的值代表什么？
# 答：在任何给定时间将数据保留在数据库中的秒数。默认情况下，不保证撤消保留。如果系统需要更多空间，则Oracle数据库可以使用最近生成的撤消数据覆盖未过期的撤消
# 2. 在撤消表空间上设置UNDO_RETENTION参数和RETENTION GUARANTEE子句，以确保保留24小时。更改表空间的属性，以免表空间不足。
# a.将UNDO_RETENTION参数更改为14400秒（4小时）。
# 注意：将UNDO_RETENTION的值增加到几天以上可能会导致undo表空间的增长不合理。
ALTER SYSTEM SET UNDO_RETENTION = 14400 SCOPE=BOTH;
# b.查找撤消表空间的名称。
SHOW PARAMETER UNDO
# c.更改撤消表空间的RETENTION GUARANTEE值。
ALTER TABLESPACE UNDOTBS1 RETENTION GUARANTEE;
# d.查找与UNDOTBS1表空间关联的数据文件的名称。注意FILE_ID值 。
select file_name, file_id from dba_data_files where tablespace_name = 'UNDOTBS1';
# e.如果需要更多空间来保留未到期的撤消和活动撤消记录，则将撤消表空间数据文件配置为自动扩展。使用您自己的FILE_ID值。
ALTER DATABASE DATAFILE 4 AUTOEXTEND ON MAXSIZE UNLIMITED;
# 问题：如果保证撤消保留，并且没有更多空间可用于活动撤消记录（由于撤消表空间已满，达到最大大小或存储设备[磁盘]上没有剩余空间），会发生什么情况？
# 答：由于撤消表空间中的空间不足，事务失败。
# 3.查看RECYCLEBIN参数的值，然后退出SQL * Plus。
show parameter recyclebin
# 4.（可选）在EM Express中查看修改的设置。
# a.以SYS用户和SYSDBA特权登录，从orcl数据库主页导航：Configuration> Initialization Parameters。
# b.在当前选项卡式页面上，输入UNDO。您会看到与步骤2b中相同的值。
# c.如果要更改初始化参数，可以选择该参数，单击“ 设置”，然后输入所需的值。此时，您不想更改任何值，因此单击取消。
# d.您可以在搜索字段中输入recyclebin以查看与步骤3中所示相同的值。
# e.要查看UNDOTBS1表空间的当前值，请导航至：存储>表空间。
# 注意：您会看到表空间是无限的，如步骤2e中所配置。您的数值可能会有所不同。
# f.单击ORCL（12.1.0.2.0）返回数据库主页。G。注销Enterprise Manager Database Express。
# 5.（可选）在Enterprise Manager Cloud Control中查看修改后的设置。
# a.以SYSMAN用户身份登录，从orcl数据库主页导航：“ 管理”>“初始化参数”>“当前”（标签页）。输入UNDO在名称字段，然后单击转到。您将看到与步骤2b和4b中相同的值。
# b.输入RECYCLEBIN在名称字段，然后单击转到。您将看到与步骤3和4d中所示相同的值。
# c.导航到管理>存储>自动撤消管理，并浏览常规和系统活动页面，包括其图形。
# d.在“系统活动”页面上找到显示最大可能查询或闪回持续时间的图表。
# e.注销企业管理器云控制。
```



## 练习14-2：恢复已删除的表

### 总览

在这种情况下，您将恢复已删除的表。

### 假设条件

该RECYCLEBIN参数设置为ON（已在前面的实践所证实）。

您打开了两个终端窗口，并以oracle OS用户身份登录。$LABS是当前目录。为orcl实例设置了环境变量。

### 任务

```sql
# 1.执行setup_14_02.sh脚本创建练习环境。输出在/tmp/setup.log文件中。
/setup_14_02.sh
# 2.执行break_14_02.sh脚本以模拟开发人员完成的工作。输出在/tmp/break.log文件中。
./break_14_02.sh
# 3.开发人员（具有BAR Oracle用户帐户）来找您，并要求您恢复删除的表。
# 他解释说该表有多次迭代，但他所需的迭代在BAR模式中名为BAR102。
# 它应该有12列，其中之一名为LOCATION_ID。BAR模式中当前有一个BAR102表。
# 将请求的表恢复到BAR102A。
# a.尝试使用SHOW RECYCLEBIN命令查看回收站的内容。
sqlplus / as sysdba
show recyclebin
# 注：该SHOW RECYCLEBIN仅命令显示属于这些对象的当前用户。因为您是DBA，并且不知道BAR用户的密码，所以SHOW RECYCLEBIN命令不会显示您有兴趣恢复的已删除表。
# b.检查dba_recyclebin视图中的对象。（可选）将SQL * Plus页面大小更改为99行。
set pages 99
select original_name, object_name, droptime
from dba_recyclebin where owner ='BAR';
# 注意：在上方，您可以看到同一对象在不同的​​时间点两次掉落。使用时间戳，您可以确定要还原的表的版本。
# c.确定哪个对象包含感兴趣的列。您的对象名称将不同。使用您自己的值。
select location_id
from BAR."BIN$CpAWzw2QDs3gU48juYsdeA==$0" where rownum = 1;
# 注意：回收站中的对象名称必须用双引号引起来，因为它可能包含特殊字符。
select location_id
from BAR."BIN$CpAWzw2LDs3gU48juYsdeA==$0" where rownum = 1;
# d.还原具有正确的列的对象。
FLASHBACK TABLE BAR."BIN$CpAWzw2QDs3gU48juYsdeA==$0" TO BEFORE DROP RENAME TO BAR102A;
# 4.通过选择第一行，确认BAR.BAR102A表已还原。（不管出现哪一行，只要有一行就可以了。）然后退出。
select * from BAR.BAR102A where rownum = 1;
exit;
# 5.使用cleanup_14_02.sh脚本清理此练习环境。
# 注意：此脚本使用PURGE DBA_RECYCLEBIN命令从回收站中删除所有对象。输出在/tmp/cleanup.log文件中。
./cleanup_14_02.sh
```

## 练习14-3：使用闪回表

### 总览

在这种实践中，您将使用闪回表来逆转恶意交易。

### 假设条件

练习14-1已完成。

您打开了两个终端窗口，并以oracle OS用户身份登录。$ LABS是当前目录。为orcl实例设置了环境变量。

### 任务

```sql
# 1.执行setup_14_03.sh脚本以创建本练习中使用的用户和表。这些表具有外键关系。输出在/tmp/setup.log文件中。
./setup_14_03.sh
# 2.确定当前时间到最近的秒。将此记录为T1：
# 注意：SYSDATE的格式不是默认格式。格式已由练习中标题为“设置RMAN的日期和时间格式” 的NLS_DATE_FORMAT环境变量更改。
sqlplus / as sysdba
select sysdate from dual;
# 3.在另一个终端窗口中，执行break_14_03.sh脚本。这模拟了一个恶意交易，该交易对BARCOPY和BARDEPT表中的数据进行加扰。BARCOPY和BARDEPT之间存在外键约束。输出在/tmp/break.log中
./break_14_03.sh
# 4.人力资源代表报告说，一名雇员错误地更改了部门名称，并扰乱了将哪些雇员分配给哪个部门。
# 该表在时间T1正确，并且自那时以来未进行任何授权更改。
# 涉及的表是BAR.BARCOPY和BAR.DEPT。
# 将表恢复到T1的状态。您必须使用自己的T1值。
# 由于存在外键关系，因此必须还原两个表。（继续SQL * Plus会话。）
ALTER TABLE BAR.BARDEPT ENABLE ROW MOVEMENT;
ALTER TABLE BAR.BARCOPY ENABLE ROW MOVEMENT;
FLASHBACK TABLE BAR.BARDEPT TO TIMESTAMP TO_TIMESTAMP('2014-12-19:10:48:47','YYYY-MM-DD:HH24:MI:SS');
FLASHBACK TABLE BAR.BARCOPY TO TIMESTAMP
TO_TIMESTAMP('2014-12-19:10:48:47','YYYY-MM-DD:HH24:MI:SS');
# 5.检查是否已正确还原表。尽管行顺序可能不同，但以下查询的结果应与您的结果匹配。然后退出SQL * Plus。
@check_14_03.sql
# 6.通过运行cleanup_14_03.sh脚本来清理练习环境。
./cleanup_14_03.sh
# 7.为了准备下一个练习，请备份您的orcl数据库，删除过时的备份，并确保未列出任何故障。然后从RMAN退出。
rman target "'/ as sysbackup'"
BACKUP DATABASE PLUS ARCHIVELOG DELETE INPUT;
delete noprompt obsolete;
list failure;
exit
```
