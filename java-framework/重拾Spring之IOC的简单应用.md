# 重拾 Spring 之 IOC 的简单应用

# Spring-IOC 核心概念

&emsp;&emsp;IOC——Spring 通过一种称作控制反转（IoC）的技术促进了低耦合。当应用了 IoC，一个对象依赖的其它对象会通过被动的方式传递进来，而不是这个对象自己创建或者查找依赖对象。你可以认为 IoC 与 JNDI 相反——不是对象从容器中查找依赖，而是容器在对象初始化时不等对象请求就主动将依赖传递给它。它的底层设计模式采用了**工厂模式**，所有的 Bean 都需要注册到 Bean 工厂中，将其初始化和生命周期的监控交由工厂实现管理。程序员只需要按照规定的格式进行 Bean 开发，然后利用 XML 文件进行 bean 的定义和参数配置，其他的动态生成和监控就不需要调用者完成，而是统一交给了平台进行管理。

&emsp;&emsp;这样做的好处是：第一、资源的集中管理，实现资源的可配置和易管理；第二、降低了使用资源双方的依赖程度，也就是我们说的耦合度。

> 声明：本次测试使用的 Spring 版本为 5.2.6.RELEASE，JDK 版本为 1.8.0_241

# Spring-IOC 的应用

&emsp;&emsp;为了能更好的通过代码进行测试验证，需要大家搭建一个简单版的 Spring 项目，我这里采用的 Maven 进行项目构建管理：

![IDEA-Spring.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1596005289128.png)

## 定义 Bean 信息

创建一个通用的 Bean ，在后面会经常使用到它，里面的内容可以非常简洁，如下：

```java
/**
 * @author: hy
 * @date: 2020/7/29 2:08 下午
 * @Description: 测试用 Bean
 * @version: v_1.0.0
 */
public class HyBlogs {

    public HyBlogs() {
        System.out.println("This is HY-Blogs!!!");
    }
}
```

### 采用基于 XML 的方式定义 Bean 信息

在 Spring.xml 文件中定义一个 Bean，指向刚才我们创建的那个通用的 Bean：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="hyBlogs" class="com.ioc.bean.HyBlogs"/>
</beans>
```

编写一个测试类通过 XML 去获取我们定义的 Bean 信息：

```java
package com.ioc.bean;

import org.junit.Before;
import org.junit.Test;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * @author: hy
 * @date: 2020/7/29 2:13 下午
 * @Description:
 * @version: v_1.0.0
 */
public class XmlTest {
    private ClassPathXmlApplicationContext context;

    @Before
    public void before() {
        context = new ClassPathXmlApplicationContext("spring.xml");
    }

    @Test
    public void getMyBean() {
        HyBlogs hyBlogs1 = context.getBean(HyBlogs.class);
        HyBlogs hyBlogs2 = (HyBlogs) context.getBean("hyBlogs");
        HyBlogs hyBlogs3 = context.getBean("hyBlogs", HyBlogs.class);
        System.out.printf("HY-Blogs1 = %s %nHY-Blogs2 = %s %nHY-Blogs3 = %s %n", hyBlogs1, hyBlogs2, hyBlogs3);
    }
}
```

**执行结果**：

```java
This is HY-Blogs!!!
HY-Blogs1 = com.ioc.bean.HyBlogs@1500955a 
HY-Blogs2 = com.ioc.bean.HyBlogs@1500955a 
HY-Blogs3 = com.ioc.bean.HyBlogs@1500955a 
```

**结果分析**：

首先读取资源配置文件 spring.xml，然后解析成 BeanDefinition，最后利用反射进行相应的实例化操作。

### 基于读取配置类的形式定义Bean信息

新建一个配置类定义相应的 Bean 信息：

```java
package com.ioc.conf;

import com.ioc.bean.HyBlogs;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author: hy
 * @date: 2020/7/29 3:18 下午
 * @Description:
 * @version: v_1.0.0
 */
@Configuration
public class IocConfig {

    @Bean
    public HyBlogs getHyBlogs() {
        return new HyBlogs();
    }
}
```

新建一个测试类进行测试：

```java
package com.ioc.bean;

import com.ioc.conf.IocConfig;
import org.junit.Before;
import org.junit.Test;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * @author: hy
 * @date: 2020/7/29 3:24 下午
 * @Description:
 * @version: v_1.0.0
 */
public class IocConfigTest {
    private AnnotationConfigApplicationContext context;

    @Before
    public void before() {
        context = new AnnotationConfigApplicationContext(IocConfig.class);
    }

    @Test
    public void getMyBean() {
        HyBlogs hyBlogs1 = context.getBean(HyBlogs.class);
        HyBlogs hyBlogs2 = context.getBean("hyBlogs", HyBlogs.class);
        HyBlogs hyBlogs3 = (HyBlogs) context.getBean("hyBlogs");

        System.out.printf("HY-Blogs1 = %s %nHY-Blogs2 = %s %nHY-Blogs3 = %s %n", hyBlogs1, hyBlogs2, hyBlogs3);
    }
}
```

**执行结果**：

```java
This is HY-Blogs!!!
HY-Blogs1 = com.ioc.bean.HyBlogs@33afa13b 
HY-Blogs2 = com.ioc.bean.HyBlogs@33afa13b 
HY-Blogs3 = com.ioc.bean.HyBlogs@33afa13b 
```

## Spring-IOC 常用注解

在前面的测试用例中，我们使用了 @Configuration、@Bean 等 Spring 中 的注解。

- **@Configuration** 相当于 xml 配置文件中的 <beans/> 节点。
- **@Bean** 则相当于 xml 配置文件的 <bean/> 节点

### 配置Bean的作用域

- 在不指定 @Scope 的情况下，所有的 bean 都是单实例的 bean ，这一点在上面的测试用例中可以看出，重复获取多个 Bean ，但是它们都指向的同一个 地址，而且是 **饿汉式** 加载(容器启动实例就创建好了)；
- 当指定 @Scope 为 prototype 时就表示为多实例的，而且还是 **懒汉式** 加载(即当 IOC 容器启动的时候，并不会创建对象，而是在每次使用的时候才会去创建对应的对象 Bean)；
- 注意：当指定多实例的时候是无法解决循环依赖的。

### 修改测试用例，测试 Bean 的作用域

修改上面的测试用例 xml 配置文件，增加 scope = prototype 的配置信息，如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="hyBlogs" class="com.ioc.bean.HyBlogs" scope="prototype"/>
</beans>
```

运行测试代码：

