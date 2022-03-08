# MySQL修改数据库表结构常用SQL汇总



> &emsp;&emsp;在快速迭代的开发过程中，你是否遇到过这样一类情况：上午刚确定完需求，下午就匆匆忙忙设计好了对应需求的数据库表结构，不料第二天早上的早会上产品经理又调整了产品需求。在小公司待久了，遇到类似情况感觉就像家常便饭一般。于是第二天再次确定好需求，针对性的去修改昨天设计好的数据库表结构了，虽然很多情况下会出现改一处而动全部，但是在公司项目不断发展迭代的过程中这又能算什么呢。
> &emsp;&emsp;由于平时在设计好了数据库表之后经常遇到再次调整的情况，所以我整理了一下经常会用到的一些SQL脚本，以供不时之需。

### 1、调整数据库表中的列

##### 添加列：

++语法++：

```sql
ALTER TABLE 数据库名.表名 ADD COLUMN 列名 数据类型(数据长度) DEFAULT 默认值 限制 COMMENT 注释 AFTER 列名;
```

> 简单解释：给指定数据库中的表添加目标列，指定列的数据类型和数据长度，设置默认值及相关限制(如唯一索引UNIQUE)，添加列的注释，将新加列加在某一列之后。

++示例++： 

```sql
ALTER TABLE `test`.`user` ADD COLUMN `id_no` VARCHAR(20) DEFAULT NULL UNIQUE COMMENT 'ID_Number' AFTER `id`;

ALTER TABLE `test`.`user` ADD COLUMN `phone_no` VARCHAR(64) DEFAULT NULL COMMENT '手机号' AFTER `id`;

ALTER TABLE `test`.`user` ADD COLUMN `nick_name` VARCHAR(64) DEFAULT NULL COMMENT '昵称';

-- 新增列设置成自增主键示例：
ALTER TABLE `test`.`user` ADD COLUMN `id` BIGINT(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID', ADD PRIMARY    KEY(`id`);
```

##### 删除列：

++语法++：

```sql
ALTER TABLE 数据库名.表名 DROP COLUMN 列名;
```

> 简单解释；删除指定数据库指定表中的目标列。

++示例++：

```sql
ALTER TABLE `test`.`user` DROP COLUMN `nick_name`;
```

##### 修改列：

> 可以修改列的名称、数据类型、数据长度限制、字符集编码、注释、列的顺序 位置、默认值等等。

++语法++：

```sql
-- 修改列名及相关属性(CHANGE)：
ALTER TABLE 数据库名.表名 CHANGE COLUMN 旧列名 新列名 数据类型(数据长度) DEFAULT 默认值 限制 COMMENT 注释 AFTER 列名;

-- 修改列属性(MODIFY)：
ALTER TABLE 数据库名.表名 MODIFY COLUMN 列名 数据类型(数据长度) DEFAULT 默认值 限制 COMMENT 注释 AFTER 列名;
```

> 注意: modify 只能修改属性，不能修改列名！

++示例++：

```sql
-- 修改列名及相关属性(CHANGE)：
ALTER TABLE `test`.`user` CHANGE COLUMN `id_no` `id_num` VARCHAR(32) DEFAULT '123' COMMENT 'ID_NUM' AFTER `id`;

ALTER TABLE `test`.`user` CHANGE COLUMN `phone_no` `phone_number` BIGINT(16) DEFAULT NULL UNIQUE COMMENT 'PHHONE_NUM' AFTER `id`

-- 修改列‘id_num’的字符集为'utf8mb4':
ALTER TABLE `test`.`user` CHANGE `id_num` `id_num` VARCHAR(32) CHARACTER SET utf8mb4 NOT NULL DEFAULT '0' COMMENT 'Id_number'; 

-- 修改列属性(MODIFY)：
ALTER TABLE `test`.`user` MODIFY COLUMN `id_num` BIGINT(20)  DEFAULT 0 COMMENT 'Id_number' AFTER `id`;

ALTER TABLE `test`.`user` MODIFY COLUMN `phone_no` BIGINT(20)  DEFAULT 0 UNIQUE COMMENT 'phone_number' AFTER `id`;
```

### 2、调整数据库表中的索引

