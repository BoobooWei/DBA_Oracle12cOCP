# 实践16:移动数据

> **Practices for Lesson 16: Moving Data**
>
> 2020.02.04 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [实践16:移动数据](#实践16移动数据)   
   - [实践16:概览](#实践16概览)   
   - [实践16-1:使用数据泵移动数据](#实践16-1使用数据泵移动数据)   
      - [Overview](#overview)   
      - [Task](#task)   
      - [Practice](#practice)   
      - [KnowledgePoint](#knowledgepoint)   
   - [实践16-2:使用SQL*Loader加载数据](#实践16-2使用sqlloader加载数据)   
      - [Overview](#overview)   
      - [Task](#task)   
      - [Practice](#practice)   
      - [KnowledgePoint](#knowledgepoint)   
   - [总结](#总结)   
      - [Oracle Data Pump概述](#oracle-data-pump概述)   
      - [SQL * Loader](#sql-loader)   

<!-- /MDTOC -->

## 实践16:概览

Practices for Lesson 16: Overview

**Background:** In the recent past, you received a number of questions about the HR schema. To analyze them without interfering in daily activities, you decide to use Data Pump export to export the HR schema to a file. When you perform the export, you are not sure into which database you will be importing this schema.

In the end, you learn that the only database for which management approves an import is the orcl database. So you perform the import with Data Pump import, remapping the HR schema to the DBA1 schema.

Then you receive two data load requests for which you decide to use SQL*Loader.

**背景1：**

最近，您收到了许多关于HR schema的问题。为了在不干扰日常活动的情况下分析它们，您决定

1. 使用Data Pump导出将HR schema的数据导出到文件中。

2. 执行导出时，不确定要将此模式导入哪个数据库，最后，您了解到管理部门批准导入的惟一数据库是xxx数库。

3. 因此使用**数据泵**导入到指定实例中，将HR schema 映射为 DBA1 schema。

**背景2:**

1. 您将收到两个数据加载请求，您决定使用`SQL*Loader`来实现。

## 实践16-1:使用数据泵移动数据

Practice 16-1: Moving Data by Using Data Pump

### Overview

In this practice, you first grant the DBA1 user the privileges necessary to provide access to the DATA_PUMP_DIR directory. You then export the HR schema so that you can then import the tables that you want into the DBA1 schema. In the practice, you import only the EMPLOYEES table at this time.

在此实践中，首先授予DBA1用户访问`DATA_PUMP_DIR`目录所需的特权。然后导出HR模式，这样就可以将需要的表导入到DBA1模式中。在实践中，只导入EMPLOYEES表。

### Task

1. First, you need to grant to the DBA1 user the appropriate privileges on the DATA_PUMP_DIR directory. Be sure you know the OS directory where the Data Pump import file will be placed.
2. Use the Data Pump export utility to export the HR schema. Specify the DBA1 user to execute the export operation.
3. Now, import the **EMPLOYEES** table from the exported **HR** schema into the **DBA1** schema.

### Practice

1. 首先，您需要向DBA1用户授予`DATA_PUMP_DIR`目录上适当的特权。确保您知道数据泵导入文件将放在哪个OS目录下。

   ```sql
   sqlplus / as sysdba
   SELECT * from dba_directories WHERE directory_name = 'DATA_PUMP_DIR';
   alter session set container=emrep;
   grant read on directory data_pump_dir to dba1;
   grant write on directory data_pump_dir to dba1;
   ```

2. 使用数据泵导出实用程序导出HR schema。指定DBA1用户来执行导出操作。

   ```sql
   expdp dba1/oracle@emrep  dumpfile=HREXP%U.dmp directory=DATA_PUMP_DIR logfile=hrexp.log SCHEMAS=HR
   ```

3. 现在，将**HR.EMPLOYEES**表导入到**DBA1**。

   ```sql
   impdp dba1/oracle@emrep DIRECTORY=data_pump_dir DUMPFILE=HREXP01.dmp REMAP_SCHEMA=hr:dba1 TABLES=hr.REGIONS LOGFILE=empimport.log
   ```



执行结果

```bash
[oracle@ocm ~]$ sqlplus / as sysdba
SQL> set serveroutput on
SQL> exec print_table(q'[SELECT * from dba_directories WHERE directory_name = 'DATA_PUMP_DIR']')
OWNER			      : SYS
DIRECTORY_NAME		      : DATA_PUMP_DIR
DIRECTORY_PATH		      : /u01/app/oracle/admin/booboo/dpdump/
ORIGIN_CON_ID		      : 1

SQL> alter session set container=emrep;

Session altered.

grant read on directory data_pump_dir to dba1;

Grant succeeded.

SQL> grant write on directory data_pump_dir to dba1;

Grant succeeded.
SQL> !ls -l /u01/app/oracle/admin/booboo/dpdump/
total 4
drwxr-x---  2 oracle oinstall   6 Feb  4 22:15 96710507BCB024C7E0553CE49B3EAF97
-rw-r-----. 1 oracle oinstall 116 Nov  3 19:31 dp.log

SQL> !ls -l /u01/app/oracle/admin/booboo/dpdump/96710507BCB024C7E0553CE49B3EAF97
total 0

SQL> !expdp dba1/oracle@emrep dumpfile=HREXP%U.dmp directory=DATA_PUMP_DIR logfile=hrexp.log SCHEMAS=HR

Export: Release 12.2.0.1.0 - Production on Tue Feb 4 22:17:17 2020

Copyright (c) 1982, 2017, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
Starting "DBA1"."SYS_EXPORT_SCHEMA_01":  dba1/********@emrep dumpfile=HREXP%U.dmp directory=DATA_PUMP_DIR logfile=hrexp.log SCHEMAS=HR
Processing object type SCHEMA_EXPORT/TABLE/TABLE_DATA
Processing object type SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/INDEX_STATISTICS
Processing object type SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
Processing object type SCHEMA_EXPORT/STATISTICS/MARKER
Processing object type SCHEMA_EXPORT/USER
Processing object type SCHEMA_EXPORT/SYSTEM_GRANT
Processing object type SCHEMA_EXPORT/ROLE_GRANT
Processing object type SCHEMA_EXPORT/DEFAULT_ROLE
Processing object type SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
Processing object type SCHEMA_EXPORT/SEQUENCE/SEQUENCE
Processing object type SCHEMA_EXPORT/TABLE/TABLE
Processing object type SCHEMA_EXPORT/TABLE/GRANT/OWNER_GRANT/OBJECT_GRANT
Processing object type SCHEMA_EXPORT/TABLE/COMMENT
Processing object type SCHEMA_EXPORT/PROCEDURE/PROCEDURE
Processing object type SCHEMA_EXPORT/PROCEDURE/ALTER_PROCEDURE
Processing object type SCHEMA_EXPORT/VIEW/VIEW
Processing object type SCHEMA_EXPORT/TABLE/INDEX/INDEX
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT
Processing object type SCHEMA_EXPORT/TABLE/TRIGGER
. . exported "HR"."EMPLOYEES"                            17.08 KB     107 rows
. . exported "HR"."LOCATIONS"                            8.437 KB      23 rows
. . exported "HR"."JOB_HISTORY"                          7.195 KB      10 rows
. . exported "HR"."JOBS"                                 7.109 KB      19 rows
. . exported "HR"."DEPARTMENTS"                          7.125 KB      27 rows
. . exported "HR"."T02"                                  6.734 KB       9 rows
. . exported "HR"."COUNTRIES"                            6.367 KB      25 rows
. . exported "HR"."REGIONS"                              5.546 KB       4 rows
. . exported "HR"."TEST"                                 5.062 KB       1 rows
Master table "DBA1"."SYS_EXPORT_SCHEMA_01" successfully loaded/unloaded
******************************************************************************
Dump file set for DBA1.SYS_EXPORT_SCHEMA_01 is:
  /u01/app/oracle/admin/booboo/dpdump/96710507BCB024C7E0553CE49B3EAF97/HREXP01.dmp
Job "DBA1"."SYS_EXPORT_SCHEMA_01" successfully completed at Tue Feb 4 22:18:24 2020 elapsed 0 00:01:03

SQL> !ls -lh /u01/app/oracle/admin/booboo/dpdump/96710507BCB024C7E0553CE49B3EAF97/HREXP01.dmp
-rw-r----- 1 oracle oinstall 752K Feb  4 22:18 /u01/app/oracle/admin/booboo/dpdump/96710507BCB024C7E0553CE49B3EAF97/HREXP01.dmp

SQL> !impdp dba1/oracle@emrep DIRECTORY=data_pump_dir DUMPFILE=HREXP01.dmp REMAP_SCHEMA=hr:dba1 TABLES=hr.employees LOGFILE=empimport.log

Import: Release 12.2.0.1.0 - Production on Tue Feb 4 22:23:55 2020

Copyright (c) 1982, 2017, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
Master table "DBA1"."SYS_IMPORT_TABLE_01" successfully loaded/unloaded
Starting "DBA1"."SYS_IMPORT_TABLE_01":  dba1/********@emrep DIRECTORY=data_pump_dir DUMPFILE=HREXP01.dmp REMAP_SCHEMA=hr:dba1 TABLES=hr.employees LOGFILE=empimport.log
Processing object type SCHEMA_EXPORT/TABLE/TABLE
Processing object type SCHEMA_EXPORT/TABLE/TABLE_DATA
. . imported "DBA1"."EMPLOYEES"                          17.08 KB     107 rows
Processing object type SCHEMA_EXPORT/TABLE/GRANT/OWNER_GRANT/OBJECT_GRANT
Processing object type SCHEMA_EXPORT/TABLE/COMMENT
Processing object type SCHEMA_EXPORT/TABLE/INDEX/INDEX
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
Processing object type SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/INDEX_STATISTICS
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/REF_CONSTRAINT
ORA-39083: Object type REF_CONSTRAINT:"DBA1"."EMP_DEPT_FK" failed to create with error:
ORA-00942: table or view does not exist

Failing sql is:
ALTER TABLE "DBA1"."EMPLOYEES" ADD CONSTRAINT "EMP_DEPT_FK" FOREIGN KEY ("DEPARTMENT_ID") REFERENCES "DBA1"."DEPARTMENTS" ("DEPARTMENT_ID") ENABLE

ORA-39083: Object type REF_CONSTRAINT:"DBA1"."EMP_JOB_FK" failed to create with error:
ORA-00942: table or view does not exist

Failing sql is:
ALTER TABLE "DBA1"."EMPLOYEES" ADD CONSTRAINT "EMP_JOB_FK" FOREIGN KEY ("JOB_ID") REFERENCES "DBA1"."JOBS" ("JOB_ID") ENABLE

Processing object type SCHEMA_EXPORT/TABLE/TRIGGER
Processing object type SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
Processing object type SCHEMA_EXPORT/STATISTICS/MARKER
ORA-39082: Object type TRIGGER:"DBA1"."SECURE_EMPLOYEES" created with compilation warnings

ORA-39082: Object type TRIGGER:"DBA1"."UPDATE_JOB_HISTORY" created with compilation warnings

Job "DBA1"."SYS_IMPORT_TABLE_01" completed with 4 error(s) at Tue Feb 4 22:24:18 2020 elapsed 0 00:00:22

-- 由于给表存在外键，所以不能单独导入该表，换一个没有外键约束的表REGIONS
SQL> !impdp dba1/oracle@emrep DIRECTORY=data_pump_dir DUMPFILE=HREXP01.dmp REMAP_SCHEMA=hr:dba1 TABLES=hr.REGIONS LOGFILE=empimport.log

Import: Release 12.2.0.1.0 - Production on Tue Feb 4 22:25:53 2020

Copyright (c) 1982, 2017, Oracle and/or its affiliates.  All rights reserved.

Connected to: Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
Master table "DBA1"."SYS_IMPORT_TABLE_01" successfully loaded/unloaded
Starting "DBA1"."SYS_IMPORT_TABLE_01":  dba1/********@emrep DIRECTORY=data_pump_dir DUMPFILE=HREXP01.dmp REMAP_SCHEMA=hr:dba1 TABLES=hr.REGIONS LOGFILE=empimport.log
Processing object type SCHEMA_EXPORT/TABLE/TABLE
Processing object type SCHEMA_EXPORT/TABLE/TABLE_DATA
. . imported "DBA1"."REGIONS"                            5.546 KB       4 rows
Processing object type SCHEMA_EXPORT/TABLE/GRANT/OWNER_GRANT/OBJECT_GRANT
Processing object type SCHEMA_EXPORT/TABLE/INDEX/INDEX
Processing object type SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
Processing object type SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/INDEX_STATISTICS
Processing object type SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
Processing object type SCHEMA_EXPORT/STATISTICS/MARKER
Job "DBA1"."SYS_IMPORT_TABLE_01" successfully completed at Tue Feb 4 22:26:15 2020 elapsed 0 00:00:21

SQL> select tname from tab where tname='REGIONS';

TNAME
--------------------------------------------------------------------------------
REGIONS

SQL> select * from REGIONS where rownum < 3;

 REGION_ID REGION_NAME
---------- -------------------------
	 1 Europe
	 2 Americas
```



### KnowledgePoint

## 实践16-2:使用SQL*Loader加载数据

Practice 16-2: Loading Data by Using SQL*Loader

### Overview

In this practice, you load data into the PRODUCT_DESCRIPTIONS table by using SQL*Loader

Express Mode. Data and control files are provided.

在这个实践中，您可以使用 SQL * Loader 将数据加载到product_description表中。

表达模式。提供数据和控制文件。

### Task

1. As the **OE** user, use SQL*Loader to load the PRODUCT_DESCRIPTIONS table from the **product_descriptions.dat** data file in Express Mode.*

   **Warning:** DO NOT execute this SQL*Loader command a second time without first executing the cleanup script in step 3. Duplicate rows will be loaded and the Primary Key Index will become unusable.

2. As the **OE** user, load data into the ******INVENTORIES** table by using SQL*Loader command line. The **lab_16_02_02.dat** data file contains rows of data for the **PRODUCT_ON_HAND** table. The **lab_16_02_02.ctl** file is the control file for this load.

   Optionally, view the lab_16_02_02.dat and lab_16_02_02.ctl files to learn more about their structure before going further.

3. Execute the **$LABS/P16/lab_16_cleanup.sh** script to remove the rows and files generated by this practice.

### Practice

1. 作为HR用户，使用SQL * Loader 从 `product_description.dat` 文件导入 product_description 表，dat数据文件在Express模式下。

   警告:如果不首先执行步骤3中的清理脚本，不要重复执行这个SQL * Loader命令。重复的行将被加载，主键索引将变得不可用。

   ```sql
   # 创建用户oe
   sqlplus dba1/oracle@emrep as sysdba
   create user oe identified by oracle_4u;
   grant connect to oe;
   grant unlimited tablespace to oe;
   grant create any table to oe;

   # 使用oe用户登陆后创建表
   sqlplus oe/oracle_4u@emrep
   create table product_descriptions (PRODUCT_ID varchar(255),LANGUAGE_ID varchar(255),TRANSLATED_NAME varchar(255),TRANSLATED_DESCRIPTION varchar(255));

   # 使用sqlldr工具导入数据
   sqlldr oe/oracle_4u@emrep TABLE=product_descriptions

   # 检查导入的数据
   sqlplus oe/oracle_4u@emrep
   select count(*) from product_descriptions;
   ```

   product_description.dat文件

   ```sql
   4001,ENG,Door,Outdoor
   4002,FRE,Porte,Porte exterieure
   4003,SPA,Puerta,Puerta exterior
   4004,GER,Tur,Auberliche Tur
   5001,ENG,Shutter,Outdoor shutter
   5002,FRE,Volet,Volet exterieur
   5003,SPA,Obturador,Obturador exterior
   5004,GER,Fenster, Fensterladen
   ```

    注意：导入前表必须先存在，否则会报错

   ```sql
   SQL*Loader-941: Error during describe of table PRODUCT_DESCRIPTIONS
   ORA-04043: object PRODUCT_DESCRIPTIONS does not exist
   ```



2. 作为OE用户，使用SQL* Loader命令行将数据加载到 INVENTORIES 表中。

   `lab_16_02_02.dat`数据文件包含PRODUCT_ON_HAND表的数据行;

   `lab_16_02_02.ctl`文件是这个加载的控制文件。

   > 可以查看`lab_16_02_02.da`t和`lab_16_02_02.ctl`文件，以了解更多关于他们的结构，然后再进一步。

   ```sql
   # 使用oe用户登陆后创建表
   sqlplus oe/oracle_4u@emrep
   create table INVENTORIES (warehouse_id  varchar(255),
   product_id  varchar(255),
   quantity_on_hand  varchar(255), remark varchar(255));

   # 使用sqlldr工具导入数据
   sqlldr userid=oe/oracle_4u@emrep control=lab_16_02_02.ctl log=lab_16_02_02.log data=lab_16_02_02.dat

   # 检查
   SELECT * FROM inventories WHERE quantity_on_hand = 7 AND	WAREHOUSE_ID>500 ;

   ```

   lab_16_02_02.dat

   ```sql
   501,1001,7
   502,1001,7
   ...省略
   ```

    lab_16_02_02.ctl
   ```sql
   -- Oracle Database 12c: Administration Workshop I
   -- Oracle Server Technologies - Curriculum Development
   --
   -- ***Training purposes only***
   -- ***Not appropriate for production use***
   --
   -- Load data into the INVENTORIES table
   --
   LOAD DATA
   infile '/home/oracle$LABS/lab_16_02_02.dat'
   INTO TABLE OE.INVENTORIES
   APPEND
   FIELDS TERMINATED BY ','
   (warehouse_id,
   product_id,
   quantity_on_hand)
   ```



3. 执行`lab_16_cleanup.sh`脚本删除由这个实践生成的行和文件。

   ```bash
   cat lab_16_cleanup.sh
   #!/bin/bash
   #-- Cleanup after Practice 16
   #-- remove *.log files
   rm $LABS/P16/*.log
   #-- remove *.bad files
   rm $LABS/P16/*.bad
   sqlplus -s oe/oracle_4U <<-EOF
   -- delete rows inserted by SQL*Loader
   DELETE FROM OE.PRODUCT_DESCRIPTIONS WHERE product_id > 4000;
   Commit;
   Delete from OE.INVENTORIES
     WHERE quantity_on_hand = 7
     AND   WAREHOUSE_ID>500 ;
   COMMIT;
   EOF
   ```


1～3执行结果

```bash
[oracle@ocm P16]$ pwd
/u01/software/labs/P16
[oracle@ocm P16]$ ll
total 24
-rw-r--r-- 1 oracle oinstall  125 Jan 22  2013 afiedt.buf
-rw-r--r-- 1 oracle oinstall  387 Feb  4 22:49 lab_16_02_02.ctl
-rw-r--r-- 1 oracle oinstall  913 Jan 22  2013 lab_16_02_02.dat
-rw-r--r-- 1 oracle oinstall  369 Jan 23  2013 lab_16_cleanup.sh
-rw-r--r-- 1 oracle oinstall  247 Jan 22  2013 product_descriptions.dat
[oracle@ocm P16]$ sqlplus oe/oracle_4u@emrep

SQL*Plus: Release 12.2.0.1.0 Production on Wed Feb 5 00:51:35 2020

Copyright (c) 1982, 2016, Oracle.  All rights reserved.

Last Successful login time: Wed Feb 05 2020 00:45:22 +08:00

Connected to:
Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production

SQL> create table product_descriptions (PRODUCT_ID varchar(255),LANGUAGE_ID varchar(255),TRANSLATED_NAME varchar(255),TRANSLATED_DESCRIPTION varchar(255));

Table created.

SQL> !sqlldr oe/oracle_4u@emrep TABLE=product_descriptions

SQL*Loader: Release 12.2.0.1.0 - Production on Wed Feb 5 00:52:12 2020

Copyright (c) 1982, 2017, Oracle and/or its affiliates.  All rights reserved.

Express Mode Load, Table: PRODUCT_DESCRIPTIONS
Path used:      External Table, DEGREE_OF_PARALLELISM=AUTO
SQL*Loader-816: error creating temporary directory object SYS_SQLLDR_XT_TMPDIR_00000 for file product_descriptions.dat
ORA-01031: insufficient privileges
SQL*Loader-579: switching to direct path for the load
SQL*Loader-583: ignoring trim setting with direct path, using value of LDRTRIM
SQL*Loader-584: ignoring DEGREE_OF_PARALLELISM setting with direct path, using value of NONE
Express Mode Load, Table: PRODUCT_DESCRIPTIONS
Path used:      Direct

Load completed - logical record count 8.

Table PRODUCT_DESCRIPTIONS:
  8 Rows successfully loaded.

Check the log file:
  product_descriptions.log
for more information about the load.

SQL> !cat product_descriptions.log

SQL*Loader: Release 12.2.0.1.0 - Production on Wed Feb 5 00:52:12 2020

Copyright (c) 1982, 2017, Oracle and/or its affiliates.  All rights reserved.

Express Mode Load, Table: PRODUCT_DESCRIPTIONS
Data File:      product_descriptions.dat
  Bad File:     product_descriptions_%p.bad
  Discard File:  none specified

 (Allow all discards)

Number to load: ALL
Number to skip: 0
Errors allowed: 50
Continuation:    none specified
Path used:      External Table

Table PRODUCT_DESCRIPTIONS, loaded from every logical record.
Insert option in effect for this table: APPEND

   Column Name                  Position   Len  Term Encl Datatype
------------------------------ ---------- ----- ---- ---- ---------------------
PRODUCT_ID                          FIRST     *   ,       CHARACTER
LANGUAGE_ID                          NEXT     *   ,       CHARACTER
TRANSLATED_NAME                      NEXT     *   ,       CHARACTER
TRANSLATED_DESCRIPTION               NEXT     *   ,       CHARACTER

Generated control file for possible reuse:
OPTIONS(EXTERNAL_TABLE=EXECUTE, TRIM=LRTRIM)
LOAD DATA
INFILE 'product_descriptions'
APPEND
INTO TABLE PRODUCT_DESCRIPTIONS
FIELDS TERMINATED BY ","
(
  PRODUCT_ID,
  LANGUAGE_ID,
  TRANSLATED_NAME,
  TRANSLATED_DESCRIPTION
)
End of generated control file for possible reuse.

SQL*Loader-816: error creating temporary directory object SYS_SQLLDR_XT_TMPDIR_00000 for file product_descriptions.dat
ORA-01031: insufficient privileges

------------------------------------------------------------------------
SQL*Loader-579: switching to direct path for the load
SQL*Loader-583: ignoring trim setting with direct path, using value of LDRTRIM
SQL*Loader-584: ignoring DEGREE_OF_PARALLELISM setting with direct path, using value of NONE
------------------------------------------------------------------------

Express Mode Load, Table: PRODUCT_DESCRIPTIONS
Data File:      product_descriptions.dat
  Bad File:     product_descriptions.bad
  Discard File:  none specified

 (Allow all discards)

Number to load: ALL
Number to skip: 0
Errors allowed: 50
Continuation:    none specified
Path used:      Direct

Table PRODUCT_DESCRIPTIONS, loaded from every logical record.
Insert option in effect for this table: APPEND

   Column Name                  Position   Len  Term Encl Datatype
------------------------------ ---------- ----- ---- ---- ---------------------
PRODUCT_ID                          FIRST     *   ,       CHARACTER
LANGUAGE_ID                          NEXT     *   ,       CHARACTER
TRANSLATED_NAME                      NEXT     *   ,       CHARACTER
TRANSLATED_DESCRIPTION               NEXT     *   ,       CHARACTER

Generated control file for possible reuse:
OPTIONS(DIRECT=TRUE)
LOAD DATA
INFILE 'product_descriptions'
APPEND
INTO TABLE PRODUCT_DESCRIPTIONS
FIELDS TERMINATED BY ","
(
  PRODUCT_ID,
  LANGUAGE_ID,
  TRANSLATED_NAME,
  TRANSLATED_DESCRIPTION
)
End of generated control file for possible reuse.


Table PRODUCT_DESCRIPTIONS:
  8 Rows successfully loaded.
  0 Rows not loaded due to data errors.
  0 Rows not loaded because all WHEN clauses were failed.
  0 Rows not loaded because all fields were null.

Bind array size not used in direct path.
Column array  rows :    5000
Stream buffer bytes:  256000
Read   buffer bytes: 1048576

Total logical records skipped:          0
Total logical records read:             8
Total logical records rejected:         0
Total logical records discarded:        0
Total stream buffers loaded by SQL*Loader main thread:        1
Total stream buffers loaded by SQL*Loader load thread:        0

Run began on Wed Feb 05 00:52:12 2020
Run ended on Wed Feb 05 00:52:14 2020

Elapsed time was:     00:00:01.55
CPU time was:         00:00:00.06

SQL> SELECT count(*) FROM PRODUCT_DESCRIPTIONS WHERE product_id > 4000;

  COUNT(*)
----------
	 8


SQL> create table INVENTORIES (warehouse_id  varchar(255),
product_id  varchar(255),
  3  quantity_on_hand  varchar(255), remark varchar(255));

Table created.

SQL> !sqlldr userid=oe/oracle_4u@emrep control=lab_16_02_02.ctl log=lab_16_02_02.log data=lab_16_02_02.dat

SQL*Loader: Release 12.2.0.1.0 - Production on Wed Feb 5 00:56:04 2020

Copyright (c) 1982, 2017, Oracle and/or its affiliates.  All rights reserved.

Path used:      Conventional
Commit point reached - logical record count 64
Commit point reached - logical record count 83

Table OE.INVENTORIES:
  83 Rows successfully loaded.

Check the log file:
  lab_16_02_02.log
for more information about the load.

SQL> !cat lab_16_02_02.log

SQL*Loader: Release 12.2.0.1.0 - Production on Wed Feb 5 00:56:04 2020

Copyright (c) 1982, 2017, Oracle and/or its affiliates.  All rights reserved.

Control File:   lab_16_02_02.ctl
Data File:      lab_16_02_02.dat
  Bad File:     lab_16_02_02.bad
  Discard File:  none specified

 (Allow all discards)

Number to load: ALL
Number to skip: 0
Errors allowed: 50
Bind array:     64 rows, maximum of 256000 bytes
Continuation:    none specified
Path used:      Conventional

Table OE.INVENTORIES, loaded from every logical record.
Insert option in effect for this table: APPEND

   Column Name                  Position   Len  Term Encl Datatype
------------------------------ ---------- ----- ---- ---- ---------------------
WAREHOUSE_ID                        FIRST     *   ,       CHARACTER
PRODUCT_ID                           NEXT     *   ,       CHARACTER
QUANTITY_ON_HAND                     NEXT     *   ,       CHARACTER


Table OE.INVENTORIES:
  83 Rows successfully loaded.
  0 Rows not loaded due to data errors.
  0 Rows not loaded because all WHEN clauses were failed.
  0 Rows not loaded because all fields were null.


Space allocated for bind array:                  49536 bytes(64 rows)
Read   buffer bytes: 1048576

Total logical records skipped:          0
Total logical records read:            83
Total logical records rejected:         0
Total logical records discarded:        0

Run began on Wed Feb 05 00:56:04 2020
Run ended on Wed Feb 05 00:56:05 2020

Elapsed time was:     00:00:00.64
CPU time was:         00:00:00.06

SQL> SELECT count(*) FROM inventories WHERE quantity_on_hand = 7 AND    WAREHOUSE_ID>500 ;

  COUNT(*)
----------
	83
```



### KnowledgePoint

## 总结

### Oracle Data Pump概述

Oracle数据泵技术支持将数据和元数据从一个数据库高速迁移到另一个数据库。

对以下主题的理解可以帮助您成功地充分利用Oracle Data Pump，以发挥最大的优势：

- [数据泵组件](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-data-pump-overview.html#GUID-47B26B0B-3C95-4182-ACDF-2EEDD577FC9E)
  Oracle数据泵由三个不同的组件组成。它们是命令行客户端，`expdp`和`impdp`，`DBMS_DATAPUMP```PL / SQL包（也称为数据泵API）和`DBMS_METADATA`PL / SQL包（也称为元数据API）。
- [数据泵如何移动数据？](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-data-pump-overview.html#GUID-3F418F02-5FE2-455A-B5AD-C1910DB3B5E0)
  这是Data Pump用来将数据移入和移出数据库的方法，以及使用每种方法时的方法。
- [将数据泵与CDB一起使用](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-data-pump-overview.html#GUID-45B17B65-20F2-4128-9A39-B1B0F5E323BB)
  Data Pump可以将数据库的全部或部分从非CDB迁移到PDB，在相同或不同CDB内的PDB之间以及从PDB迁移到非CDB。
- [数据泵导出和导入操作所需的角色](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-data-pump-overview.html#GUID-8B6975D3-3BEC-4584-B416-280125EEC57E)
  许多数据泵导出和导入操作都要求用户同时具有一个`DATAPUMP_EXP_FULL_DATABASE`或多个`DATAPUMP_IMP_FULL_DATABASE`角色。
- [在执行数据泵作业期间会发生什么？](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-data-pump-overview.html#GUID-6BDC1CC8-8596-402D-B016-602985B97AB6)
  数据泵作业使用主表，主进程和工作进程执行工作并跟踪进度。
- [监视作业状态](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-data-pump-overview.html#GUID-E365D74E-12CD-495C-BA23-5A55F679C7E7)
  Data Pump导出和导入客户端实用程序可以以日志记录方式或交互式命令方式附加到作业。
- [文件分配](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-data-pump-overview.html#GUID-FE7E746D-A343-463E-A4E2-A6AD1349FE4B)
  了解Data Pump如何分配和处理文件将帮助您充分利用“导出”和“导入”。
- [在不同数据库版本之间进行导出和导入](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-data-pump-overview.html#GUID-BAA3B679-A758-4D55-9820-432D9EB83C68)
  Data Pump可用于在数据库软件的不同版本之间迁移数据库的全部或任何部分。
- [SecureFiles LOB注意事项](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-data-pump-overview.html#GUID-9030BC32-193B-4455-8DBB-4271DD44FA7A)
  当您使用数据泵导出来导出SecureFiles LOB时，产生的行为取决于几件事，包括Export `VERSION`参数的值，是否存在ContentType以及LOB是否已归档和是否缓存了数据。
- [数据泵退出代码](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-data-pump-overview.html#GUID-34D0DEE7-3530-42DC-BE01-C2588CC73CE5)
  数据泵在日志文件和进程退出代码中报告导出和导入操作的结果。
- [审核数据泵作业](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-data-pump-overview.html#GUID-4443B80B-0446-4010-B8CA-2524659516BC)
  对数据泵作业执行审核，以监视和记录特定的用户数据库操作。
- [数据泵如何处理时间戳数据？](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-data-pump-overview.html#GUID-7EE60BAB-0D3D-46EB-8E11-4FDDA68EF14E)
  本部分描述了可能影响成功完成涉及时间戳数据类型`TIMESTAMP WITH TIMEZONE`和的导出和导入作业的因素`TIMESTAMP WITH LOCAL TIMEZONE`。
- [字符集和全球化支持注意事项](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-data-pump-overview.html#GUID-72F01BF0-61F5-4775-8C7B-4E227F244866)
  数据泵导出和导入的全球化支持行为。
- [具有数据绑定排序规则的Oracle Data Pump行为](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-data-pump-overview.html#GUID-B1A6BBA2-0269-48CC-8A0E-8E3955A231C0)
  Oracle Data Pump支持数据绑定排序规则（DBC）。

### SQL * Loader

以下是有关SQL * Loader实用程序的主题。

[SQL * Loader概念](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-sql-loader-concepts.html#GUID-DD843EE2-1FAB-4E72-A115-21D97A501ECC)

描述SQL * Loader及其功能，以及数据加载概念（包括对象支持）。它讨论了SQL * Loader的输入，数据库准备以及SQL * Loader的输出。

[SQL * Loader命令行参考](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-sql-loader-commands.html#GUID-CD662CD8-DAA7-4A30-BC84-546E4C40DB31)

描述SQL * Loader使用的命令行语法。它讨论了命令行参数，抑制SQL * Loader消息，调整绑定数组的大小等等。

[SQL * Loader控制文件参考](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-sql-loader-control-file-contents.html#GUID-7F8983A0-CA5D-41D9-A096-CB1858CEDB4C)

描述用于配置SQL * Loader并向SQL * Loader描述如何将数据映射到Oracle格式的控制文件语法。它提供了详细的语法图以及有关指定数据文件，表和列，数据的位置，要加载的数据的类型和格式等信息。

[SQL * Loader字段列表参考](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-sql-loader-field-list-contents.html#GUID-DB309002-461D-42F7-8C94-727B32FA8B85)

描述SQL * Loader控制文件的字段列表部分。字段列表提供有关正在加载的字段的信息，例如位置，数据类型，条件和定界符。

[加载对象，LOB和集合](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/loading-objects-using-oracle-sql-loader.html#GUID-A1828462-FD32-457C-976F-C85BA3A995DA)

描述如何加载各种格式的列对象。它还讨论了如何加载对象表，`REF`列，LOB和集合。

[常规和直接路径载荷](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-sql-loader-conventional-and-direct-loads.html#GUID-321928FB-C86C-4F1F-9250-05111A988B7B)

描述常规路径负载和直接路径负载之间的差异。直接路径加载是一种高性能选项，可显着减少加载大量数据所需的时间。

[SQL * Loader Express](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-sql-loader-express-mode.html#GUID-8C235861-2A8B-4196-9705-E6FFED0C0C99)

描述SQL * Loader表示模式，它使您可以快速轻松地装入简单数据类型。

- [SQL * Loader概念](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-sql-loader-concepts.html#GUID-DD843EE2-1FAB-4E72-A115-21D97A501ECC)
  这些部分描述SQL * Loader概念。
- [SQL * Loader命令行参考](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-sql-loader-commands.html#GUID-CD662CD8-DAA7-4A30-BC84-546E4C40DB31)
  您可以使用命令行参数来启动SQL * Loader。
- [SQL * Loader控制文件参考](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-sql-loader-control-file-contents.html#GUID-7F8983A0-CA5D-41D9-A096-CB1858CEDB4C)
  SQL * Loader控制文件是一个文本文件，其中包含SQL * Loader作业的数据定义语言（DDL）指令。
- [SQL * Loader字段列表参考](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-sql-loader-field-list-contents.html#GUID-DB309002-461D-42F7-8C94-727B32FA8B85)
  SQL * Loader控制文件的字段列表部分提供有关要加载的字段的信息，例如位置，数据类型，条件和定界符。
- [加载对象，LOB和集合](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/loading-objects-using-oracle-sql-loader.html#GUID-A1828462-FD32-457C-976F-C85BA3A995DA)
  您可以使用SQL * Loader加载各种格式的列对象，并加载对象表，REF列，LOB和集合。
- [常规和直接路径载荷](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-sql-loader-conventional-and-direct-loads.html#GUID-321928FB-C86C-4F1F-9250-05111A988B7B)
- [SQL * Loader Express](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/sutil/oracle-sql-loader-express-mode.html#GUID-8C235861-2A8B-4196-9705-E6FFED0C0C99)