```java
package com.ioc.bean;

import org.junit.Before;
import org.junit.Test;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * @author: hy
 * @date: 2020/7/29 2:13 下午
 * @Description: 使用 XML 配置文件的方式创建 Bean
 * @version: v_1.0.0
 */
public class XmlTest {
    private ClassPathXmlApplicationContext context;

    @Before
    public void before() {
        context = new ClassPathXmlApplicationContext("spring.xml");
    }

    @Test
    public void getMyBean() {
        HyBlogs hyBlogs1 = context.getBean(HyBlogs.class);
        HyBlogs hyBlogs2 = (HyBlogs) context.getBean("hyBlogs");
        HyBlogs hyBlogs3 = context.getBean("hyBlogs", HyBlogs.class);
        System.out.printf("HY-Blogs1 = %s %nHY-Blogs2 = %s %nHY-Blogs3 = %s %n", hyBlogs1, hyBlogs2, hyBlogs3);
    }
}
```

**执行结果**：

```java
This is HY-Blogs!!!
This is HY-Blogs!!!
This is HY-Blogs!!!
HY-Blogs1 = com.ioc.bean.HyBlogs@5bb21b69 
HY-Blogs2 = com.ioc.bean.HyBlogs@6b9651f3 
HY-Blogs3 = com.ioc.bean.HyBlogs@38bc8ab5 
```

**结果分析**：

根据指向结果可以看出，我们通过代码获取了 3 次 Bean ，实际上同一个 Bean 也被创建了 3 个实例，对应的地址也是不一样的。

还可以修改上面通过注解方式的示例代码，修改后如下：

```java
package com.ioc.conf;

import com.ioc.bean.HyBlogs;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Scope;

/**
 * @author: hy
 * @date: 2020/7/29 3:18 下午
 * @Description:
 * @version: v_1.0.0
 */
@Configuration
public class IocConfig {

    @Bean
    @Scope(value = "prototype")
    public HyBlogs hyBlogs() {
        return new HyBlogs();
    }
}
```

主要就是新增了 @Scope(value = "prototype") 注解。

运行测试代码：

```java
package com.ioc.bean;

import com.ioc.conf.IocConfig;
import org.junit.Before;
import org.junit.Test;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * @author: hy
 * @date: 2020/7/29 3:24 下午
 * @Description:
 * @version: v_1.0.0
 */
public class IocConfigTest {
    private AnnotationConfigApplicationContext context;

    @Before
    public void before() {
        context = new AnnotationConfigApplicationContext(IocConfig.class);
    }

    @Test
    public void getMyBean() {
        HyBlogs hyBlogs1 = context.getBean(HyBlogs.class);
        HyBlogs hyBlogs2 = context.getBean("hyBlogs", HyBlogs.class);
        HyBlogs hyBlogs3 = (HyBlogs) context.getBean("hyBlogs");

        System.out.printf("HY-Blogs1 = %s %nHY-Blogs2 = %s %nHY-Blogs3 = %s %n", hyBlogs1, hyBlogs2, hyBlogs3);

        //判定新创建的几个 Bean 对象是否相等
        //返回 true 则表示是单实例只创建了一个对象，多次调用返回的地址相同
        //返回 false 则表示是多实例，每次调用都会创建一个新的对象。
        System.out.println(hyBlogs1 == hyBlogs2);
        System.out.println(hyBlogs2 == hyBlogs3);
        System.out.println(hyBlogs1 == hyBlogs3);
    }
}
```

**执行结果**：

```java
This is HY-Blogs!!!
This is HY-Blogs!!!
This is HY-Blogs!!!
HY-Blogs1 = com.ioc.bean.HyBlogs@6dbb137d 
HY-Blogs2 = com.ioc.bean.HyBlogs@3c9d0b9d 
HY-Blogs3 = com.ioc.bean.HyBlogs@43301423 
false
false
false
```

**结果分析**：

上面我们指定了 @Scope 作用域为 prototype 所以多次获取 Bean 会创建多个 Bean，不再是单实例了，从打印出来的地址也可以看出不是同一个对象了。
@Scope 注解指定的作用域取值有：singleton —— 单实例的(默认)，prototype —— 多实例的，request —— 同一次请求，session —— 同一个会话级别。

### Bean 的懒加载 @Lazy

前面我们说到了 **单实例** 是 **饿汉式** 的，**多实例** 则是 **懒汉式** 的，其实可以设置 **单实例** 采用 **懒汉式** 进行加载。

Bean 的懒加载 @Lazy(主要针对 **单实例** 的 bean 在容器启动的时候，不创建对象，而在第一次使用的时候才会创建该对象，<fonr style="color:red">**多实例 bean 没有懒加载这一说法**</font>)。

XML 配置文件修改方式如下：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="hyBlogs" class="com.ioc.bean.HyBlogs" lazy-init="true"/>
</beans>
```

xml 配置文件方式需要给对应需要懒加载的 bean 添加上 lazy-init="true" 的属性信息。

注解修改方式如下：

```java
package com.ioc.conf;

import com.ioc.bean.HyBlogs;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Lazy;

/**
 * @author: hy
 * @date: 2020/7/29 3:18 下午
 * @Description:
 * @version: v_1.0.0
 */
@Configuration
public class IocConfig {

    @Bean
    @Lazy
    public HyBlogs hyBlogs() {
        return new HyBlogs();
    }
}
```

由于 @Lazy 注解默认的 value 值就是 true ，所以不需要显示指定其 value = true .

### 包扫描 @CompentScan

很多时候我们需要将大量的对象交给 Spring 进行管理，比如主要处理业务逻辑的 Service ，或者负责提供接口的 Controller 都会统一交给 Spring 进行管理。这时，我们就需要使用到 Spring 中的包扫描了。

#### 1、通过 XML 配置文件配置包扫描

经常使用 xml 配置文件的同学应该对这个不陌生，在 xml 配置文件中如下配置一番变可以起到包扫描的效果了，如下示例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.ioc">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
</beans>
```

采用如上的扫描配置后，我们就可以将刚才在 xml 配置文件中单独配置的 bean 定义给移除了，因为扫描对应的包下面的对象已经交给了 Spring 进行管理，不需要再单独配置它了。因此执行上面 xml 对应的测试用例照样运行正常。

#### 2、通过注解的方式配置包扫描

在配置类上写 @CompentScan 注解来进行包扫描。

- 常规用法：在 basePackages 包下面具有 @Controller @Service @Repository @Component 注解的组件都会被加载到 Spring 容器中。
  
  ```java
  package com.ioc.conf;
  ```

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

