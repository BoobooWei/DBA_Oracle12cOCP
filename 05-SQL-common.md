# 05-SQL-01

> 2019.09.22 BooBoo Wei

## 注意点

之前已经学习过oracle的sql语句，此处只记录难点。

[SQL学习仓库链接](https://github.com/BoobooWei/booboo_oracle)

Oracle 创建测试大表写法

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

