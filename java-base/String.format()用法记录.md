# String.format() 用法记录



&emsp;&emsp;这几天在编写示例代码的时候经常会用到 System.out.printf() ，因为方便打印各种参数以及执行结果。为啥要用这个方法呢，最常用的不是 System.out.println() 吗？其实 System.out.printf() 对应的实现中用到了和 String.format() 相同的 Formatter.class ，所以平时使用过 String.format() 的同学在使用 System.out.printf() 自然可以很顺利的完成编写。但是，可能有的同学没有使用过 String.format() 更不常用 System.out.printf() ，所以使用起来对某些占位符比较陌生，接下来我们一起来对这里面的占位符以及使用进行梳理 ，帮助大家巩固他们的用法。

### System.out.printf()

```java
// java.util.Formatter 和 String.format() 采用的 Formatter 相同
private Formatter formatter;

/**
 * 使用指定的格式字符串和参数将格式化字符串写入此输出流。 始终使用的语言环境是 local.getdefault() 返回的语言环境，而与之前对该对象调用其他格式化方法无关。
 *
 * @param  format - 格式字符串语法中描述的格式字符串 
 * @param  args — 格式说明符在格式字符串中引用的参数。如果参数多于格式说明符，则忽略额外的参数。参数的数量是可变的，可以为零。参数的最大数量受到 Java™虚拟机规范定义的 Java 数组的最大维数的限制。空参数的行为取决于转换。 
 * @return  此输出流 
 * @throws  java.util.IllegalFormatException 如果格式字符串包含非法语法、与给定参数不兼容的格式说明符、给定格式字符串的参数不足或其他非法条件。有关所有可能的格式化错误的规范，请参阅formatter类规范的详细信息部分。 
 * @throws  NullPointerException —— 如果格式为null
 */
public PrintStream format(String format, Object ... args) {
    try {
        synchronized (this) {
            ensureOpen();
            if ((formatter == null)
                    || (formatter.locale() != Locale.getDefault()))
                formatter = new Formatter((Appendable) this);
            formatter.format(Locale.getDefault(), format, args);
        }
    } catch (InterruptedIOException x) {
        Thread.currentThread().interrupt();
    } catch (IOException x) {
        trouble = true;
    }
    return this;
}
```

### String.format()

```java
/**
 * 使用指定的格式字符串和参数返回格式化字符串。 始终使用的语言环境是 local.getdefault() 返回的语言环境。
 * 
 * @param  格式字符串
 * @param  格式说明符在格式字符串中引用的参数。如果参数多于格式说明符，则忽略额外的参数。参数的数量是可变的，可以为零。参数的最大数量受到 Java™ 虚拟机规范定义的 Java 数组的最大维数的限制。空参数的行为取决于转换。
 *
 * @throws  java.util.IllegalFormatException 如果格式字符串包含非法语法、与给定参数不兼容的格式说明符、给定格式字符串的参数不足或其他非法条件。有关所有可能的格式化错误的规范，请参阅 formatter 类规范的详细信息部分。
 *
 * @return  一个格式化的字符串
 *
 * @see  java.util.Formatter
 */
public static String format(String format, Object... args) {
    return new Formatter().format(format, args).toString();
}
```

由上面的源码可以看出，他们的具体实现几乎一致。

&emsp;&emsp;在平时的开发过程中，可能会遇到这样的需求：要求给前端输出一段提示性语句，但是语句中包含动态的数据(客户昵称、性别、地址等等)。由于只是在某个场景下的固定格式的提示性语句，所以我们可以选择返回一段字符串给前端进行展示即可，至于动态数据，我们采用 String.format() 处理一下不就好了，是不是感觉简单多啦。

**String.format() 字符串格式化的方式有下面两种**：

- format(String format, Object… args) 新字符串使用本地语言环境，制定字符串格式和参数生成格式化的新字符串。

- format(Locale locale, String format, Object… args) 使用指定的语言环境，制定字符串格式和参数生成格式化的字符串。

