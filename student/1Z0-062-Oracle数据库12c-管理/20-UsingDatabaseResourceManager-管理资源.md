# 实践20:管理资源



> **Practices for Lesson 20: Using Database Resource Manager**
>
> 2020.01.29 BoobooWei

[toc]

## 实践20:概览

Practices for Lesson 20: Overview

You received complaints that certain batch jobs are using too many system resources and that a specific user is known to start data warehouse processes during regular business hours. You decide to use the Database Resource Manager for better system-resource utilization and control.

Your first effort to balance the situation includes creating an APPUSER consumer group and assigning it to the default DEFAULT_PLAN resource plan. You then map a couple of Oracle users and your major OS user to resource groups. Activate the resource plan and test your assignments. Regularly click Show SQL to review all statements that are new to you.

您收到的投诉是，某些批处理作业使用了太多的系统资源，并且已知某个特定用户会在正常工作时间启动数据仓库流程。您决定使用数据库资源管理器来更好地利用和控制系统资源。

要平衡这种情况，首先要创建一个APPUSER使用者组，并将其分配给默认的DEFAULT_PLAN资源计划。然后将几个Oracle用户和主要OS用户映射到资源组。激活资源计划并测试你的任务。定期单击Show SQL查看所有新出现的语句。

## 实践20-1:管理资源

Practice 20-1: Managing Resources

### Overview

In this practice you use Enterprise Manager Cloud Control and SQL*Plus to configure a resource plan with consumer groups to balance the resource usage among different users and applications.

**Assumptions**

Users SH, OE, and PM are unlocked and the password for each is set to oracle_4U.

### Task

In this practice, you create an APPUSER consumer group and assign it to the default DEFAULT_PLAN resource plan. Then you map a few Oracle users and your major OS user to resource groups. Activate the resource plan and test your assignments.

Log in as the DBA1 user (with oracle_4U password, connect as SYSDBA) and perform the necessary tasks through Enterprise Manager Cloud Control or through SQL*Plus. All scripts for this practice are in the $LABS/P20 directory.

Whenever you open a new terminal window, execute the oraenv script to set environment variables for the orcl database.

1. Using Cloud Control, create a resource group called APPUSER. At this point, do not add users to the group.

   | **Step** | **Window/Page Description**                      | **Choices or Values**                                        |
   | -------- | ------------------------------------------------ | ------------------------------------------------------------ |
   | a.       | Cloud Control orcl  Database home                | Click **Administration > Resource Manager**                  |
   | b.       | Database Login                                   | Select Credential **Preferred**.  Select Preferred Credential Name: **SYSDBA  Database Credentials**  Click **Login**. |
   | c.       | Getting Started with  Database  Resource Manager | Click **Consumer  Groups**.                                  |
   | d.       | Consumer Groups                                  | Click **Create**.                                            |
   | e.       | Create Resource Consumer Group                   | Enter Consumer Group:  **APPUSER** Verify Scheduling Policy: **Round Robin** Click **Show SQL**. |
   | f.       | Show SQL                                         | Click **Return**.                                            |
   | g.       | Create Resource Consumer Group                   | Click **OK**.                                                |

   *Question 1:* What does the ROUND-ROBIN parameter value mean?

   *Possible Answer:* ROUND-ROBIN indicates that CPU resources are fairly allocated to the PPUSER consumer group, according to the active resource plan directives.

