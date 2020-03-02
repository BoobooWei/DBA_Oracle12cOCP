# 实践21:自动化任务

> **Practices for Lesson 21: Using Oracle Scheduler to Automate Tasks**
>
> 2020.01.29 BoobooWei

[toc]

## 实践21:概览

Practices for Lesson 21: Overview

In these practices, you explore Oracle Scheduler capabilities.

在这些实践中，您将探索Oracle调度器功能。

## 实践21-1:创建调度程序组件

Practice 21-1: Creating Scheduler Components

### Overview

In this practice, you create Scheduler components such as programs, jobs, and schedules.

### Task

In this practice, you use Enterprise Manager Cloud Control to create Scheduler objects in the

ORCL database instance and automate tasks.

1. Create a simple job that runs a SQL script by using Enterprise Manager Cloud Control.

2. Log in to SQL*Plus as the DBA1 user. Grant the CONNECT, RESOURCE, and DBA roles to the HR user.

3. Return to Enterprise Manager Cloud Control. On the Scheduler Jobs page, re-order the jobs by **Last Run Date** by clicking the column name until they are in descending order. If the job does not appear on the Scheduler Jobs page, click the Refresh button until it succeeds. Also, you may not see it “running,” but with the Last Run Status of SUCCEEDED.

   If you do not see the job on the All page, check the History page. 

4. Create a program called LOG_SESS_COUNT_PRGM that logs the current number of database sessions into a table. 

5. Create a schedule named SESS_UPDATE_SCHED owned by HR that executes every three seconds. Use SQL*Plus and the DBMS_SCHEDULER.CREATE_SCHEDULE procedure to create the schedule.

6. Using Enterprise Manager Cloud Control, create a job named LOG_SESSIONS_JOB that uses the LOG_SESS_COUNT_PRGM program and the SESS_UPDATE_SCHED schedule. Make sure that the job uses FULL logging.

7. In your SQL*Plus session, check the HR.SESSION_HISTORY table for rows.

8. Use Enterprise Manager Cloud Control to alter the SESS_UPDATE_SCHED schedule from every three seconds to every three minutes. Then use SQL*Plus to verify that the rows are now being added every three minutes by querying the HR.SESSION_HISTORY table, ordered by the SNAP_TIME column.

9. In your SQL*Plus session, query the **HR.SESSION_HISTORY** table, ordered by the **SNAP_TIME** column. (Wait for three minutes after you update the schedule.)

10. **This is your mandatory cleanup task.** Use Enterprise Manager Cloud Control to drop the LOG_SESSIONS_JOB and CREATE_LOG_TABLE_JOB jobs, the LOG_SESS_COUNT_PRGM program, and the SESS_UPDATE_SCHED schedule. Use SQL*Plus to drop the SESSION_HISTORY table, and exit from your session.

    **Note:** Make sure that you do not delete the wrong schedule.

### Practice

### KnowledgePoint

## 实践21-2:创建轻量级调度程序作业

Practice 21-2: Creating Lightweight Scheduler Jobs

### Overview

In this practice, you create and execute lightweight scheduler jobs.

### Task

In this optional practice, you create and run a lightweight scheduler job. View the metadata for a lightweight scheduler job. Navigate to your $LABS/P21 directory.

1. Create a job template for the lightweight job. The template must be a PL/SQL procedure or a PL/SQL block. Run the cr_test_log.sql script to create the TEST_LOG table. Then run prog_1.sql. The prog_1.sql script in the $LABS/P21 directory creates a job template.

   **Note:** The job template has a subset of the attributes of a scheduler program. Most of the attributes of a template cannot be changed for the job.

2. Create a lightweight job by using the PL/SQL API. The job will run the my_prog template daily with an interval of 2, starting immediately.

3. Verify that the job was created by querying the USER_SCHEDULER_JOBS view.

4. Access the Enterprise Manager Cloud Control Scheduler Jobs page, find the MY_LWT_JOB job, and view the attributes.

5. On the Scheduler Jobs, All page, delete the MY_LWT_JOB job.

### Practice

### KnowledgePoint

## 实践21-3:监控调度程序

Practice 21-3: Monitoring the Scheduler

### Overview

In this practice, you view Scheduler components.

### Task

In this practice, use Enterprise Manager Cloud Control to view Scheduler components. Click **Show SQL** regularly to review all statements that are new to you.

Log in as the DBA1 user (with oracle_4U password, connect as SYSDBA). Perform the necessary tasks either through Enterprise Manager Cloud Control or through SQL*Plus. All scripts for this practice are in the $LABS/P21 directory.

1. Log in to the orcl database target as the DBA1 user with the oracle_4U password.

2. To view the Scheduler jobs, navigate to **Administration** > **Oracle Scheduler** > **Jobs**. Are there any jobs?

   *Answer:* There are some jobs.

3. Are there any programs? Navigate to **Administration** > **Oracle Scheduler** > **Programs**.

   *Answer:* There are some existing programs.

4. Are there any schedules? Navigate to **Administration** > **Oracle Scheduler** > **Schedules.**

   *Answer:* There are four schedules: DAILY_PURGE_SCHEDULE, FILE_WATCHER_SCHEDULE, PMO_DEFERRED_GIDX_MAINT_SCHED and BSLN_MAINTAIN_STATS_SCHED.

5. List the Scheduler Windows. Are there any existing windows? Which resource plan is associated with each window? Navigate to **Administration** > **Oracle Scheduler** > **Windows**.

    *Answer:* There are several windows. All are enabled, except WEEKNIGHT_WINDOW and

   WEEKEND_WINDOW.

6. Click the **MONDAY_WINDOW** link. Answer the questions, and then click **OK**.

   *Question 1:* At which time does this window open? **10 PM**

   *Question 2:* For how long does it stay open? **4 hours**

7. List the Scheduler Job Classes. Are there any Job Classes? Navigate to **Administration** > **Oracle Scheduler** > **Job Classes**.

   *Answer:* There are many job classes.

   Question 2: Which resource consumer group is associated with the DEFAULT_JOB_CLASS

   job class?

   *Possible Answer*: None

8. On the Scheduler Job classes page, click the ORA$AT_JCURG_OS link.

   *Question 1:* Which resource consumer group is associated with the job class?

   *Answer:* `ORA$AT_JCURG_OS` is associated with `ORA$AUTOTASK`.

   *Question 2:* For which task is this job class used?

   *Answer:* For automatic optimizer statistics collection

9. Click **OK**, and then log out of Enterprise Manager Cloud Control.

### Practice

### KnowledgePoint