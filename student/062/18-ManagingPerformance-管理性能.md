# 实践18:管理性能

> **Practices for Lesson 18: Managing Performance**
>
> 2020.01.29 BoobooWei

[toc]

## 实践18:概览

Practices for Lesson 18: Overview

**Background:** Users are complaining about slower-than-normal performance for operations involving the human resources and order-entry applications. When you question other members of the DBA staff, you find that maintenance was recently performed on some of the tables belonging to the HR schema. You need to troubleshoot and make changes as appropriate to resolve the performance problems. SQL script files are provided for you in the $LABS/P18 directory. Other directories are individually named.

背景:用户抱怨人力资源和订单输入应用程序的运行速度低于正常水平。当您询问其他DBA员工时，您会发现最近对属于HR模式的一些表进行了维护。您需要排除故障并做出适当的更改来解决性能问题。在$LABS/P18目录中为您提供了SQL脚本文件。其他目录是单独命名的。

## 实践18-1:管理性能

Practice 18-1: Managing Performance

### Overview

### Task

1. Log in to SQL*Plus as the DBA1 user and perform maintenance on tables in the HR schema by running the **lab_18_01_01.sql** script.

2. You get calls from HR application users saying that a particular query is taking longer than normal to execute. The query is in the **lab_18_01_02.sql** script. 

3. Using Cloud Control, locate the HR session in which the above statement was just executed, and view the execution plan for that statement.

    

   | **Step** | **Window/Page Description** | **Choices or Values**                                        |
   | -------- | --------------------------- | ------------------------------------------------------------ |
   | a.       | Cloud Control               | Navigate to the **orcl** database target.                    |
   | b.       | orcl database home          | Click **Performance >  Search Sessions.**                    |
   | c.       | Database Login              | Connect by using  Preferred SYSDBA  Credentials.             |
   | d.       | Search Sessions             | Select “**DB  User**” in the Filter field  menu. Enter **HR**.  Click **Go.** |
   | e.       | Search Sessions             | Click the **SID** number in the Results listing.             |
   | f.       | Session Details             | Click the **hash value  link** to the right  of the “Previous SQL”  label in the  Application section. |
   | g.       | SQL Details                 | Click the Plan tab to see the execution plan for the query.<br/>Select the Tabular radio button. |

4. Using Cloud Control, check to see the status of the **EMPLOYEE** table’s index on

   **EMPLOYEE_ID**. See if it is **VALID**.

   | **Step** | **Window/Page Description** | **Choices or Values**                                        |
   | -------- | --------------------------- | ------------------------------------------------------------ |
   | a.       | SQL Details                 | Click **Schema** > **Database Objects** >  **Indexes**.      |
   | b.       | Indexes                     | Select **Table Name** in  the “Search By” menu.  Enter **HR** in the Schema  field.  Enter EMPLOYEES in the Object  field. Click **Go**. |
   | c.       | Indexes                     | In the Index  column, click the  **EMP_EMP_ID_PK** index.    |

   | **Step** | **Window/Page Description**  | **Choices or Values**                                        |
   | -------- | ---------------------------- | ------------------------------------------------------------ |
   | d.       | View Index: HR.EMP_EMP_ID_PK | In the General section, check  the status of the index.  You should see a value  of **UNUSABLE**. |

5. Now that you have seen one index with a non-VALID status, you decide to check all indexes. Using SQL*Plus, as the **HR** user find out which HR schema indexes do not have STATUS of VALID. To do this, you can query a data dictionary view with a condition on the STATUS column.

   a.  Go to the SQL*Plus session where you are still logged in as the HR user, and run this query:

   ```sql
   SQL> COL INDEX_NAME FORMAT A20
   SQL> COL TABLE_NAME FORMAT A20
   SQL> select index_name, table_name, status
   2> from user_indexes where status <> 'VALID';
   ```

   b.  You notice that the output lists six indexes, all on the **EMPLOYEES** table. This is a problem you need to fix.

