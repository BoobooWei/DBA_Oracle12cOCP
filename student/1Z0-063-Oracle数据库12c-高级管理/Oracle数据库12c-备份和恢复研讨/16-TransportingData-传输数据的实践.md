# 第16课：传输数据的实践

> 2020.04.19 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [第16课：传输数据的实践](#第16课：传输数据的实践)   
   - [实践概述](#实践概述)   
   - [练习16-1：传输表空间](#练习16-1：传输表空间)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   

<!-- /MDTOC -->

## 实践概述

在这些实践中，您将传输表空间。您还将执行将表空间从一个平台迁移到另一个平台所需的步骤（尽管在培训环境中，每个学生都可以访问一个主机，而不是几个主机）。


## 练习16-1：传输表空间

### 总览

在这种实践中，您将转移一个表空间，其中包含在不同平台之间转移表空间的所有步骤（尽管在培训环境中，您仅在一个平台上使用一个主机）。

### 假设条件

两个终端窗口打开。$LABS是当前目录。ORCL是您的源数据库，RCAT是您的目标数据库。该实践表明何时需要切换环境变量。

### 任务

```sql
# 1.通过执行$ LABS目录中的setup_16_01.sh脚本进行设置。
# 在此脚本中：
## 创建了新的表空间和用户。
## 用户创建一个表并填充它。
## 输出在/tmp/setup.log文件中。
. oraenv
./setup_16_01.sh

# 2.在您的ORCL源数据库上启动SQL * Plus会话，并验证跨平台传输表空间的先决条件。
# a.以SYS用户身份登录，并验证源数据库处于读写模式。
sqlplus / as sysdba
SELECT NAME, LOG_MODE, OPEN_MODE, CURRENT_SCN FROM
V$DATABASE;
# b.为了执行跨平台表空间传输，您必须知道要将数据传输到的目标平台的确切名称。
col platform_name format a30
SELECT PLATFORM_ID, PLATFORM_NAME, ENDIAN_FORMAT FROM V$TRANSPORTABLE_PLATFORM
WHERE UPPER(PLATFORM_NAME) LIKE '%LINUX%';
# c.将BARTBS表空间设置为只读。这是导出表空间元数据所必需的。然后退出。
ALTER TABLESPACE bartbs READ ONLY;
# 3.启动一个RMAN会话，并连接到您的ORCL源数据库作为目标实例。
. oraenv
rman target "'/ as sysbackup'"
# 4.通过将BACKUP命令与TO PLATFORM子句一起使用来备份源表空间。
# 使用DATAPUMP子句指示必须为表空间元数据创建表空间的导出转储文件。
BACKUP TO PLATFORM 'Linux x86 64-bit' FORMAT '/u01/backup/test.bck' DATAPUMP FORMAT '/u01/backup/test.dmp' TABLESPACE bartbs;
# 5.允许对BARTBS表空间进行读写操作。然后退出RMAN。
alter tablespace bartbs READ WRITE;
# 注意：通常，从源数据库断开连接后，可以使用操作系统实用程序将备份集和数据泵导出转储文件移动到目标主机。在此培训示例中，您不需要这样做，因为您只有一个主机。
# 6.以SYS 用户身份，连接到RCAT 目标主机作为目标。（您将以SYS而不是SYSBACKUP身份登录，因为您将在目标数据库中创建BAR用户）。
. oraenv
rman target /
# 7.确保目标数据库以读写模式打开。
SELECT NAME, LOG_MODE, OPEN_MODE, CURRENT_SCN FROM V$DATABASE;
# 8.创建BAR用户并授予CREATE SESSION特权。
CREATE USER BAR IDENTIFIED BY oracle_4U;
GRANT CREATE SESSION TO BAR;
# 9.将RESTORE命令与FOREIGN TABLESPACE子句一起使用。该FORMAT子句指定的文件目的地。使用DUMP FILE FROM BACKUPSET子句从转储文件还原元数据，这是将表空间插入目标数据库所必需的。
RESTORE FOREIGN TABLESPACE bartbs FORMAT '/u01/backup/rcat/bartbs.dbf' FROM BACKUPSET '/u01/backup/test.bck' DUMP FILE FROM BACKUPSET '/u01/backup/test.dmp';
# 10.确认表空间存在于目标数据库中。然后退出
select tablespace_name, status from dba_tablespaces;
exit
# 11.通过执行cleanup_16_01.sh脚本来清理练习环境。该脚本将删除原始和传输的表空间以及备份和转储文件。输出在/tmp/cleanup.log文件中。
./cleanup_16_01.sh
```
