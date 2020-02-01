# 实践8:管理空间

> **Practices for Lesson 8: Managing Space**
>
> 2020.01.29 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:0 updateOnSave:1 -->

[实践8:管理空间](#实践8管理空间)   
&emsp;[实践8:概览](#实践8概览)   
&emsp;[实践8-1:管理存储](#实践8-1管理存储)   
&emsp;&emsp;[Overview](#overview)   
&emsp;&emsp;[Task](#task)   
&emsp;&emsp;[Practice](#practice)   
&emsp;&emsp;[KnowledgePoint](#knowledgepoint)   

<!-- /MDTOC -->

 ## 实践8:概览

Practices for Lesson 8: Overview

**Background:** To prepare for an upcoming merger, you want to set the warning and critical thresholds to a lower value than the default. Ensure that you receive early warnings to give you more time to react. When you finish your test case, drop the tablespace that you used.

背景:为了准备即将到来的合并，需要将警告和临界阈值设置为比默认值更低的值。确保你得到了早期的警告，给你更多的时间来做出反应。当您完成您的测试用例时，删除您使用的表空间。

## 实践8-1:管理存储

Practice 8-1: Managing Storage

### Overview

In this practice you will set a tablespace fullness threshold so as to be warned when a tablespace has reached the fullness threshold tolerated.

在这个实践中，您将设置一个表空间的饱和阈值，以便在一个表空间达到容忍的饱和阈值时得到警告。

### Task

Access the  orcl database as the SYS user (with the oracle_4U password, connect as SYSDBA) and perform the necessary tasks through Enterprise Manager Cloud Control or through SQL*Plus. All scripts for this practice are in the $LABS/P8 directory.

1. Using the DBMS_SERVER_ALERT.SET_THRESHOLD procedure, reset the database-wide threshold values for the Tablespace Space Usage metric. Connect to a SQL*Plus session and execute the following procedure:

   ```sql
   $ . oraenv
   ORACLE_SID = [orcl] ? orcl
   The Oracle base for
   ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1 is
   /u01/app/oracle
   $ cd $LABS/P8
   $ sqlplus / as sysdba
   …
   Connected to:
   …
   SQL> exec DBMS_SERVER_ALERT.SET_THRESHOLD(-
   > dbms_server_alert.tablespace_pct_full,-
   > NULL,NULL,NULL,NULL,1,1,NULL,-
   > dbms_server_alert.object_type_tablespace,NULL);
   PL/SQL procedure successfully completed.
   SQL>
   ```



2. From your SQL*Plus session, check the database-wide threshold values for the Tablespace Space Usage metric by using the following command (output formatted for clarity):

   ```sql
   SQL> SELECT warning_value,critical_value
   2	FROM	dba_thresholds
   3	WHERE	metrics_name='Tablespace Space Usage'
   4	AND	object_name IS NULL;
   ```


   WARNING_VALUE CRITICAL_VALUE
------------- --------------
   85	97
   ```



3. Create a new tablespace called TBSALERT with a 120 MB file called tbsalert.dbf. Make sure that this tablespace is locally managed and uses Automatic Segment Space Management. Do *not* make it auto-extensible, and do *not* specify any thresholds for this tablespace.

   ```sql
   SQL> CREATE TABLESPACE tbsalert
   2	DATAFILE '/u01/app/oracle/oradata/orcl/tbsalert.dbf'
   3	SIZE 120M REUSE LOGGING EXTENT MANAGEMENT LOCAL
   4	SEGMENT SPACE MANAGEMENT AUTO; Tablespace created.
   SQL> SELECT autoextensible FROM dba_data_files
   2	WHERE tablespace_name='TBSALERT';


   AUT
   --- NO
   ```



4.  In Enterprise Manager Cloud Control, navigate to the orcl database home page. Then Select **Administration** > **Storage** > **Tablespaces**.

5. Select the **New** radio button. Enter **SYS** in the Username field, **oracle_4U** in the Password field, and choose **SYSDBA** in the Role field. Then click **Login**.

6. Change the Tablespace Space Usage thresholds of the TBSALERT tablespace. Set its warning level to 55 percent and its critical level to 70 percent.

7. Return to your SQLPlus session and check the new threshold values for the TBSALERT tablespace. In your SQLPlus session, enter (output formatted):

8. In your SQL*Plus session, query the REASON and RESOLUTION columns from DBA_ALERT_HISTORY for the TBSALERT tablespace. Exit from SQL*Plus.

9. Review and execute the **$LABS/P8/seg_advsr_setup.sh** script that creates and populates new tables in the TBSALERT tablespace.

10. Check the fullness level of the TBSALERT tablespace by using Enterprise Manager Cloud Control or SQL*Plus. The current level should be around 60 percent. Wait a few minutes and check that the warning level is reached for the TBSALERT tablespace. (*If you are too fast and receive errors, just use your browser’s Refresh button, or select your destination again.)*

11. In your SQL*Plus session, execute the inserts below to add more data to TBSALERT. Wait a few moments and view the critical level through a query in SQL*Plus and Enterprise Manager Cloud Control. Verify that TBSALERT fullness is around 75 percent.

12.  In your SQL*Plus session, execute the following delete statements to delete rows from tables in TBSALERT. These statements will take several minutes to complete. Then exit your SQL*Plus session.

13.  Now, run the Segment Advisor for the TBSALERT tablespace in Enterprise Manager Cloud Control. Make sure that you run the Advisor in Comprehensive mode without time limitation. Accept and implement its recommendations. After the recommendations have been implemented, check whether the fullness level of TBSALERT is below 55 percent.

14.  Wait a few minutes and check that there are no outstanding alerts for the TBSALERT tablespace. Navigate to the **Oracle Database** > **Monitoring** > **Incident Manager** > **Events without incidents**.

15. Retrieve the history of the TBSALERT Tablespace Space Usage metric for the last 24 hours.

16. Verify that the TBSALERT tablespace fullness has decreased below the threshold because space has been reclaimed. In Enterprise Manager Cloud Control, navigate to **Administration** > **Storage** > **Tablespaces**.

17. Log in to SQL*Plus as the SYS user. In SQL*Plus, reset the TBSALERT tablespace Tablespace Space Usage metric. Exit from SQL*Plus.

18. **Note: This is a mandatory cleanup step**. Review, and then execute the seg_advsr_cleanup.sh script in the $LABS/P8 directory to drop your TBSALERT tablespace.

### Practice

作为`SYS`用户访问数据库，并通过`Enterprise Manager Cloud Control`或`SQL*Plus`执行必要的任务。这个实践的所有脚本都在`$LABS/P8`目录中。

1. 调用存储过程`DBMS_SERVER_ALERT.SET_THRESHOLD`，为表空间使用度量重置数据库范围的阈值。连接到一个SQL*Plus会话并执行以下过程:

   ```sql
   cd $LABS/P8
   sqlplus sys/oracle@booboopdb1 as sysdba
   begin
   DBMS_SERVER_ALERT.SET_THRESHOLD(
    metrics_id=>dbms_server_alert.tablespace_pct_full,
    warning_operator=>NULL,
    warning_value=>NULL,
    critical_operator=>NULL,
    critical_value=>NULL,
    observation_period=>1,
    consecutive_occurrences=>1,
    INSTANCE_NAME=>NULL,
    object_type=>dbms_server_alert.object_type_tablespace,
    object_name=>NULL);
   end;
   /
   ```

2. 在SQL*Plus会话中，使用以下命令检查表空间使用度量的数据库范围阈值(为清晰起见，输出格式化):

   ```sql
   select object_name,metrics_name, WARNING_OPERATOR,status from dba_thresholds ;
   ```

   查询结果：

   ```sql
   SYS@booboopdb1>exec print_table('select object_name,metrics_name, WARNING_OPERATOR,status from dba_thresholds')
   OBJECT_NAME		      : TEMP
   METRICS_NAME		      : Tablespace Space Usage
   WARNING_OPERATOR	      : DO NOT CHECK
   STATUS			      : VALID
   -----------------
   OBJECT_NAME		      : UNDOTBS1
   METRICS_NAME		      : Tablespace Space Usage
   WARNING_OPERATOR	      : DO NOT CHECK
   STATUS			      : VALID
   -----------------
   OBJECT_NAME		      : UNDOTBS1_TEMP
   METRICS_NAME		      : Tablespace Space Usage
   WARNING_OPERATOR	      : DO NOT CHECK
   STATUS			      : INVALID
   -----------------

   PL/SQL procedure successfully completed.
   ```



3. 在SQL*Plus会话中，使用以下命令检查表空间使用度量的数据库范围阈值(为清晰起见，输出格式化):

   ```sql
   CREATE TABLESPACE tbsalert
   	DATAFILE '/u01/app/oracle/oradata/booboo/booboopdb1/tbsalert.dbf'
   	SIZE 120M REUSE LOGGING EXTENT MANAGEMENT LOCAL
   	SEGMENT SPACE MANAGEMENT AUTO;
   SELECT autoextensible FROM dba_data_files
   	WHERE tablespace_name='TBSALERT';
   ```

   执行结果：

   ```sql
   AUT
   ---
   NO
   ```

   说明该表空间文件不会自动增长。

4. dfd

### KnowledgePoint