6. You decide to use Cloud Control to reorganize all the indexes in the HR schema that are marked as UNUSABLE.

    

   | **Step** | **Window/Page Description**        | **Choices or Values**                                        |
   | -------- | ---------------------------------- | ------------------------------------------------------------ |
   | a.       | View Index: HR.EMP_EMP_ID_PK       | Select **Reorganize** in  the Actions menu.  Click **Go**.   |
   | b.       | Reorganize Objects: Objects        | Click **Add**.                                               |
   | c.       | Objects: Add                       | Select **Indexes** in the Type menu. Enter **HR** in the Schema  field.  Enter **EMP_** in the Object  Name field. Click **Search**. |
   | d.       | Objects: Add                       | In the Available Objects section, select all the  indexes that match the UNUSABLE indexes in Step 5.  Click **OK**. |
   | e.       | Reorganize Objects: Objects        | Check that the six  unusable indexes are listed. Click **Next**. |
   | f.       | Reorganize Objects: Options        | Accept the default options. Click  **Next**.                 |
   | g.       | Processing: Generating Script      | Displays briefly.                                            |
   | h.       | Reorganize Objects: Impact  Report | The Script Generation Information section  should show no warnings or errors.  Click **Next.** |
   | i.       | Reorganize Objects: Schedule       | In the Host  Credentials section, select **New.**  Enter Username: **oracle**  Enter password: ****  Click **Test.**  When return is Test Successful, click **Next.** |
   | j.       | Reorganize Objects: Review         | Click **Submit Job.**                                        |
   | k.       | Job Activity                       | A Confirmation message appears.  Click the **REORGANIZE** job name  listed in the  Confirmation Message. |
   | l.       | Job Run: REORGANIZE_ORCL_*         | Refresh the Browser until the job  shows  **Succeeded**.     |

7. Return to the SQL*Plus session where the HR user is logged in and run the **lab_18_01_07.sql** script to execute the same kind of query. Then repeat the steps to see the plan of the last SQL statement executed by this session.

   a.  Enter the following at the SQL*Plus prompt:

    ```sql
   SQL> @lab_18_01_07.sql
    ```

   Repeat the tasks listed in step 3 to view the execution plan for the query. Now the icon indicates the use of an index. Select the **Tabular** radio button. Note that the plan now uses an index unique scan.

   b.  Exit the SQL*Plus session.

8. What is the difference in execution plans, and why?

  *Answer:* The statement execution uses a unique index scan instead of a full table scan, because the index is usable after you reorganized the indexes.

9. Simulate a working load on your instance by running the **l****ab_18_01_09.sql** script as the SYS user. **Note the SID value that is reported**.                      

   This script takes about 20 minutes to complete. So, run it in a separate terminal window and continue with this practice exercise while it runs. Remember to set your environment appropriately by using oraenv in the new terminal window before connecting to SQL*Plus.

   **Note:** Because this script generates a fairly heavy load in terms of CPU and disk I/O, you may notice that response time is slower.

   ```sql
   $ sqlplus DBA1/oracle_4U@orcl as sysdba
   SQL> @lab_18_01_09.sql
   ```

10. Go back to Cloud Control and examine the performance of your database.

    

   | **Step** | **Window/Page Description** | **Choices or Values**                                        |
   | -------- | --------------------------- | ------------------------------------------------------------ |
   | a.       | orcl database home          | Click **Performance >  Performance Home** to investigate system  performance. |
   | b.       | Database Instance: orcl     | View the Average Active session Graph  at the bottom of the page.  **Note:** You may need  to wait a minute  or two to see  the effects of the load generation script  appear on the graphs. |
   
   *Question 1*: In the **Average Active Sessions** graph, which are the two main categories that active sessions are waiting for?
   *Answer:* In this example, it looks like Configuration issues and User I/O are quite high. CPU is also showing high wait activity. Your results may differ from what is shown here.
   
   | **Step** | **Window/Page Description**            | **Choices or Values**                                      |
   | -------- | -------------------------------------- | ---------------------------------------------------------- |
   | c.       | Database Instance: orcl                | Click **Configuration** in the legend.                     |
   | d.       | Active Sessions Waiting: Configuration | Examine the Active  Sessions Waiting: Configuration graph. |
   
   *Question 2:* In the Configuration category of waits, what is one of the contributors to the wait time?
   
   *Answer:* Log buffer space is the highest in this example.
   
   | **Step** | **Window/Page Description**               | **Choices or Values**                                        |
   | -------- | ----------------------------------------- | ------------------------------------------------------------ |
   | a.       | Active Sessions Waiting: Configuration    | Click the browser back  button.                              |
   | b.       | Database Instance: orcl  Performance Home | Click **Settings**.                                          |
   | c.       | Performance Page  Settings                | In Detail Chart  Settings, select **I/O**  for Default  View.  Select **I/O Function** for I/O Chart  Setting. Click **OK**. |
   | d.       | Database Instance: orcl  Performance Home | Scroll down to I/O Megabytes per  Second by  I/O Function graph. |
   
    
   
   **Note:** The graph you see may vary from the screenshot.
   
   *Question 3: Which process is doing the most writing to the disk?*
   
   *Answer:* LGWR
   
    
   
   | **Step** | **Window/Page Description**               | **Choices or Values**                                        |
   | -------- | ----------------------------------------- | ------------------------------------------------------------ |
   | e.       | Database Instance: orcl  Performance Home | Click **Top Activity** link below and right  of the Average Active Sessions graph.  You may need to scroll to the bottom to see the  horizontal scroll bar. |
   | f.       | Top Activity                              | Click the **SQL  ID** of the first DELETE  statement listed in the Top SQL region. |
   | g.       | SQL Details: 0qqwcxx1quwuv                |                                                              |
   