> &emsp;&emsp;MySQL 索引的建立对于 MySQL 的高效运行是很重要的，索引可以大大提高 MySQL 的检索速度。

#### 创建索引

##### 普通索引：

++方式一语法++：

```sql
CREATE INDEX indexName ON mytable(COLUMN_NAME(length)); 
```

> 索引列如果是 CHAR，VARCHAR 类型，length 可以小于字段实际长度；如果是 BLOB 和 TEXT 类型，必须指定 length。

++方式一示例++：

```sql
-- 给 phone_number 列添加索引，此列类型是 bigint 所以不需要指定长度。
CREATE INDEX p_index ON `test`.`user`(`phone_number`);

-- 给 id_num 列添加索引，此列类型是 varchar 所以这里指定列长度16。
CREATE INDEX id_index ON `test`.`user`(`id_num`(16));
```

++方式二语法++：

```sql
-- 修改表结构(添加索引)
ALTER TABLE tableName ADD INDEX indexName(columnName);
```

++方式二示例++：

```sql
ALTER TABLE `test`.`user` ADD INDEX p_index(`phone_number`);
```

##### 唯一索引：

> 唯一索引与前面的普通索引类似，不同之处：索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。

++方式一语法++：

```sql
CREATE UNIQUE INDEX indexName ON tableName(COLUMN_NAME(length));
```

++方式一示例++：

```sql
-- 唯一索引列数据类型是 bigint 不能加 length 条件:
CREATE UNIQUE INDEX unionId_Index ON `test`.`user`(`id`);

-- 唯一索引列数据类型是 varchar 可以加 length 条件(也可以不加)，但索引的 length <= 列值的 length :
CREATE UNIQUE INDEX unionIdNum_Index ON `test`.`user`(`id_num`(20));

-- 添加多列组合唯一索引：
CREATE UNIQUE INDEX unionIdNum_Index ON `test`.`user`(`id_num`,`id`);
```

++方式二语法++：

```sql
ALTER TABLE tableName ADD UNIQUE indexName (COLUMN_NAME(length));
```

++方式二示例++：

```sql
-- 修改表结构的方式添加唯一索引并限制索引长度(索引列是varchar类型)：
ALTER TABLE `test`.`user` ADD UNIQUE unionIdNum_Index (`id_num`(20));

-- 修改表结构的方式添加唯一索引，索引列是 bigint 类型，因此不能加 length 限制：
ALTER TABLE `test`.`user` ADD UNIQUE unionId_Index (`id`);

-- 修改表结构的方式添加多列组合唯一索引(varchar列可指定长度length)：
ALTER TABLE `test`.`user` ADD UNIQUE unionId_Index (`id`, `id_num`(20));
-- 
```

##### 主键索引：

> 主键索引值必须是唯一的，且不能为NULL。

++语法++：

```sql
ALTER TABLE tableName ADD PRIMARY KEY (column_list); 
```

++示例++：

```sql
-- 设置单列主键：
ALTER TABLE `test`.`user` ADD PRIMARY KEY (`id`); 

-- 设置多列组合主键(注意：组合列不能出现 NULL 且必须唯一)：
ALTER TABLE `test`.`user` ADD PRIMARY KEY (`id`, `id_num`); 
```

##### 全文索引：

> 旧版的 MySQL 全文索引只能用在 MyISAM 表格的 char、varchar 和 text 的字段上。 
> 不过从 MySQL5.6 开始，InnoDB 引擎也加入了全文索引。
> 注意：只有 char、varchar、text 类型字段能创建全文索引。

++语法++：

```sql
ALTER TABLE tableName ADD FULLTEXT index_name (column_list);
```

++示例++：

```sql
-- 设置单列全文索引：
ALTER TABLE `test`.`user` ADD FULLTEXT f_index (`id_num`);

-- 设置多列组合全文索引：
ALTER TABLE `test`.`user` ADD FULLTEXT f_index2 (`id_num`,`phone_number`);
```

#### 删除索引

> 注意：没有 DROP UNIQUE / FULLTEXT 这种方式，统一 DROP INDEX 即可。

++方式一语法++：

```sql
DROP INDEX [indexName] ON mytable; 
```

++方式一示例++：

