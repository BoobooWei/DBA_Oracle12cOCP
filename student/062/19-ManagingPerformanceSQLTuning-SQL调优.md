# 实践19:管理性能之SQL调优

> **Practices for Lesson 19: Managing Performance: SQL Tuning**
>
> 2020.01.29 BoobooWei
[toc]

## 实践19:概览

Practices for Lesson 19: Overview

By default, Automatic SQL Tuning executes automatically during each nightly maintenance window. For this practice, you simulate the execution of Automatic SQL Tuning, and explore its results.

默认情况下，在每个夜间维护窗口期间自动执行SQL调优。对于这个实践，您将模拟自动SQL调优的执行，并研究其结果。

## 实践19-1:使用自动SQL调优

Practice 19-1: Using Automatic SQL Tuning

### Overview

In this practice, you manually launch Automatic SQL Tuning to automatically tune a small application workload. You then investigate the outcome and configuration possibilities.

**Assumptions**

ADMIN Super Administrator user has been created in Enterprise Manager Cloud Control.

DBA1 user with SYSDBA privileges has been created in orcl database.

### Task

1. In Cloud Control, Configure the Automatic SQL Tuning Task to Implement SQL Profiles Automatically.

   | **Step** | **Window/Page Description** | **Choices or Values**                                        |
   | -------- | --------------------------- | ------------------------------------------------------------ |
   | a.       | EMCC Cloud Control          | Login as **ADMIN** user.                                     |
   | b.       | Summary page                | Navigate to the **orcl** database home page.                 |
   | c.       | orcl database home          | Click **Administration > Oracle Scheduler > Automated Maintenance Tasks**. |
   | d.       | Database Login              | Login with **S****YSDBA** credentials to database.  Use DBA1/oracle_4U AS SYSDBA |
   | e.       | Automated Maintenance Tasks | Verify Status is **Enabled.**  Click **Configure.**          |

   | **Step** | **Window/Page Description**                | **Choices or Values**                              |
   | -------- | ------------------------------------------ | -------------------------------------------------- |
   | f.       | Automated Maintenance Tasks  Configuration | Click **Configure**  next to Automatic SQL Tuning. |

   | **Step** | **Window/Page Description**    | **Choices or Values**                                        |
   | -------- | ------------------------------ | ------------------------------------------------------------ |
   | g.       | Automatic SQL Tuning  Settings | Click **Yes** for Automatic Implementation of  SQL Profiles. Click **Show SQL.** |

   | **Step** | **Window/Page Description** | **Choices or Values**                      |
   | -------- | --------------------------- | ------------------------------------------ |
   | h.       | Show SQL                    | View the SQL statement.  Click **Return.** |

   | **Step** | **Window/Page Description**    | **Choices or Values** |
   | -------- | ------------------------------ | --------------------- |
   | i.       | Automatic SQL Tuning  Settings | Click **Apply.**      |

2. Review and execute the **$LABS/P19/ast_setup.sh** script .This script creates the AST user, turns off automatic maintenance tasks, and drops any existing profiles on queries executed by the AST user.

3. Execute the **ast_workload_stream.sh** script. This script executes a query that is not correctly optimized multiple times. The query in question uses hints that force the optimizer to pick a suboptimal execution plan. The script executes for approximately 60 seconds.

4. Automatic SQL Tuning is implemented by using an automated task that runs during maintenance windows. However, you are not going to wait for the next maintenance window to open. This might take too long. Instead, you will force the opening of your next maintenance window now. This will automatically trigger the Automatic SQL Tuning task. Review and execute the **ast_run.sh** script to do that. It takes about 10 minutes for the script to execute.

5. Execute the **ast_workload_stream.sh** script again. What do you observe? 

   You should see that the execution time for ast_workload_stream.sh is much faster than the original execution. This is probably due to the fact that Automatic SQL Tuning implemented a profile for your statement automatically.

6. Log in as the AST user and force the creation of an AWR snapshot.