11.  Kill the session that is generating the load. Use the session ID recorded in step 9. The session ID is listed in the **SID** column of the Detail for Selected 5 Minute Interval.

    

   | **Step** | **Window/Page Description**  | **Choices or Values**                                        |
   | -------- | ---------------------------- | ------------------------------------------------------------ |
   | a.       | SQL Details: 0qqwcxx1quwuv   | Click the **SID** number for the  session ID recorded earlier. This  is found under  the heading **Detail for Selected 5 Minute  Interval**. |
   | b.       | Session Details:  *nn* (SYS) | Click **Kill Session**.                                      |
   | c.       | Confirmation                 | Click **Yes**.                                               |
   | d.       | Session Details:  *nn* (SYS) | **Note:** If you remain  on this Session Details page long enough for  a few automatic refreshes to be done, you may see a warning, “WARNING, Session has expired.” or a SQL Error saying the session is marked for kill.  This warning means that  you are attempting to refresh information  about a session that has already been killed. You can ignore  this warning. |

   | **Step** | **Window/Page Description**  | **Choices or Values**                                        |
   | -------- | ---------------------------- | ------------------------------------------------------------ |
   | e.       | Session Details:  *nn* (SYS) | Click **Top Activity** in  the navigation history  at the top of the page. |
   | f.       | Top Activity                 | View the Top Activity graph.  Note that the session activity in the database has declined  considerably. |

12.  Log out of Enterprise Manager Cloud Control.



### Practice

### KnowledgePoint

## 实践18-2:使用自动内存管理

Practice 18-2: Using Automatic Memory Management

### Overview

In this practice you review memory management capabilities. Note that the values you see may differ slightly from what is shown in this activity guide.

### Task

1. Log in to SQL*Plus for the orcl instance as the DBA1 user with the oracle_4U password and make a copy of your server parameter file (SPFILE).

2. Still connected as the DBA1 user in SQL*Plus, set the following parameters to the given value in your SPFILE only! Use the amm_parameters.sql file located in your

   $LABS/P18 directory to set the parameters.

3. Execute the amm_setup.sql script.

4. Log in as the AMM user with the oracle_4U password. Execute the amm_setup2.sql script to re-create the TABSGA table and insert rows.

   a. Modify the TABSGA table to “parallel 64” and create a TESTPGA procedure (which creates a workload) by pressing Enter to continue the script.

   b.  Confirm that there are no errors and query the dynamic memory components again by pressing Enter to continue the script.

   c.  To view the query results, press Enter to continue the script.

   d.  Exit the script, but remain in the SQL*Plus session.

5. Connect as SYSDBA in your SQL*Plus session, and shut down and restart your database instance. Reconnect as the AMM user with the oracle_4U password:

6. As the AMM user, determine the current settings for the various memory buffers as well as the list of resized operations that were performed since you started your instance.

   a.  You can use the **amm_components.sql** script for that purpose.

   b.  To view the query result, press Enter to continue the script.

   c.  View the memory components (ordered by descending START_TIME) by pressing Enter to continue the script.

   d.  To view the query result, press Enter to continue the script.

7. Remain connected as the AMM user in your SQL*Plus session and execute the following query. Immediately after that, determine the component sizes and resized operations. You can use the amm_query1.sql script for that purpose. What do you observe?

   a.  Execute the amm_query1.sql script. You can see that the large pool has a much bigger size, whereas the buffer cache is smaller. This memory transfer was automatically done by the system.

8. Repeat the query by using the amm_query2.sql script. What do you observe?

   *Possible Answer:* The same trend continues.

9. Still connected as the AMM user in your SQL*Plus session, execute the amm_query3.sql script. Immediately afterward, determine the memory component sizes and the list of resize operations. What do you observe?

   *Possible Answer:* The same action of growing and shrinking the memory components

   *Alternative Answer:* The memory grows and shrinks until the memory allocation meets the needs of the database activity, and then remains nearly constant.

10. In Enterprise Manager Cloud Control, look at the memory variations that happened during this practice. What do you observe?

     

    | **Step** | **Window/Page Description** | **Choices or Values**                                        |
    | -------- | --------------------------- | ------------------------------------------------------------ |
    | a.       | Cloud Control               | Log in to Enterprise Manager Cloud Control and navigate to the orcl Database home page. |
    | b.       | orcl Database home          | Click **Performance >  Advisors Home > Memory Advisors.**    |
    | c.       | Memory Advisors             | Scroll down and examine the two  graphs.                     |

    *Question:* What changes do you see to the components of the SGA?

    *Answer*: You should see modifications of the memory components in the second graph, indicating that the large pool grew and shrank.

