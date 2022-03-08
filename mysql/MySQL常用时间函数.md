# MySQL常用时间函数

&emsp;&emsp;由于平时在项目某些需求中需要处理筛查某些时间段内的数据，所以难免会拿 MySQL 开刀，直接在数据查询阶段处理好后直接得到想要的结果就不用二次处理了。可能这里有些同学会觉得 SQL 查询不是说尽量不要作计算类的操作么？平时听到很多讲优化故事的地方会见到这种说法，但是在实际开发过程中给我的感觉并不完全是这样，面对同样的问题也需要不同的分析，需要考虑实际的运用场景，脱离实际讲理论不就是在耍流氓么。

&emsp;&emsp;今天咋们不讨论优化，只是针对 MySQL 的一个入门，描述一些常用日期时间相关的 SQL 查询，如果不懂啥是SQL，可以提前去了解一下，这里的这些查询是需要有一点 SQL 基础的，不然看不懂就别怪我咯。

### 一、时间类型的常用查询和相关函数

&emsp;&emsp;在 MySQL 查询数据的时候，通常我们会遇到查询一段时间的数据，这时候就涉及到时间的取值。MySQL 带有的一些时间函数可以让我们更轻松的获取对应的时间进行查询。

**注意：下面示例中所使用的 NOW() 函数对应的日期是 2020-04-29，时间则是动态的。**

#### 1、MySQL 中的时间函数有如下一些：

##### (1)、NOW

###### 语法：

```SQL
NOW() -- 获取当前 MySQL 服务器日期时间
```

###### 示例：

```SQL
SELECT NOW(); -- 2020-04-28 22:38:46
```

##### (1.1)、CURRENT_TIMESTAMP

###### 语法：

```SQL
CURRENT_TIMESTAMP() -- 获取当前 MySQL 服务器日期时间
```

###### 示例：

```SQL
SELECT CURRENT_TIMESTAMP(); -- 2020-04-28 22:38:46
```

##### (1.2)、LOCALTIME

###### 语法：

```SQL
LOCALTIME() -- 获取当前 MySQL 服务器日期时间
```

###### 示例：

```SQL
SELECT LOCALTIME(); -- 2020-04-28 22:38:46
```

##### (1.3)、LOCALTIMESTAMP

###### 语法：

```SQL
LOCALTIMESTAMP() -- 获取当前 MySQL 服务器日期时间
```

###### 示例：

```SQL
SELECT LOCALTIMESTAMP(); -- 2020-04-28 22:38:46
```

##### (1.4)、CURRENT_TIMESTAMP

###### 语法：

```SQL
CURRENT_TIMESTAMP() -- 获取当前 MySQL 服务器日期时间
```

###### 示例：

```SQL
SELECT CURRENT_TIMESTAMP(); -- 2020-04-28 22:38:46
```

注：NOW() 和 CURRENT_TIMESTAMP、CURRENT_TIMESTAMP、LOCALTIME、LOCALTIMESTAMP、CURRENT_TIMESTAMP 没有任何区别，他们都表示的是SQL开始执行时的 MySQL 服务器系统时间。

⚠️注意：以上时间函数除了 NOW 后面必须加上英文括号()以外，其他几个均可以不加英文括号也可以正常执行。

##### (1.5)、SYSDATE

###### 语法：

```SQL
SYSDATE() -- 获取当前 SQL 执行时的 MySQL 服务器系统时间(动态取值)
```

###### 示例：

```SQL
SELECT SYSDATE(); -- 2020-04-28 22:38:46
```

注：NOW() 和 SYSDATE() 的主要区别是：NOW() 是在 SQL 开始执行时的系统时间，SYSDATE() 是在 SQL 执行过程中动态获取的系统时间，下面来测试看一下结果：

```SQL
SELECT SYSDATE(6), CURRENT_TIMESTAMP(6);

-- SYSDATE(6)的结果: 2020-04-29 14:00:50.498157
-- CURRENT_TIMESTAMP(6)的结果: 2020-04-29 14:00:50.498087
-- 由上面的结果可以看到，SYSDATE(6) 大于 CURRENT_TIMESTAMP(6);
```

注：NOW()、CURRENT_TIMESTAMP()、LOCALTIME()、LOCALTIMESTAMP()、CURRENT_TIMESTAMP()、SYSDATE() 这几个时间函数都可以传入要获取时间的精度。
比如：NOW(3),NOW(6),这样获取到的时间会带上毫秒数值或微秒数值(3-毫秒；6-微秒)。

