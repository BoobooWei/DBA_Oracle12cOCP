# 07-Manage

> 2019.10.27 BoobooWei

[toc]

## 复习上一节课内容

1. 启动数据库实例
2. 查看数据库实例明细
3. 启动默认监听
4. 查看监听
5. 连接数据库服务

 ```bash
[oracle@oracle01 ~]$ sqlplus / as sysdba

SQL*Plus: Release 12.2.0.1.0 Production on Sat Oct 26 18:15:13 2019

Copyright (c) 1982, 2016, Oracle.  All rights reserved.

Connected to an idle instance.

SQL> startup
ORACLE instance started.

Total System Global Area  855638016 bytes
Fixed Size		    8798504 bytes
Variable Size		  327159512 bytes
Database Buffers	  515899392 bytes
Redo Buffers		    3780608 bytes
Database mounted.
Database opened.
SQL> show pdbs;

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 2 PDB$SEED			  READ ONLY  NO
	 3 BOOBOOPDB1			  READ WRITE NO
	 4 BOOBOOPDB2			  MOUNTED
	 5 BOOBOOPDB3			  MOUNTED
	 6 BOOBOOPDB4			  MOUNTED

[oracle@oracle01 ~]$ lsnrctl start

LSNRCTL for Linux: Version 12.2.0.1.0 - Production on 26-OCT-2019 18:45:28

Copyright (c) 1991, 2016, Oracle.  All rights reserved.

Starting /u01/app/oracle/product/12.2.0/db_1/bin/tnslsnr: please wait...

TNSLSNR for Linux: Version 12.2.0.1.0 - Production
System parameter file is /u01/app/oracle/product/12.2.0/db_1/network/admin/listener.ora
Log messages written to /u01/app/oracle/diag/tnslsnr/oracle01/listener/alert/log.xml
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=oracle01)(PORT=1521)))
Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=ol7-122.localdomain)(PORT=1521)))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 12.2.0.1.0 - Production
Start Date                26-OCT-2019 18:46:05
Uptime                    0 days 0 hr. 0 min. 55 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Listener Parameter File   /u01/app/oracle/product/12.2.0/db_1/network/admin/listener.ora
Listener Log File         /u01/app/oracle/diag/tnslsnr/oracle01/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=oracle01)(PORT=1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
The listener supports no services
The command completed successfully
[oracle@oracle01 ~]$ lsnrctl status

LSNRCTL for Linux: Version 12.2.0.1.0 - Production on 26-OCT-2019 18:48:33

Copyright (c) 1991, 2016, Oracle.  All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=ol7-122.localdomain)(PORT=1521)))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 12.2.0.1.0 - Production
Start Date                26-OCT-2019 18:46:05
Uptime                    0 days 0 hr. 3 min. 5 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Listener Parameter File   /u01/app/oracle/product/12.2.0/db_1/network/admin/listener.ora
Listener Log File         /u01/app/oracle/diag/tnslsnr/oracle01/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=oracle01)(PORT=1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcps)(HOST=oracle01)(PORT=5500))(Security=(my_wallet_directory=/u01/app/oracle/admin/booboo/xdb_wallet))(Presentation=HTTP)(Session=RAW))
Services Summary...
Service "93227d66515623b9e0553ce49b3eaf97" has 1 instance(s).
  Instance "booboo", status READY, has 1 handler(s) for this service...
Service "932283ef357f23fbe0553ce49b3eaf97" has 1 instance(s).
  Instance "booboo", status READY, has 1 handler(s) for this service...
Service "9322894c27292419e0553ce49b3eaf97" has 1 instance(s).
  Instance "booboo", status READY, has 1 handler(s) for this service...
Service "93228eb75d13244ce0553ce49b3eaf97" has 1 instance(s).
  Instance "booboo", status READY, has 1 handler(s) for this service...
Service "booboo" has 1 instance(s).
  Instance "booboo", status READY, has 1 handler(s) for this service...
Service "boobooXDB" has 1 instance(s).
  Instance "booboo", status READY, has 1 handler(s) for this service...
Service "booboopdb1" has 1 instance(s).
  Instance "booboo", status READY, has 1 handler(s) for this service...
Service "booboopdb2" has 1 instance(s).
  Instance "booboo", status READY, has 1 handler(s) for this service...
Service "booboopdb3" has 1 instance(s).
  Instance "booboo", status READY, has 1 handler(s) for this service...
Service "booboopdb4" has 1 instance(s).
  Instance "booboo", status READY, has 1 handler(s) for this service...
The command completed successfully


=======
[oracle@oracle01 admin]$ sqlplus hr/Oracle123@oracle01:1521/booboopdb1

SQL> column  tname format a20
SQL> select * from tab;

TNAME		     TABTYPE  CLUSTERID
-------------------- ------- ----------
COUNTRIES	     TABLE
DEPARTMENTS	     TABLE
EMPLOYEES	     TABLE
EMP_DETAILS_VIEW     VIEW
JOBS		     TABLE
JOB_HISTORY	     TABLE
LOCATIONS	     TABLE
REGIONS 	     TABLE

8 rows selected.

SQL> show user;
USER is "HR"
 ```



