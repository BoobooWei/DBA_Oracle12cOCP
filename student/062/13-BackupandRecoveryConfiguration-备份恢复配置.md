# 实践13:备份恢复配置

> **Practices for Lesson 13: Backup and Recovery: Configuration**
>
> 2020.02.04 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [实践13:备份恢复配置](#实践13备份恢复配置)   
   - [实践13:概览](#实践13概览)   
   - [实践13-1:配置数据库进行恢复](#实践13-1配置数据库进行恢复)   
      - [Overview](#overview)   
      - [Task](#task)   
      - [Practice](#practice)   
      - [KnowledgePoint](#knowledgepoint)   
         - [管理控制文件](#管理控制文件)   
         - [管理重做日志](#管理重做日志)   
         - [管理存档的重做日志文件](#管理存档的重做日志文件)   

<!-- /MDTOC -->

## 实践13:概览

Practices for Lesson 13: Overview

Configure your database to reduce the chances of failure or data loss. To do so, perform the following tasks:

•  Ensure redundancy of control files.

•  Review the fast recovery area configuration.

•  Ensure that there are at least two redo log members in each group.

•  Place your database in ARCHIVELOG mode.

•  Configure redundant archive log destinations.



配置数据库以减少失败或数据丢失的机会。为此，请执行以下任务:

* 确保控制文件的冗余。
* 检查快速恢复区域的配置。
* 确保每个组中至少有两个重做日志成员。
* 将数据库置于ARCHIVELOG模式。
* 配置冗余的存档日志目的地。

## 实践13-1:配置数据库进行恢复

Practice 13-1: Configuring Your Database for Recovery

### Overview

In this practice, you verify that your database is configured properly to support recovery operations in the event of a failure.

### Task

1. Verify that the control files are multiplexed.

2. Review the fast recovery area configuration and change the size to 8 GB.

3. Check how many members each redo log group has. Ensure that there are at least two redo log members in each group. One set of members should be stored in the fast recovery area.

4. You notice that for each redo log group, the “Archived” column has no value. This means that your database is not retaining copies of redo logs to use for database recovery, and in the event of a failure, you will lose all data since your last backup. Place your database in ARCHIVELOG mode, so that redo logs are archived.

   You do not need to specify a naming convention or a destination for the archived redo log files because you are using a fast recovery area.

   **Note:** If you add archive log destinations, you must create the directory if it does not already exist.

   Use SQL*Plus to set the database in ARCHIVELOG mode.

5. Configure redundant archive log destinations.

### Practice

1. 验证控制文件是多路复用的。

   ```sql
   SELECT NAME FROM V$CONTROLFILE;
   ```

   a. 登陆 Enterprise Manager Database Express

   b. 选择 `存储引擎 > 控制文件`

   c. 可以看到当前有两个控制文件

   ```sql
   SQL> SELECT NAME FROM V$CONTROLFILE;

   NAME
   --------------------------------------------------------------------------------
   /u01/app/oracle/oradata/booboo/control01.ctl
   /u01/app/oracle/oradata/booboo/control02.ctl
   ```



2. 检查快速恢复区配置并将大小更改为8gb。

   ```sql
   mkdir  /u01/app/oracle/fast_recovery_area

   sqlplus / as sysdba
   show parameter db_recovery_file_
   alter system set "db_recovery_file_dest_size"='8G' scope=both sid='*';
   alter system set "db_recovery_file_dest"='/u01/app/oracle/fast_recovery_area' scope=both sid='*';
   show parameter db_recovery_file_
   ```



   a. Enterprise Manager Database Express 选择 `配置 > 初始化参数`

   b. 选择 `归档和恢复` ，找到以`db_recovery_file`开头的参数

   c. 检查快速恢复区是否开启。未开启，因为值为null和0

   d. 修改这两个参数，注意这两个参数都是动态参数，无需重启服务。

   ```sql
   SQL> show parameter db_recovery_file_

   NAME				     TYPE	 VALUE
   ------------------------------------ ----------- ------------------------------
   db_recovery_file_dest		     string
   db_recovery_file_dest_size	     big integer 0

   SQL> alter system set "db_recovery_file_dest_size"='8G' scope=both sid='*';
   System altered.
   SQL> alter system set "db_recovery_file_dest"='/u01/app/oracle/fast_recovery_area' scope=both sid='*';
   SQL> show parameter db_recovery_file_

   NAME				     TYPE	 VALUE
   ------------------------------------ ----------- ------------------------------
   db_recovery_file_dest		     string	 /u01/app/oracle/fast_recovery_
   						 area
   db_recovery_file_dest_size	     big integer 8G
   ```



3. 检查每个重做日志组有多少成员。确保每个组中至少有两个重做日志成员。一组成员应该存储在快速恢复区域。

   ```sql
   select group#,member from v$logfile;
   ALTER DATABASE
     ADD LOGFILE MEMBER
       '/u01/app/oracle/oradata/booboo/redo01-1.log'
     TO GROUP 1;
   alter system switch logfile;
   ```

   a. 选择 `存储 > 重做日志组`

   b. 观察每个组有多少个成员，`成员计数`列显示为`1`

   c. 选择其中一个组点击 `添加成员`

4. 您注意到，对于每个重做日志组，`已归档`列没有值。这意味着您的数据库没有保留用于数据库恢复的重做日志副本，并且在发生故障时，您将丢失自上次备份以来的所有数据。将数据库置于ARCHIVELOG模式，以便归档重做日志。

   您不需要为存档的重做日志文件指定命名约定或目标，因为您使用的是快速恢复区域。

   注:如果您添加存档日志目的地，您必须创建目录，如果它还不存在。

   使用`SQLPlus`将数据库设置为ARCHIVELOG模式。

   ```sql
   sqlplus / as sysdba
   archive log list
   shutdown immediate
   startup mount
   alter database archivelog;
   alter database open;
   archive log list
   ```

   执行结果

   ```sql
   SQL> archive log list
   Database log mode	       No Archive Mode
   Automatic archival	       Disabled
   Archive destination	       USE_DB_RECOVERY_FILE_DEST
   Oldest online log sequence     30
   Current log sequence	       32
   SQL> shutdown immediate
   Database closed.
   Database dismounted.
   ORACLE instance shut down.
   SQL> startup mount
   ORACLE instance started.

   Total System Global Area  838860800 bytes
   Fixed Size		    8798312 bytes
   Variable Size		  490737560 bytes
   Database Buffers	  331350016 bytes
   Redo Buffers		    7974912 bytes
   Database mounted.
   SQL> alter database archivelog;

   Database altered.

   SQL> alter database open;

   Database altered.

   SQL> archive log list;
   Database log mode	       Archive Mode
   Automatic archival	       Enabled
   Archive destination	       USE_DB_RECOVERY_FILE_DEST
   Oldest online log sequence     30
   Next log sequence to archive   32
   Current log sequence	       32
   ```

   现在，您的数据库处于ARCHIVELOG模式，它将继续存档每个在线重做日志文件的副本，然后再将其用于其他重做数据。
   注意：记住这将消耗磁盘上的空间，并且必须定期将旧的归档日志备份到其他存储。

5. 配置冗余归档日志目的地。

   ```bash
   mkdir /u01/app/oracle/oradata/booboo/archive_dir2
   sqlplus / as sysdba
   ALTER SYSTEM SET log_archive_dest_1='LOCATION=/u01/app/oracle/fast_recovery_area/EMCDB/archivelog' SCOPE=both;
   ALTER SYSTEM SET log_archive_dest_2='LOCATION=/u01/app/oracle/oradata/booboo/archive_dir2' SCOPE=both;
   ALTER SYSTEM SWITCH LOGFILE;
   ALTER SYSTEM SWITCH LOGFILE;
   ALTER SYSTEM SWITCH LOGFILE;
   ALTER SYSTEM SWITCH LOGFILE;
   SELECT name FROM v$archived_log ORDER BY stamp;
   ```



   a. 创建一个新目录`/u01/app/oracle/oradata/orcl/archive_dir2`

   b. 设置 `LOG_ARCHIVE_DEST_1`参数指向 FRA；设置`LOG_ARCHIVE_DEST_2`指向新目录

   c. 手动切换日志，查看视图`V$ARCHIVED_LOG`

   执行结果

   ```sql
   [oracle@ocm ~]$ mkdir /u01/app/oracle/oradata/booboo/archive_dir2
   [oracle@ocm ~]$ sqlplus / as sysdba
   SQL> show parameter log_archive_dest_1

   NAME				     TYPE	 VALUE
   ------------------------------------ ----------- ------------------------------
   log_archive_dest_1		     string
   log_archive_dest_10		     string
   log_archive_dest_11		     string
   log_archive_dest_12		     string
   log_archive_dest_13		     string
   log_archive_dest_14		     string
   log_archive_dest_15		     string
   log_archive_dest_16		     string
   log_archive_dest_17		     string
   log_archive_dest_18		     string
   log_archive_dest_19		     string
   SQL> ALTER SYSTEM SET log_archive_dest_1='LOCATION=/u01/app/oracle/fast_recovery_area/EMCDB/archivelog' SCOPE=both;

   System altered.
   SQL> ALTER SYSTEM SET log_archive_dest_2='LOCATION=/u01/app/oracle/oradata/booboo/archive_dir2' SCOPE=both;

   System altered.

   SQL> ALTER SYSTEM SWITCH LOGFILE;

   System altered.

   SQL> /

   System altered.

   SQL> /
   /

   System altered.

   SQL>
   System altered.

   SQL> /

   System altered.

   SQL> SELECT name FROM v$archived_log ORDER BY stamp;

   NAME
   --------------------------------------------------------------------------------
   /u01/app/oracle/fast_recovery_area/EMCDB/archivelog/1_32_1031256251.dbf
   /u01/app/oracle/oradata/booboo/archive_dir2/1_32_1031256251.dbf
   /u01/app/oracle/fast_recovery_area/EMCDB/archivelog/1_33_1031256251.dbf
   /u01/app/oracle/oradata/booboo/archive_dir2/1_33_1031256251.dbf
   /u01/app/oracle/fast_recovery_area/EMCDB/archivelog/1_34_1031256251.dbf
   /u01/app/oracle/oradata/booboo/archive_dir2/1_35_1031256251.dbf
   /u01/app/oracle/oradata/booboo/archive_dir2/1_34_1031256251.dbf
   /u01/app/oracle/fast_recovery_area/EMCDB/archivelog/1_35_1031256251.dbf
   /u01/app/oracle/fast_recovery_area/EMCDB/archivelog/1_36_1031256251.dbf
   /u01/app/oracle/oradata/booboo/archive_dir2/1_36_1031256251.dbf
   ```



### KnowledgePoint

#### 管理控制文件

[管理控制文件](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-control-files.html#GUID-5F6F9382-6921-4992-9789-E63B18E99C5F)

您可以创建，备份和删除控制文件。

- [什么是控制文件？](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-control-files.html#GUID-98A05D29-DD80-4D87-9615-76CBCF8FE694)
  每个Oracle数据库都有一个**控制文件**，该**文件**是一个小的二进制文件，记录了数据库的物理结构。
- [控制文件](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-control-files.html#GUID-737B0C7F-CED8-4B00-8428-DA11B35F9E9B)
  准则您可以遵循准则来管理数据库的控制文件。
- [创建控制文件](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-control-files.html#GUID-9CBB8320-E2F6-4BDB-9B1C-A7B801F97F73)
  您可以创建，复制，重命名和重定位控制文件。
- [创建控制文件](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-control-files.html#GUID-4D4D22BE-1619-46BD-B22C-7409B35923AA)
  后的[故障排除在](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-control-files.html#GUID-4D4D22BE-1619-46BD-B22C-7409B35923AA)发出该`CREATE CONTROLFILE`语句之后，您可能会遇到一些错误。
- [备份控制文件](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-control-files.html#GUID-204AF8CF-6C51-4D0F-ADE2-BA804352EA93)
  使用该`ALTER DATABASE BACKUP CONTROLFILE`语句备份控制文件。
- [使用当前副本](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-control-files.html#GUID-D6662264-5149-4394-BE10-28C3047A8343)
  恢复控制文件您可以从当前备份或多路复用副本恢复控制文件。
- [删除控制文件](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-control-files.html#GUID-AB003834-2392-4B99-ADBD-497636A2E096)
  您可以删除控制文件，但数据库始终应至少具有两个控制文件。
- [控制文件数据字典视图](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-control-files.html#GUID-B0AB9343-301E-45C8-AACC-5BC9C85CB2CD)
  您可以查询一组数据字典视图以获取有关控制文件的信息。

#### 管理重做日志

[管理重做日志](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-the-redo-log.html#GUID-BC1F1762-0BB1-4218-B7AF-6160C395AAE4)

您可以通过完成以下任务来管理重做日志，这些任务包括创建重做日志组和成员，重定位和重命名重做日志成员，删除重做日志组和成员以及强制日志切换。

- [什么是重做日志？](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-the-redo-log.html#GUID-79189E7E-7431-49CA-B342-3B971186503A)
  恢复操作最关键的结构是**重做日志**，它由两个或多个预分配的文件组成，这些文件在发生更改时存储对数据库所做的所有更改。Oracle数据库的每个实例都有一个相关的重做日志，以在实例发生故障时保护数据库。
- [规划重做日志](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-the-redo-log.html#GUID-D77BCD54-908E-49F8-A80D-C8D3E9465D70)
  在配置数据库实例重做日志时，您可以遵循准则。
- [创建重做日志组和成员](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-the-redo-log.html#GUID-081AE57B-A821-4F26-97B2-4B141FDB3946)
  在数据库创建过程中计划数据库的重做日志，并创建所有必需的重做日志文件组和成员。但是，在某些情况下，您可能需要创建其他组或成员。例如，将组添加到重做日志可以纠正重做日志组可用性问题。
- [重定位和重命名重做日志成员](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-the-redo-log.html#GUID-F593023B-3071-4776-95F2-9B8B6C6E670B)
  您可以使用操作系统命令重定位重做日志，然后使用该`ALTER DATABASE`语句使数据库知道其新名称（位置）。
- [删除重做日志组和成员](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-the-redo-log.html#GUID-54E1DA00-B999-4463-B868-85ADDC108D5D)
  在某些情况下，您可能希望删除整个重做日志成员组。
- [强制](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-the-redo-log.html#GUID-6A7FB6CB-AF29-40BC-BDB3-B514C502B2F6)
  日志切换当LGWR停止写入一个重做日志组并开始写入另一个重做日志组时，将发生日志切换。默认情况下，当当前重做日志文件组填满时，日志切换会自动发生。
- [验证重做日志文件中的块](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-the-redo-log.html#GUID-E8BC43CA-D047-4AEF-AB4A-2566BE44D73A)
  您可以将数据库配置为使用校验和来验证重做日志文件中的块。
- [清除](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-the-redo-log.html#GUID-AA0EBC79-6670-409E-AED1-666FCEF3F25F)
  重做日志文件当数据库打开时，重做日志文件可能会损坏，并最终停止数据库活动，因为归档无法继续。
- [FORCE LOGGING设置的优先级](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-the-redo-log.html#GUID-27E9B63B-03E4-43E5-BA59-AB0EDDDE5CE6)
  您可以在各种级别进行设置`FORCE LOGGING`和[设置](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-the-redo-log.html#GUID-27E9B63B-03E4-43E5-BA59-AB0EDDDE5CE6)`NOLOGGING`，例如数据库，可插拔数据库（PDB），表空间或数据库对象。当`FORCE LOGGING`被设置在一个或多个级别的优先级`FORCE LOGGING`设置决定了在重做日志中记录。
- [重做日志数据字典视图](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-the-redo-log.html#GUID-05FCB15E-BE40-40EB-B756-4D280461AD7E)
  您可以查询一组数据字典视图以获取有关重做日志的信息。

#### 管理存档的重做日志文件

[管理存档的重做日志文件](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-archived-redo-log-files.html#GUID-5EE4AC49-E1B2-41A2-BEE7-AA951EAAB2F3)

您可以通过完成诸如在`NOARCHIVELOG` 或 `ARCHIVELOG` 模式之间进行选择以及指定存档目标之类的任务来管理存档的重做日志文件。

- [什么是存档的重做日志？](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-archived-redo-log-files.html#GUID-9BD00F2E-DC98-4871-8D26-FDB40623909C)
  Oracle Database使您可以将已填充的重做日志文件组保存到一个或多个脱机目标，这些目标统称为**归档重做日志**。
- [在NOARCHIVELOG和ARCHIVELOG模式](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-archived-redo-log-files.html#GUID-2FF3967A-44E6-4EA1-8759-5DB2D67CBDE1)
  之间进行选择您必须在`NOARCHIVELOG`或`ARCHIVELOG`模式下运行数据库。
- [控制归档](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-archived-redo-log-files.html#GUID-BE84B19D-7BE2-40D5-B962-5EF54E53095C)
  您可以为数据库设置归档模式，并调整归档器进程的数量。
- [指定存档目标](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-archived-redo-log-files.html#GUID-FC700EB5-72AF-414F-8D36-327D266F0BA8)
  在可以存档重做日志之前，必须确定要存档的目标，并熟悉各种目标状态。
- [关于日志传输模式](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-archived-redo-log-files.html#GUID-2A6DDDDA-4127-4574-8F2D-07AC926FCEBB)
  将归档日志传输到目的地的两种模式是**常规归档传输**和**备用传输**模式。正常传输包括将文件传输到本地磁盘。备用传输涉及通过网络将文件传输到本地或远程备用数据库。
- [管理存档目标失败](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-archived-redo-log-files.html#GUID-B2D14414-7CC4-4265-9CF4-2AADFE1F29DA)
  有时，存档目标可能会失败，从而在自动存档模式下运行时引起问题。Oracle数据库提供了可帮助您最大程度减少与目标故障相关的问题的过程。
- [控制由Archivelog进程生成的跟踪输出](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-archived-redo-log-files.html#GUID-CDFE66B4-7448-4FC8-A676-58B20FF5CC25)
  后台进程始终在适当的时候写入跟踪文件。对于archivelog流程，您可以控制生成到跟踪文件的输出。
- [查看有关存档的重做日志的](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-archived-redo-log-files.html#GUID-62E48748-F8A8-4740-90B1-5B3708A63386)
  信息您可以使用动态性能视图或`ARCHIVE` `LOG` `LIST`命令显示有关存档的重做日志的信息。
