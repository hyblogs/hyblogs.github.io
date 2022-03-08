# MySQL常用小数处理函数

&emsp;&emsp;前面记录了在 MySQL 中处理日期数据，整理了一些常见的日期处理方式(如：日期时间的格式化、日期和时间戳之间的转换等等)。有需要的小伙伴们可以移步这里：[MySQL常用时间函数](https://www.hyblogs.com/archives/mysql-03)。今天我们一起来整理一下在开发过程中经常会遇到的小数位处理，虽然有时候建议在程序中处理这类数据，但是如果条件允许的话，直接在 SQL 端输出想要的数据就不需要二次处理了，相对程序中处理会更省事儿。(>^ω^<)

### ROUND(X,D) 函数

> &emsp;&emsp;表示将值 X 四舍五入为小数点后 D 位的数值，D为小数点后小数位数。若要保留 X 值小数点左边的 D 位。其中 D 可不传，默认为 0，D 可以是负数，这时是指定小数点左边的 D 位整数位为 0，同时小数位均为 0。

#### 示例：

```sql
SELECT 
    ROUND('1.98755678909876543435667788') AS num0,
    ROUND('1.98755678909876543435667788', 1) AS num1,
    ROUND('1.98755678909876543435667788', 2) AS num2,
    ROUND('1.98755678909876543435667788', 3) AS num3,
    ROUND('1.98755678909876543435667788', 4) AS num4,
    ROUND('1.98755678909876543435667788', 5) AS num5,
    ROUND('12453321.98755678909876543435667788', -1) AS num6,
    ROUND('776534455378651.98755678909876543435667788', -2) AS num7,
    ROUND('4567765456771.98755678909876543435667788', -3) AS num8;
```

执行结果：

| num0 | num1 | num2 | num3  | num4   | num5    | num6     | num7            | num8          |
| ---- | ---- | ---- | ----- | ------ | ------- | -------- | --------------- | ------------- |
| 2    | 2    | 1.99 | 1.988 | 1.9876 | 1.98756 | 12453320 | 776534455378700 | 4567765457000 |

#### 效果图：

![ROUND(X,D)](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1591439146794.png)

### TRUNCATE(X,D) 函数

> &emsp;&emsp;D 表示，舍去小数点后 D 位，并且 D 同样可以为负数，D 是必须传入的参数，可以是 0 ，但不能不传。PS: 不进行四舍五入处理。

#### 示例

```sql
SELECT 
    TRUNCATE(2.987667, 0) AS num0,
    TRUNCATE(2.012342, 1) AS num1,
    TRUNCATE(2.1753243, 2) AS num2,
    TRUNCATE(2.4654345, 3) AS num3,
    TRUNCATE(2.5546656, 4) AS num4,
    TRUNCATE(12322.6453, -1) AS num5,
    TRUNCATE(4321342.9012, -3) AS num6;
```

执行结果：

| num0 | num1 | num2 | num3  | num4   | num5  | num6    |
| ---- | ---- | ---- | ----- | ------ | ----- | ------- |
| 2    | 2    | 2.17 | 2.465 | 2.5546 | 12320 | 4321000 |

#### 效果图

![TRUNCATE(X,D)](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1591440796524.png)

> &emsp;&emsp;此处，我们可以比较得出 ROUND(X,D) 和 TRUNCATE(X,D) 的主要区别是 ROUND(X,D) 函数是四舍五入处理，而 TRUNCATE(X,D) 函数不进行四舍五入处理。

### FLOOR(X) 函数

> 表示向下取整，只返回值 X 的整数部分，小数部分舍弃。

#### 示例：

```sql
SELECT 
    FLOOR(2) AS num0,
    FLOOR(2.0) AS num1,
    FLOOR(2.1) AS num2,
    FLOOR(2.4) AS num3,
    FLOOR(2.5) AS num4,
    FLOOR(2.6) AS num5,
    FLOOR(2.9) AS num6;
```

执行结果：

| num0 | num1 | num2 | num3 | num4 | num5 | num6 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 2    | 2    | 2    | 2    | 2    | 2    | 2    |

#### 效果图：

![FLOOR(X)](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1591439580977.png)

### CEILING(X) 函数

> 表示向上取整，只返回值 X 的整数部分，小数部分舍弃。

示例：

```sql
SELECT 
    CEILING(2) AS num0,
    CEILING(2.0) AS num1,
    CEILING(2.1) AS num2,
    CEILING(2.4) AS num3,
    CEILING(2.5) AS num4,
    CEILING(2.6) AS num5,
    CEILING(2.9) AS num6;
```

执行结果：

| num0 | num1 | num2 | num3 | num4 | num5 | num6 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 2    | 2    | 3    | 3    | 3    | 3    | 3    |

#### 效果图：

![CEILING(X)](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1591440379407.png)

### CEIL(X) 函数

> 表示返回不小于给定数的下一个整数。(返回大于或等于浮点数的整数)

#### 示例：

```sql
SELECT 
    CEIL(1) AS num0,
    CEIL(2.0) AS num1,
    CEIL(2.01) AS num2,
    CEIL(2.98) AS num3,
    CEIL(72.0123) AS num4,
    CEIL(242.1753) AS num5,
    CEIL(48766799.4654345) AS num6;
```

执行结果：

| num0 | num1 | num2 | num3 | num4 | num5 | num6     |
| ---- | ---- | ---- | ---- | ---- | ---- | -------- |
| 1    | 2    | 3    | 3    | 73   | 243  | 48766800 |

#### 效果图：

![CEIL(X)](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1591453921930.png)

> &emsp;&emsp;至此，我们可以很鲜明的看出 FLOOR(X) 、 CEILING(X) 以及 CEIL(X) 对小数处理的区别。

### FORMAT(X,D) 函数

> &emsp;&emsp;表示将值 X 格式化，强制保留值 X 到小数点后 D 位，截断时会进行四舍五入处理。整数部分超过三位的时候以逗号分割，并且返回的结果是 String 字符串类型。如果 D 小于 0 将按照 0 进行处理。

#### 示例：

```sql
SELECT 
    FORMAT(2313222.987667, 0) AS num0,
    FORMAT(8765672.012342, 1) AS num1,
    FORMAT(2432342.1753243, 2) AS num2,
    FORMAT(48766799.4654345, 3) AS num3,
    FORMAT(987672.5546656, 4) AS num4,
    FORMAT(1232872.6453, -1) AS num5,
    FORMAT(4321342.9012, -3) AS num6;
```

执行结果：

| num0      | num1        | num2         | num3           | num4         | num5      | num6      |
| --------- | ----------- | ------------ | -------------- | ------------ | --------- | --------- |
| 2,313,223 | 8,765,672.0 | 2,432,342.18 | 48,766,799.465 | 987,672.5547 | 1,232,873 | 4,321,343 |

#### 效果图：

![FORMAT(X,D)](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1591441467142.png)

### CONVERT(VALUE,TYPE)/CAST(expr AS type) 函数

> &emsp;&emsp;CONVERT(VALUE,TYPE) 与 CAST(expr AS type) 函数都是类型转换，支持四舍五入，只是语法方面稍有不同而已。

TYPE 可选值:

- 二进制，同带 binary 前缀的效果 : BINARY
- 字符型，可带参数 : CHAR()
- 日期 : DATE
- 时间: TIME
- 日期时间型 : DATETIME
- 浮点数 : DECIMAL
- 整数 : SIGNED
- 无符号整数 : UNSIGNED

#### 采用 CONVERT(VALUE,TYPE) 函数的示例：

> &emsp;&emsp;此处只针对性的展示对数字类型的值的处理示例，其余的如：日期，字符串、时间等等类型的处理可以自己去尝试一下哦。

```sql
SELECT 
    CONVERT(1, DECIMAL(10, 2)) AS num0,
    CONVERT(-1, DECIMAL(10, 2)) AS num1,
    CONVERT(1.125, DECIMAL(10, 2)) AS num2,
    CONVERT(-1.125, DECIMAL(10, 2)) AS num3,
    CONVERT(1.125, DECIMAL) AS num4,
    CONVERT(-1.125, DECIMAL) AS num5,
    CONVERT(1.425, SIGNED) AS num6,
    CONVERT(-1.425, SIGNED) AS num7,
    CONVERT(1.525, SIGNED) AS num8,
    CONVERT(-1.525, SIGNED) AS num9,
    CONVERT(1.525, UNSIGNED) AS num10,
    CONVERT(-1.525, UNSIGNED) AS num11,
    CONVERT(1.425, UNSIGNED) AS num12,
    CONVERT(-1.425, UNSIGNED) AS num13;
```

执行结果：

| num0 | num1 | num2 | num3  | num4 | num5 | num6 | num7 | num8 | num9 | num10 | num11 | num12 | num13 |
| ---- | ---- | ---- | ----- | ---- | ---- | ---- | ---- | ---- | ---- | ----- | ----- | ----- | ----- |
| 1    | -1   | 1.13 | -1.13 | 1    | -1   | 1    | -1   | 2    | -2   | 2     | 0     | 1     | 0     |

#### 效果图：

![CONVERT(VALUE,TYPE)](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1591454908938.png)

#### 采用 CAST(expr AS type) 函数的示例：

> &emsp;&emsp;此处只针对性的展示对数字类型的值的处理示例，其余的如：日期，字符串、时间等等类型的处理可以自己去尝试一下哦。

```sql
SELECT 
    CAST(1 AS DECIMAL(10, 2)) AS num0,
    CAST(-1 AS DECIMAL(10, 2)) AS num1,
    CAST(1.125 AS DECIMAL(10, 2)) AS num2,
    CAST(-1.125 AS DECIMAL(10, 2)) AS num3,
    CAST(1.125 AS DECIMAL) AS num4,
    CAST(-1.125 AS DECIMAL) AS num5,
    CAST(1.425 AS SIGNED) AS num6,
    CAST(-1.425 AS SIGNED) AS num7,
    CAST(1.525 AS SIGNED) AS num8,
    CAST(-1.525 AS SIGNED) AS num9,
    CAST(1.525 AS UNSIGNED) AS num10,
    CAST(-1.525 AS UNSIGNED) AS num11,
    CAST(1.425 AS UNSIGNED) AS num12,
    CAST(-1.425 AS UNSIGNED) AS num13;
```

执行结果：

| num0 | num1 | num2 | num3  | num4 | num5 | num6 | num7 | num8 | num9 | num10 | num11 | num12 | num13 |
| ---- | ---- | ---- | ----- | ---- | ---- | ---- | ---- | ---- | ---- | ----- | ----- | ----- | ----- |
| 1    | -1   | 1.13 | -1.13 | 1    | -1   | 1    | -1   | 2    | -2   | 2     | 0     | 1     | 0     |

#### 效果图：

![CAST(expr AS type)](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1591455271968.png)

> &emsp;&emsp;由此可见，采用 CONVERT(VALUE,TYPE) 与 CAST(expr AS type) 函数对相同的数据执行结果都是一致的，只是语法不同而已。

> **这里，我抛出一种看着不正常的情况**，如下图：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1591455774349.png)