```sql
-- 删除 user 表中名为 p_index 的索引：
DROP INDEX p_index ON `test`.`user`;
```

++方式二语法++：

```sql
ALTER TABLE tableName DROP INDEX indexName;
```

++方式二示例++：

```sql
ALTER TABLE `test`.`user` DROP INDEX p_index;
```

++删除主键索引语法++：

```sql
ALTER TABLE tableName DROP PRIMARY KEY;
```

++删除主键索引示例++：

```sql
ALTER TABLE `test`.`user` DROP PRIMARY KEY;
-- 此处若主键列是自增列则会报错，如下所示：
-- ERROR 1075 - Incorrect table definition; there can be only one auto column and it must be defined as a key。

-- 经过查询资料得知，自增列主键需要先删除自增后才能删除主键。
-- 1、删除表中 id 列的自增（修改列即可）：
ALTER TABLE `test`.`user` CHANGE `id` `id` BIGINT(20) NOT NULL COMMENT '主键ID';
-- 2、删除表中的主键：
ALTER TABLE `test`.`user` DROP PRIMARY KEY;
```

#### 查询索引

++语法++：

```sql
SHOW INDEX FROM tableName;
```

++示例++：

```sql
-- 查询表 user 中的所有索引信息：
SHOW INDEX FROM `test`.`user`;
```

返回信息如下：

| Table | Non_unique | Key_name | Seq_in_index | Column_name  | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
| ----- | ---------- | -------- | ------------ | ------------ | --------- | ----------- | -------- | ------ | ---- | ---------- | ------- | ------------- |
| user  | 1          | p_index  | 1            | phone_number | A         | 0           | (Null)   | (Null) | YES  | BTREE      |         |               |

### 3、调整数据库表属性

#### 修改表名称

++语法++：

```sql
ALTER TABLE tableName RENAME newTableName;
```

++示例++：

```sql
-- 将数据库表’user‘更名为‘u_user’：
ALTER TABLE `test`.`user` RENAME `u_user`;
```

#### 修改表字符集

> 修改表中指定列的字符集可食用修改列的方法进行修改，具体修改语法及示例 请看上文。

++语法++：

```sql
ALTER TABLE tableName DEFAULT CHARACTER SET 目标字符集(如：utf8mb4);
```

++示例++：

```sql
-- 设置表‘user’的默认字符集为 utf8mb4 (utf8 在 MySQL 中并不算真正意义上的 UTF-8 字符集，所以推荐使用 utf8mb4)
-- 而且 MySQL 中的 utf8 不能存储 emoji 表情，utf8mb4 则可以存储。
ALTER TABLE `test`.`user` DEFAULT CHARACTER SET utf8mb4;
```

#### 修改表引擎

**方式一**：

> 修改表结构(效率高，按行复制表记录到新表中；但是会对原表加锁，系统 I/O 开销较大)

++语法++：

```sql
ALTER TABLE tableName ENGINE = 目标引擎(如：INNODB、MYISAM等);
```

++示例++：

```sql
-- 修改表‘user’的引擎为‘MyISAM’:    
ALTER TABLE `test`.`user` ENGINE = MyISAM;
```

**方式二**：

> 使用 MySQL 自带工具 mysqldump 导出到文件，修改文件中的 CREATE TABLE 引擎语句，同时修改表名，且默认在 create table 前加上 drop table 语句；修改完了再将表文件导入数据库即完成修改。

**方式三**：

> 有时候由于需要保留原表，因此需要重新创建一个新表，表结构和原表结构一致，修改新表的引擎后将原表的数据写入到新表即可，这里需要注意数据一致性，最好在处理数据的时候将原表加锁，并且数据量过多时最好分批写入数据到新表，并写入的时候加上事务处理！

++示例++：

```sql
-- 先查看表‘user’的相关信息：
SHOW TABLE STATUS LIKE 'user';

-- 得出结果(前两列)：
|Name    |Engine    |
-----------------
|user    |InnoDB    |
```

```sql
-- 删除(如果新表已存在)并创建新表：
DROP TABLE IF EXISTS `user_copy`;
CREATE TABLE `user_copy` LIKE `user`;
-- 修改新表引擎类型：
ALTER TABLE `user_copy` ENGINE = MyISAM;
-- 开启事务处理：
START TRANSACTION;
INSERT INTO `user_copy` select * from `user` WHERE id BETWEEN 1 AND 3;
COMMIT;
```

