# Double 转 BigDecimal 缺失精度



&emsp;&emsp;在我们日常的开发过程中难免会处理一些业务相关的数据，其中不乏对 带小数位 的数据进行相关计算。在对带小数位的数据进行相关计算的过程中可能会出现一些我们意想不到的结果，比如：<br/>

> 某一天接到任务说我们项目需要开发一个模块去联合多家购物平台进行返利推广赚取佣金给公司造血。小子二话不说就开始写起了代码，其中平台A对预估返利佣金计算方式是：++预估佣金(单位：元) = 佣金比率(百分比) * 商品折扣价(单位：元)++。其中平台A只返回 佣金比率 和 商品折扣价 ，佣金比率 和 商品折扣价 都是Java中的Double类型。系统设计时依据相关设计规范(规定金额类数据处理都采用BigDecimal进行计算)给预估佣金设计成了BigDecimal类型。当然，小子计算时也很自然的采用了一下方式：

```
BigDecimal commission = new BigDecimal(double0).divide(new BigDecimal("100")).multiply(new BigDecimal(double1));
```

> 其中 commission 指预估佣金(单位：元)，double0 指佣金比率(百分比)，double1 指商品折扣价(单位：元)。示例值：<br/>

```
double double0 = 35.43d; 
double double1 = 699.90d;
```

> 执行代码后得到的结果是：

```
commission = 247.974569999999989954915236012311716926227360101568099328804067909004515968263149261474609375
```

> 顿时，小子觉得这计算结果不太对啊，咋冒出来这么多小数位了，用计算器 验证了一把，计算器结果是：247.97457。怎么代码计算出来和实际不符呢？采用的计算类型也是按规则用的 BigDecimal 啊，咋就不对呢？小子百思不得其解，只好苦苦寻觅...

> 小子按照自己的步骤，一步一步探索着，直到写下了如下代码：

```
System.out.println(new BigDecimal(double0));
```

> 看到上面代码打印结果的一刹那差不多就找到缘由了，上面代码打印结果是：

```
35.42999999999999971578290569595992565155029296875
```

> 虽说小子找到了上面计算结果小数位异常的原因，但是新的问题又出现了，为啥好好的 35.43 怎么就变成 35.42999999999999971578290569595992565155029296875 了呢。

首先，咋们先来了解一下 BigDecimal的double参数构造，在官方JDK文档中对这个构造是这么描述的：

![BigDecimal](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/2020-11-16-27_1605539623697.png)

中文翻译版是这样的：

![BigDecimal](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/2020-11-16-27_1605539623697.png)

> 看到这里，出现上面问题的缘由基本上捋清楚了，小子感觉 BigDecimal的double 构造函数简直就是个坑啊！

根据官方文档说明，咋们是可以避免这一切的，比如咋们可以采用 BigDecimal的String构造函数。但是呢，我现在拿到的是 Double，我岂不是还得先把 Double 转 String 然后在运用 BigDecimal 的 String 构造函数进行计算，这未免太麻烦了吧～

#### 解决办法：

查看 BigDecimal相关的函数应该不难发现有这个一个函数存着，那就是：BigDecimal.valueOf(double);
其中官方文档说明是这样的：

![BigDecimal.valueOf](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/2020-11-16-37_1605539631017.png)

翻译成中文后是这样的：

![BigDecimal.valueOf](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/2020-11-16-25_1605539621230.png)

源码是这样的：

```
public static BigDecimal valueOf(double val) {
    return new BigDecimal(Double.toString(val));
}
```

首先，double会被自动转换成String类型，然后再运用 BigDecimal 的 String 构造方法进行处理，这样就不会造成小数位异常了。

> 小子机灵一试：

```
System.out.println(BigDecimal.valueOf(double0));
```

> 得到的结果为：

```
35.43
```

> 正好符合预期。小子顺势把上面最初的那段代码进行了修正：

```
BigDecimal commission = BigDecimal.valueOf(double0).divide(new BigDecimal("100")).multiply(BigDecimal.valueOf(double1));
```

> 运行计算后得到的结果为：

```
247.97457
```

> 这才符合正常情况嘛，至此，小子遇到的问题也圆满解决了～

#### 总结

&emsp;&emsp;平时开发过程中，对于金额的计算一定要使用BigDecimal类型进行处理，但是一定要注意小数精度问题，用法不对很容易出现上面类似的问题，进行构造BigDecimal数据时最好使用String类型，将Double类型转换成BigDecimal类型时一定要用 BigDecimal.valueof(double d) 函数进行处理，避免使用函数不当造成小数位精度丢失等不必要的麻烦。

&emsp;&emsp;平时遇到JDK相关问题时，请先看看相关的JDK-API文档，英语不好的同学可以先使用中文版，后面逐渐过渡到英文版使用吧。