/**

* @author: hy
* @date: 2020/7/29 3:18 下午
* @Description: Spring IOC 测试配置
* @version: v_1.0.0
  */
  @Configuration
  @ComponentScan(basePackages = "com.ioc")
  public class IocConfig {

}

```
上面的配置增加了 @ComponentScan(basePackages = "com.ioc") 注解配置。这时候，还需要在我们的通用 Bean 上面配置 @Component 注解，不然扫描不到这个 Bean 。如下：
```java
package com.ioc.bean;

import org.springframework.stereotype.Component;

/**
 * @author: hy
 * @date: 2020/7/29 2:08 下午
 * @Description: 测试用 Bean
 * @version: v_1.0.0
 */
@Component
public class HyBlogs {

    public HyBlogs() {
        System.out.println("This is HY-Blogs!!!");
    }
}
```

- 排除用法：excludeFilters(排除 @Controller 注解和 HyBlogs)
  
  ```java
  @Configuration
  @ComponentScan(basePackages = {"com.ioc"},excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION,value = {Controller.class}),
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE,value = {HyBlogs.class})
  })
  public class IocConfig {
  ```

}

```
如上述配置后，由于配置了排除 excludeFilters 将 HyBlogs.class 排除在外了。所以，此时在获取 HyBlogs 实例的时候就会出错了，找不到这个 Bean 了。

- 包含用法：includeFilters
<font style="color:red;">**注意：若使用包含，需要把useDefaultFilters 属性设置为 false（true 表示扫描全部的）**</font>。
```java
@Configuration
@ComponentScan(basePackages = {"com.ioc"}, includeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION,value = {Controller.class, Service.class})
}, useDefaultFilters = false)
public class IocConfig {

}
```

如上配置代码后，如果我们此时想要获取上面定义的通用实例对象，必须将上面定义的 @Component 注解替换成 @Service ，因为我们只采用了包含扫描 @Service 注解和 @Controller 注解。

如下所示，修改 @Component 为 @Service ：

```java
@Service
public class HyBlogs {

    public HyBlogs() {
        System.out.println("This is HY-Blogs!!!");
    }
}
```

- 自定义 Filter

自定义一个 HyBlogsFilter 实现 TypeFilter ，代码如下：

```java
package com.ioc.filter;

import org.springframework.core.type.ClassMetadata;
import org.springframework.core.type.classreading.MetadataReader;
import org.springframework.core.type.classreading.MetadataReaderFactory;
import org.springframework.core.type.filter.TypeFilter;

import java.io.IOException;

/**
 * @author: hy
 * @date: 2020/7/29 11:07 下午
 * @Description: 自定义过滤器
 * @version: v_1.0.0
 */
public class HyBlogsFilter implements TypeFilter {

    private static final String FILTER_CLASS_NAME = "Hy";

    @Override
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        if (classMetadata.getClassName().contains(FILTER_CLASS_NAME)) {
            return true;
        }
        return false;
    }
}
```

对应的配置注解需要修改成自定义的过滤器，如下配置示例：

```java
@Configuration
@ComponentScan(basePackages = {"com.ioc"}, includeFilters = {
        @ComponentScan.Filter(type = FilterType.CUSTOM,value = HyBlogsFilter.class)
}, useDefaultFilters = false)
public class IocConfig {

}
```

此时，同样需要将 useDefaultFilters 属性值设置为 false ，避免全部扫描。如果将 includeFilters 换成 excludeFilters 则需要按需求进行修改过滤器，否则不会将我们通用的 Bean 交给 Spring 容器进行管理。

### 条件注解 @Conditional

只有当满足指定条件后才进行对应的 Bean 的创建及其示例化操作。在 SpringBoot 中有很多这种注解，比如：@ConditionalOnBean、@ConditionalOnClass、@ConditionalOnJava 等等很多。

接下来再创建一个对象 MyBlogs ，等到 Spring 容器中有 HyBlogs 这个 Bean 的时候才实例化 MyBlogs 这个 Bean .

```java
package com.ioc.bean;

/**
 * @author: hy
 * @date: 2020/7/30 2:41 下午
 * @Description:
 * @version: v_1.0.0
 */
public class MyBlogs {

    public MyBlogs() {
        System.out.println("This is MyBlogs!");
    }
}
```

新建一个 HyBlogsCondition 实现 Condition 接口：

```java
package com.ioc.condition;

import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.type.AnnotatedTypeMetadata;

/**
 * @author: hy
 * @date: 2020/7/30 2:44 下午
 * @Description: 条件判定
 * @version: v_1.0.0
 */
public class HyBlogsCondition implements Condition {

    private static final String BEAN_NAME_HYBLOGS = "hyBlogs";

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        //判断容器中是否有 HyBlogs 对应的 Bean 实例
        if (context.getBeanFactory().containsBean(BEAN_NAME_HYBLOGS)) {
            return true;
        }
        return false;
    }
}
```

在配置类中添加条件配置项，即“只有当容器中有 HyBlogs 的时候才实例化MyBlogs”，如下：

```java
package com.ioc.conf;

import com.ioc.bean.HyBlogs;
import com.ioc.bean.MyBlogs;
import com.ioc.condition.HyBlogsCondition;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Conditional;
import org.springframework.context.annotation.Configuration;

/**
 * @author: hy
 * @date: 2020/7/29 3:18 下午
 * @Description: Spring IOC 测试配置
 * @version: v_1.0.0
 */
@Configuration
public class IocConfig {

    @Bean
    public HyBlogs hyBlogs() {
        return new HyBlogs();
    }

    @Bean
    @Conditional(HyBlogsCondition.class)
    public MyBlogs myBlogs() {
        return new MyBlogs();
    }
}
```

执行之前的测试用例 IocConfigTest ：

```java
public class IocConfigTest {
    private AnnotationConfigApplicationContext context;

    @Before
    public void before() {
        context = new AnnotationConfigApplicationContext(IocConfig.class);
    }

    @Test
    public void getMyBean() {
        HyBlogs hyBlogs1 = context.getBean(HyBlogs.class);
        HyBlogs hyBlogs2 = context.getBean("hyBlogs", HyBlogs.class);
        HyBlogs hyBlogs3 = (HyBlogs) context.getBean("hyBlogs");

        System.out.printf("HY-Blogs1 = %s %nHY-Blogs2 = %s %nHY-Blogs3 = %s %n", hyBlogs1, hyBlogs2, hyBlogs3);
    }
}
```

**执行结果**：

```java
This is HY-Blogs!!!
This is MyBlogs!
HY-Blogs1 = com.ioc.bean.HyBlogs@5cee5251 
HY-Blogs2 = com.ioc.bean.HyBlogs@5cee5251 
HY-Blogs3 = com.ioc.bean.HyBlogs@5cee5251 
```

**结果分析**：

根据执行结果可以看到，先实例化来 HyBlogs 这个 Bean ，然后再实例化了 MyBlogs 这个 Bean .

### 向 IOC 容器中添加组件

下面将展示几种向 IOC 容器中添加组件的方式，也是 Spring 中创建 Bean  的几种方式。

**1、 通过使用 @ComponentScan 包扫描注解 + @Controller、@Service、@Repository、@Component 注解，针对我们自己写的组件可以通过该方式来加载到容器中；**

&emsp;&emsp;这种方式就是上面所展示的包扫描的方式实现的对应包下面的所有实例都会交给 Spring 容器进行 Bean 的管理。

这几种注解的使用场景一般如下：

- @Component : 侧重于通用的Bean类；
- @Service：标识该类用于业务逻辑；
- @Controler：标识该类为Spring MVC的控制器类；
- @Repository: 标识该类是一个实体类，只有属性和Setter,Getter；

**2、通过 @Bean 的方式来导入组件(适用于导入第三方组件)；**

&emsp;&emsp;这种方式就是上面的使用 @Configuration 注解，然后在配置类中定义的一些 @Bean 对应的将 Bean 对象交给 Spring 容器了。这里创建的 Bean 名称默认为方法的名称，比如 hyBlogs。也可以采用 @Bean("xxxx") 进行自定义。

**3、通过 @Import 注解的方式导入组件；**

- **通过使用 @Import 注解直接导入组件(导入组件的 id 为全限定类名)**
  
  ```java
  @Configuration
  @Import({HyBlogs.class, MyBlogs.class})
  public class IocConfig {
  ```

}