##### (2)、CURDATE

###### 语法：

```SQL
CURDATE() -- 获取当前 MySQL 服务器日期(仅获取日期，不含具体时间)
```

###### 示例：

```SQL
SELECT CURDATE(); -- 2020-04-28
```

类似的函数还有：CURRENT_DATE 和 CURRENT_DATE();

##### (3)、CURTIME

###### 语法：

```SQL
CURTIME() -- 获取当前 MySQL 服务器时间(仅获取时间，不含日期，和CURDATE 相对)
```

###### 示例：

```SQL
SELECT CURTIME(); -- 22:45:07
```

类似的函数还有：CURRENT_TIME 和 CURRENT_TIME();

##### (4)、DATE

###### 语法：

```SQL
DATE(date) -- 获取日期时间或者日期的日期部分
```

###### 示例：

```SQL
SELECT DATE('2020-04-28 22:48:56 237'); -- 2020-04-28
SELECT DATE('2020-04-28'); -- 2020-04-28
```

##### (5)、EXTRACT

###### 语法：

```SQL
EXTRACT(unit FROM date) -- 获取返回日期/时间的单独部分，比如年、月、日、小时、分钟等等

-- date 参数是合法的日期表达式。

-- unit 参数可以是下列的值：

-- MICROSECOND --> 返回时间或日期时间表达式expr的微秒，这个数字范围为 0 到 999999
-- SECOND --> 返回时间的秒，取值范围为0〜59
-- MINUTE --> 返回时间的分钟，范围为0至59
-- HOUR --> 返回时间的小时部分。返回值的范围为0至23的小时值。然而，TIME值的范围实际上要大得多，所以HOUR可以返回大于23的值
-- DAY --> 返回给定日期的月份的日期部分
-- WEEK --> 返回日期的周号。 WEEK() 的双参数使您能够指定星期是从星期天还是星期一开始，以及返回值是在0到53还是从1到53的范围内。如果省略mode参数，则值使用了default_week_format系统变量
-- MONTH --> 返回日期的月份，1月至12月的范围为1至12，对于包含月份为零的日期（如“0000-00-00”或“2008-00-00”），返回0
-- QUARTER --> 返回日期的一年中的季度，范围为1到4
-- YEAR --> 返回年份，范围为1000到9999，或者对于“零”日期返回0
-- SECOND_MICROSECOND --> 返回对应日期时间当前秒数及微秒数值(如：2020-04-29 09:54:09 返回值：9000000)
-- MINUTE_MICROSECOND --> 返回对应日期时间当前分钟数、秒数及微秒数值(如：2020-04-29 09:54:09 返回值：5409000000)
-- MINUTE_SECOND --> 返回对应日期时间当前分钟数及秒数值(如：2020-04-29 09:54:09 返回值：5409)
-- HOUR_MICROSECOND --> 返回对应日期时间当前小时分钟秒及微秒数值(如：2020-04-29 09:54:09 返回值：95409000000)
-- HOUR_SECOND --> 返回对应日期时间当前小时分钟秒数值(如：2020-04-29 09:54:09 返回值：95409)
-- HOUR_MINUTE --> 返回对应日期时间的当前小时及分钟数值(如：2020-04-29 09:54:09 返回值：954)
-- DAY_MICROSECOND --> 返回对应日期时间当天的小时分钟秒及微秒数值(如：2020-04-29 09:54:09 返回值：95409000000)
-- DAY_SECOND --> 返回对应日期时间当天的小时分钟秒数值(如：2020-04-29 09:54:09 返回值：95409)
-- DAY_MINUTE --> 返回对应日期时间当天的小时及分钟数值(如：2020-04-29 09:54:09 返回值：954)
-- DAY_HOUR --> 返回对应时间当天的小时数值(0~23)
-- YEAR_MONTH --> 返回对应日期时间的年和月(如：202004)
```

###### 示例：

```SQL
-- 获取当前年月:
SELECT EXTRACT(YEAR_MONTH FROM NOW()); -- 202004

-- 获取当前周是一年中的第几周:
SELECT EXTRACT(WEEK FROM NOW()); -- 17

-- 获取当前 MySQL 服务器日期是哪个季度:
SELECT EXTRACT(QUARTER FROM NOW()); -- 2 (1-春季；2-夏季；3-秋季；4-冬季；)
```

##### (6)、DATE_ADD

###### 语法：

