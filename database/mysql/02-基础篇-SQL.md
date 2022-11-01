## SQL通用语法

- SQL语句可以单行或多行书写，以分号结尾
- SQL语句可以使用空格、缩进来增强语句的可读性
- MySQL数据库的SQL语句不区分大小写 ，关键字建议使用大写
- 注释：
  - 单行注释：  --注释内容 或 #注释内容(MySQL特有)
  - 多行注释：/* 注释内容 */

## SQL分类

| 分类 | 全称                       | 说明                                                   |
| ---- | -------------------------- | ------------------------------------------------------ |
| DDL  | Data Definition Language   | 数据定义语言，用来定义数据库对象（数据库、表、字段）   |
| DML  | Data Manipulation Language | 数据操作语言，用来对数据库表中的数据进行增改           |
| DQl  | Data Query Language        | 数据查询语言，用来查询数据库中表的记录                 |
| DCL  | Data Control Language      | 数据控制语言，用来创建数据库用户，控制数据库的访问权限 |

## DDL

### 数据库操作

- 查询所有数据库 `SHOW DATABASES;`
- 查询当前数据库`SELECT DATABASE();`
- 创建 `create database [if not exists] 数据库名 [default charset 字符集][collate 排序规则];`
- 删除`drop database [if exists] 数据库名`
- 使用 `use 数据库名;`

### 表操作

- 查询当前数据库所有表`show tables;`
- 查询表结构`desc 表名;`
- 查询指定表的建表语句`show create table 表名;`
- 创建表

```sql
create table 表名(
	字段1 字段1类型[comment 字段1注释],
	字段2 字段2类型[comment 字段2注释],
  ...
	字段n 字段n类型[comment 字段n注释]
)[comment 表注释];
-- [...]为可选参数，最后一个字段后面没有逗号
```

- 数据类型

  - 数值类型

  | 类型         | 大小    | 有符号范围                 | 无符号范围                 | 描述                 |
  | ------------ | ------- | -------------------------- | -------------------------- | -------------------- |
  | TINYINT      | 1 byte  | (-128, 127)                | (0, 255)                   | 小整数值             |
  | SMALLINT     | 2 bytes | (-32768, 32767)            | (0, 65535)                 | 大整数值             |
  | MEDIUMINT    | 3 bytes | (-8388608, 8388607)        | (0, 16777215)              | 大整数值             |
  | INT或INTEGER | 4 bytes | (-214783648, 214783647)    | (0, 4294967295)            | 大整数值             |
  | BIGINT       | 8 bytes | (-2^63, 2^63-1)            | (0, 2^64-1)                | 极大整数值           |
  | FLOAT        | 4 bytes |                            |                            | 单精度浮点数值       |
  | DOUBLE       | 8 bytes |                            |                            | 双精度浮点数值       |
  | DECIMAL      |         | 依赖于M(精度)和D(标度)的值 | 依赖于M(精度)和D(标度)的值 | 小数值（精确定点数） |

  - 字符串类型

  | 类型       | 大小               | 描述                         |
  | ---------- | ------------------ | ---------------------------- |
  | CHAR       | 0-255 bytes        | 定长字符串                   |
  | VARCHAR    | 0-65535 bytes      | 变长字符串                   |
  | TINYBLOB   | 0-255 bytes        | 不超过255个字符的二进制数据  |
  | TINYTEXT   | 0-255 bytes        | 短文本字符串                 |
  | BLOB       | 0-65535 bytes      | 二进制形式的长文本数据       |
  | TEXT       | 0-65535 bytes      | 长文本数据                   |
  | MEDIUMBLOB | 0-16777215 bytes   | 二进制形式的中等长度文本数据 |
  | MEDIUMTEXT | 0-16777215 bytes   | 中等长度文本数据             |
  | LONGBLOB   | 0-4294967295 bytes | 二进制形式的极大文本数据     |
  | LONGTEXT   | 0-4292967295 bytes | 极大文本数据                 |

  - 日期时间类型

  | 类型      | 大小 | 范围                                       | 格式                | 描述                   |
  | --------- | ---- | ------------------------------------------ | ------------------- | ---------------------- |
  | DATE      | 3    | 1000-01-01 至 9999-12-31                   | YYYY-MM-DD          | 日期值                 |
  | TIME      | 3    | -838:59:59 至 838:59:59                    | HH:MM:SS            | 时间值或持续时间       |
  | YEAR      | 1    | 1901 至 2155                               | YYYY                | 年份值                 |
  | DATETIME  | 8    | 1000-01-01 00:00:00 至 9999-12-31 23:59:59 | YYYY-MM-DD HH:MM:SS | 混合日期和时间值       |
  | TIMESTAMP | 4    | 1970-01-01 00:00:00 至 2038-01-19 03:14:07 | YYYY-MM-DD HH:MM:SS | 混合日期和时间值时间戳 |

- 添加字段 `alter table 表名 字段名 类型(长度) [comment 注释] [约束]`

  - 为emp表增加一个新的字段“昵称”，为nickname, 类型为varchar(20)

  `alter table emp add nickname varchar(20) comment '昵称'`

- 修改字段数据类型`alter table 表名 modify 字段名 新数据类型(长度);`

- 修改字段名和字段类型 `alter table 表名 change 旧字段名 新字段名 类型(长度) [comment 注释] [约束]`

  - 将emp表的nickname字段修改为username, 类型为varchar(30)

  `alter table emp change nickname username varchar(30) comment '昵称';`

- 删除字段 `alter table 表名 drop 字段名;`

  - 将emp表的字段username删除

  `alter table emp drop username;`

- 修改表名`alter table 表名 rename to 新表名;`

  - 将emp表的表名修改为employee; `alter table emp rename to employee;`