```
如上所示，可以直接将 bean 导入，此时获取 Bean 时需要通过 Class 的方式获取，如下测试用例：
```java
public class IocConfigTest {
    private AnnotationConfigApplicationContext context;

    @Before
    public void before() {
        context = new AnnotationConfigApplicationContext(IocConfig.class);
    }

    @Test
    public void getMyBean() {
        HyBlogs hyBlogs1 = context.getBean(HyBlogs.class);

        System.out.printf("HY-Blogs1 = %s%n", hyBlogs1);
    }
}
```

我这里测试不能通过 name 的方式获取到 Bean，只能通过 class 的方式才能获取到 Bean ，因为放入到 Spring 容器的时候是将 class 放入的，获取也只能通过 class 的方式获取吧。

- **通过使用 @Import 注解的 ImportSelector 类实现组件的导入(导入组件的 id 为全限定类名)，自定义的 BlogsImportSelector 需要实现 ImportSelector 接口。**
  
  ```java
  package com.ioc.imp;
  ```

import org.springframework.context.annotation.ImportSelector;
import org.springframework.core.type.AnnotationMetadata;

/**

* @author: hy

* @date: 2020/7/30 3:30 下午

* @Description:

* @version: v_1.0.0
  */
  public class BlogsImportSelector implements ImportSelector {
   @Override
   public String[] selectImports(AnnotationMetadata importingClassMetadata) {
  
       //返回全限定类名的数组
       return new String[]{"com.ioc.bean.HyBlogs", "com.ioc.bean.MyBlogs"};
  
   }
  }
  
  ```
  对应的测试用例如下：
  ```java
  @Configuration
  @Import({BlogsImportSelector.class})
  public class IocConfig {
  ```

}

```
此处依然是根据 class 的方式加入到 Spring 容器中，所以获取 bean 的时候也只能通过 class 的方式进行获取。

- **通过使用 @Import 注解的 ImportBeanDefinitionRegistrar 导入组件(可以指定 bean 的名称)，自定义 BlogsImportBeanDefinitionRegistrar 需要实现 ImportBeanDefinitionRegistrar 接口。**
```java
package com.ioc.imp;

import com.ioc.bean.HyBlogs;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.RootBeanDefinition;
import org.springframework.context.annotation.ImportBeanDefinitionRegistrar;
import org.springframework.core.type.AnnotationMetadata;

/**
 * @author: hy
 * @date: 2020/7/30 3:36 下午
 * @Description:
 * @version: v_1.0.0
 */
public class BlogsImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        //创建一个 bean 定义对象
        RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(HyBlogs.class);
        //把 bean 定义对象导入到容器中
        registry.registerBeanDefinition("hyBlogs",rootBeanDefinition);
    }
}
```

这一次根据了 name 和 class 两种方式放入了 Spring 容器，获取的时候可以通过 class 或者 name 进行获取 bean 了。

对应的配置类注解修改成如下：

```java
@Configuration
@Import({BlogsImportBeanDefinitionRegistrar.class})
public class IocConfig {

}
```

对应的测试用例如下：

```java
public class IocConfigTest {
    private AnnotationConfigApplicationContext context;

    @Before
    public void before() {
        context = new AnnotationConfigApplicationContext(IocConfig.class);
    }

    @Test
    public void getMyBean() {
        HyBlogs hyBlogs1 = context.getBean(HyBlogs.class);
        HyBlogs hyBlogs2 = (HyBlogs) context.getBean("hyBlogs");

        System.out.printf("HY-Blogs1 = %s%nHY-Blogs2 = %s%n", hyBlogs1, hyBlogs2);
    }
}
```

执行结果：

```java
This is HY-Blogs!!!
HY-Blogs1 = com.ioc.bean.HyBlogs@6a01e23
HY-Blogs2 = com.ioc.bean.HyBlogs@6a01e23
```

采用 @Import 注解实现 bean 的注册，在 SpringBoot 中比较常见的有 @EnableXXX 这类注解，他们就使用到了 @Import 相关的方法注册他们自定义的一些 bean，有兴趣的可以去瞅瞅。

**4、通过实现 FactoryBean 接口来实现注册组件；**

创建一个自定义的 FactoryBean，如下：

```java
package com.ioc.factory;

import com.ioc.bean.HyBlogs;
import org.springframework.beans.factory.FactoryBean;
import org.springframework.stereotype.Component;

/**
 * @author: hy
 * @date: 2020/7/30 3:48 下午
 * @Description: 自定义工厂 Bean
 * @version: v_1.0.0
 */
