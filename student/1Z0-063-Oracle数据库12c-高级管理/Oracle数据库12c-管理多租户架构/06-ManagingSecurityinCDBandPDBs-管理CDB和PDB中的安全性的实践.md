# 第6课：管理CDB和PDB中的安全性的实践

> 2010.05.02 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [第6课：管理CDB和PDB中的安全性的实践](#第6课：管理cdb和pdb中的安全性的实践)   
   - [第6课：概述的练习](#第6课：概述的练习)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
   - [练习6-1：管理普通用户和本地用户](#练习6-1：管理普通用户和本地用户)   
      - [总览](#总览)   
      - [任务](#任务)   
   - [练习6-2：管理本地角色和公共角色](#练习6-2：管理本地角色和公共角色)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习6-3：管理本地和通用特权](#练习6-3：管理本地和通用特权)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习6-4：使普通用户可以查看有关PDB对象的信息](#练习6-4：使普通用户可以查看有关pdb对象的信息)   
      - [总览](#总览)   
      - [任务](#任务)   

<!-- /MDTOC -->

## 第6课：概述的练习

### 总览

在这种实践中，您将管理用户，特权和角色。

### 假设条件

* 练习3-1成功创建了cdb2。
* 练习3-3成功创建了pdb2_1。
* 练习4-4已成功将pdb2_1重命名为pdb2。
* 练习3-4成功创建了pdb2_2。



如果无法成功创建触发器，请执行以下追赶脚本：

```sql
cd /home/oracle/solutions/catchup_04_03
./cr_trig.sh
```

如果无法成功创建永久和临时表空间，请执行以下追赶脚本：

```sql
cd /home/oracle/solutions/catchup_05_02
./cr_TABLESPACES.sh
```

## 练习6-1：管理普通用户和本地用户

### 总览

在这种实践中，您将管理CDB和PDB中的普通用户和本地用户。

### 任务

```SQL
# 1.查看cdb2中的所有普通用户和本地用户。
ORACLE_SID = cdb2
sqlplus / as sysdba
col username format a20
select USERNAME, COMMON, CON_ID from cdb_users;
select USERNAME, COMMON,CON_ID from cdb_users where username = 'SYSTEM';
select distinct username from cdb_users where common='YES';
select username,con_id from cdb_users where common='NO';
# 2.创建一个普通用户C##_USER。
 create user C##_USER identified by x CONTAINER=ALL ;
# 3.查看新用户C##_USER
select distinct username from cdb_users where username='C##_USER';
# 4.将CREATE SESSION授予用户C##_USER。
GRANT CREATE SESSION TO c##_user CONTAINER=ALL;
# 5. 连接到root，pdb2和pdb2_2 as c##_user user;
connect c##_user/x@pdb2
connect c##_user/x@pdb2_2
connect c##_user/x@cdb2
# 6. 在根容器中创建本地用户LOCAL_USER。
connect / as sysdba
create user local_user identified by x CONTAINER=CURRENT;
# 报错
# ERROR at line 1:
# ORA-65049: creation of local user or role is not allowed in
# CDB$ROOT
# 请注意，根目录中不可以授权任何本地用户！！
# 7. 创建一个普通用户并授予CREATE SESSION作为本地特权。
# a. 创建用户并授予特权。
create user C##_USER2 identified by x CONTAINER=ALL;
GRANT CREATE SESSION TO c##_user2;
# 尝试通过该用户登陆pdb2，pdb2_2,cdb2
CONNECT c##_user2/x@pdb2
CONNECT c##_user2/x@pdb2_2
CONNECT c##_user2/x@cdb2
# 注意，即使该用户是普通用户，权限也是需要在根路径下进行授予。
# b.删除该用户
CONNECT / as sysdba
DROP USER c##_user2;
# 8. 在pdb2中创建本地用户LOCAL_USER_PDB2
# a.查看pdb2中的所有用户
connect sys/oracle_4U@PDB2 as sysdba
col username format a30
select USERNAME, COMMON, CON_ID from cdb_users order by 1;
# 注意，您查看了当前PDB的所有普通用户和本地用户。
select username,common from dba_users;
# b.尝试在PDB2中创建一个公共用户C##_USER_PDB2。
create user c##_user_pdb2 identified by x CONTAINER=ALL;
# c.创建本地用户LOCAL_USER_PDB2在PDB2。
create user local_user_pdb2 identified by x CONTAINER=CURRENT;
select USERNAME,COMMON,CON_ID from cdb_users order by 1;
grant create session to local_user_pdb2;
# d.以LOCAL_USER_PDB2的身份连接到PDB2。
connect local_user_pdb2/x@PDB2
# e.以LOCAL_USER_PDB2的身份连接到PDB2_2。
connect local_user_pdb2/x@PDB2_2
# 请注意，它失败是因为LOCAL_USER_PDB2在根目录中不存在
# f.PDB的普通用户和本地用户概述：
connect sys/oracle_4U@PDB2_2 as sysdba
col username format a30
select USERNAME, COMMON, CON_ID from cdb_users order by 1;
select USERNAME, COMMON from dba_users order by username;
```



## 练习6-2：管理本地角色和公共角色

### 总览

在这种实践中，您将管理在CDB和PDB中创建为公共或本地角色，并被授予公共和/或本地角色。

### 假设条件

从先前的练习6-1 分别在 `cdb2` 和 `PDB2` 中成功创建了 `C##_USER` 和 `LOCAL_USER_PDB2` 。

### 任务

```SQL
# 1.管理CDB和PDB中角色的创建。
# a.列出CDB中的所有预定义角色。
connect / as sysdba
col role format a30
select ROLE, COMMON, CON_ID from cdb_roles order by role;

# b.在根目录中列出所有预定义的角色。
select ROLE, COMMON from dba_roles order by role;

# c.Create a common C##_ROLE in root.
create role c##_role container=ALL;

# d. Create a local LOCAL_ROLE in root.
create role local_role container=CURRENT;  
# CDB$ROOT中不允许创建本地用户或角色

# e.列出PDB PDB2中的所有预定义角色。
connect system / oracle_4U@PDB2
col role format a30
select ROLE, COMMON, CON_ID from cdb_roles;
# 您只能查看PDB的所有公共角色和本地角色。
select ROLE,COMMON from dba_roles order by role;
# f.在PDB2中创建通用角色。
create role c##_role_PDB2 container=ALL;
# 您收到一条错误消息，因为无法从PDB创建通用角色。
# g.在PDB2中创建本地角色。
create role local_role_PDB2 container=CURRENT;
select ROLE, COMMON from dba_roles order by role;

# 2.授予公共或本地角色为公共或本地角色。
# a.从根向普通用户授予普通角色。
connect / as sysdba
grant c##_role to c##_user;  
col grantee format A16
col GRANTED_ROLE format A16
select GRANTEE, GRANTED_ROLE, COMMON, CON_ID
from cdb_role_privs where grantee='C##_USER';
# 请注意，公共角色是在本地授予公共用户的。授予的角色仅适用于根。
connect c##_user/x
select * from session_roles;
connect c##_user/x@PDB2
select * from session_roles;
# b.现在，从根目录将通用角色授予通用用户，以适用于所有容器。
connect / as sysdba
grant c##_role to c##_user container=all;
select GRANTEE, GRANTED_ROLE, COMMON, CON_ID
from cdb_role_privs where grantee='C##_USER';
connect c##_user/x
select * from session_roles;
connect c##_user/x@PDB2
select * from session_roles;
# c.撤消公共用户的公共角色，以便该角色不能在任何容器中使用。
connect / as sysdba
revoke c##_role from c##_user container=all;
connect c##_user/x
select * from session_roles;
connect c##_user/x@PDB2  
select * from session_roles;

# d.从根目录向本地用户授予通用角色。
connect / as sysdba
grant c##_role to local_user_pdb2;
# 请注意，该用户的根目录未知。它是PDB2中的本地用户。
# e.从PDB2向本地用户授予通用角色。
connect system/oracle_4U@PDB2
grant c##_role to local_user_PDB2;
select GRANTEE, GRANTED_ROLE, COMMON, CON_ID
from cdb_role_privs where grantee='LOCAL_USER_PDB2';
# 请注意，仅在PDB PDB2中向用户授予本地公共角色（公共列= NO）。
# f.以本地用户身份测试连接。
connect local_user_pdb2/x@PDB2
select * from session_roles;
# g.从适用于所有容器的PDB2向本地用户授予公共角色。
connect system/oracle_4U@PDB2
grant c##_role to local_user_pdb2 container=all;
# 注意，不能从PDB全局授予公共角色。
# h.从PDB2向本地用户授予本地角色。
grant local_role_pdb2 to local_user_pdb2;
select GRANTEE, GRANTED_ROLE, COMMON, CON_ID
from cdb_role_privs where grantee='LOCAL_USER_PDB2';
# i.以本地用户身份测试连接。
connect local_user_pdb2/x@PDB2
select * from session_roles;  
```

## 练习6-3：管理本地和通用特权

### 总览

在这种实践中，您将管理在CDB和PDB中授予的普通或本地特权。

### 假设条件

`C##_USER` 和 `LOCAL_USER_PDB2` 被成功地从以前的做法5-2中创建
PDB2 的 cdb2 。

### 任务

```SQL
# 1.检查特权是创建为普通特权还是本地特权。
connect / as sysdba
desc sys.system_privilege_map
desc sys.table_privilege_map
# 请注意，没有COMMON列。特权既不能创建为通用特权，也不能创建为本地特权，但可以授予它们为通用特权或本地特权。

# 2.检查如何将CREATE SESSION系统特权授予C##_USER和LOCAL_USER_PDB2
connect system/oracle_4U
col grantee format a18
col privilege format a14
select GRANTEE, PRIVILEGE, COMMON, CON_ID
from cdb_sys_privs
where grantee in ('C##_USER', 'LOCAL_USER_PDB2');

connect system/oracle_4U@PDB2
select GRANTEE, PRIVILEGE, COMMON
from dba_sys_privs
where grantee in ('C##_USER', 'LOCAL_USER_PDB2');

# 3.向普通用户C##_USER授予系统特权CREATE TABLE和UNLIMITED TABLESPACE，以适用于任何容器。这将是一项普通特权。
connect system/oracle_4U
grant CREATE TABLE, UNLIMITED TABLESPACE to C##_USER CONTAINER=ALL;

# 4.向普通用户C##_USER授予系统特权CREATE SEQUENCE，使其仅适用于root用户。这将是本地特权。
col grantee format a12
grant CREATE SEQUENCE to C##_USER CONTAINER=CURRENT; Grant succeeded.
select GRANTEE, PRIVILEGE, COMMON, CON_ID 2 from cdb_sys_privs
where grantee = 'C##_USER';

# 5.向普通用户C##_USER授予系统特权CREATE SYNONYM，使其仅适用于PDB2。这将是本地特权。
connect system/oracle_4U@PDB2
col grantee format a18
grant CREATE SYNONYM to C##_USER CONTAINER=CURRENT;
select GRANTEE, PRIVILEGE, COMMON, CON_ID from cdb_sys_privs where grantee = 'C##_USER';  

# 6.向普通用户C##_USER授予系统特权CREATE VIEW，使其仅适用于root 用户，但已连接到PDB2中。
col grantee format a18  
grant CREATE VIEW to C##_USER CONTAINER=ALL;
# 请注意，您不能从PDB授予通用特权。

# 7.向本地用户LOCAL_USER_PDB2授予系统特权CREATE ANY TABLE，以适用于任何容器。
connect system/oracle_4U
col grantee format a18
grant CREATE ANY TABLE to LOCAL_USER_PDB2 CONTAINER=ALL;
# 请注意，该用户的root用户身份未知。它是PDB2中的本地用户。

# 8.向本地用户LOCAL_USER_PDB2授予系统特权CREATE ANY SEQUENCE，以仅在root 用户中适用。
grant CREATE ANY SEQUENCE to LOCAL_USER_PDB2 CONTAINER=CURRENT;

# 9.向本地用户LOCAL_USER_PDB2授予系统特权UNLIMITED TABLESPACE，使其仅适用于PDB2。这将是本地特权。
connect system/oracle_4U@PDB2
col grantee format a18
grant UNLIMITED TABLESPACE to LOCAL_USER_PDB2;
select GRANTEE, PRIVILEGE, COMMON, CON_ID from cdb_sys_privs where grantee = 'LOCAL_USER_PDB2';  

# 10.向本地用户LOCAL_USER_PDB2授予系统特权DROP ANY VIEW，使其仅适用于root用户，但已连接至PDB2。
grant DROP ANY VIEW to LOCAL_USER_PDB2 CONTAINER=ALL;
请注意，您不能授予将在另一个容器中应用的本地特权。  
```

## 练习6-4：使普通用户可以查看有关PDB对象的信息

### 总览

在这种实践中，您将管理普通用户的CONTAINER_DATA属性，以使普通用户可以查看有关特定PDB中PDB对象的信息。

### 任务

```SQL
# 1.查找有关默认（用户级别）和特定于对象的CONTAINER_DATA属性的信息，这些属性在DBA_CONTAINER_DATA数据字典视图中明确设置为DEFAULT以外的值。
CONNECT / as sysdba
COLUMN USERNAME FORMAT A10
COLUMN DEFAULT_ATTR FORMAT A7
COLUMN OWNER FORMAT A8
COLUMN OBJECT_NAME FORMAT A10
COLUMN ALL_CONTAINERS FORMAT A3
COLUMN CONTAINER_NAME FORMAT A10
COLUMN CON_ID FORMAT 999
set pages 100
set line 200
SELECT USERNAME, DEFAULT_ATTR, OWNER, OBJECT_NAME,
            ALL_CONTAINERS, CONTAINER_NAME, CON_ID
     FROM CDB_CONTAINER_DATA
     WHERE username NOT IN
           ('GSMADMIN_INTERNAL', 'APPQOSSYS', 'DBSNMP')
     ORDER BY OBJECT_NAME;
# 2.创建普通用户 c##jfv 并授予c##jfv 的系统权限CREATE SESSION和SET CONTAINER 。
CREATE USER c##jfv IDENTIFIED BY oracle_4U CONTAINER=ALL;
GRANT CREATE SESSION, SET CONTAINER TO c##jfv CONTAINER=ALL;

# 3.然后在V_$SESSION视图上授予c##jfv对象特权SELECT 。
GRANT SELECT ON sys.v_$session TO c##jfv CONTAINER=ALL;

# 4.创建另一个以SYS用户身份连接到pdb1_1的会话，并保持连接状态。
ORACLE_SID = cdb2
sqlplus sys/oracle_4U@pdb1_1 as sysdba
SHOW CON_NAME
ALTER SESSION SET CONTAINER=pdb1_1;
SHOW CON_NAME

# 5.在第一个会话中，您应该看到pdb1_1的一行。
SELECT username, con_id FROM v_$session
WHERE username IS NOT NULL AND username <> 'DBSNMP';

# 6.仍然在第一个会话中，以普通用户c##jfv身份连接。普通用户看不到V_$SESSION中与pdb1_1相关的任何信息。
CONNECT c##jfv/oracle_4U
SELECT username, con_id FROM sys.v_$session
WHERE username IS NOT NULL AND username <> 'DBSNMP';

# 7.启用普通用户c##jfv来查看V_$SESSION中与pdb1_1相关的信息。
CONNECT / AS SYSDBA
ALTER USER c##jfv
         SET CONTAINER_DATA = (CDB$ROOT, PDB1_1, PDB2_2)
         FOR V_$SESSION
         CONTAINER=CURRENT;

# 8.以普通用户c##jfv的身份连接pdb1_1以查看V$SESSION中与以下内容有关的信息：
CONNECT c##jfv/oracle_4U
SELECT username, con_id FROM sys.v_$session
WHERE username IS NOT NULL AND username <> 'DBSNMP';

# 9.查看对象上公共用户C##JFV的CONTAINER_DATA属性集
CONNECT / AS SYSDBA
Connected.
COLUMN USERNAME FORMAT A25
COLUMN DEFAULT_ATTR FORMAT A7
COLUMN OWNER FORMAT A15
COLUMN OBJECT_NAME FORMAT A15
COLUMN ALL_CONTAINERS FORMAT A3
COLUMN CONTAINER_NAME FORMAT A10
COLUMN CON_ID FORMAT 999
set pages 100
set line 200
SELECT USERNAME, DEFAULT_ATTR, OWNER, OBJECT_NAME,
            ALL_CONTAINERS, CONTAINER_NAME, CON_ID
     FROM CDB_CONTAINER_DATA
     WHERE username NOT IN
            ('GSMADMIN_INTERNAL', 'APPQOSSYS', 'DBSNMP')
     ORDER BY OBJECT_NAME;

```