## 管理网络

老师初步介绍网络，比较浅，只需掌握：

1. 配置监听
2. 查看监听
3. 通过监听服务连接数据库

```bash
[oracle@oracle01 ~]$ cd $ORACLE_HOME/network/admin
[oracle@oracle01 admin]$ pwd
/u01/app/oracle/product/12.2.0/db_1/network/admin
[oracle@oracle01 admin]$ ll
total 16
-rw-r-----. 1 oracle oinstall  343 Sep 22 02:02 listener.ora
drwxr-xr-x. 2 oracle oinstall   64 Sep 21 23:58 samples
-rw-r--r--. 1 oracle oinstall 1441 Aug 28  2015 shrept.lst
-rw-r-----. 1 oracle oinstall  198 Sep 22 02:02 sqlnet.ora
-rw-r-----. 1 oracle oinstall  430 Sep 22 02:34 tnsnames.ora
[oracle@oracle01 admin]$ cat listener.ora 
# listener.ora Network Configuration File: /u01/app/oracle/product/12.2.0/db_1/network/admin/listener.ora
# Generated by Oracle configuration tools.

LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = ol7-122.localdomain)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )
```

Tips-监听名字`LISTENER`为何不建议修改？

```
查看监听的命令 lsnrctl status 默认查看的就是`LISTENER`；
如果改变名称，则需要使用 lsnrctl status yourname
```





##  系统视图

### 性能视图`v$fixed_table`

```sql
SQL> column Name format a30
SQL> select name from v$fixed_table where name like '%USER%';

NAME
------------------------------
X$LOGMNR_USER$
X$RO_USER_ACCOUNT
X$KFVACFSREALMUSER
X$DIAG_DDE_USER_ACTION_DEF
X$DIAG_DDE_USER_ACTION
X$DIAG_EM_USER_ACTIVITY
X$DIAG_VEM_USER_ACTLOG
X$DIAG_VEM_USER_ACTLOG1
GV$IM_USER_SEGMENTS
V$IM_USER_SEGMENTS
GV$PWFILE_USERS
V$PWFILE_USERS
GV$ASM_ACFS_SEC_REALM_USER
V$ASM_ACFS_SEC_REALM_USER
GV$ASM_USER
V$ASM_USER
GV$ASM_USERGROUP
V$ASM_USERGROUP
GV$ASM_USERGROUP_MEMBER
V$ASM_USERGROUP_MEMBER
GV$RO_USER_ACCOUNT
V$RO_USER_ACCOUNT

22 rows selected.
```



### 字典视图`dict`

```sql
SQL> column table_name format a100
SQL> select table_name from dict where rownum<10;

TABLE_NAME
----------------------------------------------------------------------------------------------------
CDB_2PC_NEIGHBORS
CDB_2PC_PENDING
CDB_ACL_NAME_MAP
CDB_ADDM_FDG_BREAKDOWN
CDB_ADDM_FINDINGS
CDB_ADDM_INSTANCES
CDB_ADDM_SYSTEM_DIRECTIVES
CDB_ADDM_TASKS
CDB_ADDM_TASK_DIRECTIVES

9 rows selected.
```

### 表视图 `tab`

```sql
SQL> column TNAME format a20
SQL> select * from tab;

TNAME		     TABTYPE  CLUSTERID
-------------------- ------- ----------
BOOBOO		     SYNONYM
COUNTRIES	     TABLE
DEPARTMENTS	     TABLE
EMPLOYEES	     TABLE
EMP_DETAILS_VIEW     VIEW
JOBS		     TABLE
JOB_HISTORY	     TABLE
LOCATIONS	     TABLE
REGIONS 	     TABLE

9 rows selected.

```



