# MySQL查询今天&昨天&本周&上周&本月&上月&本年&上年度等日期段数据



### MySQL日期时间格式化

 &emsp;&emsp;MySQL常用时间函数及其格式化可参考之前的文档：++[MySQL常用时间函数及其格式化](https://www.hyblogs.com/archives/mysql-03)++。里面列举了各种日期格式的格式化及日期时间和字符串的相互转换等等内容。

### 根据不同日期时间段查询数据

&emsp;&emsp;下面是列举了一些在MySQL使用环境下按照今天/昨天/本周/上周/本月/上月/本年/去年等的一些查询筛选条件。

```sql
-- 查询近24小时内的数据
SELECT * FROM 表名 WHERE 时间字段名 >=(NOW() - INTERVAL 24 HOUR);

-- 查询今天的数据
SELECT * FROM 表名 WHERE TO_DAYS(时间字段名) = TO_DAYS(NOW());

-- 查询昨天的数据
SELECT * FROM 表名 WHERE TO_DAYS( NOW() ) - TO_DAYS( 时间字段名) <= 1;

-- 查询近7天的数据
SELECT * FROM 表名 WHERE DATE_SUB(CURDATE(), INTERVAL 7 DAY) <= DATE(时间字段名);

-- 查询当前这周的数据(方式一)
SELECT * FROM 表名 WHERE YEARWEEK(DATE_FORMAT(时间字段名,'%Y-%m-%d')) = YEARWEEK(NOW());

-- 查询当前这周的数据(方式二)
SELECT * FROM 表名 WHERE WEEKOFYEAR(FROM_UNIXTIME(时间字段名,'%y-%m-%d')) = WEEKOFYEAR(NOW());

-- 查询上周的数据
SELECT * FROM 表名 WHERE YEARWEEK(DATE_FORMAT(时间字段名,'%Y-%m-%d')) = YEARWEEK(NOW()) - 1;

-- 查询近30天的数据
SELECT * FROM 表名 where DATE_SUB(CURDATE(), INTERVAL 30 DAY) <= DATE(时间字段名);

-- 查询本月数据(方式一)
SELECT * FROM 表名 WHERE DATE_FORMAT(时间字段名, '%Y%m') = DATE_FORMAT(CURDATE() , '%Y%m');

-- 查询本月数据(方式二)
SELECT * FROM 表名 WHERE MONTH(FROM_UNIXTIME(时间字段名,'%y-%m-%d')) = MONTH(NOW());

-- 查询上个月的数据(方式一)
SELECT * FROM 表名 WHERE PERIOD_DIFF(DATE_FORMAT(NOW() , '%Y%m'), DATE_FORMAT(时间字段名, '%Y%m')) = 1;

-- 查询上个月的数据(方式二)
SELECT * FROM 表名 WHERE DATE_FORMAT(时间字段名,'%Y-%m') = DATE_FORMAT(DATE_SUB(CURDATE(), INTERVAL 1 MONTH), '%Y-%m');

-- 查询本季度数据
SELECT * FROM 表名 WHERE QUARTER(时间字段名) = QUARTER(NOW());

-- 查询上季度数据
SELECT * FROM 表名 WHERE QUARTER(时间字段名) = QUARTER(DATE_SUB(NOW(), INTERVAL 1 QUARTER));

-- 查询距离当前现在6个月的数据
SELECT * FROM 表名 WHERE 时间字段名 BETWEEN DATE_SUB(NOW(), INTERVAL 6 MONTH) AND NOW();

-- 查询本年数据
SELECT * FROM 表名 WHERE YEAR(时间字段名)=YEAR(NOW());

-- 查询上年数据
SELECT * FROM 表名 WHERE YEAR(时间字段名) = YEAR(DATE_SUB(NOW(), INTERVAL 1 YEAR));

-- 查询本年度本月份的数据
SELECT * FROM 表名 WHERE YEAR(FROM_UNIXTIME(时间字段名,'%y-%m-%d')) = YEAR(NOW()) AND MONTH(FROM_UNIXTIME(时间字段名,'%y-%m-%d')) = MONTH(NOW());
```
