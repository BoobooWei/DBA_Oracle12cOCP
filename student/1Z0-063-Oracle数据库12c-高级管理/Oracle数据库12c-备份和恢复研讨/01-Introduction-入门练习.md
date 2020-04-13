# 第1课：入门练习

> 2020-03-02 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [第1课：入门练习](#第1课：入门练习)   
   - [实践概述](#实践概述)   
   - [练习1-1：探索课程环境](#练习1-1：探索课程环境)   
      - [Overview](#overview)   
      - [Tasks](#tasks)   

<!-- /MDTOC -->

## 实践概述

在本练习中，您将探索在课程练习中将使用的环境。

## 练习1-1：探索课程环境

### Overview

在此可选练习中，您将探索练习环境的某些元素。在以下实践中，将引入更多元素，并且您将更改此配置。

学生有一台装有Linux操作系统的机器。预装的是：

• 具有1个活动数据库实例的Oracle Database 12 *c*（12.2）：

• booboo（主数据库）

学生有第二台装有Linux操作系统的机器。预装的是：

• 具有1个活动数据库实例的Oracle Database 12 *c*（12.2）

• emrep（用于Oracle Enterprise Manager Cloud Control）

•Oracle企业管理器云控制

分级软件包括：

•Oracle安全备份 Oracle Secure Backup

### Tasks

1. 查看Oracle数据库环境变量
2. 查看Oracle后台进程
3. 查看Oracle监听服务情况

```bash
export|grep ORACLE
ps -ef|grep pmon
lsnrctl status
```

登陆到数据库`sqlplus / as sysdba`

```sql
--查看数据库实例名，归档模式等信息
SELECT name, log_mode, db_unique_name,open_mode FROM v$database;
select status from v$instance;
--查看登陆用户
select user from dual;
--通过服务名连接到booboopdb1
conn sys/oracle@booboopdb1 as sysdba
--通过服务名连接到db01
conn sys/oracle@db01 as sysdba
```