> &emsp;&emsp;那么当 type 是 UNSIGNED 时，带符号(减号)的数字字符串处理将出现上述结果，各位可以去尝试一下，虽然 CONVERT / CAST 函数是用来转换类型的，但是也不可随意使用，请大家注意一下类似问题，避免踩坑！

### 总结

> &emsp;&emsp;今天的 MySQL 常用浮点数小数位处理的方法到这里就要结束啦，接下来将对上述提到的函数进行归一到表格中，方便大家查询使用：

| 函数                                     | 说明                                                                     |
| -------------------------------------- | ---------------------------------------------------------------------- |
| FLOOR(X)                               | 返回不大于值 X 的最大整数。                                                        |
| CEIL(X)、CEILING(X)                     | 返回不小于值 X 的最小整数。                                                        |
| TRUNCATE(X,D)                          | 返回数值 X 保留到小数点后 D 位的值，截断时不进行四舍五入处理。                                     |
| ROUND(X)                               | 返回离值 X 最近的整数，截断时要进行四舍五入处理。                                             |
| ROUND(X,D)                             | 保留值 X 小数点后 D 位的值，截断时要进行四舍五入处理。                                         |
| FORMAT(X,D)                            | 将数值 X 格式化，将 X 保留到小数点后 D 位，截断时要进行四舍五入处理。                                |
| CONVERT(VALUE,TYPE)/CAST(expr AS type) | CONVERT(VALUE,TYPE) 与 CAST(expr AS type) 函数都是类型转换，支持四舍五入，只是语法方面稍有不同而已。 |

> 注：本次示例验证的系统环境是 CentOS 7.x ，数据库版本是 v_5.7.29.
