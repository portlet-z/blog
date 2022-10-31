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



## DQL

## DCL

