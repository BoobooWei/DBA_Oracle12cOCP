# 第6课：创建备份的练习

> 2020-04-09 BoobooWei

## 第6课：概述的练习

在该实践中，您将配置块更改跟踪，创建增量备份，备份控制文件以及备份存档的重做日志文件。

## 练习6-1：配置块更改跟踪

### 总览

在这该实践中，您将配置块更改跟踪（BCT）。

尽管BCT是可选的，但它可以将增量备份所需的时间从扫描数据库中的所有块的时间减少到与自上次备份以来已更改的块数成比例的时间。

注意：BCT文件只能包含8个位图，因此，如果新的增量备份基于父级备份，则增量备份超过8个就无法优化备份。开发增量备份时，请考虑8位图限制战略。

例如，如果先进行0级数据库备份，再进行7次差异增量备份，则块更改跟踪文件现在将包含8个位图。如果然后进行累积的1级增量备份，RMAN将无法优化备份，因为与父级0备份相对应的位图将被跟踪当前更改的位图覆盖。

### 假设条件

以前的实践已经完成。

您已打开一个终端窗口。为数据库实例设置了环境变量。

### 任务

配置块更改跟踪以将BCT文件放置在默认数据文件创建目标中。

```SQL
!mkdir /home/oracle/oracle_bct
# 确保将DB_CREATE_FILE_DEST初始化参数设置为正确的位置
ALTER SYSTEM SET DB_CREATE_FILE_DEST = '/home/oracle/oracle_bct';
SHOW PARAMETER DB_CREATE_FILE_DEST

# 启用块更改跟踪
ALTER DATABASE ENABLE BLOCK CHANGE TRACKING;
```

<img src='https://i.loli.net/2020/04/10/OVNHRJprSewbmA7.jpg' alt='OVNHRJprSewbmA7'/>


## 练习6-2：使用增量备份