## 参数文件

[11g 参数文件](https://github.com/BoobooWei/booboo_oracle/blob/master/D-体系结构和存储引擎-03-物理结构_参数文件parameter_files.md) 

[12c 参数文件]( https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/creating-and-configuring-an-oracle-database.html#GUID-7302C60F-E96E-4202-AC81-25A6C93EEFA3  )

- [*《 Oracle数据库SQL语言参考》*](https://www.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/12.2/admin&id=SQLRF00902)中有关该`ALTER` `SYSTEM`命令的 信息
-  [在CDB中使用ALTER SYSTEM SET语句]( https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/administering-a-cdb-with-sql-plus.html#GUID-E47348D6-5350-4890-ACD6-7BA1C1DD4E95 ) 

12c 参数文件 与 11g 参数文件的不同之处

```bash
1. pdb 继承 cdb的参数
2. pdb 参数可以变更，保存在CDB级别的 pdb_spfile$ 系统表中，对参数文件没有影响
3. pdb 参数哪些可以变更，当 ISPDB_MODIFIABLE 列TRUE用于V$SYSTEM_PARAMETER视图中的参数时，可以修改PDB的初始化参数。
4. 文本初始化参数文件（PFILE）不能包含特定于PDB的参数值。

-- 查询可被PDB修改的所有初始化参数：
SELECT NAME FROM V$SYSTEM_PARAMETER WHERE ISPDB_MODIFIABLE='TRUE' ORDER BY NAME;

-- 可被动态修改 且 pdb 中可被覆盖的 参数
SELECT NAME,ISPDB_MODIFIABLE,ISSYS_MODIFIABLE FROM V$SYSTEM_PARAMETER WHERE ISPDB_MODIFIABLE='TRUE' and ISSYS_MODIFIABLE<>'FALSE' ORDER BY NAME;

-- 不可动态修改 且 pdb 中可被覆盖的 参数
SELECT NAME,ISPDB_MODIFIABLE,ISSYS_MODIFIABLE FROM V$SYSTEM_PARAMETER WHERE ISPDB_MODIFIABLE='TRUE' and ISSYS_MODIFIABLE='FALSE' ORDER BY NAME;
```

### 动态参数修改

```sql
-- pdb中查看当前动态参数memory_max_target
SQL> select num,name,type,value,isdefault,ISBASIC,ISPDB_MODIFIABLE from v$system_parameter where name='open_cursors';

       NUM NAME 		      TYPE VALUE		ISDEFAULT ISBAS
---------- -------------------- ---------- -------------------- --------- -----
ISPDB
-----
      3328 open_cursors 		 3 300			FALSE	  TRUE
TRUE

-- 可以在pdb中直接修改
SQL> alter system set open_cursors=500 scope=both;

System altered.

SQL> show parameter open_cursors;

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
open_cursors			     integer	 500
SQL> select num,name,type,value,isdefault,ISBASIC,ISPDB_MODIFIABLE from v$parameter where name='open_cursors';

       NUM NAME 		      TYPE VALUE		ISDEFAULT ISBAS
---------- -------------------- ---------- -------------------- --------- -----
ISPDB
-----
      3328 open_cursors 		 3 500			FALSE	  TRUE
TRUE


SQL> select num,name,type,value,isdefault,ISBASIC,ISPDB_MODIFIABLE from v$system_parameter where name='open_cursors';

       NUM NAME 		      TYPE VALUE		ISDEFAULT ISBAS
---------- -------------------- ---------- -------------------- --------- -----
ISPDB
-----
      3328 open_cursors 		 3 500			FALSE	  TRUE
TRUE
```



### 静态参数修改

```sql
-- pdb中查看当前静态参数memory_max_target
SQL> show pdbs

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 3 BOOBOOPDB1			  READ WRITE NO

SQL> column name format a20
SQL> column value format a20
SQL> select NAME,VALUE,ISSYS_MODIFIABLE from v$parameter where name='memory_max_target';

NAME		     VALUE		  ISSYS_MOD
-------------------- -------------------- ---------
memory_max_target    0			  FALSE

-- 查看memory_max_target的ISPDB_MODIFIABLE的值为FALSE，代表pdb中不可以修改
SQL> select num,name,type,value,isdefault,ISBASIC,ISPDB_MODIFIABLE from v$system_parameter where name='memory_max_target';

       NUM NAME 		      TYPE VALUE		ISDEFAULT ISBAS
---------- -------------------- ---------- -------------------- --------- -----
ISPDB
-----
      1376 memory_max_target		 6 0			TRUE	  FALSE
FALSE

-- pdb中修改该参数值
SQL> alter system set memory_max_target=1g scope=spfile;
alter system set memory_max_target=1g scope=spfile
*
ERROR at line 1:
ORA-65040: operation not allowed from within a pluggable database


-- pdb中修改open_links
SQL> select num,name,type,value,isdefault,ISBASIC,ISPDB_MODIFIABLE from v$system_parameter where name='open_links';

       NUM NAME 						    TYPE VALUE		      ISDEFAULT ISBAS ISPDB
---------- -------------------------------------------------- ---------- -------------------- --------- ----- -----
      3288 open_links						       3 4		      TRUE	FALSE TRUE

SQL> alter system set open_links=10 scope=spfile;

System altered.

-- cdb中查看pdb_spfile$表中记录的参数值
SQL> select name,value$ from pdb_spfile$ where name='open_cursors';

NAME		     VALUE$
-------------------- --------------------
open_cursors	     500

-- 通过cdb，重启pdb后生效
SQL> STARTUP PLUGGABLE DATABASE booboopdb1 FORCE
Pluggable Database opened.

-- pdb中查看参数值
SQL> show parameter open_cursors

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
open_cursors			     integer	 500


--文本初始化参数文件（PFILE）不能包含特定于PDB的参数值。
```

### 查看参数

```sql
SQL> SELECT NAME,ISPDB_MODIFIABLE,ISSYS_MODIFIABLE FROM V$SYSTEM_PARAMETER WHERE ISPDB_MODIFIABLE='TRUE' and ISSYS_MODIFIABLE<>'FALSE' ORDER BY NAME;

NAME						   ISPDB ISSYS_MOD
-------------------------------------------------- ----- ---------
approx_for_aggregation				   TRUE  IMMEDIATE
approx_for_count_distinct			   TRUE  IMMEDIATE
approx_for_percentile				   TRUE  IMMEDIATE
asm_diskstring					   TRUE  IMMEDIATE
awr_pdb_autoflush_enabled			   TRUE  IMMEDIATE
cell_offload_compaction 			   TRUE  IMMEDIATE
cell_offload_decryption 			   TRUE  IMMEDIATE
cell_offload_parameters 			   TRUE  IMMEDIATE
cell_offload_plan_display			   TRUE  IMMEDIATE
cell_offload_processing 			   TRUE  IMMEDIATE
cell_offloadgroup_name				   TRUE  IMMEDIATE
commit_logging					   TRUE  IMMEDIATE
commit_wait					   TRUE  IMMEDIATE
commit_write					   TRUE  IMMEDIATE
containers_parallel_degree			   TRUE  IMMEDIATE
cpu_count					   TRUE  IMMEDIATE
create_stored_outlines				   TRUE  IMMEDIATE
cursor_bind_capture_destination 		   TRUE  IMMEDIATE
cursor_invalidation				   TRUE  IMMEDIATE
cursor_sharing					   TRUE  IMMEDIATE
db_block_checking				   TRUE  IMMEDIATE
db_cache_size					   TRUE  IMMEDIATE
db_create_file_dest				   TRUE  IMMEDIATE
db_create_online_log_dest_1			   TRUE  IMMEDIATE
db_create_online_log_dest_2			   TRUE  IMMEDIATE
db_create_online_log_dest_3			   TRUE  IMMEDIATE
db_create_online_log_dest_4			   TRUE  IMMEDIATE
db_create_online_log_dest_5			   TRUE  IMMEDIATE
db_file_multiblock_read_count			   TRUE  IMMEDIATE
db_index_compression_inheritance		   TRUE  IMMEDIATE
db_securefile					   TRUE  IMMEDIATE
db_unrecoverable_scn_tracking			   TRUE  IMMEDIATE
ddl_lock_timeout				   TRUE  IMMEDIATE
default_sharing 				   TRUE  IMMEDIATE
deferred_segment_creation			   TRUE  IMMEDIATE
dst_upgrade_insert_conv 			   TRUE  IMMEDIATE
enable_automatic_maintenance_pdb		   TRUE  IMMEDIATE
enable_ddl_logging				   TRUE  IMMEDIATE
encrypt_new_tablespaces 			   TRUE  IMMEDIATE
fixed_date					   TRUE  IMMEDIATE
global_names					   TRUE  IMMEDIATE
heat_map					   TRUE  IMMEDIATE
inmemory_clause_default 			   TRUE  IMMEDIATE
inmemory_expressions_usage			   TRUE  IMMEDIATE
inmemory_force					   TRUE  IMMEDIATE
inmemory_query					   TRUE  IMMEDIATE
inmemory_size					   TRUE  IMMEDIATE
inmemory_virtual_columns			   TRUE  IMMEDIATE
java_jit_enabled				   TRUE  IMMEDIATE
job_queue_processes				   TRUE  IMMEDIATE
listener_networks				   TRUE  IMMEDIATE
local_listener					   TRUE  IMMEDIATE
log_archive_min_succeed_dest			   TRUE  IMMEDIATE
long_module_action				   TRUE  IMMEDIATE
max_datapump_jobs_per_pdb			   TRUE  IMMEDIATE
max_dump_file_size				   TRUE  IMMEDIATE
max_idle_time					   TRUE  IMMEDIATE
max_iops					   TRUE  IMMEDIATE
max_mbps					   TRUE  IMMEDIATE
max_pdbs					   TRUE  IMMEDIATE
max_string_size 				   TRUE  IMMEDIATE
nls_length_semantics				   TRUE  IMMEDIATE
nls_nchar_conv_excp				   TRUE  IMMEDIATE
object_cache_max_size_percent			   TRUE  DEFERRED
object_cache_optimal_size			   TRUE  DEFERRED
olap_page_pool_size				   TRUE  DEFERRED
open_cursors					   TRUE  IMMEDIATE
optimizer_adaptive_plans			   TRUE  IMMEDIATE
optimizer_adaptive_reporting_only		   TRUE  IMMEDIATE
optimizer_adaptive_statistics			   TRUE  IMMEDIATE
optimizer_capture_sql_plan_baselines		   TRUE  IMMEDIATE
optimizer_dynamic_sampling			   TRUE  IMMEDIATE
optimizer_features_enable			   TRUE  IMMEDIATE
optimizer_index_caching 			   TRUE  IMMEDIATE
optimizer_index_cost_adj			   TRUE  IMMEDIATE
optimizer_inmemory_aware			   TRUE  IMMEDIATE
optimizer_mode					   TRUE  IMMEDIATE
optimizer_secure_view_merging			   TRUE  IMMEDIATE
optimizer_use_invisible_indexes 		   TRUE  IMMEDIATE
optimizer_use_pending_statistics		   TRUE  IMMEDIATE
optimizer_use_sql_plan_baselines		   TRUE  IMMEDIATE
parallel_degree_limit				   TRUE  IMMEDIATE
parallel_degree_policy				   TRUE  IMMEDIATE
parallel_force_local				   TRUE  IMMEDIATE
parallel_instance_group 			   TRUE  IMMEDIATE
parallel_max_servers				   TRUE  IMMEDIATE
parallel_min_time_threshold			   TRUE  IMMEDIATE
pdb_file_name_convert				   TRUE  IMMEDIATE
pdb_lockdown					   TRUE  IMMEDIATE
pga_aggregate_limit				   TRUE  IMMEDIATE
pga_aggregate_target				   TRUE  IMMEDIATE
plscope_settings				   TRUE  IMMEDIATE
plsql_ccflags					   TRUE  IMMEDIATE
plsql_code_type 				   TRUE  IMMEDIATE
plsql_debug					   TRUE  IMMEDIATE
plsql_optimize_level				   TRUE  IMMEDIATE
plsql_v2_compatibility				   TRUE  IMMEDIATE
plsql_warnings					   TRUE  IMMEDIATE
query_rewrite_enabled				   TRUE  IMMEDIATE
query_rewrite_integrity 			   TRUE  IMMEDIATE
recyclebin					   TRUE  DEFERRED
remote_dependencies_mode			   TRUE  IMMEDIATE
remote_listener 				   TRUE  IMMEDIATE
remote_recovery_file_dest			   TRUE  IMMEDIATE
resource_limit					   TRUE  IMMEDIATE
resource_manager_plan				   TRUE  IMMEDIATE
result_cache_mode				   TRUE  IMMEDIATE
result_cache_remote_expiration			   TRUE  IMMEDIATE
resumable_timeout				   TRUE  IMMEDIATE
session_cached_cursors				   TRUE  DEFERRED
sessions					   TRUE  IMMEDIATE
sga_min_size					   TRUE  IMMEDIATE
sga_target					   TRUE  IMMEDIATE
shadow_core_dump				   TRUE  IMMEDIATE
shared_pool_size				   TRUE  IMMEDIATE
shared_servers					   TRUE  IMMEDIATE
skip_unusable_indexes				   TRUE  IMMEDIATE
smtp_out_server 				   TRUE  IMMEDIATE
sort_area_retained_size 			   TRUE  DEFERRED
sort_area_size					   TRUE  DEFERRED
spatial_vector_acceleration			   TRUE  IMMEDIATE
sql_trace					   TRUE  IMMEDIATE
sqltune_category				   TRUE  IMMEDIATE
star_transformation_enabled			   TRUE  IMMEDIATE
statistics_level				   TRUE  IMMEDIATE
temp_undo_enabled				   TRUE  IMMEDIATE
timed_os_statistics				   TRUE  IMMEDIATE
timed_statistics				   TRUE  IMMEDIATE
undo_retention					   TRUE  IMMEDIATE
undo_tablespace 				   TRUE  IMMEDIATE
workarea_size_policy				   TRUE  IMMEDIATE
xml_db_events					   TRUE  IMMEDIATE

132 rows selected.

SQL> SELECT NAME,ISPDB_MODIFIABLE,ISSYS_MODIFIABLE FROM V$SYSTEM_PARAMETER WHERE ISPDB_MODIFIABLE='TRUE' and ISSYS_MODIFIABLE='FALSE' ORDER BY NAME;

NAME						   ISPDB ISSYS_MOD
-------------------------------------------------- ----- ---------
O7_DICTIONARY_ACCESSIBILITY			   TRUE  FALSE
blank_trimming					   TRUE  FALSE
commit_point_strength				   TRUE  FALSE
common_user_prefix				   TRUE  FALSE
db_domain					   TRUE  FALSE
db_files					   TRUE  FALSE
db_performance_profile				   TRUE  FALSE
nls_calendar					   TRUE  FALSE
nls_comp					   TRUE  FALSE
nls_currency					   TRUE  FALSE
nls_date_format 				   TRUE  FALSE
nls_date_language				   TRUE  FALSE
nls_dual_currency				   TRUE  FALSE
nls_iso_currency				   TRUE  FALSE
nls_language					   TRUE  FALSE
nls_numeric_characters				   TRUE  FALSE
nls_sort					   TRUE  FALSE
nls_territory					   TRUE  FALSE
nls_time_format 				   TRUE  FALSE
nls_time_tz_format				   TRUE  FALSE
nls_timestamp_format				   TRUE  FALSE
nls_timestamp_tz_format 			   TRUE  FALSE
open_links					   TRUE  FALSE
pdb_os_credential				   TRUE  FALSE
rollback_segments				   TRUE  FALSE
sql92_security					   TRUE  FALSE
undo_management 				   TRUE  FALSE
utl_file_dir					   TRUE  FALSE

28 rows selected.
```



[参考链接]( https://www.cnblogs.com/askscuti/p/10878906.html)

## 启动和关闭

[Starting Up and Shutting Down]( https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/starting-up-and-shutting-down.html#GUID-045DE684-6680-4099-A49E-2F5B5FA59670 )

## 跟踪文件

[Managing Diagnostic Data]( https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/managing-diagnostic-data.html#GUID-8DEB1BE0-8FB9-4FB2-A19A-17CF6F5791C3 )

## 课程目标

```
1. Start and stop the Oracle database instance and components
2. Modify database initialization parameters
3. Describe the stages of database startup
4. Describe database shutdown options 
5. View the alert log
6. Access dynamic performance views
```



### 1. 启动和停止数据库实例和组件

### 2. 修改数据库初始化参数

### 3. 描述数据库启动的阶段

### 4. 描述数据关闭的参数含义

### 5. 查看告警日志

### 6. 访问动态性能视图