```sql
-- 查看复制出来的新表‘user_copy’的相关信息：
SHOW TABLE STATUS LIKE 'user_copy';

-- 得到如下结果(前两列)：
|Name        |Engine|
--------------------
|user_copy    |MyISAM|
```

### 4、调整数据库属性

**修改数据库名称**

> MySQL 数据库改名，官方没有直接修改数据库名称的命令，只能变相修改实现。

**方式一**：

> 如果所有表都是 MyISAM 类型的话，可以改文件夹的名字。
> 1、关闭mysqld；
> 2、把 data 目录中的 db_name 目录重命名为 new_db_name；
> 3、开启mysqld；

**方式二**：

> 使用 MySQL自带的工具 mysqldump 导出备份数据库，然后新建新的数据库后再导入数据，这样比较麻烦，不太推荐这种做法。

**方式三**：

> 新建数据库，在新的数据库里重命名所有旧数据库中的表，再删除旧的数据库。具体操作步骤：创建新的数据库→重命名数据表名称→删除旧的数据库。
> 
> 注：此方法不太灵活，需要对每一个表进行重命名操作，可以用 Shell 进行优化。

++示例++：

```sql
-- 创建新数据库‘test2’：
CREATE DATABASE test2;
-- 重命名旧数据库中的对应表到新数据库：
RENAME TABLE 
`test`.`user` TO `test2`.`user`,
`test`.`user_copy` TO `test2`.`user_copy`;
-- 删除旧数据库：
DROP DATABASE test;
```

**方式四**：

> 这种方式就是对方式三的优化，采用 Shell 的方式对重命名表到新数据库时可以循环操作，不必每个表写一遍对应的 SQL 了。
> 
> 以下 Shell 只是示例，具体的可以依据实际情况进行调整：

```shell
#!/bin/bash
mysqlconn=”mysql -u xxxx -pxxxx -S /var/lib/mysql/mysql.sock -h localhost”
olddb=”db_name”
newdb=”new_db_name”

#$mysqlconn -e “CREATE DATABASE $newdb”
params=$($mysqlconn -N -e “SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE table_schema='$olddb'”)

for name in $params; do
$mysqlconn -e “RENAME TABLE $olddb.$name to $newdb.$name”;
done;

#$mysqlconn -e “DROP DATABASE $olddb”
```

**修改字符编码**

++语法++：

```sql
ALTER DATABASE 数据库名 CHARACTER SET 目标字符集 COLLATE 字符集对应的排序规则;
```

++示例++：

```sql
-- 咋们看看数据库各层目前的字符编码情况吧：
SHOW VARIABLES LIKE '%char%';

-- 结果如下：
Variable_name    Value
character_set_client    utf8mb4
character_set_connection    utf8mb4
character_set_database    utf8
character_set_filesystem    binary
character_set_results    utf8mb4
character_set_server    utf8
character_set_system    utf8
character_sets_dir    /usr/local/mysql57/share/charsets/

-- 修改数据库字符编码为'utf8mb4':
ALTER DATABASE test CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

-- 咋们再看看修改后的数据库各层的字符编码情况：
Variable_name    Value
character_set_client    utf8mb4
character_set_connection    utf8mb4
character_set_database    utf8mb4
character_set_filesystem    binary
character_set_results    utf8mb4
character_set_server    utf8
character_set_system    utf8
character_sets_dir    /usr/local/mysql57/share/charsets/

-- 由上面的结果可以看出，character_set_database 对应的字符编码由 utf8 变成了 utf8mb4，即修改成功。
```

### 5、总结

&emsp;&emsp;以上对数据库表对应的操作在文章前面部分的比较常用，后面部分使用到的频率相对会低一些。

&emsp;&emsp;以上操作 SQL 部分的执行系统环境是 CentOS 7.x ，对应的 MySQL 版本是 5.7.29 ，单机环境，其他版本或环境未实际测试；且 Shell 部分未实际测试，仅供参考！

**温馨提示：执行上述操作时请注意数据安全问题，有必要提前备份好需要操作的数据库或数据表相关信息，切记！！！**