@Component
public class BlogsFactoryBean implements FactoryBean<HyBlogs> {
    @Override
    public HyBlogs getObject() throws Exception {
        return new HyBlogs();
    }

    @Override
    public Class<?> getObjectType() {
        return HyBlogs.class;
    }

    @Override
    public boolean isSingleton() {
        return false;
    }
}
```

修改配置类，如下：

```java
@Configuration
public class IocConfig {

    /**
     * 这里注入的是 blogsFactoryBean
     * 在容器中通过 id 为 blogsFactoryBean 获得 bean 时为该工厂的 getObject 返回的对象
     * 如果想获得这个工厂 bean，则需要在查询时的 id 前面加一个&，即 &blogsFactoryBean
     */
    @Bean("blogsFactoryBean")
    public BlogsFactoryBean blogsFactoryBean() {
        return new BlogsFactoryBean();
    }
}
```

编写测试用例，如下：

```java
public class IocConfigTest {
    private AnnotationConfigApplicationContext context;

    @Before
    public void before() {
        context = new AnnotationConfigApplicationContext(IocConfig.class);
    }

    @Test
    public void getMyBean() {
        HyBlogs hyBlogs1 = (HyBlogs) context.getBean("blogsFactoryBean");
        HyBlogs hyBlogs2 = (HyBlogs) context.getBean("blogsFactoryBean");

        System.out.printf("HY-Blogs1 = %s%nHY-Blogs2 = %s%n", hyBlogs1, hyBlogs2);
    }
}
```

执行结果：

```java
This is HY-Blogs!!!
This is HY-Blogs!!!
HY-Blogs1 = com.ioc.bean.HyBlogs@4445629
HY-Blogs2 = com.ioc.bean.HyBlogs@45b9a632
```

结果分析：

由于上面自定义 bean 工厂的时候，isSingleton() 方法默认返回的 false ，所以并不是单例的 bean，此时若将 isSingleton() 返回值修改成 true 就表示将 bean 设置成单例的。修改后再次运行结果如下：

```java
This is HY-Blogs!!!
HY-Blogs1 = com.ioc.bean.HyBlogs@4445629
HY-Blogs2 = com.ioc.bean.HyBlogs@4445629
```

可以看到，修改 isSingleton() 方法返回值为 true 后，获取到的 bean 变成了单例了。

眼尖的同学看到了上面的注释中说了可以在获取时的 id 前面加一个&，即 &blogsFactoryBean 就可以获取到我们自定义的 bean 工厂本身了，咋们试试吧，如下：

```java
@Test
public void getMyBean() {
    HyBlogs hyBlogs1 = (HyBlogs) context.getBean("blogsFactoryBean");
    HyBlogs hyBlogs2 = (HyBlogs) context.getBean("blogsFactoryBean");
    BlogsFactoryBean blogsFactoryBean = (BlogsFactoryBean) context.getBean("&blogsFactoryBean");

    System.out.printf("HY-Blogs1 = %s%nHY-Blogs2 = %s%n", hyBlogs1, hyBlogs2);

    System.out.printf("BlogsFactoryBean = %s %n", blogsFactoryBean);
}
```

执行结果：

```java
This is HY-Blogs!!!
HY-Blogs1 = com.ioc.bean.HyBlogs@4445629
HY-Blogs2 = com.ioc.bean.HyBlogs@4445629
BlogsFactoryBean = com.ioc.factory.BlogsFactoryBean@45b9a632 
```

可以看到，成功获取到了我们自定义的 bean 工厂。

### Bean 的生命周期

&emsp;&emsp;我们将 Bean 交给了 Spring 容器进行管理，那么 Spring 容器自然而然管理着 Bean 对象的创建到销毁整个过程。

Spring Bean 的生命周期有如下几个阶段：

- 1、Spring 启动，查找并加载需要被 Spring 管理的 bean，进行 Bean 的实例化；
- 2、Bean 实例化后对将 Bean 的引入和值注入到 Bean 的属性中；
- 3、如果 Bean 实现了 BeanNameAware 接口的话，Spring 将 Bean 的 Id 传递给 setBeanName() 方法；
- 4、如果 Bean 实现了 BeanFactoryAware 接口的话，Spring 将调用 setBeanFactory() 方法，将 BeanFactory 容器实例传入；
- 5、如果 Bean 实现了 ApplicationContextAware 接口的话，Spring 将调用 Bean 的 setApplicationContext() 方法，将 bean 所在应用上下文引用传入进来；
- 6、如果 Bean 实现了 BeanPostProcessor 接口，Spring 就将调用他们的 postProcessBeforeInitialization() 方法；
- 7、如果 Bean 实现了 InitializingBean 接口，Spring 将调用他们的 afterPropertiesSet() 方法。类似的，如果 bean 使用 init-method 声明了初始化方法，该方法也会被调用；
- 8、如果 Bean 实现了 BeanPostProcessor 接口，Spring 就将调用他们的 postProcessAfterInitialization() 方法；
- 9、此时，Bean 已经准备就绪，可以被应用程序使用了。他们将一直驻留在应用上下文中，直到应用上下文被销毁。
- 10、如果 bean 实现了 DisposableBean 接口，Spring 将调用它的 destory() 接口方法；同样，如果 bean 使用了 destory-method 声明销毁方法，该方法也会被调用。

#### 1、通过使用 @Bean 注解的 initMethod 和 destroyMethod 属性指定对象的初始化和销毁

```java
public class HyBlogs {

    public HyBlogs() {
        System.out.println("This is HY-Blogs!!!");
    }

    public void init() {
        System.out.println("HyBlogs is init~");
    }

    public void destroy() {
        System.out.println("HyBlogs is destroy~");
    }
}
```

修改配置类，在 @Bean 注解中添加 initMethod 和 destroyMethod 属性及其对应的实际方法名称，如下：

```java
@Configuration
public class IocConfig {

    @Bean(initMethod = "init", destroyMethod = "destroy")
    public HyBlogs hyBlogs() {
        return new HyBlogs();
    }
}
```

#### 2、通过实现 InitializingBean 和 DisposableBean 这2个接口，重写 ++属性赋值++ 和 ++销毁++ 两个生命阶段

```java
public class HyBlogs implements InitializingBean, DisposableBean {

