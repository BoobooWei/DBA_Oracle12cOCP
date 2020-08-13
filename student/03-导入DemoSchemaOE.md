# 导入DemoSchemaOE

> 2020.02.08 BoobooWei

## 1. 下载Demo

[Schema Demo Github](https://github.com/oracle/db-sample-schemas)

## 2. 解压查看Readme

```bash
[oracle@ocm db-sample-schemas-19.2]$ pwd
/u01/software/db-sample-schemas-19.2
[oracle@ocm db-sample-schemas-19.2]$ ll db-sample-schemas-19.2/
total 104
drwxr-xr-x 2 oracle oinstall    85 Feb  8 14:24 bus_intelligence
-rw-r--r-- 1 oracle oinstall   117 Aug 23 22:13 CONTRIBUTING.md
drwxr-xr-x 2 oracle oinstall   326 Feb  8 14:24 customer_orders
-rw-r--r-- 1 oracle oinstall  3633 Aug 23 22:13 drop_sch.sql
drwxr-xr-x 2 oracle oinstall   197 Feb  8 14:24 human_resources
drwxr-xr-x 2 oracle oinstall    79 Feb  8 14:24 info_exchange
-rw-r--r-- 1 oracle oinstall  1050 Aug 23 22:13 LICENSE.md
-rw-r--r-- 1 oracle oinstall  2740 Aug 23 22:13 mk_dir.sql
-rw-r--r-- 1 oracle oinstall 27756 Aug 23 22:13 mkplug.sql
-rw-r--r-- 1 oracle oinstall  7166 Aug 23 22:13 mksample.sql
-rw-r--r-- 1 oracle oinstall  6592 Aug 23 22:13 mkunplug.sql
-rw-r--r-- 1 oracle oinstall  6123 Aug 23 22:13 mkverify.sql
drwxr-xr-x 3 oracle oinstall  8192 Feb  8 15:07 order_entry
drwxr-xr-x 2 oracle oinstall  4096 Feb  8 14:24 product_media
-rw-r--r-- 1 oracle oinstall  5682 Aug 23 22:13 README.md
-rw-r--r-- 1 oracle oinstall  5263 Aug 23 22:13 README.txt
drwxr-xr-x 2 oracle oinstall  4096 Feb  8 14:24 sales_history
[oracle@ocm db-sample-schemas-19.2]$ ls db-sample-schemas-19.2/order_entry/
2002                     empdept.xsl       oe_p_cs.sql       oe_p_ko.sql.bak   oe_p_us.sql
bi_oe_oi.ctl             filelist.xml      oe_p_cs.sql.bak   oe_p_nl.sql       oe_p_us.sql.bak
bi_oe_oi.dat             loe_v3.sql        oe_p_cus.sql      oe_p_nl.sql.bak   oe_p_whs.sql
bi_oe_or.ctl             loe_v3.sql.bak    oe_p_cus.sql.bak  oe_p_n.sql        oe_p_whs.sql.bak
bi_oe_or.dat             oc_comnt.sql      oe_p_dk.sql       oe_p_n.sql.bak    oe_p_zhs.sql
bi_oe_pi.ctl             oc_comnt.sql.bak  oe_p_dk.sql.bak   oe_p_ord.sql      oe_p_zhs.sql.bak
bi_oe_pi.dat             oc_cre.sql        oe_p_d.sql        oe_p_ord.sql.bak  oe_p_zht.sql
ccus_v3.sql              oc_cre.sql.bak    oe_p_d.sql.bak    oe_p_pd.sql       oe_p_zht.sql.bak
ccus_v3.sql.bak          oc_drop.sql       oe_p_el.sql       oe_p_pd.sql.bak   oe_views.sql
cidx_v3.sql              oc_drop.sql.bak   oe_p_el.sql.bak   oe_p_pi.sql       oe_views.sql.bak
cidx_v3.sql.bak          oc_main.sql       oe_p_esa.sql      oe_p_pi.sql.bak   pcus_v3.sql
cmnt_v3.sql              oc_main.sql.bak   oe_p_esa.sql.bak  oe_p_pl.sql       pcus_v3.sql.bak
cmnt_v3.sql.bak          oc_popul.sql      oe_p_e.sql        oe_p_pl.sql.bak   poe_v3.sql
coe_v3.sql               oc_popul.sql.bak  oe_p_e.sql.bak    oe_p_ptb.sql      poe_v3.sql.bak
coe_v3.sql.bak           oe_analz.sql      oe_p_frc.sql      oe_p_ptb.sql.bak  pord_v3.sql
coe_xml.sql              oe_analz.sql.bak  oe_p_frc.sql.bak  oe_p_pt.sql       pord_v3.sql.bak
coe_xml.sql.bak          oe_comnt.sql      oe_p_f.sql        oe_p_pt.sql.bak   PurchaseOrders.dmp
cord_v3.sql              oe_comnt.sql.bak  oe_p_f.sql.bak    oe_p_ro.sql       purchaseOrder.xml
cord_v3.sql.bak          oe_cre.sql        oe_p_hu.sql       oe_p_ro.sql.bak   purchaseOrder.xsd
createFolders.sql        oe_cre.sql.bak    oe_p_hu.sql.bak   oe_p_ru.sql       purchaseOrder.xsl
createFolders.sql.bak    oe_drop.sql       oe_p_inv.sql      oe_p_ru.sql.bak   pwhs_v3.sql
createResources.sql      oe_drop.sql.bak   oe_p_inv.sql.bak  oe_p_sf.sql       pwhs_v3.sql.bak
createResources.sql.bak  oe_idx.sql        oe_p_i.sql        oe_p_sf.sql.bak   xdb03usg.sql
createUser.sql           oe_idx.sql.bak    oe_p_i.sql.bak    oe_p_sk.sql       xdb03usg.sql.bak
createUser.sql.bak       oe_main.sql       oe_p_itm.sql      oe_p_sk.sql.bak   xdbConfiguration.sql
cwhs_v3.sql              oe_main.sql.bak   oe_p_itm.sql.bak  oe_p_s.sql        xdbConfiguration.sql.bak
cwhs_v3.sql.bak          oe_oc_.log        oe_p_iw.sql       oe_p_s.sql.bak    xdbSupport.sql
doe_v3.sql               oe_p_ar.sql       oe_p_iw.sql.bak   oe_p_th.sql       xdbSupport.sql.bak
doe_v3.sql.bak           oe_p_ar.sql.bak   oe_p_ja.sql       oe_p_th.sql.bak   xdbUtilities.sql
doe_xml.sql              oe_p_ca.sql       oe_p_ja.sql.bak   oe_p_tr.sql       xdbUtilities.sql.bak
doe_xml.sql.bak          oe_p_ca.sql.bak   oe_p_ko.sql       oe_p_tr.sql.bak

```

## 3. 通过`oe_main.sql`开始导入OE schema

```sql
[oracle@ocm order_entry]$ cd db-sample-schemas-19.2/order_entry/
[oracle@ocm order_entry]$ pwd
/u01/software/db-sample-schemas-19.2/db-sample-schemas-19.2/order_entry
[oracle@ocm order_entry]$ sed -i 's@__SUB__CWD__@\/u01\/software\/db-sample-schemas-19.2\/db-sample-schemas-19.2\/order_entry\/@g' *.sql
[oracle@ocm order_entry]$ sqlplus dba1/oracle@emrep
> @oe_main.sql
oracle@ocm order_entry]$ sqlplus / as sysdba

SQL*Plus: Release 12.2.0.1.0 Production on Sat Feb 8 15:00:57 2020

Copyright (c) 1982, 2016, Oracle.  All rights reserved.


Connected to:
Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production

SQL> conn dba1/oracle@emrep
Connected.
SQL> @oe_main.sql

specify password for OE as parameter 1:
Enter value for 1: oe

specify default tablespeace for OE as parameter 2:
Enter value for 2: EXAMPLE

specify temporary tablespace for OE as parameter 3:
Enter value for 3: TEMP

specify password for HR as parameter 4:
Enter value for 4: hr

specify password for SYS as parameter 5:
Enter value for 5: WLS3Gg5_2

specify directory path for the data files as parameter 6:
Enter value for 6: /u01/app/oracle/oradata/booboo/booboopdb1/

writeable directory path for the log files as parameter 7:
Enter value for 7: 
SP2-0137: DEFINE requires a value following equal sign

specify version as parameter 8:
Enter value for 8: v3

specify connect string as parameter 9:
Enter value for 9: localhost:1521/emrep

Enter value for log_path: /tmp/

...此处省略
```

## 4. 检查

```sql
SQL> conn oe/oe@emrep
Connected.
SQL> select tname from tab;

TNAME
--------------------------------------------------------------------------------
ACCOUNT_MANAGERS
ACTION_TABLE
BOMBAY_INVENTORY
CATEGORIES_TAB
COUNTRIES
CUSTOMERS
CUSTOMERS_VIEW
DEPARTMENTS
EMPLOYEES
INVENTORIES
JOBS

TNAME
--------------------------------------------------------------------------------
JOB_HISTORY
LINEITEM_TABLE
LOCATIONS
OC_CORPORATE_CUSTOMERS
OC_CUSTOMERS
OC_INVENTORIES
OC_ORDERS
OC_PRODUCT_INFORMATION
ORDERS
ORDERS_VIEW
ORDER_ITEMS

TNAME
--------------------------------------------------------------------------------
PRODUCTS
PRODUCT_DESCRIPTIONS
PRODUCT_INFORMATION
PRODUCT_PRICES
PRODUCT_REF_LIST_NESTEDTAB
PROMOTIONS
PURCHASEORDER
SUBCATEGORY_REF_LIST_NESTEDTAB
SYDNEY_INVENTORY
TORONTO_INVENTORY
WAREHOUSES

33 rows selected.
```

## 5. 使用`oe_drop.sql`删除OE schema，不删除用户



Tips：

1. 学会看帮助文档 README.md
2. 导入的过程中，弄明白每一个传参的含义，例如：第八个参数为何要填写`v3`


## 其他

### Centos 7 装插件rlwrap
```sql
# 安装插件rlwrap，sqlplus可以光标回退，命令回显，使用方向键操作命令行
wget ftp://ftp.pbone.net/mirror/archive.fedoraproject.org/epel/7.2019-05-29/x86_64/Packages/r/rlwrap-0.43-2.el7.x86_64.rpm
yum localinstall -y rlwrap-0.43-2.el7.x86_64.rpm
 
 
# 在.bashrc中增加调用sqlplus的别名
vi .bashrc
------------------------------------
alias sqlplus='rlwrap sqlplus'
 
# 使.bashrc新增的内容生效
source .bashrc
# 启动sqlplus
sqlplus /nolog
sqlplus / as sysdba
 
# 美化结果集的脚本文件
vi /home/oracle/login.sql
-------------------------
set linesize 120
set pagesize 500
 
 
# 增加环境变量使sqlplus永远能读取/home/oracle/login.sql
vi .bashrc
---------------------------
export SQLPATH=/home/oracle
 
 
# 使.bashrc新增的内容生效
source .bashrc
```

### OCP考试练习环境的创建

> 再练习一次

```bash
# 1.为cdb1的新PDB数据文件创建存放目录。
cd $ORACLE_BASE/oradata/orcl
mkdir ocp
 
# 2.运行SQL * Plus，并使用CREATE PLUGGABLE DATABASE与用户连接到根目录
sqlplus / as sysdba
CREATE PLUGGABLE DATABASE ocp ADMIN USER ocp
IDENTIFIED BY oracle_4U ROLES=(CONNECT)
FILE_NAME_CONVERT=('/u01/app/oracle/oradata/orcl/pdbseed',
  '/u01/app/oracle/oradata/orcl/ocp');
 
 
 
# 3.检查ocp的打开模式。
col con_id format 999
col name format A10
select con_id, NAME, OPEN_MODE,DBID, CON_UID from V$PDBS;
 
 
# 4.打开ocp。
alter pluggable database ocp open;
 
 
# 登陆
sqlplus / as sysdba
alter session set container=ocp;
conn ocp/oracle as sysdba
show con_name
# 使用OCP sysdba用户登陆到 pdb中
sqlplus sys/WLS3Gg5_2@127.0.0.1:1521/ocp.ilt.example.com as sysdba
 
 
# 使用oe业务用户登陆到pdb中
conn oe/oracle@127.0.0.1:1521/ocp.ilt.example.com
show con_name
```

### 检查

```
# 检查用户是否创建成功
column account_status format a20
column username format a20
column password format a20
select username,password,account_status from dba_users where username='OE';
 
USERNAME         PASSWORD         ACCOUNT_STATUS
-------------------- -------------------- --------------------
OE                    OPEN
 
 
# 创建用户的表
column OWNER format a20
column TABLE_NAME format a20
select owner, table_name from dba_tables where owner='OE';
 
OWNER            TABLE_NAME
-------------------- --------------------
OE           CUSTOMERS
OE           WAREHOUSES
OE           ORDER_ITEMS
OE           ORDERS
OE           INVENTORIES
OE           PRODUCT_INFORMATION
OE           PRODUCT_DESCRIPTIONS
OE           PROMOTIONS
 
 
# 查看OE的系统权限
select privilege from dba_sys_privs where GRANTEE='OE';
 
PRIVILEGE
----------------------------------------
CREATE MATERIALIZED VIEW
UNLIMITED TABLESPACE
CREATE VIEW
QUERY REWRITE
CREATE SYNONYM
CREATE SESSION
CREATE DATABASE LINK
 
 
# 查看用户被授予的对象权限
col GRANTEE for a15;
col PRIVILEGE for a20;
col owner for a15;
SELECT GRANTEE,PRIVILEGE,OWNER,TABLE_NAME FROM DBA_TAB_PRIVS WHERE GRANTEE='OE';
 
 
 
GRANTEE     PRIVILEGE        OWNER       TABLE_NAME
--------------- -------------------- --------------- --------------------
OE      EXECUTE          SYS         DBMS_STATS
OE      READ             SYS         SUBDIR
OE      READ             SYS         SS_OE_XMLDIR
OE      WRITE            SYS         SUBDIR
OE      WRITE            SYS         SS_OE_XMLDIR
 
 
# 修改秘密
alter user oe identified by oracle;
 
 
# 登陆
conn oe/oracle@127.0.0.1:1521/ocp.ilt.example.com
show con_name
 
CON_NAME
------------------------------
OCP
 
 
# 查看OE用户有权限的表
SQL>  select table_name from tabs;
 
TABLE_NAME
--------------------------------------------------------------------------------
CUSTOMERS
WAREHOUSES
ORDER_ITEMS
ORDERS
INVENTORIES
PRODUCT_INFORMATION
PRODUCT_DESCRIPTIONS
PROMOTIONS
 
8 rows selected.
```