```SQL
DATE_ADD(date,INTERVAL expr type) -- 将日期时间添加指定的日期时间间隔

-- date 参数是合法的日期时间表达式；

-- expr 参数是添加的时间间隔；

-- type 参数可以是下列值(对应的含义请看上文)：

-- MICROSECOND、SECOND、MINUTE、HOUR、DAY、WEEK、MONTH、QUARTER、YEAR、SECOND_MICROSECOND、MINUTE_MICROSECOND、MINUTE_SECOND、HOUR_MICROSECOND、HOUR_SECOND、HOUR_MINUTE、DAY_MICROSECOND、DAY_SECOND、DAY_MINUTE、DAY_HOUR、YEAR_MONTH
```

###### 示例：

```SQL
-- 获取 3 天后的日期时间:
SELECT DATE_ADD(NOW(),INTERVAL 3 DAY); -- 2020-05-02 11:50:07
-- 获取明年的今天这个时间:
SELECT DATE_ADD(NOW(),INTERVAL 1 YEAR); -- 2021-04-29 11:49:41
```

##### (7)、DATE_SUB

###### 语法：

```SQL
DATE_SUB(date,INTERVAL expr type) -- 将日期时间减去指定的日期时间间隔

-- date 参数是合法的日期表达式；

-- expr 参数是添加的时间间隔；

-- type 参数可以是下列值(对应的含义请看上文)：

-- MICROSECOND、SECOND、MINUTE、HOUR、DAY、WEEK、MONTH、QUARTER、YEAR、SECOND_MICROSECOND、MINUTE_MICROSECOND、MINUTE_SECOND、HOUR_MICROSECOND、HOUR_SECOND、HOUR_MINUTE、DAY_MICROSECOND、DAY_SECOND、DAY_MINUTE、DAY_HOUR、YEAR_MONTH
```

###### 示例：

```SQL
-- 获取 3 天前的日期时间:
SELECT DATE_SUB(NOW(),INTERVAL 3 DAY); -- 2020-04-26 11:53:40
```

##### (8)、DATEDIFF

###### 语法：

```SQL
DATEDIFF(date1,date2) -- 获取两个日期之间的天数。(只有日期部分参与计算)

-- date1 和 date2 参数是合法的日期或日期-时间表达式
```

###### 示例：

```SQL
-- 距离 2020 年 五一 小长假还有多少天:
SELECT DATEDIFF('2020-05-01', NOW()); -- 2
```

##### (9)、DATE_FORMAT

###### 语法：

```SQL
DATE_FORMAT(date,format) -- 格式化日期时间

-- date 参数是合法的日期时间;

-- format 规定日期时间的输出格式;

-- format 参数格式：

-- %a：缩写星期名
-- %b：缩写月名
-- %c：月，数值
-- %D：带有英文前缀的月中的天
-- %d：月的天，数值(00-31)
-- %e：月的天，数值(0-31)
-- %f：微秒
-- %H：小时 (00-23)
-- %h：小时 (01-12)
-- %I：小时 (01-12)
-- %i：分钟，数值(00-59)
-- %j：年的天 (001-366)
-- %k：小时 (0-23)
-- %l：小时 (1-12)
-- %M：月名
-- %m：月，数值(00-12)
-- %p：AM 或 PM
-- %r：时间，12-小时（hh:mm:ss AM 或 PM）
-- %S：秒(00-59)
-- %s：秒(00-59)
-- %T：时间, 24-小时 (hh:mm:ss)
-- %U：周 (00-53) 星期日是一周的第一天
-- %u：周 (00-53) 星期一是一周的第一天
-- %V：周 (01-53) 星期日是一周的第一天，与 %X 使用
-- %v：周 (01-53) 星期一是一周的第一天，与 %x 使用
-- %W：星期名
-- %w：周的天 （0=星期日, 6=星期六）
-- %X：年，其中的星期日是周的第一天，4 位，与 %V 使用
-- %x：年，其中的星期一是周的第一天，4 位，与 %v 使用
-- %Y：年，4 位
-- %y：年，2 位
```

###### 示例：

```SQL
-- 格式化当前日期时间:
SELECT DATE_FORMAT(NOW(), '%Y年%m月%d日 %k时%i分%s秒'); -- 2020年04月29日 13时28分04秒
```

#### 2、日期时间选取函数

注：以下函数中的参数 expr/date 均是合法的日期或日期-时间表达式

##### (1)、DATE(expr) -- 获取日期

###### 示例：

```SQL
SELECT DATE(NOW()); -- 结果：2020-04-29
SELECT DATE('2020-04-29 13:34:32'); -- 结果：2020-04-29
```

##### (2)、TIME(expr) -- 获取时间