11. Log out of Enterprise Manager Cloud Control.

12. To clean up your environment, shut down your database instance, restore the original SPFILE, and restart your orcl database instance by executing the amm_cleanup.sh script.

### Practice

### KnowledgePoint

## 实践18-3:监控服务

Practice 18-3: Monitoring Services

In this practice, you create and monitor services.

### Overview

In your database there are several running applications. You want to monitor the resources that are being used by each application. Create a service configuration for each application or application function that uses your database.

In this practice, you create the following configuration in the orcl database:

| **Service Name** | **Usage**      | **Response Time (sec)–**       |
| ---------------- | -------------- | ------------------------------ |
| SERV1            | Client service | **Warning/Critical**  0.4, 1.0 |

### Task

1. Use the DBMS_SERVICE package to create a service called SERV1. Then make sure that you add the service name to your tnsnames.ora file.                   

   a.  The recommended method for adding a service name to the tnsnames.ora file is to use Oracle Net Manager. For this practice, in the interest of time, execute the sv1_add.sh script to add the service name.

   b.  Review the tnsnames.ora file in $ORACLE_HOME/network/admin to confirm that the following lines are included. The script substituted the output of the hostname command for <hostname> below. The output of the host name command is shown in step 1a.

   c.  Use the DBMS_SERVICE.CREATE_SERVICE procedure to create a service. (The command is entered on one line.)

2. After you have created your service, try connecting to your database by using your service name.

   *Question:* What happens? Why?

   *Answer:* You cannot connect by using your service because, although it is defined, it is not started on your instance.

   a.  You can verify this by looking at the DBA_SERVICES view and by looking at the services known to the listener.

   *Question:* How would you make sure that you can connect by using your service?

   *Answer:* You must start your service on your instance.

3. Start the service on your instance and connect to your instance by using your service.

4. Create a workload for the SERV1 service. You will create a user for this activity and start a workload.

   a.  Execute the sv1_load.sh script as SYSDBA. This script creates a new SV_USER user.

   b.  Connect to your instance as the SV_USER user using the SERV1 service. Create workload activity by executing the sv1_load2.sql script. If this script finishes before you completed the next step, then use the sv1_sel.sql script to execute the following query: SELECT COUNT(*) FROM DBA_OBJECTS,DBA_OBJECTS,DBA_OBJECTS

   **Note:** Do not wait for the script to complete before proceeding to the next step.

5. After the execution starts, access the Top Consumers page from the Performance tabbed page in Cloud Control, and determine the amount of resources SERV1 is using. Also, check the statistics on your service with V$SERVICE_STATS from a SQL*Plus session, connected as SYSDBA.

   | **Step** | **Window/Page Description** | **Choices or Values**                                        |
   | -------- | --------------------------- | ------------------------------------------------------------ |
   | a.       | Cloud Control               | Log in to Enterprise Manager Cloud Control and navigate to the orcl Database home page. |
   | b.       | orcl Database               | In the Performance section, view the **Services**  tab.      |

   | **Step** | **Window/Page Description** | **Choices or Values**                                        |
   | -------- | --------------------------- | ------------------------------------------------------------ |
   | c.       | orcl Database home          | Click **Performance >  Performance Home.**                   |
   | d.       | Performance Home            | Scroll down to view the Active Session  graph aggregated by service  by clicking the  **Services** tab. |

   | **Step** | **Window/Page Description** | **Choices or Values**                                     |
   | -------- | --------------------------- | --------------------------------------------------------- |
   | e.       | Performance Home            | Scroll down to Additional Links. Click **Top Consumers**. |
   | f.       | Top Consumers: Overview tab | Review the graphs.                                        |

    The names and number of services listed in the Top Services Graph depends on the number and type of connections to the database.

   | **Step** | **Window/Page Description**    | **Choices or Values**                            |
   | -------- | ------------------------------ | ------------------------------------------------ |
   | g.       | Top Consumers: Overview        | Click the **Top Services** tab.                  |
   | h.       | Top Consumers:Top Services     | Click the **SERV1** link in Service column.      |
   | i.       | Service: SERV1: Modules tab    | Click the **Statistics** tab.                    |
   | j.       | Service: SERV1: Statistics tab | View detailed statistics for the  SERV1 service. |

6. If the sv1_load2.sql script finishes before you complete this step, then use the sv1_sel.sql script to continue creating a workload. When you have completed the tasks, make sure that you stop your running workload by pressing Ctrl + C in your terminal window.

7. Clean up from this practice by running the sv1_cleanup.sh script in the $LABS/P18 directory.

### Practice

### KnowledgePoint