当然，默认情况下，我们使用的是第一种。

### 常用的转换符

| 转换符 | 详细说明                    | 替换参数示例          |
| --- | ----------------------- | --------------- |
| %s  | 字符串类型                   | “我喜欢~”          |
| %c  | 字符类型                    | ‘m’             |
| %b  | 布尔类型                    | true            |
| %d  | 整数类型（十进制）               | 88              |
| %x  | 整数类型（十六进制）              | FF              |
| %o  | 整数类型（八进制）               | 77              |
| %f  | 浮点类型                    | 8.888           |
| %a  | 十六进制浮点类型                | FF.35AE         |
| %e  | 指数类型                    | 9.38e+5         |
| %g  | 通用浮点类型（f和e类型中较短的）       | 1.88995         |
| %h  | 散列码                     | A               |
| %%  | 百分比类型                   | ％(%特殊字符%%才能显示%) |
| %n  | 换行符                     |                 |
| %tx | 日期与时间类型（x代表不同的日期与时间转换符) |                 |

### 搭配转换符

| 标志符 | 说明                              | 示例                          | 结果              |
| --- | ------------------------------- | --------------------------- | --------------- |
| +   | 为正数或者负数添加符号                     | ("%+d",15)                  | +15             |
| −   | 左对齐                             | ("%-5d",15)                 | 15              |
| 0   | 数字前面补0                          | ("%04d", 99)                | 0099            |
| 空格  | 在整数之前添加指定数量的空格                  | ("% 4d", 99)                | 99              |
| ,   | 以“,”对数字分组                       | ("%,f", 9999.99)            | 9,999.990000    |
| (   | 使用括号包含负数                        | ("%(f", -99.99)             | (99.990000)     |
| #   | 如果是浮点数则包含小数点，如果是16进制或8进制则添加0x或0 | ("%#x", 99)<br/>("%#o", 99) | 0x63<br/>0143   |
| <   | 格式化前一个转换符所描述的参数                 | ("%f和%< 3.2f", 99.45)       | 99.450000和99.45 |
| $   | 被格式化的参数索引                       | ("%1$d,%2$s", 99,"abc")     | 99,abc          |

### 日期和时间转换符

| 转换符 | 说明                           | 示例                          |
| --- | ---------------------------- | --------------------------- |
| c   | 包括全部日期和时间信息                  | 星期六 十月 27 14:21:20 CST 2020 |
| F   | “年-月-日”格式                    | 2020-10-27                  |
| D   | “月/日/年”格式                    | 10/27/20                    |
| r   | “HH:MM:SS PM”格式（12时制）        | 02:25:51 下午                 |
| T   | “HH:MM:SS”格式（24时制）           | 14:28:16                    |
| R   | “HH:MM”格式（24时制）              | 14:28                       |
| --- | ---                          | ---                         |
| H   | 2位数字24时制的小时（不足2位前面补0）        | 15                          |
| I   | 2位数字12时制的小时（不足2位前面补0）        | 03                          |
| k   | 2位数字24时制的小时（前面不补0）           | 15                          |
| l   | 2位数字12时制的小时（前面不补0）           | 3                           |
| M   | 2位数字的分钟（不足2位前面补0）            | 03                          |
| S   | 2位数字的秒（不足2位前面补0）             | 09                          |
| L   | 3位数字的毫秒（不足3位前面补0）            | 015                         |
| N   | 9位数字的毫秒数（不足9位前面补0）           | 562000000                   |
| p   | 小写字母的上午或下午标记                 | 中：下午<br/>英：pm               |
| z   | 相对于GMT的RFC822时区的偏移量          | +0800                       |
| Z   | 时区缩写字符串                      | CST                         |
| s   | 1970-1-1 00:00:00 到现在所经过的秒数  | 1193468128                  |
| Q   | 1970-1-1 00:00:00 到现在所经过的毫秒数 | 1193468128984               |

### 示例

```java
public class MyCollection {
    public static void main(String[] args) {
        System.out.printf("Hello,%s %n", "Boys And Girls");
        System.out.printf("Hi,%s:%s.%s %n", "boys", "and", "girls");
        System.out.printf("字母a的大写是：%c %n", 'A');
        System.out.printf("3>7的结果是：%b %n", false);
        System.out.printf("100的一半是：%d %n", 100 / 2);
        System.out.printf("100的16进制数是：%x %n", 100);
        System.out.printf("100的8进制数是：%o %n", 100);
        System.out.printf("50元的书打8.5折扣是：%f 元%n", 50 * 0.85);
        System.out.printf("上面价格的16进制数是：%a %n", 50 * 0.85);
        System.out.printf("上面价格的指数表示：%e %n", 50 * 0.85);
        System.out.printf("上面价格的指数和浮点数结果的长度较短的是：%g %n", 50 * 0.85);
        System.out.printf("上面的折扣是%d%% %n", 85);
        System.out.printf("字母A的散列码是：%h %n", 'A');

        System.out.println("==========================================");

        //$的使用
        System.out.println(String.format("格式参数$的使用：%1$d,%2$s", 99, "abc"));
        //+的使用
        System.out.printf("显示正负数的符号：%+d 与 %d %n", 99, -99);
        //补O的使用
        System.out.printf("最牛的特工编号是：%03d %n", 7);
        //空格的使用
        System.out.printf("Tab键的效果是：% 8d %n", 7);
        //,的使用
        System.out.printf("整数分组的效果是：%,d %n", 9989997);
        //空格和小数点后面个数
        System.out.printf("一本书的价格是：% 20.6f 元 %n", 49.8);

        System.out.println("==========================================");

        Date date = new Date();
        //c的使用
        System.out.printf("完整的日期和时间信息：%tc %n", date);
        //f的使用
        System.out.printf("年-月-日格式：%tF %n", date);
        //d的使用
        System.out.printf("月/日/年格式：%tD %n", date);
        //r的使用
        System.out.printf("HH:MM:SS PM格式（12时制）：%tr %n", date);
        //t的使用
        System.out.printf("HH:MM:SS格式（24时制）：%tT %n", date);
        //R的使用
        System.out.printf("HH:MM格式（24时制）：%tR %n", date);
        //混合使用
        System.out.printf("年-月-日 HH:MM:SS 格式(24小时)：%tF %tT %n", date, date);

        System.out.println("==========================================");

        //b的使用，月份简称
        System.out.println(String.format(Locale.US, "英文月份简称：%tb", date));
        System.out.printf("本地月份简称：%tb%n", date);
        //B的使用，月份全称
        System.out.println(String.format(Locale.US, "英文月份全称：%tB", date));
        System.out.printf("本地月份全称：%tB%n", date);
        //a的使用，星期简称
        System.out.println(String.format(Locale.US, "英文星期的简称：%ta", date));
        //A的使用，星期全称
        System.out.printf("本地星期的简称：%tA %n", date);
        //C的使用，年前两位
        System.out.printf("年的前两位数字（不足两位前面补0）：%tC %n", date);
        //y的使用，年后两位
        System.out.printf("年的后两位数字（不足两位前面补0）：%ty %n", date);
        //j的使用，一年的天数
        System.out.printf("一年中的天数（即年的第几天）：%tj %n", date);
        //m的使用，月份
        System.out.printf("两位数字的月份（不足两位前面补0）：%tm %n", date);
        //d的使用，日（二位，不够补零）
        System.out.printf("两位数字的日（不足两位前面补0）：%td %n", date);
        //e的使用，日（一位不补零）
        System.out.printf("月份的日（前面不补0）：%te %n", date);

        System.out.println("==========================================");

        //H的使用
        System.out.printf("2位数字24时制的小时（不足2位前面补0）:%tH %n", date);
        //I的使用
        System.out.printf("2位数字12时制的小时（不足2位前面补0）:%tI %n", date);
        //k的使用
        System.out.printf("2位数字24时制的小时（前面不补0）:%tk %n", date);
        //l的使用
        System.out.printf("2位数字12时制的小时（前面不补0）:%tl %n", date);
        //M的使用
        System.out.printf("2位数字的分钟（不足2位前面补0）:%tM %n", date);
        //S的使用
        System.out.printf("2位数字的秒（不足2位前面补0）:%tS %n", date);
        //L的使用
        System.out.printf("3位数字的毫秒（不足3位前面补0）:%tL %n", date);
        //N的使用
        System.out.printf("9位数字的毫秒数（不足9位前面补0）:%tN %n", date);
        //p的使用
        System.out.println(String.format(Locale.US, "小写字母的上午或下午标记(英)：%tp", date));
        System.out.printf("小写字母的上午或下午标记（中）：%tp %n", date);
        //z的使用
        System.out.printf("相对于GMT的RFC822时区的偏移量:%tz %n", date);
        //Z的使用
        System.out.printf("时区缩写字符串:%tZ %n", date);
        //s的使用
        System.out.printf("1970-1-1 00:00:00 到现在所经过的秒数：%ts %n", date);
        //Q的使用
        System.out.printf("1970-1-1 00:00:00 到现在所经过的毫秒数：%tQ %n", date);
    }
}
```

**执行结果**：

```java
Hello,Boys And Girls 
Hi,boys:and.girls 
字母a的大写是：A 
3>7的结果是：false 
100的一半是：50 
100的16进制数是：64 
100的8进制数是：144 
50元的书打8.5折扣是：42.500000 元
上面价格的16进制数是：0x1.54p5 
上面价格的指数表示：4.250000e+01 
上面价格的指数和浮点数结果的长度较短的是：42.5000 
上面的折扣是85% 
字母A的散列码是：41 
==========================================
格式参数$的使用：99,abc
显示正负数的符号：+99 与 -99 
最牛的特工编号是：007 
Tab键的效果是：       7 
整数分组的效果是：9,989,997 
一本书的价格是：           49.800000 元 
==========================================
完整的日期和时间信息：星期一 七月 27 21:18:21 CST 2020 
年-月-日格式：2020-07-27 
月/日/年格式：07/27/20 
HH:MM:SS PM格式（12时制）：09:18:21 下午 
HH:MM:SS格式（24时制）：21:18:21 
HH:MM格式（24时制）：21:18 
年-月-日 HH:MM:SS 格式(24小时)：2020-07-27 21:18:21 
==========================================
英文月份简称：Jul
本地月份简称：七月
英文月份全称：July
本地月份全称：七月
英文星期的简称：Mon
本地星期的简称：星期一 
年的前两位数字（不足两位前面补0）：20 
年的后两位数字（不足两位前面补0）：20 
一年中的天数（即年的第几天）：209 
两位数字的月份（不足两位前面补0）：07 
两位数字的日（不足两位前面补0）：27 
月份的日（前面不补0）：27 
==========================================
2位数字24时制的小时（不足2位前面补0）:21 
2位数字12时制的小时（不足2位前面补0）:09 
2位数字24时制的小时（前面不补0）:21 
2位数字12时制的小时（前面不补0）:9 
2位数字的分钟（不足2位前面补0）:18 
2位数字的秒（不足2位前面补0）:21 
3位数字的毫秒（不足3位前面补0）:903 
9位数字的毫秒数（不足9位前面补0）:903000000 
小写字母的上午或下午标记(英)：pm
小写字母的上午或下午标记（中）：下午 
相对于GMT的RFC822时区的偏移量:+0800 
时区缩写字符串:CST 
1970-1-1 00:00:00 到现在所经过的秒数：1595855901 
1970-1-1 00:00:00 到现在所经过的毫秒数：1595855901903 
```

### 总结

&emsp;&emsp;今天的记录到这里就结束了，有需要的同学可以保存下来，需要的时候方便查找。有兴趣的同学可以自己下来依次尝试一下各个替换符带来的实现效果哦～
