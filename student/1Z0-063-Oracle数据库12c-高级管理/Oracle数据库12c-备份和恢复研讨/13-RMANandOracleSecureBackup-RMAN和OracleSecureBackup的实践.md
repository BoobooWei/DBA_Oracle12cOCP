# 第13课：RMAN和Oracle Secure Backup的实践

> 2020-04-19 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [第13课：RMAN和Oracle Secure Backup的实践](#第13课：rman和oracle-secure-backup的实践)   
   - [实践概述](#实践概述)   
   - [练习13-1：安装Oracle Secure Backup](#练习13-1：安装oracle-secure-backup)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   
   - [练习13-2：配置Oracle Secure Backup](#练习13-2：配置oracle-secure-backup)   
      - [总览](#总览)   
      - [假设条件](#假设条件)   
      - [任务](#任务)   

<!-- /MDTOC -->

## 实践概述

在这些实践中，您将安装并使用Oracle Secure Backup。

## 练习13-1：安装Oracle Secure Backup

### 总览

在这种实践中，您从暂存区安装Oracle Secure Backup。

### 假设条件

您已启动并运行一个orcl数据库实例，并执行了RMAN早期实践中的配置。

将打开两个终端窗口，并且您以root OS用户身份登录。$LABS是当前目录。为orcl实例设置了环境变量。

Oracle Secure Backup软件已在/stage/software中暂存和解压缩目录。

### 任务

```SQL
# 1.以root用户身份登录， 并确认/usr/local/oracle/backup目录存在。这是Oracle Secure Backup主目录，建议从该目录开始安装。（如果要选择另一个目录，则OSB产品文档以及所有相关的培训文件将不会同步，并且无法按原样工作。）
su - root
cd /usr/local/oracle/backup
# 2.要在您的培训环境中安装OSB软件：
## 1. 在目录中执行stage/<osb_installmedia>/setup 。
## 2.如果询问，请接受默认设置，即不编辑obparameters文件。
## 3.接受默认设置以立即安装。
## 4.选择安装选项a（管理服务器，介质服务器和客户端）。
## 5.将加密密钥库的密码设置为oracle_4U。
## 6.将管理员用户的密码设置为oracle_4U。
## 7.不要输入管理员用户的电子邮件地址，只需按Enter键即可。
## 8.接受默认设置，即不连接到磁带库。
## 9.接受默认设置，即不连接到磁带机。
## 10.退出root OS用户帐户。
/stage/software/osb-10.4.0.3.0_linux.x64_release/setup
exit
```

## 练习13-2：配置Oracle Secure Backup

### 总览

在这种做法中，您配置Oracle Secure Backup和RMAN，然后启动到磁带的备份以测试您的配置。

### 假设条件

打开两个终端窗口，您以oracle OS用户身份登录。$ LABS是当前目录。为orcl实例设置了环境变量。

您已经完成了先前的练习13-1，并按照指示使用目录和密码安装了软件。

### 任务

```SQL
# 1.执行setup_13_osb.sh脚本（调用osb_in.sh脚本来更新osb_out.sh 与脚本您的主机名，然后执行 osb_out.sh 脚本）。
# 脚本创建两个虚拟测试库，一些虚拟测试驱动器，插入卷并创建一个预先授权的oracle OSB用户。
# 本实践中使用的这些虚拟测试设备仅用于培训目的。他们不支持供生产使用。
# 输出在/tmp/setup.log文件中。（可选）查看setup_13_osb.sh文件。
cat setup_13_osb.sh
./setup_13_osb.sh
# 2.使用obtool命令行，查看刚刚创建的元素。
# 注意：OSB命令与Linux一样区分大小写。
# a.以admin OSB用户身份登录 obtool。作为预授权的oracle用户，您无需输入密码。
obtool
# b.使用lsmf --long命令查看RMAN-DEFAULT媒体系列。
lsmf --long
# c.使用lsdev命令查看当前配置的设备。
lsdev
# d.使用lsvol命令列出插入vlib2库的卷。
lsvol -L vlib2
# e.退出obtool实用程序。
exit
# 3.在Web工具中，有选择地浏览您创建的元素，然后配置数据库备份存储选择器。（以下第5步为必填项。）
# 注意：您也可以使用obtool实用程序创建该对象。练习使用图形工具进行学习。对于在上一步中使用obtool脚本创建的对象来说，情况也是如此：您也可以在Web工具中创建它们。
# a.登录到Web工具。URL为：https：// < 主机名>。
# b.首次登录Web工具时，需要确认安全例外。（Firefox ：我了解风险>添加例外>确认安全性例外）
# c.输入admin作为User Name，输入oracle_4U作为Password，然后单击Login。
# d.在OSB主页上，您可以查看过去24小时内的作业及其状态。页面的下部显示现有设备。单击首选项。
# e.为“扩展命令输出” 选择打开。单击“ 应用”，然后单击“ 主页”。
# f.滚动到页面底部。您会看到一个新区域，其中显示了最近的对象命令及其状态。
# 4.（可选）查看为您创建的元素。单击配置，然后单击您感兴趣的任何元素。
# a.在“配置：主机”页面上，查看在软件安装期间选择的角色。查看完主机后，单击“配置”（带标签的页面或面包屑）以返回到“配置概述”页面。
# b.在“配置：设备”页面上，查看使用obtool脚本创建的库和磁带驱动器。（可选）选择一个设备并对其执行ping操作。然后返回到“配置概述”页面。
# c.在“配置：媒体系列”页面上，查看在软件安装过程中创建的RMAN-DEFAULT媒体系列。返回到“配置概述”页面。
# d.在“配置：用户”页面上，选择oracle用户，然后单击“ 编辑”。
# 您将看到oracle OS用户映射到oracle OSB用户的页面。
# e.单击预授权访问。
# 您会看到oracle用户已获得命令行（cmdline）和 rman 操作。
# f.查看完Oracle OSB用户后，单击左上方的Configure（配置）。
# 5.导航到配置>数据库备份存储选择器，单击添加并配置数据库备份存储选择器，如屏幕快照所示。
# 注意：您必须单击主机名才能应用工作。一种。点击应用。（您应该看到一条成功消息。）滚动到页面底部，然后查看obtool命令。
# a.点击应用。（您应该看到一条成功消息。）
# b.返回OSB主页，但不要退出Web工具。
# 6.切换到终端窗口。以SYSBACKUP角色登录到RMAN客户端，并配置用于备份到磁带的通道。在一行中输入命令，或使用运行{...}来配置您的频道。
rman target "'/ as sysbackup'"
CONFIGURE CHANNEL DEVICE TYPE 'SBT_TAPE' PARMS 'ENV=(OB_DEVICE=vdrive1)';
# 7.（可选）查看所有参数，并记下有关SBT_TAPE的参数，这些参数是在通道配置中创建的。
show all;
# 8.在RMAN中，将USERS表空间备份到磁带的设备类型。
backup device type SBT_TAPE tablespace users;
# 9.在RMAN中查看您的磁带备份，然后退出。
list backup device type SBT_TAPE;
# 注意：设备类型为SBT_TAPE，备份存储在卷 000001上；第一个备份包含数据文件，第二个备份包含SPFILE和控制文件。
# 10.返回Web工具并刷新页面，然后单击显示完成的作业。
# 11.（可选）使用lsj命令在obtool中查看作业。
obtool
lsj -A
# 12.退出所有窗口和工具。
```
