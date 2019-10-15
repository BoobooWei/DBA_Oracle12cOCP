# 02-CreateDatabase

> 2019.09.22 

[TOC]

## 讲义

[Database Configuration Assistant (DBCA) : Creating Databases in Silent Mode](https://oracle-base.com/articles/misc/database-configuration-assistant-dbca-silent-mode)

[ALTER PLUGGABLE DATABASE](https://docs.oracle.com/database/121/SQLRF/statements_2008.htm#SQLRF55667)

## 注意点

### dbca静默安装-不使用容器 


安装数据库booboo

```bash
dbca -silent -createDatabase -templateName General_Purpose.dbc -gdbName booboo -sid booboo -sysPassword oracle -systemPassword oracle -datafileDestination /u01/app/oracle/oradata -characterSet we8mswin1252 -nationalCharacterSet al16utf16 -responseFile NO_VALUE
```

卸载数据库

```bash
dbca -silent -deleteDatabase -sourceDB booboo -sysDBAUserName sys -sysDBAPassword oracle
```


### dbca图形化安装-使用容器

直接执行dbca进行图形化安装，我在安装的过程中选择了4个pdb，前缀为booboo。

1. 查看当前所有的pdbs
2. 关闭除了booboopdb1之外的所有pdb
3. 保存当前pdb的状态，让下次数据库启动的时候只开启booboopdb1
4. 切换session到booboopdb1

```sql
show pdbs
alter pluggable database booboopdb4 close;
alter pluggable database all save state;
show pdbs
alter session set container=booboopdb1;
show pdbs
```

操作记录

```bash
SQL> show parameter name

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
cdb_cluster_name		     string	 booboo
cell_offloadgroup_name		     string
db_file_name_convert		     string
db_name 			     string	 booboo
db_unique_name			     string	 booboo
global_names			     boolean	 FALSE
instance_name			     string	 booboo
lock_name_space 		     string
log_file_name_convert		     string
pdb_file_name_convert		     string
processor_group_name		     string

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
service_names			     string	 booboo
SQL> show pdbs

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 2 PDB$SEED			  READ ONLY  NO
	 3 BOOBOOPDB1			  READ WRITE NO
	 4 BOOBOOPDB2			  READ WRITE NO
	 5 BOOBOOPDB3			  READ WRITE NO
	 6 BOOBOOPDB4			  READ WRITE NO
	 
SQL> alter pluggable database booboopdb4 close;

Pluggable database altered.

SQL> alter pluggable database booboopdb3 close;

Pluggable database altered.

SQL> alter pluggable database booboopdb2 close;

Pluggable database altered.

SQL> alter pluggable database all save state;

Pluggable database altered.




SQL> show pdbs

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 2 PDB$SEED			  READ ONLY  NO
	 3 BOOBOOPDB1			  READ WRITE NO
	 4 BOOBOOPDB2			  MOUNTED
	 5 BOOBOOPDB3			  MOUNTED
	 6 BOOBOOPDB4			  MOUNTED

SQL> alter session set container=booboopdb1;

Session altered.

SQL> show pdbs

    CON_ID CON_NAME			  OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
	 3 BOOBOOPDB1			  READ WRITE NO
```