    public HyBlogs() {
        System.out.println("This is HY-Blogs!!!");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("HyBlogs is destroy ~");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("HyBlogs is set properties ~");
    }
}
```

#### 3、通过 JSR250 规范提供的注解 @PostConstruct 和  @PreDestroy 标注的方法

@PostConstruct 该注解被用来修饰一个非静态的 void() 方法。被修饰的方法会在服务器加载 Servlet 的时候运行，并且只会被服务器执行一次。@PostConstruct 在构造函数之后执行，init() 方法之前执行。该注解的方法在整个 Bean 初始化中的执行顺序为： Constructor(构造方法) -> @Autowired(依赖注入) -> @PostConstruct(注释的方法)

@PreDestroy 注解被用来修饰一个非静态的 void() 方法，被修饰的方法会在服务器卸载 Servlet 的时候运行，并且只会被服务器调用一次，类似于 Servlet 的 destroy() 方法。被 @PreDestroy 修饰的方法会在 destroy() 方法之后运行，在 Servlet 被彻底卸载之前。

修改我们通用的 Bean，添加对应的方法及注解，如下：

```java
public class HyBlogs {

    public HyBlogs() {
        System.out.println("This is HY-Blogs!!!");
    }

    @PostConstruct
    public void init() {
        System.out.println("HyBlogs is init~");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("HyBlogs is destroy~");
    }
}
```

配置类对应的改成最简单的注入我们的 Bean 到 Spring 容器即可：

```java
@Configuration
public class IocConfig {

    @Bean
    public HyBlogs hyBlogs() {
        return new HyBlogs();
    }
}
```

这里需要说明一下，单元测试里面需要添加 context.close() 方法的调用，否则不会执行 destroy() 方法，因为只有在容器关闭的时候才会去执行销毁方法，如下修改：

```java
public class IocConfigTest {
    private AnnotationConfigApplicationContext context;

    @Before
    public void before() {
        context = new AnnotationConfigApplicationContext(IocConfig.class);
    }

    @Test
    public void getMyBean() {
        HyBlogs hyBlogs1 = context.getBean(HyBlogs.class);
        HyBlogs hyBlogs2 = (HyBlogs) context.getBean("hyBlogs");

        System.out.printf("HY-Blogs1 = %s%nHY-Blogs2 = %s%n", hyBlogs1, hyBlogs2);

        //调用 close() 方法关闭容器，对应的 bean 也会被销毁
        context.close();
    }
}
```

执行结果：

```java
This is HY-Blogs!!!
HyBlogs is init~
HY-Blogs1 = com.ioc.bean.HyBlogs@18ce0030
HY-Blogs2 = com.ioc.bean.HyBlogs@18ce0030
HyBlogs is destroy~
```

可以看到上面的执行结果中有执行到 **销毁** 的方法 destroy();

这里需要特别说明一下：<font style="color:red;">只有调用了 close() 方法关闭容器才会触发容器去销毁 Bean ，即调用 destroy() 方法。重点：Spring 容器针对多实例的 Bean 不负责销毁，即在容器关闭的时候不会调用多例的 Bean 的 destroy() 方法；仅针对 **单例** 的 Bean 负责销毁处理。</font>这里大家可以去尝试一下多例 bean 的 destroy() 是否在容器关闭时调用了。

#### 综合了解一下 Spring 中 Bean 的生命周期

修改我们的通用 Bean 对象，让其实现一些接口，好让我们了解更完整的 Bean 生命周期，如下修改：

```java
package com.ioc.bean;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.*;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

/**
 * @author: hy
 * @date: 2020/7/29 2:08 下午
 * @Description: 测试用 Bean
 * @version: v_1.0.0
 */
public class HyBlogs implements BeanNameAware, BeanFactoryAware, ApplicationContextAware, InitializingBean, DisposableBean {

    private String name;

    public String getName() {
        System.out.println("HyBlogs 执行了 getName() 方法～");
        return name;
    }

    public void setName(String name) {
        System.out.println("HyBlogs 执行了 setName() 方法～");
        this.name = "HY-Blogs";
    }

    public HyBlogs() {
        System.out.println("HyBlogs 执行了无参构造方法～");
    }

    @PostConstruct
    public void postConstruct() {
        System.out.println("HyBlogs 执行了 postConstruct() 方法～");
    }

    @PreDestroy
    public void preDestroy() {
        System.out.println("HyBlogs 执行了 preDestroy() 方法~");
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.printf("HyBlogs 执行了 setBeanFactory(%s) 方法～%n", beanFactory);
    }

    @Override
    public void setBeanName(String name) {
        System.out.printf("HyBlogs 执行了 setBeanName(%s) 方法～%n", name);
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("HyBlogs 执行了 destroy() 方法～");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("HyBlogs 执行了 afterPropertiesSet() 方法～");
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.printf("HyBlogs 执行了 setApplicationContext(%s) 方法～%n", applicationContext);
    }

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("HyBlogs 执行了 finalize() 方法～");
    }

    public void myInitMethod() {
        System.out.println("HyBlogs 执行了 myInitMethod() 方法～");
    }

    public void myDestroyMethod() {
        System.out.println("HyBlogs 执行了 myDestroyMethod() 方法～");
    }
}
```

修改配置类，设置自定义的 initMethod 及 destroyMethod 如下：

```java
@Configuration
public class IocConfig {

    @Bean(initMethod = "myInitMethod", destroyMethod = "myDestroyMethod")
    public HyBlogs hyBlogs() {
        return new HyBlogs();
    }
}
```

单元测试用例可以不作大的修改，我这里只修改了打印的文字，然后直接运行即可。

**执行结果**：

```java
HyBlogs 执行了无参构造方法～
HyBlogs 执行了 setBeanName(hyBlogs) 方法～
HyBlogs 执行了 setBeanFactory(org.springframework.beans.factory.support.DefaultListableBeanFactory@7ce6a65d: defining beans [org.springframework.context.annotation.internalConfigurationAnnotationProcessor,org.springframework.context.annotation.internalAutowiredAnnotationProcessor,org.springframework.context.annotation.internalCommonAnnotationProcessor,org.springframework.context.event.internalEventListenerProcessor,org.springframework.context.event.internalEventListenerFactory,iocConfig,hyBlogs]; root of factory hierarchy) 方法～
HyBlogs 执行了 setApplicationContext(org.springframework.context.annotation.AnnotationConfigApplicationContext@12f40c25, started on Thu Jul 30 17:23:49 CST 2020) 方法～
HyBlogs 执行了 postConstruct() 方法～
HyBlogs 执行了 afterPropertiesSet() 方法～
HyBlogs 执行了 myInitMethod() 方法～
根据 Class 获取实例1：HY-Blogs1 = com.ioc.bean.HyBlogs@6b26e945
根据 name 获取实例2：HY-Blogs2 = com.ioc.bean.HyBlogs@6b26e945
HyBlogs 执行了 preDestroy() 方法~
HyBlogs 执行了 destroy() 方法～
HyBlogs 执行了 myDestroyMethod() 方法～
```

这里先将生命周期执行的过程列出来，后面会继续补充一些知识点～

### 后置处理器

> 注意：测试下面这些示例对时候，需要开启包扫描，否则下面这些 bean 不会被扫描到，你也可以尝试其他方式进行测试。

#### 1、BeanPostProcessor：也称为Bean后置处理器，它是 Spring 中定义的接口，在 Spring 容器的创建过程中（具体为 Bean 初始化前后）会回调 BeanPostProcessor 中定义的两个方法。分别是 postProcessBeforeInitialization（初始化之前）和 postProcessAfterInitialization（初始化之后）

接下来我们自定义一个我们自己的后置处理器 BlogsPostProcessor，实现 BeanPostProcessor 接口，如下：

```java
package com.ioc.processor;

