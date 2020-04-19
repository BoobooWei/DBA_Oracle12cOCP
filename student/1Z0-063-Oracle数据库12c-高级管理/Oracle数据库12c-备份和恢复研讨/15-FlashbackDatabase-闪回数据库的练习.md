# 第15课：闪回数据库的练习

> 2020-04-19 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [第15课：闪回数据库的练习](#第15课：闪回数据库的练习)   
   - [实践概述](#实践概述)   
   - [练习15-1：启用闪回记录](#练习15-1：启用闪回记录)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习15-2：执行闪回数据库](#练习15-2：执行闪回数据库)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   

<!-- /MDTOC -->

## 实践概述

在这些实践中，您将启用闪回日志记录并执行闪回数据库。

## 练习15-1：启用闪回记录

### 总览

在这种情况下，您将启用闪回日志记录。

### 假设条件

您打开了两个终端窗口，并以oracle OS用户身份登录。$LABS是当前目录。环境变量指向orcl实例。

### 任务

```SQL
# 1.登录到SQL * Plus，然后确定是否启用了闪回日志记录。
sqlplus / as sysdba
SELECT flashback_on FROM v$database;
# 2.创建一个有保证的还原点，并检查您当前的FLASHBACK_ON状态。
CREATE RESTORE POINT rp1 GUARANTEE FLASHBACK DATABASE;
SELECT FLASHBACK_ON FROM V$DATABASE;
# 3.使用您选择的工具启用闪回日志记录。
ALTER DATABASE FLASHBACK ON;
# 4.确认已启用闪回日志记录。
SELECT FLASHBACK_ON FROM V$DATABASE;

# 问题：您可以使用其他哪个工具来启用数据库的闪回日志记录？
# 可能的答案：企业管理器云控制
# 问题：企业管理器云控制的导航步骤是什么？
# 可能的答案：可用性>备份和恢复>恢复设置。
```

## 练习15-2：执行闪回数据库

### 总览

在这种情况下，对数据库进行一些不正确的更新后，您将闪回数据库。这种做法是出于学习目的。如果在生产环境中具有与此类似的场景，则可能会选择其他解决方案以将闪回限制为受影响的对象，而不是选择整个数据库的闪回。

### 假设条件

您已完成练习15-1。

您打开了两个终端窗口，并以oracle OS用户身份登录。$LABS是当前目录。为orcl实例设置了环境变量。

### 任务

```sql
# 1.您可以通过多种方式执行闪回数据库操作。您可以使用保证的还原点，SCN，时间值，线程等。本示例使用SCN，但您也可以使用在先前的实践中创建的RP1还原点。
SELECT current_scn FROM v$database;
确定您当前的SCN。是的 。您将需要在以后的练习步骤中使用它。
# 2.查看人力资源数据。在此练习期间，您将使用此信息进行比较
# a.确定HR.EMPLOYEES表中SALARY列的总和。
SELECT sum(salary) FROM hr.employees;
SELECT count(*) FROM hr.employees where department_id=90;
# b.确定部门90中的员工总数。
# 3.执行lab_15_02_03.sql脚本以更新HR模式中的表。在这种情况下，它会通过闪回数据库来创建问题，从中“恢复”。
@lab_15_02_03.sql
# 4.提交数据并确定当前的SCN。
COMMIT;
SELECT current_scn FROM v$database;
# 5. 再次查询HR模式中的数据，并将结果与​​您在步骤2中的查询中收到的值进行比较。
# a.确定HR.EMPLOYEES表中SALARY列的总和。
SELECT sum(salary) FROM hr.employees;
# b.确定部门90中的员工总数。
SELECT count(*) FROM hr.employees where department_id=90;
# 6.您需要还原数据库，以便数据与开始此练习时的数据相同。出于培训目的，请使用闪回数据库进行此操作。
# 问题：对于闪回数据库操作，数据库必须处于哪种状态？
# 可能的答案：必须安装数据库。
# a.关闭数据库实例，然后以MOUNT模式启动它。
shutdown immediate
startup mount
exit
# b.登录到RMAN并使用FLASHBACK DATABASE命令将数据库闪回到步骤1中记下的SCN。退出RMAN。
rman target "'/ as sysbackup'" nocatalog
flashback database to scn=4368114; (Enter your SCN number)
exit
# c.通过查询HR.EMPLOYEES验证数据库已正确地闪回。
sqlplus / as sysdba
alter database open read only;
SELECT sum(salary) FROM hr.employees;
SELECT count(*) FROM hr.employees where department_id=90;
# d.验证闪回所需状态后，打开数据库进行读/写操作。
shutdown immediate
startup mount
alter database open resetlogs;
# 7.禁用闪回日志记录。
ALTER DATABASE FLASHBACK OFF;
# 8.删除RP1保证的还原点。然后退出SQL * Plus。
DROP RESTORE POINT rp1;
```
