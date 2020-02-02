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

1. Using the DBMS_SERVER_ALERT.SET_THRESHOLD procedure, reset the database-wide threshold values for the Tablespace Space Usage metric. Connect to a SQL*Plus session and execute the following procedure.


2. From your SQL*Plus session, check the database-wide threshold values for the Tablespace Space Usage metric by using the following command (output formatted for clarity):


3. Create a new tablespace called TBSALERT with a 120 MB file called tbsalert.dbf. Make sure that this tablespace is locally managed and uses Automatic Segment Space Management. Do *not* make it auto-extensible, and do *not* specify any thresholds for this tablespace.


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
   sqlplus / as sysdba
   conn sys/oracle@booboopdb1 as sysdba
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
   select * from dba_thresholds where object_name is null;
   ```

   查询结果：默认 `warnning：85%；critical：97%`

   ```sql
   SYS@booboo>exec print_table('select * from dba_thresholds where object_name is null');
   METRICS_NAME		      : Tablespace Space Usage
   WARNING_OPERATOR	      : GE
   WARNING_VALUE		      : 85
   CRITICAL_OPERATOR	      : GE
   CRITICAL_VALUE		      : 97
   OBSERVATION_PERIOD	      : 1
   CONSECUTIVE_OCCURRENCES       : 1
   INSTANCE_NAME		      : database_wide
   OBJECT_TYPE		      : TABLESPACE
   OBJECT_NAME		      :
   STATUS			      : VALID
   ```



3. 使用名为`tbsalert.dbf`的120 MB文件创建一个名为`TBSALERT`的新表空间。确保这个表空间是本地管理的，并使用自动段空间管理。不能自动扩展，不要为这个表空间指定任何阈值。

   ```sql
   CREATE TABLESPACE tbsalert
   	DATAFILE '/u01/app/oracle/oradata/booboo/booboopdb1/tbsalert.dbf'
   	SIZE 120M REUSE LOGGING EXTENT MANAGEMENT LOCAL
   	SEGMENT SPACE MANAGEMENT AUTO;

   SELECT autoextensible FROM dba_data_files
   	WHERE tablespace_name='TBSALERT';
   ```

   执行结果为`NO`，说明该表空间文件不会自动增长。

   ```sql
   AUT
   ---
   NO
   ```



4. 登陆企业云管理 Enterprise Manager Cloud Control,访问 booboopdb1数据库的网页，然后选择 管理>存储>表空间。

5. 输入用户名密码登陆`sys/oracle as sysdba`

6. 点击表空间 TBSALERT，编辑该表空间，设置它的警告值为55%，严重级别为70%。

7. 返回SQL*Plus，检查新的阈值配置是否已经生效:

   ```sql
   SYS@booboo>conn sys/oracle@booboopdb1 as sysdba
   Connected.
   SYS@booboopdb1>exec print_table(q'[select * from dba_thresholds where object_name='TBSALERT']');
   METRICS_NAME		      : Tablespace Space Usage
   WARNING_OPERATOR	      : GE
   WARNING_VALUE		      : 55
   CRITICAL_OPERATOR	      : GE
   CRITICAL_VALUE		      : 70
   OBSERVATION_PERIOD	      : 2
   CONSECUTIVE_OCCURRENCES       : 1
   INSTANCE_NAME		      : database_wide
   OBJECT_TYPE		      : TABLESPACE
   OBJECT_NAME		      : TBSALERT
   STATUS			      : VALID
   ```

8. 在SQL*Plus会话中，从`DBA_ALERT_HISTORY`中查询`TBSALERT`表空间的`REASON` 和 `RESOLUTION`列。

   ```sql
   select REASON,RESOLUTION from DBA_ALERT_HISTORY where object_name = 'TBSALERT';
   ```

   执行结果：

   ```sql
   SYS@booboopdb1>exec print_table(q'[select REASON,RESOLUTION from DBA_ALERT_HISTORY where object_name = 'TBSALERT']')
   REASON			      : Threshold is updated on metrics "Tablespace Space Usage"
   RESOLUTION		      : cleared
   ```



9. 执行`$LABS/P8/seg_advsr_setup.sh`脚本，在`TBSALERT`表空间中创建和填充新表。

   ```bash
   LAB=/home/oracle/labs
   # 根据自己的pdb连接方式修改脚本
   vim $LAB/P8/seg_advsr_setup.sh
   sqlplus / as sysdba << EOF
   alter system set disk_asynch_io = FALSE scope = spfile;
   shutdown immediate
   startup
   show parameter disk_asynch_io;
   exit;
   EOF

   # 批量建表和插入数据
   vim $LAB/P8/seg_advsr_setup02.sh
   sqlplus / as sysdba << EOF
   connect sys/oracle@booboopdb1 as sysdba
   set echo on
   create table employees1 tablespace tbsalert as select * from hr.employees;
   create table employees2 tablespace tbsalert as select * from hr.employees;
   create table employees3 tablespace tbsalert as select * from hr.employees;
   create table employees4 tablespace tbsalert as select * from hr.employees;
   create table employees5 tablespace tbsalert as select * from hr.employees;

   alter table employees1 enable row movement;
   alter table employees2 enable row movement;
   alter table employees3 enable row movement;
   alter table employees4 enable row movement;
   alter table employees5 enable row movement;

   BEGIN
   FOR i in 1..10 LOOP
      insert into employees1 select * from employees1;
      insert into employees2 select * from employees2;
      insert into employees3 select * from employees3;
      insert into employees4 select * from employees4;
      insert into employees5 select * from employees5;
      commit;
    END LOOP;
   END;
   /
   insert into employees1 select * from employees1;
   insert into employees2 select * from employees2;
   insert into employees3 select * from employees3;
   commit;
   exit;
   EOF
   ```

10. 使用Enterprise Manager Cloud Control或`SQL*Plus`检查`TBSALERT`表空间的填充级别。目前的水平应该在`60%`左右。等待几分钟，检查`TBSALERT`表空间是否达到警告级别。

   ```sql
   --使用sql查询表空间使用占比，其中125829120 为表空间总大小
   select sum(bytes) *100 /125829120 from dba_extents where tablespace_name='TBSALERT';
   --查询指定表空间的总大小
   SELECT
       tablespace_name, sum(bytes)
   FROM
       dba_data_files
   where tablespace_name='TBSALERT'
   group by tablespace_name;
   --查询每个表空间的总大小，使用占比，空闲占比
   select t1.tablespace_name,t1.tablespace_total_bytes,t2.tablespace_used_bytes,
   t1.tablespace_total_bytes - t2.tablespace_used_bytes tablespace_free_bytes,
   t2.tablespace_used_bytes / t1.tablespace_total_bytes tablespace_used_ratio,
   (t1.tablespace_total_bytes - t2.tablespace_used_bytes) / t1.tablespace_total_bytes tablespace_free_ratio
   from
   (SELECT tablespace_name, sum(bytes) tablespace_total_bytes
   FROM dba_data_files group by tablespace_name) t1
   join
   (select tablespace_name, sum(bytes) tablespace_used_bytes from dba_extents group by tablespace_name) t2
   on t1.tablespace_name = t2.tablespace_name;
   ```

   执行结果

   ```sql
   SYS@booboopdb1>select sum(bytes) *100 /125829120 from dba_extents where tablespace_name='TBSALERT';

   SUM(BYTES)*100/125829120
   ------------------------
   		      60
   ```

	查看告警记录

	```sql
	select reason from dba_outstanding_alerts where object_name='TBSALERT';
	```

11. 在您的`SQL*Plus`会话中，执行以下插入以向`TBSALERT`添加更多数据。稍等片刻，通过`SQL*Plus`和Enterprise Manager Cloud Control中的查询查看临界级别。确认TBSALERT使用率应该在`75%`左右。

    ```sql
    --执行插入
    insert into employees4 select * from employees4;
    commit;
    insert into employees5 select * from employees5;
    commit;
    --查看表空间占比
    select sum(bytes) *100 /125829120 from dba_extents where tablespace_name='TBSALERT';
    --查看告警记录
    select reason from dba_outstanding_alerts where object_name='TBSALERT';
    ```



12. 在`SQL*Plus`会话中，执行以下`delete`语句来删除`TBSALERT`表中的行。这些陈述需要几分钟才能完成。然后退出`SQL*Plus`会话。

    ```sql
    delete employees1;
    commit;
    delete employees2;
    commit;
    delete employees3;
    commit;
    ```



13. 在Enterprise Manager Cloud Control中运行`TBSALERT`表空间的`Segment Advisor`。确保您在不受时间限制的综合模式下运行Advisor工具。接受并执行其建议。建议实施后，检查`TBSALERT`的使用率是否低于`55%`。



14. 等待几分钟，检查`TBSALERT`表空间没有未完成的警报。导航到**Oracle数据库** > **监控** > **意外事件管理器** > **无事件事件** 。

15. 检索`TBSALERT`表空间使用度量在过去24小时内的历史记录。

16. 验证`TBSALERT`表空间的使用率已降低到阈值以下，因为空间已被回收。在企业管理云控制中，导航到**管理** > **存储** > **表空间**。

17. 作为`SYS`用户登录到`SQL*Plus`。在SQL*Plus中，重置TBSALERT表空间使用度量，退出。

    ```sql
    EXEC DBMS_SERVER_ALERT.SET_THRESHOLD(9000,NULL,NULL,NULL,NULL,1,1,NULL,5,'TBSALERT')
    SELECT warning_value,critical_value
    FROM	dba_thresholds
    WHERE metrics_name='Tablespace Space Usage'
    AND	object_name='TBSALERT';
    ```



18. **注意:这是一个强制性的清理步骤**。检查，然后执行脚本`$LABS/P8/seg_advsr_cleanup.sh`来删除`TBSALERT`表空间。

    ```bash
    $ cat seg_advsr_cleanup.sh
    #!/bin/sh
    # For training only, execute as oracle OS user


    sqlplus /nolog <<EOF
    connect / as sysdba
    alter system set disk_asynch_io = TRUE scope = spfile;
    shutdown immediate;
    startup
    alter session set container=booboopdb1;
    drop tablespace tbsalert including contents and datafiles;
    exit
    EOF

    ```



### KnowledgePoint
