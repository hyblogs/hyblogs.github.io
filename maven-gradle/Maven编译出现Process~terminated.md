# Maven编译出现Process terminated

&emsp;&emsp;今天在编译某个 Maven 项目的时候突然出现了 `Process terminated` 的错误提示，咋眼一看心生疑惑：我平时按照这样执行都好好的，为啥今天就不行了呢。

## 原因一

我只是很单纯地执行了个“clean”命令，怎么就被突然终止了呢？于是找到了较为完整的执行过程日志信息，如下：

```java
[INFO] Scanning for projects...
[ERROR] [ERROR] Some problems were encountered while processing the POMs:
[ERROR] 'packaging' with value 'jar' is invalid. Aggregator projects require 'pom' as packaging. @ line 3, column 110
 @ 
[ERROR] The build could not read 1 project -> [Help 1]
[ERROR]   
[ERROR]   The project com.hy:hy-blogs:0.0.1-SNAPSHOT (/Users/hy/code_repository/hy-blogs/pom.xml) has 1 error
[ERROR]     'packaging' with value 'jar' is invalid. Aggregator projects require 'pom' as packaging. @ line 3, column 110
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/ProjectBuildingException
```

看完这一段日志，仿佛明白了点儿什么，因为上面的日志中已经说明了是什么错误异常导致的执行“clean”被终止。**重点就是这句**：

```java
[ERROR]     'packaging' with value 'jar' is invalid. Aggregator projects require 'pom' as packaging. @ line 3, column 110
```

大概意思就是说不能指定打包类型为 `jar`，必须要指定为 `pom`。于是乎，我便跟着指引，将 pom.xml 文件中的 “packaging” 配置为了 “pom”，如下：

```xml
...
<groupId>com.hy</groupId>
<artifactId>hy-blogs</artifactId>
<version>0.0.1-SNAPSHOT</version>
<packaging>pom</packaging>
...
```

## 原因二

当我再次执行“clean”的时候，本以为它会顺利执行完成，结果没想到还是提示 `Process terminated` ，这就纳闷儿了，我不是按照提示修改了，咋还会这样呢。没办法，只能继续看看日志了，日志如下：

```java
[INFO] Scanning for projects...
[ERROR] [ERROR] Some problems were encountered while processing the POMs:
[ERROR] 'dependencies.dependency.version' for org.springframework.boot:spring-boot-starter-web:jar is missing. @ line 20, column 21
[ERROR] 'dependencies.dependency.version' for org.springframework.boot:spring-boot-starter-test:jar is missing. @ line 25, column 21
 @ 
[ERROR] The build could not read 1 project -> [Help 1]
[ERROR]   
[ERROR]   The project com.hy:office:0.0.1-SNAPSHOT (/Users/hy/code_repository/hy-blogs/office/pom.xml) has 2 errors
[ERROR]     'dependencies.dependency.version' for org.springframework.boot:spring-boot-starter-web:jar is missing. @ line 20, column 21
[ERROR]     'dependencies.dependency.version' for org.springframework.boot:spring-boot-starter-test:jar is missing. @ line 25, column 21
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/ProjectBuildingException
```

大概意思就是我的 `jar` 包没配置好版本 `version`。噢，原来如此，因为我的项目父模块是 `hy-blogs`，其中一个子模块是 `office`，我的所有依赖都改为了由 **依赖关系管理器** 来进行管理，将原本的 **依赖关系** 放到了 **依赖关系管理** 节点里面了，如下：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${mybatis.version}</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

因为我的 `office` 模块也依赖了 `spring-boot-starter-web` 和 `spring-boot-starter-test`，形成了 `office` 模块继承 `hy-blogs` 模块的形式。如果我父模块不指定版本 `version` 当子模块实际依赖使用的时候，肯定找不到对应要使用的是哪个版本，所以就报错终止了我的执行，这才是原因所在。

其实最开始父模块和子模块的依赖都是这样的：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.5</version>
    <relativePath/>
</parent>

<!-- 中间省略无关内容 -->

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>${mybatis.version}</version>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

害～ 自己手贱，调整了依赖关系。。。。

所以，我只能按照日志提示给我父模块 `pom` 依赖的 `jar` 指定了版本 `version` ，如下：

```xml
<properties>
    <java.version>1.8</java.version>
    <spring.version>2.5.5</spring.version>
    <mybatis.version>2.2.0</mybatis.version>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${mybatis.version}</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <version>${spring.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

这样加上版本之后，子模块在执行的过程中就知道是运行哪个版本了，所以就不会出错了。

经过这番折腾，现在执行 `clean` 就没问题了，终于看到 `BUILD SUCCESS` 了。如下：

```java
[INFO] Scanning for projects...
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Build Order:
[INFO] 
[INFO] hy-blogs                                                           [pom]
[INFO] office                                                             [jar]
[INFO] 
[INFO] --------------------------< com.hy:hy-blogs >---------------------------
[INFO] Building hy-blogs 0.0.1-SNAPSHOT                                   [1/2]
[INFO] --------------------------------[ pom ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:3.1.0:clean (default-clean) @ hy-blogs ---
[INFO] 
[INFO] ---------------------------< com.hy:office >----------------------------
[INFO] Building office 0.0.1-SNAPSHOT                                     [2/2]
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:3.1.0:clean (default-clean) @ office ---
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary for hy-blogs 0.0.1-SNAPSHOT:
[INFO] 
[INFO] hy-blogs ........................................... SUCCESS [  0.210 s]
[INFO] office ............................................. SUCCESS [  0.003 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.438 s
[INFO] Finished at: 2021-10-03T16:17:22+08:00
[INFO] ------------------------------------------------------------------------
```

## 总结

经过这次折腾，总结起来就是出现这样的问题，多半是配置文件搞错了，按照日志提示进行调整就好啦！首先，可能是你的 Maven 项目对应的 pom.xml 文件配置有问题；其次，也有可能是你的 Maven 组件使用的 settings.xml 文件配置错误噢Y(^_^)Y。
