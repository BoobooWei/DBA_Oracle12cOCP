# 实践4:管理实例

> **Practices for Lesson 4: Managing the Database Instance**
>
> 2020.01.29 BoobooWei

[toc]

## 实践4:概览

Practices for Lesson 4: Overview

**Background:** The Oracle software has been installed and a database has been created. You want to ensure that you can start and stop the database instance and see the application data.

背景:已安装Oracle软件，并创建了数据库。您希望确保能够启动和停止数据库实例并查看应用程序数据。

## 实践4-1:使用Oracle Enterprise Manager Cloud Control管理Oracle实例

Practice 4-1: Managing the Oracle Instance by Using Oracle Enterprise Manager Cloud Control

## 实践4-2:使用Oracle Enterprise Manger Database Express管理Oracle实例

Practice 4-2: Managing the Oracle Instance by Using Oracle Enterprise Manger Database Express

## 实践4-3:使用SQL*Plus管理Oracle实例

Practice 4-3: Managing the Oracle Instance by Using SQL*Plus

### Overview

In this practice, you use SQL*Plus to view and change instance parameters.

### Tasks

1. Optimize SQL*Plus：configuring `$ORACLE_HOME/sqlplus/admin/glogin.sql`
2. Set the JOB_QUEUE_PROCESSES initialization parameter to 1000 by using SQL*Plus.
3. Using SQL*Plus, shut down and restart the orcl database instance. 
4. Use the **SHOW PARAMETER** command to verify the settings for **SGA_MAX_SIZE**, **DB_CACHE_SIZE**, and **SHARED_POOL_SIZE**.
5. Check the value of **JOB_QUEUE_PROCESSES**.
6. Exit from SQL*Plus.

### Practice

1. 优化 SQL*Plus：配置 `$ORACLE_HOME/sqlplus/admin/glogin.sql`

	```bash
	cat > $ORACLE_HOME/sqlplus/admin/glogin.sql << ENDF
	define _editor=vi
	set serveroutput on size 1000000
	set trimspool on
	set long 5000
	set linesize 100
	set pagesize 9999
	column plan_plus_exp format a80
	set sqlprompt '&_user.@&_connect_identifier.>'
	ENDF
	```

2. 设置 JOB_QUEUE_PROCESSES 的初始化参数值为1000

   ```sql
   sqlplus / as sysdba
   SHOW PARAMETER job
   ALTER SYSTEM SET job_queue_processes=1000 SCOPE=BOTH;
   SHOW PARAMETER job
   ```

3. 使用 SQL*Plus 关闭并重启orcl实例

   ```sql
   shutdown immediate 
   startup
   ```

4. 使用 **SHOW PARAMETER** 命令查看参数 **SGA_MAX_SIZE**, **DB_CACHE_SIZE**, **SHARED_POOL_SIZE** 的值

   ```sql
   show parameter sga_max_size
   show parameter db_cache_size
   show parameter shared_pool_size
   ```

5. 检查 **JOB_QUEUE_PROCESSES** 参数的值

   ```sql
   show parameter job_queue_processes
   ```

6. 退出 SQL*Plus 

   ```sql
   exit
   ```


### KnowledgePoint

[11g 参数文件](https://github.com/BoobooWei/booboo_oracle/blob/master/D-体系结构和存储引擎-03-物理结构_参数文件parameter_files.md) 

[12c 参数文件]( https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/creating-and-configuring-an-oracle-database.html#GUID-7302C60F-E96E-4202-AC81-25A6C93EEFA3  )

- [*《 Oracle数据库SQL语言参考》*](https://www.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/12.2/admin&id=SQLRF00902)中有关该`ALTER` `SYSTEM`命令的 信息
-  [在CDB中使用ALTER SYSTEM SET语句]( https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/administering-a-cdb-with-sql-plus.html#GUID-E47348D6-5350-4890-ACD6-7BA1C1DD4E95 ) 

#### 12c 参数文件 与 11g 参数文件的不同之处

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

#### 动态参数修改

```sql
-- pdb中查看当前动态参数open_cursors
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

#### 静态参数修改

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

#### 查看参数

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

## 实践4-4:通过使用自动诊断存储库命令查看警报日志接口(ADRCI)
Practice 4-4: Viewing the Alert Log by Using the Automatic Diagnostic Repository Command

Interface (ADRCI)

### Overview

In this practice, you use command-line tools to view the orcl instance alert log and find the startup phases.

### Tasks

1. In the alert log, view the phases that the database went through during startup. What are they?
2. Scroll through the log and review the phases of the database during startup. Use the vi search commands to find the appropriate lines. Your alert log may differ from what is shown in this practice.

### Practice

1. 在警报日志中，查看数据库在启动期间经历的阶段。

   ```bash
   [oracle@oracle01 ~]$ adrci
   
   ADRCI: Release 12.2.0.1.0 - Production on Thu Jan 30 02:27:35 2020
   
   Copyright (c) 1982, 2017, Oracle and/or its affiliates.  All rights reserved.
   
   ADR base = "/u01/app/oracle"
   adrci> show alert
   
   Choose the home from which to view the alert log:
   
   1: diag/rdbms/booboo/booboo
   2: diag/clients/user_oracle/host_2874269298_107
   3: diag/tnslsnr/oracle01/listener
   Q: to quit
   
   Please select option: 1
   Output the results to file: /tmp/alert_88137_1406_booboo_1.ado
   ```

   **Note: 默认会使用vi命令打开alert文件。**

2. 滚动日志并在启动期间检查数据库的各个阶段。使用vi搜索命令查找适当的行。您的警报日志可能与此实践中显示的有所不同。

   ```bash
   2019-09-22 02:03:08.120000 -07:00
   Starting ORACLE instance (normal) (OS id: 71748)
   ......
   alter database "booboo" open resetlogs
   ......
   2020-01-29 18:00:00.047000 +08:00
   Closing scheduler window
   Closing Resource Manager plan via scheduler window
   Clearing Resource Manager CDB plan via parameter
   2020-01-29 23:08:35.821000 +08:00
   Resize operation completed for file# 3, old size 563200K, new size 573440K
   
   --输入:q 退出
   --退出vi后继续进入adrci的交互界面
   --输入Q
   Please select option: Q
   adrci> exit
   ```

### KnowledgePoint

[自动诊断存储库命令ADRCI](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/diagnosing-and-resolving-problems.html#GUID-EA97EEF6-9207-4536-B808-3B91DACA7AD6)