import com.ioc.bean.HyBlogs;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.stereotype.Component;

/**
 * @author: hy
 * @date: 2020/7/30 5:33 下午
 * @Description: 自定义的 Bean 后置处理器
 * @version: v_1.0.0
 */
@Component
public class BlogsPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if(bean instanceof HyBlogs){
            System.out.println("马上开始初始化 HyBlogs 了，请留意一下哦～莫打摆子了！");
        }
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if(bean instanceof HyBlogs){
            System.out.println("HyBlogs 已经初始化完成了，快注意点儿～");
        }
        return bean;
    }
}
```

注意：

- 1、接口中这两个方法不能返回 null，如果返回 null 那么在后续初始化方法将报空指针异常或者通过 getBean() 方法获取不到 bean 实例对象，因为后置处理器从 Spring IoC 容器中取出 bean 实例对象没有再次放回 IoC 容器中。
- 2、BeanFactory 和 ApplicationContext 两个容器对待 bean 的后置处理器稍微有些不同。
- 2.1、ApplicationContext 容器会自动检测 Spring 配置文件中那些 bean 所对应的 Java 类实现了 BeanPostProcessor 接口，并自动把它们注册为后置处理器。在创建 bean 过程中调用它们，所以部署一个后置处理器跟普通的 bean 没有什么太大区别。
- 2.2、BeanFactory 容器注册 bean 后置处理器时必须通过代码显示的注册，在 IoC 容器继承体系中的 ConfigurableBeanFactory 接口中定义了注册方法。

#### 2、BeanFactoryPostProcessor：Bean 工厂的后置处理器；触发时机：在 bean 定义注册之后， bean 实例化之前

自定义 BlogsFactoryPostProcessor 作为 Bean 工厂的后置处理器，实现 BeanFactoryPostProcessor 接口，如下示例：

```java
package com.ioc.processor;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.stereotype.Component;

/**
 * @author: hy
 * @date: 2020/7/30 5:59 下午
 * @Description: 自定义的 Bean 工厂的后置处理器
 * @version: v_1.0.0
 */
@Component
public class BlogsFactoryPostProcessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        System.out.println("调用了 BlogsFactoryPostProcessor 的 postProcessBeanFactory 方法");
        for (String beanName : beanFactory.getBeanDefinitionNames()) {
            if ("hyBlogs".equals(beanName)) {
                BeanDefinition beanDefinition = beanFactory.getBeanDefinition(beanName);
                beanDefinition.setLazyInit(true);
            }
        }
    }
}
```

#### 3、BeanDefinitionRegistryPostProcessor：Bean 定义的后置处理器，它继承了 BeanFactoryPostProcessor；触发时机：在 bean 的定义注册之前

接下来我们自定义 BlogsDefinitionRegistryPostProcessor 作为我们自己的 Bean 定义的后置处理器，如下示例：

```java
package com.ioc.processor;

import com.ioc.bean.HyBlogs;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.BeanDefinitionRegistryPostProcessor;
import org.springframework.beans.factory.support.RootBeanDefinition;
import org.springframework.stereotype.Component;

/**
 * @author: hy
 * @date: 2020/7/30 6:04 下午
 * @Description: 自自定义的 Bean 定义的后置处理器
 * @version: v_1.0.0
 */
@Component
public class BlogsDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        System.out.println("调用 BlogsDefinitionRegistryPostProcessor.postProcessBeanDefinitionRegistry() 方法");
        System.out.println("bean 定义的数据量:" + registry.getBeanDefinitionCount());
        RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(HyBlogs.class);
        registry.registerBeanDefinition("hyBlogs", rootBeanDefinition);
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        System.out.println("调用 BlogsDefinitionRegistryPostProcessor.postProcessBeanFactory() 方法");
        System.out.println(beanFactory.getBeanDefinitionCount());
    }
}
```

### Aware 接口

&emsp;&emsp;Spring 提供了大量的 Aware 接口，使得我们可以使用 Spring 的一些底层提供的容器和资源。比如：获取 ApplicationContext 就可以实现 ApplicationContextAware 接口，获取 BeanFactory 就可以实现 BeanFactoryAware，这些 Aware 接口的回调是在 Bean 初始化 initializeBean() 方法中进行回调的。

比如我们要使用 Spring 底层的 ApplicationContext，则需要实现 ApplicationContextAware 接口，如下示例：

```java
package com.ioc.award;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

/**
 * @author: hy
 * @date: 2020/7/30 6:10 下午
 * @Description: 获取 Spring 上下文
 * @version: v_1.0.0
 */
@Component
public class BlogsApplicationContextAware implements ApplicationContextAware {

    private ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("哇哦～我获取到 Spring 容器啦，哈哈哈～");
        context = applicationContext;
    }
}
```

### Lifecycle 接口

&emsp;&emsp;每个对象都有自己生命周期的需求，主要方法： isAutoStartup() 返回 true 的时候，Spring 容器启动时会去执行 start() 方法。isRunning() 返回 true 的时候，容器销毁时会调用stop() 方法。比如 eruaka 启动的入口就是通过实现 SmartLifecycle 接口来实现的。

接下来咋们自定义 BlogsLifecycle 实现 SmartLifecycle 接口，如下：

```java
package com.ioc.lifecycle;

import org.springframework.context.SmartLifecycle;
import org.springframework.stereotype.Component;

/**
 * @author: hy
 * @date: 2020/7/30 6:16 下午
 * @Description: 
 * @version: v_1.0.0
 */
@Component
public class BlogsLifecycle implements SmartLifecycle {

