# 05-SQL-01

> 2019.09.22 BooBoo Wei

[toc]

## 注意点

之前已经学习过oracle的sql语句，此处只记录难点。

[SQL学习仓库链接](https://github.com/BoobooWei/booboo_oracle)

### Oracle 创建测试大表写法

```sql
CREATE TABLE t1
    AS
        SELECT
            level id,
            'level-name'
            || TO_CHAR(level) name
        FROM
            dual
        CONNECT BY
            level <= 1000;
```

### Oracle 12C 与 11g 函数区别

#### 列转行`vm_concat` vs `listagg`

```sql
-- 12C
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
    
-- 11g
SELECT
    table_name,
    index_name,
    vm_concat(column_name,
        ',')
FROM
    user_ind_columns
GROUP BY
    table_name,
    index_name;

-- mysql group_concat
SELECT
    table_name,
    index_name,
    vm_concat(column_name,
        ',')
FROM
    user_ind_columns
GROUP BY
    table_name,
    index_name;
```

