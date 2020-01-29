# 06-SQL-advanced

> 2019.10.13 BooBoo Wei

## 注意点

之前已经学习过oracle的sql语句，此处只记录难点。

[SQL学习仓库链接](https://github.com/BoobooWei/booboo_oracle/blob/master/B-SQL语句-08-DDL创建表和管理表.md )

[`v$database`]( https://docs.oracle.com/database/121/REFRN/GUID-C62A7B96-2DD4-4E70-A0D9-26EE4BFBE256.htm#REFRN30047 )

## 管理5大对象

> 表 、视图、序列、索引、同义词
>
> table view sequence index synonym 美 [ˈsɪnənɪm] 

### 表的分类

- 用户表:由用户创建和维护的表的集和；包含用户信息
- 数据字典:由oracle服务器创建和维护的表的集和；包含数据库信息；用户记录oracle自己工作属性和状态的

| 数据字典分类 | 前缀  | 描述                                               | 备注                                              |
| ------------ | ----- | -------------------------------------------------- | ------------------------------------------------- |
| 字典表       | user_ | 包含有关用户拥有对象的信息                         | 当前用户所拥有的rw                                |
| 字典表       | all_  | 包含所有用户可以访问的表的信息（对象表和相关的表） | 当前用户所拥有的rw以及有权力查看ro的对象的信息    |
| 字典表       | dba_  | 受限制视图，只能被DBA角色的人访问                  | 数据库管理员才有权限查看                          |
| 动态性能视图 | v$    | 动态视图，数据库服务器性能，内存和锁               | 初始化在内存中，c语言的结构数组，作为排错和优化的 |


 

1. 使用`hr`用户连接数据库
2. 通过视图`user_tables`查看`hr`用户拥有`rw`权限的**表**
3. 通过视图`all_tables`查看`hr`用户拥有`rw` 和 `ro`权限的**表**
4. 通过视图`user_views`查看`hr`用户拥有`rw`权限的**视图**
5. 通过视图`all_views`查看`hr`用户拥有`rw` 和 `ro`权限的**视图**
6. 通过视图 `user_sequences`查看`hr`用户拥有`rw`权限的**序列**
7. 查看`EMPLOYEES_SEQ`**序列**的当前值`currval`和下一个值`nextval`
8. 通过视图`v$database`查看当前数据库的SCN序列号`current_scn` 
9. 通过视图`user_indexes`查看`hr`用户拥有`rw`权限的**索引**
10. 通过视图`user_ind_columns`查看`hr`用户拥有`rw`权限的**索引**对应的表和列
11. 通过视图`user_synonyms`查看`hr`用户拥有`rw`权限的**同义词**
12. 通过视图`all_synonyms`查看`hr`用户拥有`rw`和 `ro`权限的**同义词**

```sql
$ sqlplus hr/Oracle123@oracle01:1521/booboopdb1

select table_name from user_tables;
select table_name from all_tables;
select view_name,text from user_views;
select view_name,text from all_views;
select sequence_name,min_value,max_value,increment_by,last_number from user_sequences;
select EMPLOYEES_SEQ.currval,EMPLOYEES_SEQ.nextval from dual;
select current_scn from v$database;
select index_name,index_type, table_name from user_indexes;
SELECT
    table_name,
    index_name,
    LISTAGG(column_name,
        ',') WITHIN GROUP(
        ORDER BY
            column_name
        )
FROM
    user_ind_columns
GROUP BY
    table_name,
    index_name;
    
select synonym_name,table_owner,table_name from user_synonyms;
select synonym_name,table_owner,table_name from all_synonyms where rownum < 5;
```

练习记录

```sql
SQL> column SEQUENCE_NAME format a20
SQL> select sequence_name,min_value,max_value,increment_by,last_number from user_sequences;

SEQUENCE_NAME	      MIN_VALUE  MAX_VALUE INCREMENT_BY LAST_NUMBER
-------------------- ---------- ---------- ------------ -----------
DEPARTMENTS_SEQ 	      1       9990	     10 	280
EMPLOYEES_SEQ		      1 1.0000E+28	      1 	207
LOCATIONS_SEQ		      1       9900	    100        3300


SQL> create synonym booboo for hr.employees;

Synonym created.

SQL> select synonym_name,table_owner,table_name from user_synonyms;

SYNONYM_NA TABLE_OWNE TABLE_NAME
---------- ---------- ----------
BOOBOO	   HR	      EMPLOYEES

SQL> select synonym_name,table_owner,table_name from all_synonyms where rownum < 2;

SYNONYM_NA TABLE_OWNE TABLE_NAME
---------- ---------- ----------
DBMS_WLM   SYS	      DBMS_WLM

```

