# 第7课：备份，恢复，闪回CDB和PDB的实践

> 2010.05.03 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [第7课：备份，恢复，闪回CDB和PDB的实践](#第7课：备份，恢复，闪回cdb和pdb的实践)   
   - [第7课：概述的练习](#第7课：概述的练习)   
      - [实践概述](#实践概述)   
      - [假设条件](#假设条件)   
   - [练习7-1：CDB冷备份](#练习7-1：cdb冷备份)   
      - [总览](#总览)   
      - [任务](#任务)   
   - [练习7-2：RMAN整个CDB备份](#练习7-2：rman整个cdb备份)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习7-3：RMAN CDB/PDB备份](#练习7-3：rman-cdbpdb备份)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习7-4：从SYSTEM PDB数据文件丢失中恢复RMAN](#练习7-4：从system-pdb数据文件丢失中恢复rman)   
      - [总览](#总览)   
      - [任务](#任务)   
   - [练习7-5：从非必要的PDB数据文件丢失中恢复RMAN](#练习7-5：从非必要的pdb数据文件丢失中恢复rman)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习7-6：SQL PDB热备份](#练习7-6：sql-pdb热备份)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习7-7：SQL控制文件备份](#练习7-7：sql控制文件备份)   
      - [总览](#总览)   
      - [任务](#任务)   
   - [练习7-8：从控制文件丢失中恢复RMAN](#练习7-8：从控制文件丢失中恢复rman)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习7-9：从重做日志文件成员丢失中恢复RMAN](#练习7-9：从重做日志文件成员丢失中恢复rman)   
      - [总览](#总览)   
      - [任务](#任务)   
   - [练习7-10：从系统根数据文件丢失中恢复RMAN](#练习7-10：从系统根数据文件丢失中恢复rman)   
      - [总览](#总览)   
      - [任务](#任务)   
   - [练习7-11：从非必要根数据文件丢失中恢复RMAN](#练习7-11：从非必要根数据文件丢失中恢复rman)   
      - [总览](#总览)   
      - [任务](#任务)   
   - [练习7-12：PDB PITR](#练习7-12：pdb-pitr)   
      - [总览](#总览)   
      - [任务](#任务)   
   - [练习7-13：PDB表空间上的PITR](#练习7-13：pdb表空间上的pitr)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习7-14：普通用户删除闪回](#练习7-14：普通用户删除闪回)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习7-15：使用RMAN备份集插入PDB](#练习7-15：使用rman备份集插入pdb)   
      - [总览](#总览)   
      - [任务](#任务)   

<!-- /MDTOC -->

## 第7课：概述的练习

### 实践概述

在以下做法中，您将在CDB和PDB上执行备份和恢复操作。

* RMAN cdb2备份
* RMAN完整和部分pdb2备份
* 从SYSTEM pdb2数据文件丢失中恢复
* 从不必要的pdb2数据文件丢失中恢复
* SQL PDB热备份
* SQL控制文件备份
* 从所有控制文件丢失中恢复
* 从重做日志成员丢失中恢复
* 从SYSTEM根数据文件丢失中恢复
* 从不必要的根数据文件丢失中恢复
* PDB时间点恢复
* PDB表空间时间点恢复
* DROP普通用户的CDB闪回
* 使用RMAN备份插入未插入的PDB

### 假设条件

* 从练习3-1成功创建了 cdb2 。
* 从实践3-3成功创建了pdb2_1 。
* 从练习4-4成功重命名pdb2_1 为 pdb2 。

如果无法成功创建触发器，请执行以下追赶脚本：

```SQL
cd /home/oracle/solutions/catchup_04_03
./cr_trig.sh
```
如果无法成功创建永久和临时表空间，请执行以下追赶脚本：

```SQL
cd /home/oracle/solutions/catchup_05_02
./cr_TABLESPACES.sh
```   


## 练习7-1：CDB冷备份

### 总览

在这种做法中，您将执行CDB冷备份，以防丢失所有进一步的备份或无法从困难的情况中恢复，可以使用它。

但是在执行此任务之前，请确保您的数据库处于ARCHIVELOG模式。

### 任务

```SQL
# 1.为了减少整个CDB备份的可插拔数据库的数量，您将删除pdb1_1 和 pdb_orcl2
ORACLE_SID = cdb2
sqlplus / AS SYSDBA
ALTER PLUGGABLE DATABASE pdb1_1 CLOSE IMMEDIATE;
DROP PLUGGABLE DATABASE pdb1_1 INCLUDING DATAFILES;
ALTER PLUGGABLE DATABASE pdb_orcl2 CLOSE IMMEDIATE;
DROP PLUGGABLE DATABASE pdb_orcl2 INCLUDING DATAFILES;
EXIT

# 2.创建备份目录。
rm -Rf /home/oracle/Safe_Database_Files/cdb2
mkdir /home/oracle/Safe_Database_Files
mkdir /home/oracle/Safe_Database_Files/cdb2

# 3.在备份所有文件之前，请关闭cdb2数据库。
sqlplus / AS SYSDBA
select log_mode from v$database;
SHUTDOWN IMMEDIATE
STARTUP MOUNT
ALTER DATABASE ARCHIVELOG;
ALTER DATABASE OPEN;
SELECT name FROM v$datafile;
SHUTDOWN IMMEDIATE
EXIT


# 4.将文件复制到备份目录。该消息仅是信息性消息。
tar -czf /home/oracle/Safe_Database_Files/cdb2/db.tar.gz /u01/app/oracle/oradata/cdb2

# 5.在使用RMAN执行备份之前，请启动cdb2数据库。
sqlplus / AS SYSDBA
startup
EXIT
```  

## 练习7-2：RMAN整个CDB备份

### 总览

在这种实践中，您将执行cdb2的整个CDB备份。

### 假设条件

该PDB2已成功创建cdb2实践后，3-3和4-4。

### 任务

```SQL
# 1.运行RMAN以具有SYSDBA或SYSBACKUP特权的用户连接到cdb2。
export NLS_DATE_FORMAT='DD-MM-YYYY HH:MI:SS'
rman target /
# 2.与往常一样，备份数据库的所有数据文件（root和所有PDB），控制文件和将db_recovery_file_dest_size设置为18GB 后，SPFILE和归档日志文件
CONFIGURE DEFAULT DEVICE TYPE TO disk;
CONFIGURE CONTROLFILE AUTOBACKUP ON;
ALTER SYSTEM SET db_recovery_file_dest_size=18G SCOPE=both;
BACKUP DATABASE PLUS ARCHIVELOG;
```

## 练习7-3：RMAN CDB/PDB备份

### 总览

在这种实践中，您将执行PDB2的全部和部分PDB备份。

### 假设条件

该PDB2已成功创建cdb2惯例3-1，3-3，4-4和之后。

### 任务

```SQL
# 1.执行整个PDB备份。
BACKUP PLUGGABLE DATABASE pdb2;
# 2.对表空间ldata执行部分PDB备份。
BACKUP TABLESPACE pdb2:ldata;
```


## 练习7-4：从SYSTEM PDB数据文件丢失中恢复RMAN

### 总览

在这种情况下，您将从基本数据文件丢失中恢复PDB。如果您在遇到问题之前已经关闭了PDB，则不需要关闭CDB。如果在问题出现时打开了PDB，则需要关闭PDB，并且由于不可能，您必须关闭CDB实例并进行安装。

### 任务

```SQL
# 1.删​​除PDB2的SYSTEM数据文件。
sqlplus system/oracle_4U@PDB2
select file_name from DBA_DATA_FILES WHERE TABLESPACE_NAME='SYSTEM';
EXIT
# 2.运行RMAN以具有SYSDBA或SYSBACKUP特权的用户连接到cdb2。
rman target /
# 3.继续执行传统过程以恢复丢失的数据文件并恢复CDB。
SHUTDOWN ABORT;
STARTUP MOUNT;
RESTORE TABLESPACE pdb2:SYSTEM;
RECOVER TABLESPACE pdb2:SYSTEM;
ALTER DATABASE OPEN;
EXIT
# 或者，您可以使用新语法来还原和恢复整个PDB，如下所示
rm /u01/app/oracle/oradata/cdb2/pdb2_1/system01.dbf
rman target /
SHUTDOWN ABORT;
STARTUP MOUNT;
RESTORE pluggable database pdb2;
RECOVER pluggable database pdb2;
ALTER DATABASE OPEN;
select name, open_mode from v$pdbs;
EXIT
```

## 练习7-5：从非必要的PDB数据文件丢失中恢复RMAN

### 总览

在这种情况下，您将从非必需的PDB数据文件中恢复。

### 假设条件

该LDATA表空间在实践中5-1创建成功。

### 任务

```SQL
# 1.删除的数据文件LDATA的表空间PDB2。
sqlplus system/oracle_4U@PDB2
select file_name from dba_data_files where tablespace_name='LDATA';
exit
rm /u01/app/oracle/oradata/cdb2/pdb2_1/ldata_01.dbf
# 2.继续执行传统过程以恢复丢失的数据文件并恢复表空间，因为它是非CDB的。
# a.将表空间置于离线模式。
sqlplus system/oracle_4U@PDB2
ALTER TABLESPACE ldata OFFLINE IMMEDIATE;
exit
# b.运行RMAN以具有SYSDBA或SYSBACKUP特权的用户连接到cdb2。
rman target /

# c.还原和恢复表空间。
RESTORE TABLESPACE pdb2:LDATA;
RECOVER TABLESPACE pdb2:LDATA;
EXIT
# 3.将表空间放回ONLINE。
sqlplus system/oracle_4U@PDB2
ALTER TABLESPACE ldata ONLINE;

```

## 练习7-6：SQL PDB热备份

### 总览

在这种实践中，您将在cdb2中执行PDB2的热备份。

### 假设条件

该PDB2已成功创建cdb2惯例3-1，3-3，4-4和之后。

### 任务

```SQL
# 1.列出属于要备份的PDB2的所有数据文件。
connect system/oracle_4U@PDB2
select file_name from dba_data_files;
# 2.在热备份中设置PDB。
ALTER PLUGGABLE DATABASE pdb2 BEGIN BACKUP;
EXIT
# 3.将可插入数据库的数据文件复制到备份目录。
mkdir /home/oracle/backup
cp /u01/app/oracle/oradata/cdb2/pdb2_1/* /home/oracle/backup
# 4.停用备份模式。
sqlplus system/oracle_4U@PDB2
ALTER PLUGGABLE DATABASE pdb2 END BACKUP;
```

## 练习7-7：SQL控制文件备份

### 总览

在这种实践中，您将使用传统的SQL命令来备份cdb2控制文件。

### 任务

```SQL
CONNECT / as sysdba
alter database backup controlfile to trace;
exit
cd /u01/app/oracle/diag/rdbms/cdb2/cdb2/trace
ls -ltr|tail -10
cat cdb2_ora_20680.trc

```

## 练习7-8：从控制文件丢失中恢复RMAN

### 总览

在这种情况下，您将从控制文件丢失中恢复CDB。

### 假设条件

练习7-2成功完成了cdb2的整个CDB备份。

### 任务

```SQL
# 1.删​​除CDB的控制文件。
sqlplus / AS SYSDBA
select name from v$controlfile;
!rm /u01/app/oracle/oradata/cdb2/control01.ctl
# 2.关闭或中止实例cdb2。
shutdown abort
exit
# 3.继续执行传统过程以还原控制文件并恢复CDB，就好像它是非CDB数据库一样。
rman target /
startup nomount;
RESTORE CONTROLFILE FROM AUTOBACKUP;
ALTER DATABASE MOUNT;
RECOVER DATABASE;
ALTER DATABASE OPEN RESETLOGS;
select name, open_mode from v$pdbs;
# 4.备份整个cdb2。
# a.使用BACKUP命令
BACKUP DATABASE PLUS ARCHIVELOG DELETE ALL INPUT;
exit
# b.如果您遇到一些空间问题，例如以下内容，请回收一些空间并增加快速恢复区域目标大小：
delete obsolete;
ALTER SYSTEM SET db_recovery_file_dest_size=20G SCOPE=both;
EXIT
```

## 练习7-9：从重做日志文件成员丢失中恢复RMAN

### 总览

在这种实践中，您将从重做日志文件成员丢失中恢复cdb2。

### 任务

```SQL
# 1.复用重做日志文件（如果尚未完成）。
sqlplus system/oracle_4U
select member from v$logfile;
ALTER DATABASE ADD LOGFILE MEMBER '/u01/app/oracle/oradata/cdb2/redo01_2.log' TO GROUP 1;
ALTER DATABASE ADD LOGFILE MEMBER '/u01/app/oracle/oradata/cdb2/redo02_2.log' TO GROUP 2;
ALTER DATABASE ADD LOGFILE MEMBER '/u01/app/oracle/oradata/cdb2/redo03_2.log' TO GROUP 3;
alter system switch logfile;
alter system switch logfile;
alter system switch logfile;
alter system switch logfile;
EXIT
# 2.删除cdb2的重做日志文件成员。
rm /u01/app/oracle/oradata/cdb2/redo01.log
# 3.继续执行传统过程以重新生成重做日志文件成员。
sqlplus system/oracle_4U
ALTER DATABASE CLEAR LOGFILE GROUP 1;
SELECT member FROM v$logfile;
! ls /u01/app/oracle/oradata/cdb2/redo*
# 如果由于重做日志文件属于当前活动组而无法成功完成操作并收到以下消息，请切换重做日志组。并重新尝试失败的语句。
alter system switch logfile;
ALTER DATABASE CLEAR UNARCHIVED LOGFILE GROUP 1;
EXIT
# 在后一种情况下，您必须执行数据库备份，因为缺少存档日志文件。
rman target /
BACKUP DATABASE PLUS ARCHIVELOG DELETE ALL INPUT;
exit
```

## 练习7-10：从系统根数据文件丢失中恢复RMAN

### 总览

在这种情况下，您将从根数据文件丢失（特别是SYSTEM数据文件）丢失中恢复。

### 任务

```SQL
# 1.从根SYSTEM表空间中删除SYSTEM数据文件。
sqlplus system/oracle_4U
SELECT file_name FROM dba_data_files WHERE TABLESPACE_NAME='SYSTEM';
exit
rm /u01/app/oracle/oradata/cdb2/system01.dbf
# 2.运行RMAN以具有SYSDBA或SYSBACKUP特权的用户连接到cdb2。
rman target /
# 3.继续执行传统过程以恢复丢失的数据文件并恢复CDB。
SHUTDOWN ABORT;
STARTUP MOUNT;
RESTORE TABLESPACE SYSTEM;
RECOVER TABLESPACE SYSTEM;
ALTER DATABASE OPEN;
# 4.备份CDB。
BACKUP DATABASE PLUS ARCHIVELOG DELETE ALL INPUT;
exit
```

## 练习7-11：从非必要根数据文件丢失中恢复RMAN

### 总览

在这种情况下，您将使用Data Recovery Advisor RMAN从不必要的根数据文件丢失中恢复

### 任务

```SQL
# 1.删​​除cdb2根目录的SYSAUX表空间的数据文件。
sqlplus / as sysdba
select file_name from dba_data_files where tablespace_name='SYSAUX';
!rm /u01/app/oracle/oradata/cdb2/sysaux01.dbf
EXIT
# 2.运行RMAN以具有SYSDBA或SYSBACKUP特权的用户连接到cdb2。
rman target /
# a.发现失败。
LIST FAILURE;
LIST FAILURE DETAIL;
# b.从RMAN Data Recovery Advisor获得建议。
ADVISE FAILURE;
# c.预览提供的脚本以修复故障。
REPAIR FAILURE PREVIEW;
# d.如果提供的脚本使您满意，请修复故障。这将执行脚本。
REPAIR FAILURE;
# 4.备份CDB。
BACKUP DATABASE PLUS ARCHIVELOG DELETE ALL INPUT;
exit;
```

## 练习7-12：PDB PITR

### 总览

在这种情况下，您将执行可插拔数据库时间点恢复。将创建一个表DJ.T1，并将行加载到PDB2可插拔数据库中的表DJ.T1中，并在PDB2_2可插拔数据库中执行类似的操作。以后，将行加载到错误的PDB中的错误表中。您必须将情况恢复到错误插入行之前的时间。

### 任务

```SQL
# 1.连接到PDB2并创建一个表空间来存储 DJ的表。
sqlplus system/oracle_4U@pdb2
CREATE TABLESPACE dj_pdb2
DATAFILE '/u01/app/oracle/oradata/cdb2/pdb2_1/dj_pdb2.f' SIZE 10m;
CREATE USER dj identified by oracle_4U default tablespace dj_pdb2;
GRANT create session, create table, unlimited tablespace TO dj;
# 2.创建一个DJ.T1表。
CREATE TABLE dj.t1(c varchar2(100)) TABLESPACE dj_pdb2;
# 3.确保在创建表时记下SCN值。
SELECT timestamp_to_scn(sysdate) FROM v$database;
# 4.将行插入DJ.T1表中。
BEGIN
FOR i in 1.. 10000 LOOP
insert into dj.t1 values ('aaaaaaaaaaaaaaaaaaaaaaaaaaa');
END LOOP;
COMMIT; END;
/
select timestamp_to_scn(sysdate) from v$database;

# 5.您意识到在错误的PDB中加载了错误的表。您在右PDB PDB2_2中创建一个表空间来存储表DJ.T1，然后再将PDB2恢复到表仍然为空的时间。
CONNECT system/oracle_4U@pdb2_2
CREATE TABLESPACE dj_pdb2_2 DATAFILE '/u01/app/oracle/oradata/cdb2/pdb2_2/dj_pdb2_2.f' SIZE 10m;
CREATE USER dj IDENTIFIED BY oracle_4U DEFAULT TABLESPACE dj_pdb2_2;
GRANT create session, create table, unlimited tablespace TO dj;
CREATE TABLE dj.t1(c varchar2(100)) tablespace dj_pdb2_2;

# 6.将行加载到DJ.T1表中。
BEGIN
FOR i in 1.. 10000 LOOP
insert into dj.t1 values ('aaaaaaaaaaaaaaaaaaaaaaaaaaa');
END LOOP;
COMMIT; END;
/
# 7.继续执行PDB PDB2的PITR（时间点恢复）到表的时间DJ.T1 仍然是空的。
# a.连接到cdb2并关闭PDB2。
rman target /
ALTER PLUGGABLE DATABASE pdb2 CLOSE;
# b.执行PDB2的PDB PITR 。
RUN {
SET UNTIL SCN = 2668939;
RESTORE PLUGGABLE DATABASE pdb2;
RECOVER PLUGGABLE DATABASE pdb2 AUXILIARY
DESTINATION='/u01/app/oracle/oradata'; ALTER PLUGGABLE DATABASE pdb2 OPEN RESETLOGS; }
EXIT

# 8.检查是否只有PDB2恢复到SCN 2668939而不是PDB2_2。
sqlplus system/oracle_4U@pdb2
SELECT * FROM DJ.T1;
CONNECT system/oracle_4U@pdb2_2
SELECT COUNT(*) FROM DJ.T1;
exit
# 9.备份CDB。
rman target /
BACKUP DATABASE PLUS ARCHIVELOG DELETE ALL INPUT;
EXIT
```

## 练习7-13：PDB表空间上的PITR

### 总览

在这种实践中，您将对非必需的PDB数据文件执行PITR。PDB2_2可插入数据库中表DJ.T2中的行已被错误地删除。您必须将情况恢复到删除并提交行之前的时间。

### 假设条件

练习3-4完成后，已成功创建PDB pdb2_2。在实践7-12中已成功备份了PDB pdb2_2。

删除OPEN_ALL_PDBS触发器。您必须删除它。禁用它是不够的，并且您会收到错误消息，阻止该操作成功完成。

### 任务

```SQL
# 1.删除OPEN_ALL_PDBS触发器。
sqlplus / as sysdba
DROP TRIGGER open_all_pdbs;
# 2.创建一个具有10000行的DJ.T2表PDB2_2，在DELETE之前检查SCN
BEGIN
       FOR i in 1.. 10000 LOOP
           insert into dj.t2 values (100);
       END LOOP;
COMMIT; END;
/
# 3.在还原和恢复表空间之前，请将其设置为脱机状态。
ALTER TABLESPACE dj_pdb2_2 OFFLINE IMMEDIATE;
EXIT
# 4.表中所有行都存在时的情况。
rman target /
# a.在PDB2_2中执行表空间时间点恢复。
RECOVER TABLESPACE pdb2_2:dj_pdb2_2
UNTIL SCN 2674656
AUXILIARY DESTINATION '/u01/app/oracle/oradata';
EXIT
# b.在线表空间
sqlplus sys/oracle_4U@PDB2_2 as sysdba
ALTER TABLESPACE dj_pdb2_2 ONLINE;
# c.Check the content of the DJ.T2 table in PDB2_2.
select count(*) from DJ.T2;
# d.备份CDB
rman target /
DELETE OBSOLETE;
BACKUP DATABASE PLUS ARCHIVELOG delete all input;
EXIT
# e.如果要在数据库启动后自动打开所有PDB，请重新创建触发器。
sqlplus / as sysdba
CREATE TRIGGER open_all_PDBs AFTER STARTUP ON DATABASE
begin
execute immediate 'alter pluggable database all open';
 end open_all_PDBs;
/
exit
```


## 练习7-14：普通用户删除闪回

### 总览

在这种情况下，删除普通用户后，您将闪回CDB。

### 假设条件

该Ç##_USER普通用户存在于cdb2。这已在实践6-2中完成。

### 任务

```SQL
# 1.将CDB cdb2设置为FLASHBACK模式。
export NLS_DATE_FORMAT='DD-MM-YYYY HH:MI:SS'
sqlplus / as sysdba
SELECT flashback_on from V$DATABASE;
SHUTDOWN IMMEDIATE
STARTUP MOUNT
ALTER SYSTEM SET DB_FLASHBACK_RETENTION_TARGET=2880 SCOPE=BOTH;
ALTER DATABASE FLASHBACK ON;
ALTER DATABASE OPEN;
# 2.删除普通用户C##_USER.
col username format A20
select USERNAME, COMMON, CON_ID from cdb_users where username='C##_USER';
select timestamp_to_scn(current_timestamp) from v$database;
DROP USER C##_USER CASCADE;
alter system switch logfile;
alter system switch logfile;
alter system switch logfile;
alter system switch logfile;
# 3.继续闪回数据库操作。
SHUTDOWN IMMEDIATE
STARTUP MOUNT
FLASHBACK DATABASE TO SCN 2321219;
# 4.在以下列方式打开CDB之前，以只读模式打开数据库以查看更改。
ALTER DATABASE OPEN READ ONLY;
select USERNAME, COMMON, CON_ID from cdb_users where username='C##_USER';
# 5.在只读中打开PDB以查看所有更改。
ALTER PLUGGABLE DATABASE ALL OPEN READ ONLY;
select USERNAME, COMMON, CON_ID from cdb_users where username='C##_USER';
# 6.使用RESETLOGS打开CDB 。
SHUTDOWN IMMEDIATE
STARTUP MOUNT
FLASHBACK DATABASE TO SCN 2321219;
ALTER DATABASE OPEN RESETLOGS;
# 7.检查C##_USER是否可以在每个容器中连接。
connect C##_USER/x Connected.
connect C##_USER/x@PDB2 Connected.
connect C##_USER/x@PDB2_2 Connected.
exit;
# 8.备份CDB。
rman target /
BACKUP DATABASE PLUS ARCHIVELOG delete all input;
exit
```

## 练习7-15：使用RMAN备份集插入PDB

### 总览

在这种实践中，您将使用RMAN备份集和未插入的PDB的XML文件将PDB插入CDB。您决定拔出pdb2_2，然后使用备份集和XML文件将其重新插入到同一cdb2中。如果您已拔出PDB并将其插入另一个CDB，则将执行相同的操作。

要执行这些操作，您将使用企业管理器云控制。

### 任务

在企业管理器云控制中操作，步骤省略。
