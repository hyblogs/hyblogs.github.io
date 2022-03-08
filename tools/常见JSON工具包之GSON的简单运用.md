&emsp;&emsp;想必大家在日常编码过程中肯定少不了使用一些JSON工具包的吧，比如 Gson、FastJson、JackSon、Json lib 等等，它们都属于 JSON 工具包。我个人比较喜欢 Gson，接下来咋们使用 Gson 进行一些常规的 Json 转换等简单运用吧。

> ⚠️注意：代码中使用了 Guava 工具包，在执行下述代码时请注意添加相关的 jar 包依赖，Guava 工具包与本次的重点 Gson 无关，仅用来快速创建 List 等场景。本次使用的 Gson 版本为 2.8.7 ，测试时请尽量保持一致。

## Gson

Gson 是一个 Java 库，是目前功能最全的 Json 解析神器。Gson 的应用主要为 toJson 与 fromJson 两个转换函数，无依赖，不需要例外额外的 jar，能够直接跑在 JDK 上。它可用于将 JSON 字符串转换为等效的 Java 对象。Gson 可以处理任意 Java 对象，包括您没有源代码的预先存在的对象。

Gson官方地址：https://github.com/google/gson

### Maven依赖引入方式

```xml
<dependency>
  <groupId>com.google.code.gson</groupId>
  <artifactId>gson</artifactId>
  <version>2.8.7</version>
</dependency>
```

### Gradle依赖引入方式

```xml
dependencies {
  implementation 'com.google.code.gson:gson:2.8.7'
}
```

### 下载 Jar 包手动导入的方式