2. Create a new plan called NEW_DEFAULT_PLAN that uses the DEFAULT_PLAN as a template. Use the Create Like action. Add the APPUSER and LOW_GROUP consumer groups to the DEFAULT_PLAN resource plan. Change the level 3 CPU resource allocation percentages: 60 percent for the APPUSER consumer group and 40 percent for the LOW_GROUP consumer group.

    

   | **Step** | **Window/Page Description**                      | **Choices or Values**                                        |
   | -------- | ------------------------------------------------ | ------------------------------------------------------------ |
   | a.       | Consumer Groups                                  | Click **Administration > Resource Manager**.                 |
   | b.       | Getting Started with  Database  Resource Manager | Click **Plans**.                                             |
   | c.       | Resource Plans                                   | Select **Default Plan**.  Select Action **Create Like**. Click **Go**. |
   | d.       | Create Resource Plan                             | Enter Plan: **NEW_DEFAULT_PLAN**                             |
   | e.       | Create Resource Plan                             | In Resource Allocations section, click  **Add/Remove**.      |
   | f.       | Select Groups/Subplans                           | Select **APPUSER**.  Click **Move** from Available Groups/Subplans. |

   | **Step** | **Window/Page Description** | **Choices or Values**                                        |
   | -------- | --------------------------- | ------------------------------------------------------------ |
   |          |                             | pane to Resource Allocations pane.  Select **LOW_GROUP**. Click **Move**.  Click **OK**. |
   | g.       | Create Resource Plan        | In Resource Allocations section:  For APPUSER, set  Shares equal to **40**.  For LOW_GROUP, set  Shares equal to **20**.  For SYS_GROUP, set Shares equal  to **30**. Click **Show SQL**. |
   | h.       | Show SQL                    | Review the PL/SQL  code. Click **Return**.                   |
   | i.       | Create Resource Plan        | Click **OK**.                                                |

3. There are two ways to assign users to consumer groups: The user can be assigned to one or more groups explicitly and an initial group defined, or the user can be mapped into an initial group based on one or more of the rules in the Consumer Group Mappings. Configure Consumer Group Mappings so that the HR Oracle user belongs to the APPUSER consumer group and the SCOTT user to the LOW_GROUP consumer group. For the SCOTT user, confirm that his ORACLE_USER attribute has a higher priority than the CLIENT_OS_USER attribute.

   a.  Log in to SQL*Plus as the DBA1 user.

   b.  Execute the **$LABS/P20/assign_hr_appuser.sql** script to assign the HR user to the APPUSER consumer group.

   c.  Execute the **$LABS/P20/assign_scott_lowgroup.sql** script to assign the SCOTT user to the LOW_GROUP consumer group.

4. Return to Enterprise Manager Cloud Control to verify the additions you made in step 3.

   a. Click **Administration > Resource Manager**.

   b.  Click **Consumer Group Mappings**. c.   HR and SCOTT now appear in the list.

5. Assign the PM Oracle user to the following consumer groups: APPUSER, LOW_GROUP, and

   SYS_GROUP without using the Consumer Group Mappings.

    

   | **Step** | **Window/Page Description**                    | **Choices or Values**                                        |
   | -------- | ---------------------------------------------- | ------------------------------------------------------------ |
   | a.       | Consumer Group Mappings                        | Click **Security >  Users**.                                 |
   | b.       | Users                                          | Enter **PM** in  the Search box. Click **Go**.               |
   | c.       | Users                                          | Select **PM** user. Click **Edit**.                          |
   | d.       | Edit User: PM                                  | Click **Consumer  Group Privileges** tab.  If you see an error  regarding the password for the PM user, enter oracle_4U in both the password fields. |
   | e.       | Edit User: PM : Consumer Group  Privileges tab | Click **Edit List**.                                         |
   | f.       | Modify Consumer Groups                         | Move **APPUSER**  to Selected  Consumer  Groups.  Move **LOW_GROUP** to  Selected Consumer  Groups. |

   | **Step** | **Window/Page Description** | **Choices or Values**                                        |
   | -------- | --------------------------- | ------------------------------------------------------------ |
   |          |                             | Move **SYS_GROUP**  to Selected Consumer  Groups. Click **OK**. |
   | g.       | Edit User: PM               | Set Default Consumer Group to **APPUSER**. Click **Show SQL**. |
   | h.       | Show SQL                    | **Note:** The PM user  is granted the  privilege of switching to any of the three  groups, but the initial group is set to APPUSER.  Click **Return**. |
   | i.       | Edit User: PM               | Click **Apply**.                                             |