###### 示例：

```SQL
SELECT TIME(NOW()); -- 结果：15:25:25
SELECT TIME(NOW(6)); -- 结果：15:26:01.174826
SELECT TIME('2020-04-29 13:34:32'); -- 结果：13:34:32
SELECT TIME('2020-04-29 13:34:32.123456'); -- 结果：13:34:32.123456
```

##### (3)、YEAR(date) -- 获取年份

###### 示例：

```SQL
SELECT YEAR(NOW()); -- 结果：2020
SELECT YEAR('2020-04-29 13:34:32.123'); -- 结果：2020
```

##### (4)、MONTH(date) -- 获取月份

###### 示例：

```SQL
SELECT MONTH(NOW()); -- 结果：4
SELECT MONTH('2020-04-29 13:34:32.123'); -- 结果：4
```

##### (5)、DAY(date) -- 获取日

###### 示例：

```SQL
SELECT DAY(NOW()); -- 结果：29
SELECT DAY('2020-04-29 13:34:32.123'); -- 结果：29
```

##### (6)、HOUR(time) -- 获取时

###### 示例：

```SQL
SELECT HOUR(NOW()); -- 结果：18
SELECT HOUR('2020-04-29 13:34:32.123'); -- 结果：13
```

##### (7)、MINUTE(time) -- 获取分

###### 示例：

```SQL
SELECT MINUTE(NOW()); -- 结果：7
SELECT MINUTE('2020-04-29 13:34:32.123'); -- 结果：34
```

##### (8)、SECOND(time) -- 获取秒

###### 示例：

```SQL
SELECT SECOND(NOW()); -- 结果：55
SELECT SECOND('2020-04-29 13:34:32.123'); -- 结果：32
```

##### (9)、MICROSECOND(expr) -- 获取毫秒

###### 示例：

```SQL
SELECT MICROSECOND(NOW(3)); -- 结果：799000
SELECT MICROSECOND('2020-04-29 13:34:32.123'); -- 结果：123000
```

 

##### (10)、QUARTER(date) -- 获取季度

###### 示例：

```SQL
SELECT MICROSECOND(NOW()); -- 结果：2
SELECT MICROSECOND('2020-04-29 13:34:32.123'); -- 结果：2
```

##### (11)、WEEK(date[,mode]) -- 获取周(计算出当前日期所在周数)

```SQL
-- date 是要获取周数的日期。
-- mode 是一个可选参数，用于确定周数计算的逻辑。它允许您指定本周是从星期一还是星期日开始，返回的周数应在0到52之间或0到53之间。
```

###### 示例：

```SQL
SELECT MICROSECOND(NOW()); -- 结果：17
SELECT MICROSECOND(NOW(), 1); -- 结果：18
SELECT MICROSECOND('2020-04-29 13:34:32.123'); -- 结果：17
SELECT MICROSECOND('2020-04-29 13:34:32.123', 1); -- 结果：18
```

##### (12)、WEEKOFYEAR(date) -- 获取周(计算出当前日期所在周数)

###### 示例：

```SQL
SELECT WEEKOFYEAR(NOW()); -- 结果：18
SELECT WEEKOFYEAR('2020-04-29 13:34:32.123'); -- 结果：18
```

##### (13)、DAYOFYEAR(date) -- 日期在年度中第几天

###### 示例：

```SQL
SELECT DAYOFYEAR(NOW()); -- 结果：120
SELECT DAYOFYEAR('2020-04-29 13:34:32.123'); -- 结果：120
```

##### (14)、DAYOFMONTH(date) -- 日期在月度中第几天

###### 示例：

```SQL
SELECT DAYOFMONTH(NOW()); -- 结果：29
SELECT DAYOFMONTH('2020-04-29 13:34:32.123'); -- 结果：29
```

##### (15)、DAYOFWEEK(date) -- 日期在周中第几天；周日为第一天

###### 示例：

```SQL
SELECT DAYOFWEEK(NOW()); -- 结果：4
SELECT DAYOFWEEK('2020-04-29 13:34:32.123'); -- 结果：4
```

##### (16)、WEEKDAY(date) -- 日期在周中第几天；周日为第一天

注：与 DAYOFWEEK() 都表示日期在周的第几天，只是参考标准不同， WEEKDAY() 周一为第0天，周日为第6天)

###### 示例：

```SQL
SELECT WEEKDAY(NOW()); -- 结果：2
SELECT WEEKDAY('2020-04-29 13:34:32.123'); -- 结果：2
```

