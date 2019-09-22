# 04-导入example数据

[TOC]

## 讲义

`6-12C-OCP-PPT教材（老师上课用）\061_SQL\D80174GC11_ep_PDF分载\D80190GC11_ep\presentation_material\Individual_PDFs\D80190GC11_les01.pdf`

Page 31


## 注意点

> 要进入pdb再导入数据

```bash
cd /u01/app/oracle/product/12.2.0/db_1/demo/schema/human_resources
sqlplus / as sysdba
SQL> alter session set container=booboopdb1;
SQL> @hr_main.sql
SQL> select table_name from dba_tables where owner='HR';
```

> sys用户连接booboopdb1库

```bash
sqlplus sys/oracle@127.0.0.1:1521/booboopdb1 as sysdba 

SQL> desc user_tables;
SQL>  select TABLE_NAME from user_tables;

TABLE_NAME
--------------------------------------------------------------------------------
COUNTRIES
REGIONS
LOCATIONS
DEPARTMENTS
JOBS
EMPLOYEES
JOB_HISTORY
```

> hr用户连接booboopdb1库

```bash
[oracle@oracle01 human_resources]$ sqlplus hr/Oracle123@127.0.0.1:1521/booboopdb1 

SQL*Plus: Release 12.2.0.1.0 Production on Sun Sep 22 04:57:12 2019

Copyright (c) 1982, 2016, Oracle.  All rights reserved.

Last Successful login time: Sun Sep 22 2019 04:23:41 -07:00

Connected to:
Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production

SQL> select * from jobs;

JOB_ID	   JOB_TITLE			       MIN_SALARY MAX_SALARY
---------- ----------------------------------- ---------- ----------
AD_PRES    President				    20080      40000
AD_VP	   Administration Vice President	    15000      30000
AD_ASST    Administration Assistant		     3000	6000
FI_MGR	   Finance Manager			     8200      16000
FI_ACCOUNT Accountant				     4200	9000
AC_MGR	   Accounting Manager			     8200      16000
AC_ACCOUNT Public Accountant			     4200	9000
SA_MAN	   Sales Manager			    10000      20080
SA_REP	   Sales Representative 		     6000      12008
PU_MAN	   Purchasing Manager			     8000      15000
PU_CLERK   Purchasing Clerk			     2500	5500

JOB_ID	   JOB_TITLE			       MIN_SALARY MAX_SALARY
---------- ----------------------------------- ---------- ----------
ST_MAN	   Stock Manager			     5500	8500
ST_CLERK   Stock Clerk				     2008	5000
SH_CLERK   Shipping Clerk			     2500	5500
IT_PROG    Programmer				     4000      10000
MK_MAN	   Marketing Manager			     9000      15000
MK_REP	   Marketing Representative		     4000	9000
HR_REP	   Human Resources Representative	     4000	9000
PR_REP	   Public Relations Representative	     4500      10500

19 rows selected.

SQL> select * from jobs;

JOB_ID	   JOB_TITLE			       MIN_SALARY MAX_SALARY
---------- ----------------------------------- ---------- ----------
AD_PRES    President				    20080      40000
AD_VP	   Administration Vice President	    15000      30000
AD_ASST    Administration Assistant		     3000	6000
FI_MGR	   Finance Manager			     8200      16000
FI_ACCOUNT Accountant				     4200	9000
AC_MGR	   Accounting Manager			     8200      16000
AC_ACCOUNT Public Accountant			     4200	9000
SA_MAN	   Sales Manager			    10000      20080
SA_REP	   Sales Representative 		     6000      12008
PU_MAN	   Purchasing Manager			     8000      15000
PU_CLERK   Purchasing Clerk			     2500	5500

JOB_ID	   JOB_TITLE			       MIN_SALARY MAX_SALARY
---------- ----------------------------------- ---------- ----------
ST_MAN	   Stock Manager			     5500	8500
ST_CLERK   Stock Clerk				     2008	5000
SH_CLERK   Shipping Clerk			     2500	5500
IT_PROG    Programmer				     4000      10000
MK_MAN	   Marketing Manager			     9000      15000
MK_REP	   Marketing Representative		     4000	9000
HR_REP	   Human Resources Representative	     4000	9000
PR_REP	   Public Relations Representative	     4500      10500

19 rows selected.
```