7. How can you confirm that a SQL Profile was automatically implemented?

   a.  In Enterprise Manager Cloud Control, navigate to **Administration > Oracle Scheduler>Automated Maintenance Tasks.**

   b.  Click **Automatic SQL Tuning**.

   c.  On the Automatic SQL Tuning Result Summary page, view the tuning results.

   d.  Look at the graphs on the Automatic SQL Tuning Result Summary page. (If you do not see any graphs, return to step 5, execute the work load twice, and then continue with step 6 and 7.)

   e.  Focus on understanding the pie chart and the bar graph next to it. You should be able to get a sense of the general findings, as well as the number of SQL profiles implemented by the task.

   f.   In the Summary Time Period section, Click **View Report** to see a detailed SQL-level report.                                                                                                                                                                                                                                     g.  Find and select the SQL statement that ran in the AST schema.

   **Note:** The Thumbs Up icon means that the profile was implemented.

   h.  Click **View Recommendations**.

   i.   Click the **Compare Explain Plans** eyeglass icon for the SQL Profile entry.

    j.   Scroll down the page.

   k.  Look at the old and new explain plans for the query.

   l.   Then click the “**Recommendations for SQL ID**” locator link (the last of the breadcrumbs on top of the page) to return to the previous screen.

   m.  Investigate a SQL profile. While still on the “Recommendations for SQL_ID” page, click the **SQL Text** link to go to the SQL Details page for this SQL.

   n.  On the SQL Details – Tuning History page note the link to SYS_AUTO_SQL_TUNING_TASK that is there to show that the SQL was tuned by this tuning task.

   o.  Click the **Plan Control** tab.

   p.  Note that a profile was created automatically for this SQL. The type of AUTO means it was automatically created.

   q.  Click the **Statistics** tab to take a look at the execution history for this SQL.

   r.   Select one of the plan hash values from the pull down Plan Hash Values. What is the time of the execution, and Elapsed Time per Execution?

   s.  Select the other plan hash value from the pull down Plan Hash Values. What is the time of the execution, and Elapsed Time per Execution?

   t.   Which of the two executed first? Which one executed more quickly?

   *The hash value 4005616876 in the example executed first, and the second hash value executed more quickly.*

   u.  Select All in the Plan Hash Values. This shows the improved plan and the original in the same graph. The bar graph for the second run with the SQL Profile applied may be so small as to be almost invisible.

8. Generate a text report for more in-depth information. From the command line, execute the **ast_task_report.sh** script. What do you observe?

   a.  Notice the first queries that fetch the execution name and object number from the advisor schema, followed by the final query that gets the text report. In the text report, look for the section about the SQL profile finding and peruse the Validation Results section. This shows you the execution statistics observed during test-execute and allows you to get a better sense of the profile’s quality. You can also use the report_auto_tuning_task API to get reports that span multiple executions of the task.

9. Investigate configuring Automatic SQL Tuning with Cloud Control.

   a.  While you are logged in to the **orcl** database target as the **D****BA1** user, navigate to **Administration > Oracle Scheduler > Automated Maintenance Tasks**.

   b.  The chart shows times in the past when each client was executed, and times in the future when they are scheduled to run again.

   c.  Select **7 days** in the Interval menu and click **Go** to see an entire week’s worth of data.

   d.  Click the **Configure** button. On the Automated Maintenance Tasks Configuration page, you can disable individual clients and change which windows they run in.

   e.  Disable the Automatic SQL Tuning client entirely and click **Show SQL**.

   f.   Review the command and then click **Return**.

   g.  On the Automated Maintenance Tasks Configuration page, click **Apply**. You should receive a success message.

   h.  Click the **Automated Maintenance Tasks** locator link at the top of the page i.   Notice the forbidden sign right next to the task name.

   j.   Click **Configure**.

   k.  **Enable** the Automatic SQL Tuning task.

   l.   Optionally, click Show SQL, review the commands and then click **Return**.

   m.  Click **Apply** to enable Automatic SQL Tuning. You should receive a success message. n.   Navigate to the Automatic SQL Tuning Settings page. If you are on the Automated Maintenance Tasks Configuration page, click the **Configure** button for Automatic SQL Tuning.

   o.  On the Automatic SQL Tuning Settings page, select **No** beside the “Automatic

   Implementation of SQL Profiles” field, and click **Show SQL**.

   p.  Review the command, click **Return**, and then click **Apply**. You should receive a success message.

10. OPTIONAL: Review the ast_manual_config.sh script to understand how you can configure Automatic SQL Tuning by using PL/SQL.   

    

### Practice

### KnowledgePoint
