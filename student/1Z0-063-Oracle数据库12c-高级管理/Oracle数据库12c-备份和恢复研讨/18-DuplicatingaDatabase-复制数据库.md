# 第18课：复制数据库

> 2020.04.19 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [第18课练习：复制数据库](#第18课练习：复制数据库)   
   - [实践概述](#实践概述)   
   - [练习18-1：复制数据库](#练习18-1：复制数据库)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   

<!-- /MDTOC -->

## 实践概述

在这些实践中，您将准备然后复制数据库。

## 练习18-1：复制数据库

### 总览

在这种实践中，您将学习如何复制活动数据库。ORCL是活动数据库，将被复制或克隆为DBTEST。这些任务包括以下内容：

* 使用Oracle Net连接，密码文件和最小的initdbtest.ora文件为将来的DBTEST数据库准备目标。
* 确认源数据库配置设置。
* 使用RMAN 复制ORCL数据库。
* 测试对克隆数据库的访问。

### 假设条件

打开两个终端窗口，您以oracle OS用户身份登录。$LABS是当前目录。为orcl数据库实例设置了环境变量。

### 任务

```sql
# 1.为dbtest数据库创建新目录。
cd /u01/app/oracle
mkdir dbtest_fra
cd dbtest_fra
mkdir orcl
cd ..
cd oradata
ls
emrep orcl rcat
mkdir dbtest
ls -l
cd $LABS
pwd
# 2.使用netca实用程序准备Oracle Net连接。将dbtest条目添加到tnsnames.ora 文件。
netca
# a.在Oracle Net Configuration Assistant：欢迎页面上，选择本地网络服务命名配置，然后单击下一步。
# b.确认已选择添加，然后单击下一步。
# c.输入dbtest作为服务名称，然后单击下一步。
# d.选择TCP作为协议，然后单击下一步。
# e.在主机名字段中输入您的主机名和域名（例如edRSr39p1.us.oracle.com）。
# 如果不确定格式，请在下一步中执行命令以查看当前活动的示例。
# f.确认已选择使用标准端口号1521，然后单击Next。
# g.选择No，不测试，因为您的dbtest实例尚不存在，然后单击 Next。
# h.确认dbtest为Net Service Name，然后单击 Next。
# i.对问题的答案为否：您是否要配置另一个网络服务名称然后单击 Next。
# j.您应该看到以下消息：网络服务名称配置完成！请点击Next。
# k.点击完成。您将看到成功的配置消息。
netca
# 创建的网络服务名称：dbtest
# 3.在$ORACLE_HOME/network/admin/tnsnames.ora文件中查看DBTEST条目。
cat $ORACLE_HOME/network/admin/tnsnames.ora
# 4.将DBTEST条目与ORCL配置进行比较，您会发现（SERVER = DEDICATED）丢失了。
# a.使用gedit或vi编辑器更新tnsnames.ora文件，以便DBTEST
 gedit $ORACLE_HOME/network/admin/tnsnames.ora
# b.完成编辑后，请确认当前输入为：
DBTEST =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST =
edRSr39p1.us.oracle.com)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = dbtest)
    )
)
# 5.为将来的DBTEST数据库创建一个密码文件，以允许OS身份验证。
# a.以oracle OS用户身份，将环境变量指向dbtest实例。
. oraenv
# b.通过使用orapwd实用程序创建$ORACLE_HOME/dbs/orapwdbtestfile
orapwd file=$ORACLE_HOME/dbs/orapwdbtest entries=15 password=oracle_4U
# c.(可选）确认文件存在。
ls $ORACLE_HOME/dbs/
# 6. 在同一目录中，使用以下条目创建一个最小的initdbtest.ora文件：
vi $ORACLE_HOME/dbs/initdbtest.ora
db_name=dbtest
remote_login_passwordfile=exclusive
# 7.将环境变量指向dbtest实例（如步骤7a中所示）。以SYSDBA身份登录到SQL * Plus ，使用initdbtest.ora文件以NOMOUNT模式启动dbtest实例。然后退出。
 . oraenv
sqlplus / as sysdba
startup NOMOUNT pfile='/u01/app/oracle/product/12.1.0/dbhome_1/dbs/initdbtest.or a'
exit
# 8.确认SQL * Plus中的ORCL源数据库配置设置。
# a.在环境变量指向orcl的情况下，以SYSBDA身份登录到SQL * Plus
show parameters compatible
# b.确认您的备份位置和大小。如果FRA小于10G，请使用以下命令将其放大：
show parameters recovery_f
ALTER SYSTEM SET db_recovery_file_dest_size = 10G SCOPE=BOTH;
# c.确认数据库处于存档日志模式。然后退出。
SELECT NAME, LOG_MODE, OPEN_MODE FROM V$DATABASE;
exit
# 注意：如果您的值不同，请与您的讲师讨论准备要复制的数据库可能需要采取的步骤。
# 9.在克隆会话中，将$TNS_ADMIN环境变量设置为oracle OS用户，因为服务器进程将尝试使用来解析AUXILIARY服务名称。
export TNS_ADMIN=/u01/app/oracle/product/12.1.0/dbhome_1/network/admin
echo $TNS_ADMIN
# 10.使用RMAN将ORCL数据库复制为DBTEST数据库。
# 请注意，对于目标和目录连接，使用服务名称，但不用于辅助连接。因此，您必须为dbtest实例设置环境变量。
 . oraenv
echo $ORACLE_SID
rman target sys/<password>@orcl auxiliary sys/<password> catalog rcatowner/oracle_4U@rcat
# 注意：建议您不要在命令行上输入密码。您仅使用这种方法来避免歧义（并使您能够复制并粘贴）。
# 11. 使用以下命令将ORCL数据库复制为DBTEST数据库：
DUPLICATE TARGET DATABASE TO dbtest FROM ACTIVE DATABASE
SPFILE PARAMETER_VALUE_CONVERT
'/u01/app/oracle/oradata/orcl','/u01/app/oracle/oradata/dbtest',
'/u01/app/oracle/fast_recovery_area','/u01/app/oracle/dbtest_fra', 'ORCL','DBTEST'
SET DB_RECOVERY_FILE_DEST_SIZE='10G';
#（可选）查看输出。查找RMAN为您执行的主要步骤：
# 在恢复目录中注册新数据库
# 克隆SPFILE
# 克隆控制文件
# 克隆数据文件
# 切换到数据文件副本
# 执行媒体恢复
# 在RESETLOGS模式下打开DBTEST
# 12.退出RMAN。
exit
# 13.确认新创建的dbtest数据库的可用性和可访问性。
. oraenv
sqlplus / as sysdba
select dbid, name, created, open_mode from v$database;
exit
```
