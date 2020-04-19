# 第19课：RMAN故障排除和调整的实践

> 2020.04.19 BoobooWei

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

- [第19课：RMAN故障排除和调整的实践](#第19课：rman故障排除和调整的实践)   
   - [实践概述](#实践概述)   
   - [练习19-1：重置培训数据库](#练习19-1：重置培训数据库)   
      - [总览](#总览)   

<!-- /MDTOC -->

## 实践概述

在这些实践中，您将为研讨会做准备。


## 练习19-1：重置培训数据库

### 总览

本课没有实际动手练习。在这种实践中，您将重置备份和恢复研讨会的培训环境。该RESET_BAR.sh脚本取下

您当前的ORCL和RCAT数据库，然后重新创建它们。

该脚本不会涉及其他数据库或企业管理器云控制。

```SQL
# 1.关闭可能从先前任务打开的所有会话和窗口。
exit
# 2.以root操作系统用户身份登录。
su - root
# 3.首次重置课程环境时，请跳过此步骤。这些命令用于删除标记重置完成的BAR.rebuild文件。
rm -f /stage/MBS/logs/BAR.rebuild
rm -f /stage/MBS/targets/D78850GC20.reset1
# 4.导航到/stage/MBS/scripts目录。
cd /stage/MBS/scripts
# 5.执行RESET_BAR.sh脚本以重建您的训练数据库。（该脚本可能需要一个或多个小时才能完成。）
./RESET_BAR.sh
```