6. Activate the NEW_DEFAULT_PLAN resource plan.

    

   | **Step** | **Window/Page Description**                      | **Choices or Values**                                        |
   | -------- | ------------------------------------------------ | ------------------------------------------------------------ |
   | a.       | Edit User: PM                                    | Click **Administration > Resource Manager**                  |
   | b.       | Getting Started with  Database  Resource Manager | Click **Plans**.                                             |
   | c.       | Resource Plans                                   | Select **NEW_DEFAULT_PLAN**. Select **Activate** in the Actions menu.  Click **GO**. |
   | d.       | Confirmation                                     | Click **Yes**.                                               |
   | e.       | Resource Plans                                   | You should see a success message.  NEW_DEFAULT_PLAN has status of ACTIVE. |

7. Test the consumer group mappings. Start two SQL*Plus sessions: the first with the system/oracle_4U@orcl connect string and the second with the scott/tiger@orcl connect string.

   a.  As the oracle user in a terminal window, execute the oraenv script to set environment variables for the orcl database.

   b.  To start a SQL*Plus session with the system/oracle_4U@orcl connect string and to set your SQL prompt to “FIRST,” enter:

   c.  As the oracle user in a second terminal window, execute the oraenv script to set environment variables for the orcl database.

   d.  To start a SQL*Plus session with the scott/tiger@orcl connect string and to set your SQL prompt to “SECOND,” enter:

   e.  In your FIRST SQL*Plus session, enter:

   ```sql
   column username format A12
   column resource_consumer_group format A24
   SELECT username, resource_consumer_group, count(username) FROM v$session
   WHERE username IS not null
   AND program IS not null
   GROUP BY username, resource_consumer_group;
   ```

    **Note:** This statement is available in the /solns/sol_qry_vsession.sql file.

   *Question:* To which consumer group does the SCOTT user belong?

   *Answer:* SCOTT is in the LOW_GROUP consumer group.

   **Note:** Your output for this step (and the following steps) may not look exactly like the output shown. The information of concern here is for the specific users being mentioned.

   f.   In the SECOND terminal window, connect as the PM user with the oracle_4U password:

   g.  In your FIRST SQL*Plus session, enter “/” to execute the previous SQL statement again.

   h.  In the SECOND terminal window, connect as the OE user with the oracle_4U password:

   i.   In your FIRST SQL*Plus session, enter “/” to execute the previous SQL statement again.

   j.   Exit both the SQL*Plus sessions.

   *Question:* When testing your OE Oracle user, you notice that OE is in the

   OTHER_GROUPS consumer group. Why is that?

   *Possible Answer:* The OE user is not explicitly assigned to another consumer resource group.

8. Revert to your original configuration by deactivating the NEW_DEFAULT_PLAN resource group, undoing all consumer group mappings, and finally by deleting the APPUSER resource group.

    

   | **Step** | **Window/Page Description**                      | **Choices or Values**                                        |
   | -------- | ------------------------------------------------ | ------------------------------------------------------------ |
   | a.       |                                                  | Click **Administration > Resource Manager**                  |
   | b.       | Getting Started with  Database  Resource Manager | Click **Plans.**                                             |
   | c.       | Resource Plans                                   | Select **INTERNAL_PLAN.**  Select **Activate** in  the Actions menu.  Click **GO.** |
   | d.       | Confirmation                                     | Click **Yes.**                                               |
   | e.       | Resource Plans                                   | You should see a success message  INTERNAL_PLAN has status of ACTIVE |

   f.   To reconfigure or undo all consumer group mappings, review and execute the rsc_cleanup.sh script from the $LABS/P20 directory:

   g.  Log out of Enterprise Manager Cloud Control.

### Practice

### KnowledgePoint