# 第12课：执行恢复的实践II

> 2020-04-19 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [第12课：执行恢复的实践II](#第12课：执行恢复的实践ii)   
   - [实践概述](#实践概述)   
   - [练习12-1：恢复参数文件的丢失](#练习12-1：恢复参数文件的丢失)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习12-2：还原控制文件](#练习12-2：还原控制文件)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习12-3：从丢失的所有控制文件中恢复](#练习12-3：从丢失的所有控制文件中恢复)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习12-4：还原密码文件](#练习12-4：还原密码文件)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习12-5：恢复临时文件](#练习12-5：恢复临时文件)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习12-6：创建加密备份](#练习12-6：创建加密备份)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习12-7：使用加密备份进行恢复](#练习12-7：使用加密备份进行恢复)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习12-8：恢复丢失的加密钱包](#练习12-8：恢复丢失的加密钱包)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   

<!-- /MDTOC -->

## 实践概述

在这些实践中，您将从许多不同的数据库故障中恢复。强烈建议您完成开始的步骤，因为这会影响以下做法。

## 练习12-1：恢复参数文件的丢失

### 总览

在这种情况下，您将通过删除initorcl.ora参数文件来创建问题。创建问题后，必须还原参数文件。

### 假设条件

假定数据库是完整备份。假定在快速恢复区域中配置了控制文件和SPFILE的自动备份。

打开两个终端窗口，您以oracle OS用户身份登录。`$LABS`是当前目录。为`orcl`实例设置了环境变量。

### 任务

```SQL
# 1.通过执行$ LABS目录中的setup_12_01.sh脚本进行设置。
# 在此脚本中：
# • 该initorcl.ora文件从删除$ ORACLE_HOME / DBS目录下;
# • 创建了新的表空间和用户。
# • 用户创建一个表并填充它。
# • 执行表空间的备份，然后更新表。
# • 输出在/tmp/setup.log文件中。
./setup_12_01.sh
# 2.通过执行$ LABS中的break_12_01.sh脚本，导致数据库故障目录。输出在/tmp/break.log文件中。
./break_12_01.sh
# 3.尝试启动数据库实例。注意错误消息。从SQL * Plus退出。
sqlplus / as sysdba
startup
exit
# 4.使用RMAN启动数据库。
rman target "'/ as sysbackup'"
startup;
# 注意：数据库已使用虚拟参数文件启动，以允许还原SPFILE。
# 5.恢复SPFILE。因为数据库是用虚拟参数文件启动的，所以必须指定自动备份的位置。在这种情况下，我们使用恢复区和DB_NAME选项指定可以在哪里找到自动备份。
restore spfile from autobackup recovery area '/u01/app/oracle/fast_recovery_area' db_name 'orcl';
# 6.关闭数据库实例，然后使用还原的SPFILE重新启动它。
shutdown;
startup;
# 7.执行cleanup_12_01.sh脚本以清除该做法。输出在/tmp/cleanup.log 文件。
./cleanup_12_01.sh
# 8.为了准备下一步操作，请备份您的orc l数据库，删除过时的备份，并确保未列出任何故障。然后从RMAN退出。
BACKUP DATABASE PLUS ARCHIVELOG DELETE INPUT;
delete noprompt obsolete;
list failure;
exit
```


## 练习12-2：还原控制文件

### 总览

在这种实践中，您首先通过删除control02.ctl控制文件来创建恢复问题。创建问题后，必须还原此单个“丢失”的控制文件。

### 假设条件

数据库的完整备份可用。已配置将控制文件和SPFILE自动备份到快速恢复区域。

打开两个终端窗口，您以oracle OS用户身份登录。`$LABS`是当前目录。为`orcl`实例设置了环境变量。

### 任务

```SQL
# 1.通过执行$ LABS目录中的setup_12_02.sh脚本进行设置。
# 在此脚本中，创建了一个新的表空间和用户。用户创建一个表并填充它。执行表空间的备份，然后更新表。
# 输出将重定向到/tmp/setup.log，并且可以在脚本执行期间在此处查看。
./setup_12_02.sh
# 2.通过执行$ LABS中的break_12_02.sh脚本，导致数据库故障,输出在/tmp/break.log文件中。
./break_12_02.sh
# 3.尝试启动数据库。查看错误消息。然后退出SQL * Plus。
sqlplus / as sysdba
startup
exit
# 4.查看警报日志。滚动到最新条目，以查看此实践中的错误。
adrci
set editor gedit
show alert
# 5. 单击关闭窗口图标（x）关闭gedit窗口，然后退出adrci。
exit
# 6.检查Data Recovery Advisor以获取解决方案。在执行之前，预览建议的解决方案。
rman target "'/ as sysbackup'"
LIST FAILURE;
ADVISE FAILURE;
REPAIR FAILURE PREVIEW;
# 7.恢复控制文件。您可以通过RMAN命令行执行命令，也可以使用REPAIR FAILURE命令为您执行任务。
# 注意：控制文件的任何现有副本均可用于还原丢失的副本。当提示您执行修复并打开数据库时，输入y或yes。
REPAIR FAILURE;
# 8.使用LIST FAILURE命令来验证故障是否已修复。
list failure;
# 9.通过在另一个终端窗口中运行cleanup_12_02.sh脚本来清理练习环境。输出在/tmp/cleanup.log文件中。
./cleanup_12_02.
# 10.为准备下一个练习，请备份您的orcl数据库，删除过时的备份，并确保未列出任何故障。
BACKUP DATABASE PLUS ARCHIVELOG DELETE INPUT;
delete noprompt obsolete;
list failure;
exit
```

## 练习12-3：从丢失的所有控制文件中恢复

### 总览

在这种情况下，您将通过删除控制文件来创建问题。创建问题后，您必须恢复控制文件。

### 假设条件

数据库的完整备份可用。配置了控制文件和SPFILE的自动备份。

打开两个终端窗口，您以oracle OS用户身份登录。`$LABS`是当前目录。为`orcl`实例设置了环境变量。

### 任务

```SQL
# 1.通过执行$LABS目录中的setup_12_03.sh脚本进行设置。
# 在此脚本中，创建了一个新的表空间和用户。
# 用户创建一个表并填充它。执行表空间的备份，然后更新表。
# 输出将重定向到/tmp/setup.log，并且可以在脚本执行期间查看。
./setup_12_03.sh
# 2.通过执行$ LABS中的break_12_03.sh脚本，导致数据库失败。
# 输出在/tmp/break.log文件中。
./break_12_03.sh
# 3.登录到SQL * Plus，然后尝试启动数据库。注意错误消息。退出
sqlplus / as sysdba
startup
exit
# 4.查看警报日志。滚动到最新条目，以查看此实践中的错误。
adrci
set editor gedit
show alert
# 5.通过单击x图标关闭gedit窗口退出查看警报日志。退出adrci
# 通过输入Q，然后退出。
exit
# 6.使用RMAN LIST FAILURE和ADVISE FAILURE命令确定故障和建议的解决方案。
rman target "'/ as sysbackup'"
list failure;
advise failure;
# 7.查看REPAIR FAILURE PREVIEW命令生成的命令。
repair failure preview;
# 8.使用RMAN命令行还原控制文件并装入数据库。
# 注意：如果此时使用REPAIR FAILURE命令，则会创建新的失败，因此必须恢复数据库，并且您需要再次执行以下RMAN命令来完成数据库恢复：
LIST FAILURE;
ADVISE FAILURE;
REPAIR FAILURE PREVIEW;
REPAIR FAILURE;
#  然后继续执行步骤12。
# 如果不确定，请完全按照说明进行以下步骤：
restore controlfile from autobackup;
ALTER DATABASE MOUNT;
# 9.尝试打开数据库。
ALTER DATABASE OPEN;
# 问题：为什么需要 RESETLOGS？
# 答案：RESETLOGS 是必需的，因为还原的控制文件中的SCN与数据文件中记录的SCN不匹配。
# 10.尝试使用RESETLOGS选项打开数据库。
ALTER DATABASE OPEN RESETLOGS;
# 问题：为什么使用 RESETLOGS选项仍无法打开数据库？
# 答案：控制文件中的SCN早于数据文件中的SCN，并且由于UNTIL原因尚未恢复数据文件。需要恢复数据库，以便控制文件可以与数据文件同步。
# 11.恢复数据库。
recover database;
# 12.使用RESETLOGS打开数据库。
ALTER DATABASE OPEN RESETLOGS;
# 13.（可选）查询V$DATABASE以查看DBID和CURRENT_SCN的值。
SELECT NAME, DBID, CURRENT_SCN, LOG_MODE, OPEN_MODE FROM V$DATABASE;
# 14.使用LIST FAILURE命令来验证故障是否已修复。
list failure;
# 15.在另一个终端窗口中，使用以下命令清理练习环境。
# $LABS 目录中的 cleanup_12_03.sh 脚本。输出在 /tmp/cleanup.log 文件中。
./cleanup_12_03.sh
# 16.为准备下一个练习，请备份您的orc l数据库，删除过时的备份，并确保未列出任何故障。然后从RMAN退出。
BACKUP DATABASE PLUS ARCHIVELOG DELETE INPUT;
delete noprompt obsolete;
list failure;
exit
```

## 练习12-4：还原密码文件

### 总览

在这种情况下，您可以从丢失数据库密码文件中恢复过来。SYSDBA特权用户对数据库进行远程访问需要数据库密码。

### 假设条件

数据库的完整备份可用。

打开两个终端窗口，您以oracle OS用户身份登录。`$LABS`是当前目录。为`orcl`实例设置了环境变量。

### 任务

```SQL
# 1.通过执行$ LABS目录中的break_12_04.sh脚本，导致数据库失败。输出将重定向到/tmp/break.log，并且可以在脚本执行期间查看。
./break_12_04.sh
# 2.尝试使用远程连接连接到数据库。注意错误消息。
sqlplus sys@orcl as sysdba
# 3.检查密码文件是否存在。Linux和UNIX系统的orcl数据库密码文件的名称为$ORACLE_HOME/dbs/orapworcl.ora
ls $ORACLE_HOME/dbs/orapw*
# 注意：保护您的密码文件和标识密码文件位置的环境变量对于系统安全至关重要。有权访问这些权限的任何用户都可能会损害连接的安全性。
# 4.（可选）查看orapwd参数的描述。在终端窗口中调用orapwd。
orapwd
# 5.使用orapwd实用程序创建一个新的密码文件。
orapwd FILE=$ORACLE_HOME/dbs/orapworcl ENTRIES=15
# 注意：当超出分配的密码条目数量时，必须创建一个新的密码文件。为避免这种必要，请分配比您认为需要的条目更多的条目。
# 6.测试远程SYSDBA登录。现在应该成功了。从SQL * Plus退出。
sqlplus sys@orcl as sysdba
# 7.（可选）查看V$PWFILE_USERS视图。
desc V$PWFILE_USERS
SELECT * FROM V$PWFILE_USERS;
# 8.退出SQL * Plus。
exit
```

## 练习12-5：恢复临时文件

### 总览

在这种情况下，您将通过删除initorcl.ora参数文件来创建问题。创建问题后，必须还原参数文件。

### 假设条件

假定数据库是完整备份。假定在快速恢复区域中配置了控制文件和SPFILE的自动备份。

打开两个终端窗口，您以oracle OS用户身份登录。`$LABS`是当前目录。为`orcl`实例设置了环境变量。

### 任务

```SQL
# 1.通过执行$ LABS目录中的setup_12_05.sh脚本进行设置。
# 在此脚本中，创建了一个新的表空间和用户。用户创建一个表，然后填充它
# 并使用ORDER BY子句执行查询。（其中一些活动使用临时文件。）执行表空间的备份，然后更新表。
# 输出将重定向到/tmp/setup.log，并且在脚本执行期间也可以在此处查看。
./setup_12_05.sh
# 2.通过执行$ LABS中的break_12_05.sh脚本，导致数据库失败。
# 输出在/tmp/break.log文件中。
./break_12_05.sh
# 3.启动数据库实例。从SQL * Plus退出。
sqlplus / as sysdba
startup
exit
# 4.查看orcl警报日志的末尾。最新的条目显示，如果需要，启动处理包括重新创建临时文件。
# 使用adrci命令行实用程序（如先前的实践所示），或以SYSMAN用户身份登录EM Cloud Control 。
# a.在orcl数据库主页上，浏览：Oracle数据库>日志>警报日志
# 单击内容，然后单击切换到文本警报日志内容。
# b.输入包括执行启动操作时间的日期和时间值，然后单击“执行”。
# c.您可以滚动浏览警报日志的所选部分，或者使用浏览器的搜索功能（Firefox：Ctrl + F，然后输入搜索词）来查看重新创建的tempfile条目，如屏幕截图所示。
# 5.注销并关闭所有窗口。
```


## 练习12-6：创建加密备份

### 总览

在这种实践中，您将创建一个加密的备份，如果备份媒体丢失，该备份可以防止数据泄露。

在此示例中，您将使用取决于加密钱包的透明加密。如果加密钱包丢失，则备份不可恢复。

为了减轻钱包的丢失或允许在另一台计算机上恢复备份，可以使用密码加密而不是透明加密，或者同时使用两者，以便钱包或密码将使备份得以恢复。

### 假设条件

打开两个终端窗口，您以oracle OS用户身份登录。`$LABS`是当前目录。为`orcl`实例设置了环境变量。

### 任务

```SQL
# 1.准备要加密的数据库。
# a.为您的orcl数据库实例设置环境变量。如果不存在，请为Oracle钱包创建一个名为$ORACLE_BASE/admin/orcl/wallet的目录。
ls $ORACLE_BASE/admin/orcl/wallet
mkdir -p $ORACLE_BASE/admin/orcl/wallet
# b.编辑$ORACLE_HOME/network/admin/sqlnet.ora文件以添加以下行：
ENCRYPTION_WALLET_LOCATION=
          (SOURCE =
              (METHOD = FILE)
                 (METHOD_DATA =
(DIRECTORY = /u01/app/oracle/admin/orcl/wallet)))

gedit /u01/app/oracle/product/12.1.0/dbhome_1/network/admin/sqlnet.ora
# 2.创建一个基于密码的密钥库并对其进行备份。
# a. 以SYSDBA或SYSKM用户身份连接到orcl数据库实例
sqlplus / as sysdba
# b.创建一个基于密码的密钥库。
ADMINISTER KEY MANAGEMENT CREATE KEYSTORE '/u01/app/oracle/admin/orcl/wallet' IDENTIFIED BY secret;
# c.确认钱包存在。
! ls -l /u01/app/oracle/admin/orcl/wallet
# d.打开密钥库。
ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY secret;
# e.（可选）在数据字典中查看有关钱包的信息。
SELECT WRL_PARAMETER, STATUS, WALLET_TYPE FROM V$ENCRYPTION_WALLET;
# f.生成主加密密钥。
ADMINISTER KEY MANAGEMENT SET KEY IDENTIFIED BY secret WITH BACKUP USING 'test';
# g.在生成主密钥之前，请验证是否已备份密钥库。
!ls -l /u01/app/oracle/admin/orcl/wallet
# h.生成另一个密钥并查看密钥库文件。
ADMINISTER KEY MANAGEMENT SET KEY IDENTIFIED BY secret WITH BACKUP;
!ls -l /u01/app/oracle/admin/orcl/wallet
# i.备份包含当前主密钥的密钥库。从SQL * Plus退出。
ADMINISTER KEY MANAGEMENT BACKUP KEYSTORE IDENTIFIED BY secret;
!ls -l /u01/app/oracle/admin/orcl/wallet
exit
# 3.使用RMAN创建带有密码的透明加密备份。使用lab_12_06_02.rman 脚本。
# a.与往常一样，在执行脚本之前先检查一下脚本是一个好主意。
cat lab_12_06_02.rman
# b.执行lab_12_06_02.rman脚本。
rman target "'/ as sysbackup'" @lab_12_06_02.rman

# 4.验证备份件是否已加密。使用lab_12_06_03.sql脚本。
cat lab_12_06_03.sql
sqlplus / as sysdba @lab_12_06_03.sql
```

## 练习12-7：使用加密备份进行恢复

### 总览

在这种情况下，您将使用加密备份来恢复丢失的数据文件。

### 假设条件

练习12-6已完成。存在一个加密钱包，并且已创建使用透明加密的完整数据库备份

打开两个终端窗口，您以oracle OS用户身份登录。`$LABS`是当前目录。为`orcl`实例设置了环境变量。

### 任务

```SQL
# 1.通过执行$ LABS目录中的setup_12_07.sh脚本进行设置。
# 在此脚本中，创建了一个新的表空间和用户。
# 用户创建一个表并填充它。执行表空间的备份，然后更新表。输出在/tmp/setup.log文件中。
./setup_12_07.sh
# 2.通过执行$LABS中的break_12_07.sh脚本，导致数据库失败。输出在/tmp/break.log文件中。
./break_12_07.sh
# 3.尝试启动数据库。注意错误消息。
sqlplus / as sysdba
startup
# 4.在另一个终端窗口中，使用LIST FAILURE和ADVISE FAILURE命令诊断问题。
rman target "'/ as sysbackup'"
LIST FAILURE;
ADVISE FAILURE;
# 5.使用REPAIR FAILURE PREVIEW命令查看修复命令。
repair failure preview;
# 6.（可选）执行修复，然后预见错误。
REPAIR FAILURE;
# 7.由于数据库已重新启动，并且加密钱包未配置为自动登录钱包，因此必须先打开加密钱包，然后才能开始恢复。以SYSDBA或 SYSKM身份登录到SQL * Plus ，打开密钥库，然后退出。
ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY secret;
exit;
# 8.在RMAN会话中，修复故障并打开数据库。请注意，已使用加密备份的一部分来还原表空间。
REPAIR FAILURE;
# 9.使用LIST FAILURE命令来验证故障是否已修复。然后退出
list failure;
exit
```


## 练习12-8：恢复丢失的加密钱包

### 总览

在这种情况下，您将找回丢失的加密钱包。

注意：如果丢失了钱包并且没有备份，则必须将数据库恢复到使用该钱包之前的某个时间点。

### 假设条件

练习12-6和12-7已完成。

打开两个终端窗口，您以oracle OS用户身份登录。`$LABS`是当前目录。为`orcl`实例设置了环境变量。

### 任务

```SQL
# 1.复制位于以下位置的密钥库（或加密钱包）
cp $ORACLE_BASE/admin/orcl/wallet/ewallet.p12 /u01/backup/orcl/ewallet.p12
ls /u01/backup/orcl/ewal*
# 注意：仅当您看到钱包的备份时，才继续。
# 2.通过执行break_12_08.sh脚本删除密钥库。输出在/tmp/break.log 文件。
./break_12_08.sh
# 3.尝试使用SQL * Plus启动数据库。
sqlplus / as sysdba
# 4.这与您从丢失的SYSAUX表空间中恢复的上一实践中收到的消息完全相同。尝试打开密钥库。
ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY secret;
# 5.还原钱包。
! cp /u01/backup/orcl/ewallet.p12 $ORACLE_BASE/admin/orcl/wallet/ewallet.p12
!ls $ORACLE_BASE/admin/orcl/wallet
# 6.打开密钥库。然后从SQL * Plus退出。
ADMINISTER KEY MANAGEMENT SET KEYSTORE OPEN IDENTIFIED BY secret;
exit
# 7.使用Data Recovery Adviser RMAN命令恢复并打开数据库。然后确认没有故障并退出RMAN。
rman target "'/ as sysbackup'"
list failure;
advise failure;
repair failure;
list failure;
exit
# 8.通过执行cleanup_12_08.sh脚本来清理练习环境。该脚本删除加密的备份并禁用加密的备份。输出在/tmp/cleanup.log 文件。
./cleanup_12_08.sh
```