    @Override
    public boolean isAutoStartup() {
        //true：Spring 容器启动时会去执行 start() 方法
        return true;
    }

    @Override
    public int getPhase() {
        return 0;
    }

    @Override
    public void stop(Runnable callback) {

    }

    @Override
    public void start() {
        System.out.println("This is BlogsLifecycle.start()");
    }

    @Override
    public void stop() {
        System.out.println("This is BlogsLifecycle.stop()");
    }

    @Override
    public boolean isRunning() {
        //true：容器销毁时会调用stop()方法
        return true;
    }
}
```

### 自动装配

- 1、 @Autowired 默认情况下：首先是按照类型进行装配，若在 IOC 容器中发现了多个相同类型的组件，那么就按照属性名称来进行装配。@Autowired 有个属性为 required，可以配置为 false，如果配置为 false 之后，当没有找到相应 bean 的时候，系统不会抛错；@Autowired可以作用在**变量、setter 方法、构造函数**上。
- 2、@Autowired 假设我们需要指定特定的组件来进行装配，我们可以通过使用 @Qualifier("hyBlogs") 来指定装配的组件，也可以在配置类上的 @Bean 加上 @Primary 注解。此时，**自动注入的策略就从 byType 转变成 byName** 了。
- 3、@Resource(JSR250规范) 功能和 @AutoWired 的功能差不多一样，但是不支持 @Primary 和 @Qualifier 的支持。
- @Inject(JSR330规范) 需要导入 jar 包依赖功能和支持 @Primary 功能，但是没有 Require=false 的功能。@Inject 是根据类型进行自动装配的，如果需要按名称进行装配，则需要配合 @Named 一起使用；@Inject可以作用在**变量、setter方法、构造函数**上。

测试用例如下：

```java
public class HyBlogs {

    private String name;

    public String getName() {
        System.out.println("HyBlogs 执行了 getName() 方法～");
        return name;
    }

    public void setName(String name) {
        System.out.println("HyBlogs 执行了 setName() 方法～");
        this.name = name;
    }

    public HyBlogs() {
        System.out.println("HyBlogs 执行了无参构造方法～");
        this.name = "HY-Blogs";
    }
}


public class MyBlogs {

    @Autowired
    private HyBlogs hyBlogs;

    public MyBlogs() {
        System.out.println("This is MyBlogs!");
    }

    public void getName() {
    System.out.printf("HyBlogs 的 name = %s%n", this.hyBlogs.getName());
    }
}
```

需要修改配置类，创建两个 bean，如下：

```java
@Configuration
public class IocConfig {

    @Bean
    public HyBlogs hyBlogs() {
        return new HyBlogs();
    }

    @Bean
    public MyBlogs myBlogs() {
        return new MyBlogs();
    }
}
```

修改单元测试：

```java
public class IocConfigTest {
    private AnnotationConfigApplicationContext context;

    @Before
    public void before() {
        context = new AnnotationConfigApplicationContext(IocConfig.class);
    }

    @Test
    public void getMyBean() {
        MyBlogs myBlogs = context.getBean(MyBlogs.class);

        myBlogs.getName();

        //调用 close() 方法关闭容器，对应的 bean 也会被销毁
        context.close();
    }
}
```

执行结果：

```java
HyBlogs 执行了无参构造方法～
This is MyBlogs!
HyBlogs 执行了 getName() 方法～
HyBlogs 的 name = HY-Blogs
```

上面这就是普通的 @Autowired 的常规用法了。

#### @Autowired + @Qualifier 的用法示例，在 @Autowired 注解上面加上 @Qualifier 注解，指定对应的值：

```java
@Service
public class MyBlogs {

    @Qualifier(value = "hyBlogs")
    @Autowired
    private HyBlogs hyBlogs;

    public MyBlogs() {
        System.out.println("This is MyBlogs!");
    }

    public void getName() {
        System.out.printf("HyBlogs 的 name = %s%n", this.hyBlogs.getName());
    }
}
```

@Qualifier 作用为限定描述符，用于细粒度选择候选者。注入 Bean 的时候可能发现有多个可注入对象，比如说一个  Service 接口有 3 个实现类，分别为 impl1，impl2，impl3，你注入 Service 的时候注入的是接口，那么就可以通过  @Qualifier(“你要注入的bean的名称”) 来选择注入具体的对象。@Qualifier 的标注对象是**成员变量、方法入参、构造函数入参**。

#### @Autowired + @Primary，在 @Bean 下面加上 @Primary 注解，示例如下：

```java
@Configuration
public class IocConfig {

    @Bean
    @Primary
    public HyBlogs hyBlogs() {
        return new HyBlogs();
    }

    @Bean
    public HyBlogs hyBlogs1() {
        return new HyBlogs();
    }

    @Bean
    public MyBlogs myBlogs() {
        return new MyBlogs();
    }
}
```

@Primary 主要用在一些特殊情况中，比如：对同一个接口，可能会有几种不同的实现类，而默认只会采取其中一种的情况下就可以使用 @Primary 注解了。用@Primary 可以告诉 Spring 在无法确定唯一 Bean 的时候优先选择哪一个具体的 Bean 实现。

### 汇总

&emsp;&emsp;上面咋们罗列了一些常用的注解及其使用方式和相关的限定条件。汇总起来大概就是如下一些：

- @Configuration：把一个类作为一个 IoC 容器，它的某个方法头上如果注册了 @Bean，就会作为这个 Spring 容器中的 Bean；
- @Scope：注解用于限定作用域，可用于方法或者类上面；
- @Lazy：用于配置是否延迟初始化；
- @Service：用于标注业务层组件；
- @Controller：用于标注控制层组件(SpringMVC);
- @Repository：用于标注数据访问组件，即 DAO 组件;
- @Component：泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注;
- @PostConstruct：用于指定初始化方法（用在方法上）；
- @PreDestory：用于指定销毁方法（用在方法上）；
- @Resource：默认按名称装配，当找不到与名称匹配的 bean 才会按类型装配；
- @Primary：自动装配时当出现多个 Bean 候选者时，被注解为 @Primary 的 Bean 将作为首选者，否则将抛出异常；
- @Autowired：默认按类型装配，如果我们想使用按名称装配，可以结合 @Qualifier 注解一起使用；
- @Autowired + @Qualifier(“personDaoBean”) 存在多个实例配合使用。

剩余的其他一些注解就留给大家去探索把～

<br/>

> 参考资料来源声明：https://www.cnblogs.com/toby-xu/p/11310127.html
