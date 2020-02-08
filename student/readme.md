# 使用学习手册的前提



## 1. 启动自带的Oracle12c企业管理工具

请参考帮助文档 [01-配置使用Oracle企业管理工具em.md ](01-配置使用Oracle企业管理工具em.md) 启动，并保存访问地址。

### CDB

* url地址：https://192.168.14.154:5500/em/login
* web 页面登陆方式：`sys/WLS3Gg5_2 as sysdba`
* 容器名：不填写

### PDB

* url地址：https://192.168.14.154:5505/em/login
* web 页面登陆方式：`sys/WLS3Gg5_2 as sysdba`

## 2. 安装配置Oracle12c云管理工具

请参考帮助文档 [02-配置使用Oracle企业云管理工具ocm.md ](02-配置使用Oracle企业云管理工具ocm.md ) 安装启动云管理工具，并保存访问地址。

* url地址： https://192.168.14.154:7301/em
* web 页面登陆方式：`sysman/WLS3Gg5_2`

## 3. 了解Demo Schema并导入OE

示例数据库模式为每个Oracle数据库版本中的示例提供了一个通用平台。样本模式是一组互连的数据库模式。该集合提供了解决复杂性的方法：

- 模式人力资源（`HR`）对于介绍基本主题很有用。此架构的扩展支持Oracle Internet Directory演示。
- 模式订单输入（`OE`）对于处理中等复杂性的问题很有用。此模式中有许多数据类型可用，包括非标量数据类型。
- Schema Online Catalog（`OC`）是在schema内部建立的对象关系数据库对象的集合`OE`。
- Schema Product Media（`PM`）专用于多媒体数据类型。
- 在主模式名称Information Exchange（`IX`）下收集的一组模式可用于演示Oracle Advanced Queuing功能。
- 架构销售历史记录（`SH`）旨在允许进行包含大量数据的演示。此架构的扩展提供了对高级分析处理的支持。

- [Schema Demo](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/comsc/lot.html)
- [Schema Demo Github](https://github.com/oracle/db-sample-schemas)