- 删除表`drop table [if exists] 表名;`

- 删除指定表，并冲下创建该表`truncate table 表名;`

## DML

- DML(Data Manipulation Language:数据操作语言)用来对数据库中表的数据记录进行增删改操作

### 添加数据

- 给指定字段添加数据 `insert into 表名(字段名1, 字段名2, ...) values(值1, 值2, ...)`
- 给全部字段添加数据 `insert into 表名 values(值1, 值2, ...)`
- 批量添加数据 
  - `insert into 表名(字段名1, 字段名2, ...) values(值1, 值2, ...), (值1, 值2, ...);`
  - `insert into 表名 values(值1, 值2, ...), (值1, 值2, ...);`
- 注意
  - 插入数据时，指定的字段顺序需要与值的顺序是一一对应的
  - 字符串和日期型数据应该包含在引号中
  - 插入的数据大小，应该在字段的规定范围内

### 修改数据

- `update 表名 set 字段名1=值1, 字段名2=值2, ... [where 条件];`
- 修改语句的条件可以有，也可以没有，如果没有条件，则会修改整张表的所有数据

### 删除数据

- `delete from 表名 [where 条件];`
- delete语句的条件可以有，也可以没有，如果没有条件，则会删除整张表的所有数据
- delete语句不能删除某一个字段的值(可以使用update)

## DQL

- DQL(Data Query Language)数据查询语言，用来查询数据库表中的记录

### DQL语法

```sql
SELECT
	字段列表
FROM
	表名
WHERE
	条件列表
GROUP BY
	分组字段列表
HAVING
	分组后条件列表
ORDER BY
	排序字段列表
LIMIT
	分页参数
# 执行顺序
# 表名 -> 条件列表 -> 分组字段列表 -> 分组后条件列表 -> 字段列表 -> 排序字段列表 -> 分页参数
```

#### 执行顺序

表名 -> 条件列表 -> 分组字段列表 -> 字段列表 -> 分组后条件列表 -> 排序字段列表 -> 分页参数

```sql
FROM 表名
WHERE 条件列表
GROUP BY 分组字段列表
HAVING 分组后条件列表
SELECT 字段列表
ORDER BY 排序字段列表
LIMIT 分页参数
```

### 基本查询

- 查询多个字段

`select 字段1, 字段2, ... from 表名;`

`select * from 表名;`

- 设置别名 `select 字段1[as 别名1] from 表名;`
- 去除重复记录 `select distinct 字段列表 from 表名;`

### 条件查询(where)

- 比较运算符
  - 大于: >
  - 大于等于: >=
  - 小于: <
  - 小于等于: <=
  - 等于: =
  - 不等于: <> 或 !=
  - 在某个范围之内(韩最小，最大值): BETWEEN ... AND ...
  - 在in之后的列表中的值，多选一: in(...)
  - 模糊匹配(_匹配单个字符，%匹配任意个字符): LIKE 占位符
  - 是NULL: IS NULL
- 逻辑运算符
  - AND 或 &&: 并且
  - OR 或 ||: 或者
  - NOT 或 ! : 非，不是

### 聚合函数

- 接收：将一列数据作为一个整体，进行纵向计算
- 语法: `select 聚合函数(字段列表) from 表名;`
- count: 统计数量
- max: 最大值
- min: 最小值
- avg: 平均值
- sum: 求和

### 分组查询

- 语法 `select 字段列表 from 表名 [where 条件] group by 分组字段名 [having 分组过滤条件];`
- where 与 having区别
  - 执行时机不同：where 与分组之前进行过滤，不满足where条件，不参与分组，而having是分组之后对结果进行过滤
  - 判断条件不同：where不能对聚合函数进行判断，而having可以
- 执行顺序：where > 聚合函数 > having
- 分组之后，查询的字段一般为聚合函数和分组字段，查询其他字段无任何意义

### 排序查询

- 语法 `select 字段列表 from 表名 order by 字段1 排序方式1, 字段2 排序方式2`
- 排序方式： asc, desc
- 如果是多个字段，当第一个字段值相同时，才会根据第二个字段排序

### 分页查询

- 语法 `select 字段列表 from 表名 limit 起始索引, 查询记录数;`
- 起始索引从0开始，起始索引=(查询页码-1) * 每页显示记录数
- 如果查询的是第一页数据，起始索引可以省略，直接简写为 limit 10

## DCL

- DCL(Data Control Language)数据控制语言，用来管理数据库用户，控制数据库的访问权限

### 管理用户

- 查询用户 `use mysql; select * from user;`
- 创建用户 `create user '用户名@主机名' identified by '密码'`
- 修改用户密码 `alter user '用户名'@'主机名' identified with mysql_native_password by '新密码';`
- 删除用户 `drop user '用户名@主机名';`
- 注意：
  - 主机名可以使用%通配
  - 这类SQL开发人员操作的比较少，主要是DBA(Data Administrator 数据库管理员)使用

### 权限控制

MySQL中定义了很多种权限，但是常用的就以下几种

| 权限                | 说明               |
| ------------------- | ------------------ |
| ALL, ALL PRIVILEGES | 所有权限           |
| SELECT              | 查询数据           |
| INSERT              | 插入数据           |
| UPDATE              | 更新数据           |
| DELTE               | 删除数据           |
| ALTER               | 修改表             |
| DROP                | 删除数据库/表/视图 |
| CREATE              | 创建数据库/表      |

- 查询权限 `show grants for '用户名@密码'`
- 授予权限 `grant 权限列表 on 数据库.表名 to '用户名@密码';`
- 撤销权限 `revoke 权限列表 on 数据库.表名 from '用户名@密码';`
- 注意
  - 多个权限之间，使用逗号分隔
  - 授权时，数据库名和表名可以使用*进行通配，代表所有