##### (17)、YEARWEEK(date)/YEARWEEK(date,mode) -- 年和周

###### 示例：

```SQL
-- 默认使用 YEARWEEK(date) 函数测试：
SELECT YEARWEEK(NOW()); -- 结果：202017
SELECT YEARWEEK('2020-04-29 13:34:32.123'); -- 结果：202017

-- 使用 YEARWEEK(date,mode) 函数测试：
SELECT YEARWEEK(NOW(), 1); -- 结果：202018
SELECT YEARWEEK('2020-04-29 13:34:32.123', 1); -- 结果：202018
```

#### 2、MySQL 中的 UNIX 时间函数

##### (1)、UNIX_TIMESTAMP

###### 语法：

```SQL
UNIX_TIMESTAMP(date) -- 将时间转换为时间戳；如果参数为空，则处理的是当前的时间(返回从'1970-01-01 00:00:00'GMT开始的到当前时间的秒数，不为空则它返回从'1970-01-01 00:00:00' GMT开始的到指定date的秒数值)

-- 计算起始时间 1970-01-01 00:00:00 GMT 可以理解成北京时间 1970-01-01 08:00:00

-- date 可以是一个 DATE 字符串、一个 DATETIME 字符串、一个 TIMESTAMP 或以 YYMMDD 或 YYYYMMDD 格式的本地时间的一个数字。
```

###### 示例：

```SQL
SELECT UNIX_TIMESTAMP(); -- 1588500169
SELECT UNIX_TIMESTAMP('2020-05-03 18:03:49'); -- 1588500229
SELECT UNIX_TIMESTAMP('2020-05-03 18:03:49.678'); -- 1588500229.678
```

⚠️注意：

- 1、MySQL 的 UNIX 时间戳对于毫秒时间的处理和上面的示例一样，用小数位处理；
- 2、对于 UNIX_TIMESTAMP(date) date 支持的时间范围是 1970 ～ 2038，超出这个范围返回值为 NULL (我这里测试得到的结果是 0 )；

##### (2)、FROM_UNIXTIME

###### 语法：

```SQL
FROM_UNIXTIME(unix_timestamp) -- 将时间戳转换为时间，返回表示 Unix 时间标记的一个字符串；
FROM_UNIXTIME(unix_timestamp,format) -- 将时间戳转换为时间，返回表示 Unix 时间标记的一个字符串，根据format字符串格式化。format可以包含与DATE_FORMAT()函数列出的条目同样的修饰符。

-- unix_timestamp 是合法的时间戳(毫秒放小数点后面)；

-- format 是规定日期时间的输出格式(同 DATE_FORMAT() 函数所使用的格式修饰符)
```

###### 示例：

```SQL
SELECT FROM_UNIXTIME(1588500229); -- 2020-05-03 18:03:49
SELECT FROM_UNIXTIME(1588500229.678); -- 2020-05-03 18:03:49.678

SELECT FROM_UNIXTIME(1588500229,'%Y-%m-%d'); -- 2020-05-03
SELECT FROM_UNIXTIME(1588500229,'%Y-%m-%d %H:%i'); -- 2020-05-03 18:03
SELECT FROM_UNIXTIME(1588500229,'%Y-%m-%d %T'); -- 2020-05-03 18:03:49

SELECT FROM_UNIXTIME(1588500229.678,'%Y-%m-%d'); -- 2020-05-03
SELECT FROM_UNIXTIME(1588500229.678,'%Y-%m-%d %H:%i'); -- 2020-05-03 18:03
SELECT FROM_UNIXTIME(1588500229.678,'%Y-%m-%d %T'); -- 2020-05-03 18:03:49
SELECT FROM_UNIXTIME(1588500229.678,'%Y-%m-%d %T:%f'); -- 2020-05-03 18:03:49:678000
```

⚠️注意：MySQL 的 UNIX 时间戳对于毫秒时间的处理和上面的示例一样，用小数位处理；但是格式化毫秒时间戳时无法单独处理，只能处理成微秒(%f)进行格式化。

### 二、总结

&emsp;&emsp;MySQL 的常用日期函数已经在上文阐述差不多了，平时在有关日期查询的时候可以看看适时运用一下 MySQL 的日期函数进行处理，没准儿能降低你的 SQL 编写难度又能提高你的 SQL 效率呢！大家赶快行动吧，多尝试尝试，今后总会用到的。

&emsp;&emsp;这次的 MySQL 日期时间函数就分析到这里了，今后有发现其他的日期函数将会不定时进行更新补充进来～