Maven中央仓库 Gson 下载连接：[gson-2.8.7](https://repo1.maven.org/maven2/com/google/code/gson/gson/2.8.7/gson-2.8.7.jar)

本站附件 Gson 下载链接：[gson-2.8.7](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/gson-2.8.7_1626615724800.jar)

如果 Maven 中央仓库访问下载较慢则建议直接下载本站附件提供的 Gson 包。

## 常规使用-Code

```java
package com.gson;

import com.google.common.collect.Lists;
import com.google.gson.Gson;
import com.google.gson.GsonBuilder;

import java.io.Serializable;
import java.lang.reflect.Modifier;
import java.util.List;

/**
 * Gson测试类
 * @author dev.hy
 */
public class GsonClass {

    public static void main(String[] args) {
        GsonEntity entity4 = new GsonEntity("<!DOCTYPE html><html><head><meta charset=\"utf-8\"><title>hyblogs-Gson</title></head><body>    <h1>dev.hy</h1>    <p>huangyong。</p></body></html>", 4);
        GsonEntity entity3 = new GsonEntity("dev.hyblogs.com", 3, Lists.newArrayList(entity4));
        GsonEntity entity2 = new GsonEntity("dev.hyblogs", 2, Lists.newArrayList(entity3), "Public-2", "Hello!!!");
        GsonEntity entity1 = new GsonEntity("dev.hy", 1, Lists.newArrayList(entity2), "Public-1", "Fuck it!!");
        GsonEntity entity0 = new GsonEntity("dev", 0, Lists.newArrayList(entity1), "Public-0", "Oh!!");

        /* 将Java对象转换成Json字符串 */
        String jsonStr0 = new GsonBuilder().create().toJson(entity0);
        System.out.println("jsonStr0：" + jsonStr0);

        /* 将Java对象转换成Json字符串--保留值为空(null)的字段 */
        String jsonStr1 = new GsonBuilder().serializeNulls().create().toJson(entity0);
        System.out.println("jsonStr1：" + jsonStr1);

        /* 将Java对象转换成Json字符串--根据Modifier选择移除的访问修饰符(private) */
        String jsonStr2 = new GsonBuilder().excludeFieldsWithModifiers(Modifier.PRIVATE).create().toJson(entity0);
        System.out.println("jsonStr2：" + jsonStr2);

        /* 将Java对象转换成Json字符串--根据Modifier选择移除的访问修饰符(public) */
        String jsonStr3 = new GsonBuilder().excludeFieldsWithModifiers(Modifier.PUBLIC).create().toJson(entity0);
        System.out.println("jsonStr3：" + jsonStr3);

        /* 将Java对象转换成Json字符串--根据Modifier选择移除的访问修饰符(protected) */
        String jsonStr4 = new GsonBuilder().excludeFieldsWithModifiers(Modifier.PROTECTED).create().toJson(entity0);
        System.out.println("jsonStr4：" + jsonStr4);

        /* 将Java对象转换成Json字符串--根据Modifier选择移除的访问修饰符(static) */
        String jsonStr5 = new GsonBuilder().excludeFieldsWithModifiers(Modifier.STATIC).create().toJson(entity0);
        System.out.println("jsonStr5：" + jsonStr5);

        /* 将Java对象转换成Json字符串--禁用Html转义 */
        String jsonStr6 = new GsonBuilder().disableHtmlEscaping().create().toJson(entity0);
        System.out.println("jsonStr6：" + jsonStr6);

        /* 将Json字符串转换成Java对象(含List对象) */
        GsonEntity gsonEntity = new Gson().fromJson(jsonStr0, GsonEntity.class);
        System.out.println("gsonEntity：" + gsonEntity);
    }

    /**
     * Gson测试实体类
     */
    static class GsonEntity implements Serializable {
        private static final long serialVersionUID = 2279477648424876248L;

        /** Name */
        private String name;
        /** Code */
        private Integer code;
        /** Children */
        List<GsonEntity> children;
        /** Public Field */
        public String publicField;
        /** Static Field */
        static int staticField;
        /** Transient Field */
        transient String transientField;

        public GsonEntity() {
        }

        public GsonEntity(String name, Integer code) {
            this.name = name;
            this.code = code;
        }

        public GsonEntity(String name, Integer code, List<GsonEntity> children) {
            this.name = name;
            this.code = code;
            this.children = children;
        }

        public GsonEntity(String name, Integer code, List<GsonEntity> children, String publicField, String transientField) {
            this.name = name;
            this.code = code;
            this.children = children;
            this.publicField = publicField;
            this.transientField = transientField;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public Integer getCode() {
            return code;
        }

        public void setCode(Integer code) {
            this.code = code;
        }

        public List<GsonEntity> getChildren() {
            return children;
        }

        public void setChildren(List<GsonEntity> children) {
            this.children = children;
        }

        public String getPublicField() {
            return publicField;
        }

        public void setPublicField(String publicField) {
            this.publicField = publicField;
        }

        public static int getStaticField() {
            return staticField;
        }

        public static void setStaticField(int staticField) {
            GsonEntity.staticField = staticField;
        }

        public String getTransientField() {
            return transientField;
        }

        public void setTransientField(String transientField) {
            this.transientField = transientField;
        }

        @Override
        public String toString() {
            return "GsonEntity{" +
                    "name='" + name + '\'' +
                    ", code=" + code +
                    ", children=" + children +
                    ", publicField='" + publicField + '\'' +
                    ", transientField='" + transientField + '\'' +
                    '}';
        }
    }
}
```

### 打印结果如下

```java
jsonStr0：{"name":"dev","code":0,"children":[{"name":"dev.hy","code":1,"children":[{"name":"dev.hyblogs","code":2,"children":[{"name":"dev.hyblogs.com","code":3,"children":[{"name":"\u003c!DOCTYPE html\u003e\u003chtml\u003e\u003chead\u003e\u003cmeta charset\u003d\"utf-8\"\u003e\u003ctitle\u003ehyblogs-Gson\u003c/title\u003e\u003c/head\u003e\u003cbody\u003e    \u003ch1\u003edev.hy\u003c/h1\u003e    \u003cp\u003ehuangyong。\u003c/p\u003e\u003c/body\u003e\u003c/html\u003e","code":4}]}],"publicField":"Public-2"}],"publicField":"Public-1"}],"publicField":"Public-0"}
jsonStr1：{"name":"dev","code":0,"children":[{"name":"dev.hy","code":1,"children":[{"name":"dev.hyblogs","code":2,"children":[{"name":"dev.hyblogs.com","code":3,"children":[{"name":"\u003c!DOCTYPE html\u003e\u003chtml\u003e\u003chead\u003e\u003cmeta charset\u003d\"utf-8\"\u003e\u003ctitle\u003ehyblogs-Gson\u003c/title\u003e\u003c/head\u003e\u003cbody\u003e    \u003ch1\u003edev.hy\u003c/h1\u003e    \u003cp\u003ehuangyong。\u003c/p\u003e\u003c/body\u003e\u003c/html\u003e","code":4,"children":null,"publicField":null}],"publicField":null}],"publicField":"Public-2"}],"publicField":"Public-1"}],"publicField":"Public-0"}
jsonStr2：{"children":[{"children":[{"children":[{"children":[{"staticField":0}],"staticField":0}],"publicField":"Public-2","staticField":0,"transientField":"Hello!!!"}],"publicField":"Public-1","staticField":0,"transientField":"Fuck it!!"}],"publicField":"Public-0","staticField":0,"transientField":"Oh!!"}
jsonStr3：{"serialVersionUID":2279477648424876248,"name":"dev","code":0,"children":[{"serialVersionUID":2279477648424876248,"name":"dev.hy","code":1,"children":[{"serialVersionUID":2279477648424876248,"name":"dev.hyblogs","code":2,"children":[{"serialVersionUID":2279477648424876248,"name":"dev.hyblogs.com","code":3,"children":[{"serialVersionUID":2279477648424876248,"name":"\u003c!DOCTYPE html\u003e\u003chtml\u003e\u003chead\u003e\u003cmeta charset\u003d\"utf-8\"\u003e\u003ctitle\u003ehyblogs-Gson\u003c/title\u003e\u003c/head\u003e\u003cbody\u003e    \u003ch1\u003edev.hy\u003c/h1\u003e    \u003cp\u003ehuangyong。\u003c/p\u003e\u003c/body\u003e\u003c/html\u003e","code":4,"staticField":0}],"staticField":0}],"staticField":0,"transientField":"Hello!!!"}],"staticField":0,"transientField":"Fuck it!!"}],"staticField":0,"transientField":"Oh!!"}
jsonStr4：{"serialVersionUID":2279477648424876248,"name":"dev","code":0,"children":[{"serialVersionUID":2279477648424876248,"name":"dev.hy","code":1,"children":[{"serialVersionUID":2279477648424876248,"name":"dev.hyblogs","code":2,"children":[{"serialVersionUID":2279477648424876248,"name":"dev.hyblogs.com","code":3,"children":[{"serialVersionUID":2279477648424876248,"name":"\u003c!DOCTYPE html\u003e\u003chtml\u003e\u003chead\u003e\u003cmeta charset\u003d\"utf-8\"\u003e\u003ctitle\u003ehyblogs-Gson\u003c/title\u003e\u003c/head\u003e\u003cbody\u003e    \u003ch1\u003edev.hy\u003c/h1\u003e    \u003cp\u003ehuangyong。\u003c/p\u003e\u003c/body\u003e\u003c/html\u003e","code":4,"staticField":0}],"staticField":0}],"publicField":"Public-2","staticField":0,"transientField":"Hello!!!"}],"publicField":"Public-1","staticField":0,"transientField":"Fuck it!!"}],"publicField":"Public-0","staticField":0,"transientField":"Oh!!"}
jsonStr5：{"name":"dev","code":0,"children":[{"name":"dev.hy","code":1,"children":[{"name":"dev.hyblogs","code":2,"children":[{"name":"dev.hyblogs.com","code":3,"children":[{"name":"\u003c!DOCTYPE html\u003e\u003chtml\u003e\u003chead\u003e\u003cmeta charset\u003d\"utf-8\"\u003e\u003ctitle\u003ehyblogs-Gson\u003c/title\u003e\u003c/head\u003e\u003cbody\u003e    \u003ch1\u003edev.hy\u003c/h1\u003e    \u003cp\u003ehuangyong。\u003c/p\u003e\u003c/body\u003e\u003c/html\u003e","code":4}]}],"publicField":"Public-2","transientField":"Hello!!!"}],"publicField":"Public-1","transientField":"Fuck it!!"}],"publicField":"Public-0","transientField":"Oh!!"}
jsonStr6：{"name":"dev","code":0,"children":[{"name":"dev.hy","code":1,"children":[{"name":"dev.hyblogs","code":2,"children":[{"name":"dev.hyblogs.com","code":3,"children":[{"name":"<!DOCTYPE html><html><head><meta charset=\"utf-8\"><title>hyblogs-Gson</title></head><body>    <h1>dev.hy</h1>    <p>huangyong。</p></body></html>","code":4}]}],"publicField":"Public-2"}],"publicField":"Public-1"}],"publicField":"Public-0"}
gsonEntity：GsonEntity{name='dev', code=0, children=[GsonEntity{name='dev.hy', code=1, children=[GsonEntity{name='dev.hyblogs', code=2, children=[GsonEntity{name='dev.hyblogs.com', code=3, children=[GsonEntity{name='<!DOCTYPE html><html><head><meta charset="utf-8"><title>hyblogs-Gson</title></head><body>    <h1>dev.hy</h1>    <p>huangyong。</p></body></html>', code=4, children=null, publicField='null', transientField='null'}], publicField='null', transientField='null'}], publicField='Public-2', transientField='null'}], publicField='Public-1', transientField='null'}], publicField='Public-0', transientField='null'}
```

## 过滤字段

通常我们在打包或解析 Json 数据的时候，往往有些字段我们不需要，所以会想到把它过滤掉，那如何使用 Gson 工具包对 Json 操作中的部分字段进行过滤呢，这里我总结了以下几种常用的方法：

### 使用 @Expose 注解

在要过滤部分属性的对象中把需要保留的字段加上 @Expose 注解，这样其他没有添加此注解的字段通通都会被过滤掉了。
<font style="color:red">
⚠️注意：要实现这这一功能有一个很关键的点，那就是在实例化 Gson 的时候不能简单的使用 `new Gson()` 这种方式了，而是需要用到 `new GsonBuilder().excludeFieldsWithoutExposeAnnotation().create()` 来创建 Gson 对象才行。
</font>

#### 示例如下

> 在下述代码的 children 字段上添加 @Expose 注解：

```java
/**
 * Gson测试实体类
 */
static class GsonEntity implements Serializable {
    private static final long serialVersionUID = 2279477648424876248L;

    /** Name */
    private String name;
    /** Code */
    private Integer code;
    /** Children */
    @Expose
    List<GsonEntity> children;

    public GsonEntity() {
    }

    public GsonEntity(String name, Integer code) {
        this.name = name;
        this.code = code;
    }

    public GsonEntity(String name, Integer code, List<GsonEntity> children) {
        this.name = name;
        this.code = code;
        this.children = children;
    }

    /* 此处忽略掉 get/set 以及 toString 方法（内容同上面的完整对象一致） */
}
```

> 测试时使用 `new GsonBuilder().excludeFieldsWithoutExposeAnnotation().create()` 创建 Gson 对象：

```java
public static void main(String[] args) {
    GsonEntity entity4 = new GsonEntity("<!DOCTYPE html><html><head><meta charset=\"utf-8\"><title>hyblogs-Gson</title></head><body>    <h1>dev.hy</h1>    <p>huangyong。</p></body></html>", 4);
    GsonEntity entity3 = new GsonEntity("dev.hyblogs.com", 3, Lists.newArrayList(entity4));
    GsonEntity entity2 = new GsonEntity("dev.hyblogs", 2, Lists.newArrayList(entity3));
    GsonEntity entity1 = new GsonEntity("dev.hy", 1, Lists.newArrayList(entity2));
    GsonEntity entity0 = new GsonEntity("dev", 0, Lists.newArrayList(entity1));

    /* 将Java对象转换成Json对象-排除掉未使用 @Expose 注解的属性 */
    String jsonStr7 = new GsonBuilder().excludeFieldsWithoutExposeAnnotation().create().toJson(entity0);
    System.out.println("jsonStr7：" + jsonStr7);
}
```

#### 运行结果

```java
jsonStr7：{"children":[{"children":[{"children":[{"children":[{}]}]}]}]}
```

> 根据结果可以看到，由于只有 children 属性加上了 @Expose，所以其余的属性通过 Json 序列化之后便没有了。

#### 序列化或反序列化时才生效

使用 @Expose 注解时，可以选择在序列化或者反序列化的时候单独使用，因为 @Expose 可以传入参数 serialize/deserialize ，如下：

```java
public static void main(String[] args) {
    GsonEntity entity4 = new GsonEntity("<!DOCTYPE html><html><head><meta charset=\"utf-8\"><title>hyblogs-Gson</title></head><body>    <h1>dev.hy</h1>    <p>huangyong。</p></body></html>", 4);
    GsonEntity entity3 = new GsonEntity("dev.hyblogs.com", 3, Lists.newArrayList(entity4));
    GsonEntity entity2 = new GsonEntity("dev.hyblogs", 2, Lists.newArrayList(entity3));
    GsonEntity entity1 = new GsonEntity("dev.hy", 1, Lists.newArrayList(entity2));
    GsonEntity entity0 = new GsonEntity("dev", 0, Lists.newArrayList(entity1));

    /* 获取Gson对象-排除掉没有添加Expose注解的属性 */
    Gson gson = new GsonBuilder().excludeFieldsWithoutExposeAnnotation().create();

    /* 将Java对象转换成Json对象 */
    String jsonStr9 = gson.toJson(entity0);
    System.out.println("jsonStr9：" + jsonStr9);

    /* 将Json对象反序列化为Java对象 */
    GsonEntity gsonEntity = gson.fromJson(jsonStr9, GsonEntity.class);
    System.out.println(gsonEntity);
}

/**
 * Gson测试实体类
 */
static class GsonEntity implements Serializable {
    private static final long serialVersionUID = 2279477648424876248L;

    /** Name-未添加Expose注解，序列化及反序列化均被移除 */
    private String name;
    /** Code-添加了Expose注解，且序列化选择了true则表示允许序列化（默认反序列化为true，所以不影响反序列化） */
    @Expose(serialize = true)
    private Integer code;
    /** Children-添加Expose注解，反序列化设为false，对应的序列化默认为true，所以反序列化后Java对象此字段为null */
    @Expose(deserialize = false)
    List<GsonEntity> children;

    public GsonEntity() {
    }

    public GsonEntity(String name, Integer code) {
        this.name = name;
        this.code = code;
    }

    public GsonEntity(String name, Integer code, List<GsonEntity> children) {
        this.name = name;
        this.code = code;
        this.children = children;
    }

    /* 此处忽略掉 get/set 以及 toString 方法（内容同上面的完整对象一致） */
}
```

#### 执行结果

```java
jsonStr9：{"code":0,"children":[{"code":1,"children":[{"code":2,"children":[{"code":3,"children":[{"code":4}]}]}]}]}
GsonEntity{name='null', code=0, children=null}
```

所以，使用 @Expose 注解的时候，可以自由控制 serialize/deserialize 对应的值，默认均为 true，当设置为 false 时则对应的序列化或反序列化会忽略掉对应的属性哦！

### 构造自定义过滤属性的 Gson

除了上面描述的使用添加注解 @Expose 的方式保留需要序列化的属性达到过滤不需要的属性的方式以外，我们还可以自己设置排除策略对属性以及对象进行排除。

#### 示例如下

```java
/* 将Java对象转换成Json对象--自定义要过滤的属性或Class */
String jsonStr8 = new GsonBuilder().setExclusionStrategies(new ExclusionStrategy() {
    /** 设置需要过滤掉(或跳过)的属性(可根据Name,Annotations,Modifier,DeclaredClass,DeclaredType等进行筛选) */
    public boolean shouldSkipField(FieldAttributes fieldAttributes) {
        return fieldAttributes.getName().contains("name") | fieldAttributes.getName().contains("code1");
    }
    /** 设置需要过滤掉(或跳过)的Class(可根据Name,Annotations等进行筛选) */
    public boolean shouldSkipClass(Class<?> aClass) {
        return aClass.getName().contains("child2");
    }
}).create().toJson(entity0);
System.out.println("jsonStr8：" + jsonStr8);
```

#### 运行结果

> 由上面的代码可以看出，我过滤掉了属性名称含有 “name” 或 “code1” 的属性，以及 ClassName 含有 “child2” 的Class，结果如下：

```java
jsonStr8：{"code":0,"children":[{"code":1,"children":[{"code":2,"children":[{"code":3,"children":[{"code":4}]}]}]}]}
```

### 通过Modifier(修饰符)批量过滤

我们除了使用上面所示的两种方法过滤掉我们不需要的属性以外，还可以通过指定声明的修饰符来过滤掉我们不需要的属性，比如这里过滤掉声明为 private 的变量。

#### 示例代码

```java
public static void main(String[] args) {
    GsonEntity entity4 = new GsonEntity("<!DOCTYPE html><html><head><meta charset=\"utf-8\"><title>hyblogs-Gson</title></head><body>    <h1>dev.hy</h1>    <p>huangyong。</p></body></html>", 4);
    GsonEntity entity3 = new GsonEntity("dev.hyblogs.com", 3, Lists.newArrayList(entity4));
    GsonEntity entity2 = new GsonEntity("dev.hyblogs", 2, Lists.newArrayList(entity3));
    GsonEntity entity1 = new GsonEntity("dev.hy", 1, Lists.newArrayList(entity2));
    GsonEntity entity0 = new GsonEntity("dev", 0, Lists.newArrayList(entity1));

    /* 将Java对象转换成Json对象--根据访问修饰符批量过滤 */
    String jsonStr9 = new GsonBuilder().excludeFieldsWithModifiers(Modifier.PRIVATE).create().toJson(entity0);
    System.out.println("jsonStr9：" + jsonStr9);
}

/**
 * Gson测试实体类
 */
static class GsonEntity implements Serializable {
    private static final long serialVersionUID = 2279477648424876248L;

    /** Name */
    private String name;
    /** Code */
    private Integer code;
    /** Children */
    List<GsonEntity> children;

    public GsonEntity() {
    }

    public GsonEntity(String name, Integer code) {
        this.name = name;
        this.code = code;
    }

    public GsonEntity(String name, Integer code, List<GsonEntity> children) {
        this.name = name;
        this.code = code;
        this.children = children;
    }

    /* 此处忽略掉 get/set 以及 toString 方法（内容同上面的完整对象一致） */
}
```

除了上面示例使用的修饰符以外，还可以使用：PUBLIC、PROTECTED、STATIC、FINAL、SYNCHRONIZED、VOLATILE、TRANSIENT、NATIVE、INTERFACE、ABSTRACT、STRICT 等等一些修饰符，大家可以自己去尝试一下哦。

#### 运行结果

```java
jsonStr9：{"children":[{"children":[{"children":[{"children":[{}]}]}]}]}
```

以上就是在使用 Gson 工具包时如何过滤掉我们不需要的属性啦！

## 未完待续...
