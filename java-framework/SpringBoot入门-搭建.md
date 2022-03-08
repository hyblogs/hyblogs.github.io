# SpringBoot 入门-搭建

&emsp;&emsp;提到 SpringBoot 大部分人都应该比较熟悉了，至今已发布了好几年了，截止目前(2020-04-23)最新的稳定版本是 V_2.2.6 了，记得第一次用 SpringBoot 开发项目的时候还是用的 V_1.5.X 的版本，一直到后面都出 V_2.0.X 的时候莫名的嫌弃之前用的版本太老了。（苦笑）

## 一、什么是 SpringBoot 呢？

 &emsp;&emsp;百度百科的解释是这样的：

&emsp;&emsp;Spring Boot是由Pivotal团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。通过这种方式，Spring Boot致力于在蓬勃发展的快速应用开发领域(rapid application development)成为领导者。

## 二、构建第一个 SpringBoot 应用程序

&emsp;&emsp;构建 SpringBoot 应用程序的方式有很多种，本次构建会用到的 IDE 工具有 IntelliJ IDEA 、Eclipse，采用 Maven 对项目进行构建，基于 JDK 1.8 版本。
&emsp;&emsp;接下来将根据常用的几种方式依次进行创建 SpringBoot 应用程序。本篇主要描述创建 SpringBoot 项目的几种方式，不描述 SpringBoot 的具体配置、具体功能开发等。

### (一)、访问 [start.spring.io](https://start.spring.io/) 官网生成项目文件

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587650130314.png)

&emsp;&emsp;如图，我们选择采用 Maven 对项目进行构建，选择 Java 作为主要语言，选用2.2.6 稳定版，打包方式选择 Jar 包，JAVA 版本选 8 (即JDK1.8)。在这里我们可以对项目的 Group 、Artifact、Name、Description、Package name 等参数进行自定义设置。
&emsp;&emsp;此时，如果我们直接构建的话，是不包含其他第三方组件依赖的，点击“GENERATE”按钮即可开始构建，构建完成后会自动下载到本地。

&emsp;&emsp;在 IntelliJ IDEA 中打开刚才下载的项目，IDEA 会自动通过 Maven 下载对应的 Jar 包，并进行项目构建。

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587710517972.png)

构建完成后即可打开启动类(如上图，我的启动类是：DemoApplication)运行启动即可！(因为 SpringBoot 内嵌有 Tomcat ，所以不需要单独打包放入 Tomcat 中运行)

⚠️注意：此处无法通过网页访问查看启动效果，那是因为我们在创建项目的时候没有添加 WEB 依赖支持。

&emsp;&emsp;如果需要 WEB 依赖支持，需要在创建项目的时候选择加入 WEB 依赖支持即可。

如下图，打开添加依赖面板：
![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587711075374.png)

点击如图按钮添加依赖：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587711294662.png)

在搜索栏搜索关键词“WEB”，出现的第一条“Spring Web”即是我们需要的 WEB 依赖支持，点击选择依赖即可。这样在我们创建项目的依赖列表里面就会有一条 WEB 依赖的记录了，如图：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587711433198.png)

此时，我们再重新构建并下载项目即可。

用 IDEA 打开最近下载的项目，找到启动类，启动项目，如图：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587713040057.png)

如上图所示，启动成功，Tomcat 对应的端口是 8080，咋们直接本地访问 [localhost:8080](http://localhost:8080),即可得到下面的页面：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587713173416.png)

可以访问，但由于目前还没有可以访问页面，所以看到提示是空白页。

这是我们可以在 resources 目录下面的 static 文件夹里面添加一个 index.html 页面，内容随意，如图：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587713573691.png)

然后重启一下服务，再次访问 [localhost:8080](http://localhost:8080)的时候就可以看到我们刚才添加的页面效果了，如图：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587713604455.png)

上面我们添加了 Web 依赖支持，如果需要其他更多的依赖支持，比如：Redis、MySQL、MyBatis 等等，那么在创建项目的时候搜索对应的组件添加到依赖列表后构建项目下载下来即可，或者打开pom.xml文件手动添加对应的组件依赖也可以的。

### (二)、通过 IntelliJ IDEA 创建 SpringBoot 项目

&emsp;&emsp;上面通过浏览器访问 [start.spring.io](https://start.spring.io/) 我们已经创建了第一个 SpringBoot 项目，但是某些同学可能不太喜欢这种方式，像我这种用了 IDEA 大概有四五年的非新用户来说，对 IDEA 非常依赖，自然希望一切都在 IDEA 中完成。接下来咋们一起在 IDEA 中创建我们的 SpringBoot 项目。

&emsp;&emsp;说干就干，开始吧～

#### 1、打开 IDEA 点击“+ Create New Project”，如图：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587716273236.png)

#### 2、选中“Spring Initializr”，然后配置项目的 JDK 和 Starter Service URL(Starter Service 可以选中默认地址)，如下图：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587716765357.png)

#### 3、点击”Next“进入项目信息设置面板，如下图：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587716953796.png)

&emsp;&emsp;在这个面板中我们可以设置项目的 Group、Artifact、Type、Language、Packaging、Java Version 等等很多配置项。这里我们需要主要的是，Type 需要选择“Maven POM(Generate a Maven pom.xml)”，这样创建的项目会生成 pom.xml 文件，方便使用 Maven 管理各种依赖配置。

#### 4、点击“Next”进入依赖组件选择面板，如下图：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587717507927.png)

&emsp;&emsp;在这里，我们可以选择我们需要用到的依赖组件，大家可以根据自己的需要进行勾选，我这里只选择 Web 依赖，如下图：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587717806088.png)

选择好后，点击“Next”进入下一步。

#### 5、选择项目存放位置及存放文件夹名称(项目名称和存放文件夹名称可以不一样)，如下图：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587717908460.png)

点击“Finish”即完成了项目的相关设置，IDEA 会自动创建 SpringBoot 项目并打开，如下图，是 IDEA 自动打开项目后大致的结构：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587718031778.png)

&emsp;&emsp;如果是第一次构建这个版本的 SpringBoot 项目，IDEA会自动检测后下载相关的依赖 Jar 包，如果网络不太好则需要等待一定的时间，我这边是之前构建过这个版本的 SpringBoot 项目，相关的 Jar 包已经在本地的 Maven 仓库存储了一份，就不需要从网上下载了。
(PS：如果觉得从 [Maven中央仓库](https://maven.apache.org/) 下载 Jar 包很慢，可以更改仓库源地址到国内的仓库，比如：[阿里云Maven仓库](https://maven.aliyun.com/) <settings.xml配置URL：http://maven.aliyun.com/nexus/content/groups/public即可>；如果自己搭建一个 Maven 私服，也是可以起到加速作用的，我之前就在阿里云上面搭建了属于自己的 Maven私服，速度还可以，后面会分享教程的哦～)

&emsp;&emsp;通过上面的5个步骤就创建完成了一个 SpringBoot 项目，然后我们可以像第一种创建方式里面一样，增加一个 index.html 后即可启动服务正常访问了。

### (三)、通过 Eclipse 创建 SpringBoot 应用

&emsp;&emsp;上面我们通过 IDEA 创建了一个 SpringBoot 应用程序，但是有部分同学不喜欢用 IDEA 作为开发工具，而是比较倾向于 Eclipse 作为首选开发工具。所以，为了兼顾使用 Eclipse 的同学，咋们增加一种使用 Eclipse 创建 SpringBoot 应用的方式。

&emsp;&emsp;说干就干，咋们一起开始吧～

首先，需要给 Eclipse 安装 STS 插件，咋们从头开始吧！

#### 1、打开 Eclipse 插件市场，搜索 sts ,选择图中标出的那个插件 然后“Install”即可，我这里已经安装了，所以显示的是“Installed”，如图：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587722102001.png)

#### 2、安装好 sts 插件后我们选择创建项目(Create a project)，然后选择 Spring Boot --> Spring Starter Project，如图：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587722528730.png)

#### 3、点击“Next”进入下一步，设置项目信息面板，如图：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587722618669.png)

在这里，我们可以设置服务地址，默认使用[https://start.spring.io](https://start.spring.io)，可以设置项目名称，存放位置，Java版本等等很多信息。

#### 4、再次点击“Next”进入下一步，选择 SpringBoot 版本以及需要依赖的组件，如图：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587722807652.png)

现在，大家可以根据自己的需要选择合适的依赖组件，我这里默认只选择一个 Web 依赖组件即可，然后点击“Finsh”，Eclipse 就开始创建项目了。

&emsp;&emsp;由于我们选择了 WEB 依赖，所以，我们可以在resources/static下面创建一个静态页面 index.html 并添加内容，然后找到项目的启动类启动即可访问，如下图所示：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587723899890.png)

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587723881068.png)

以上便是采用 Eclipse 作为 IDE 工具进行创建 Spring Boot 项目的全部过程了，接下来咋们再试试另外的方式哦～

### (四)、采用 SpringBoot CLI 创建 SpringBoot 项目

#### 1、下载 SpringBoot CLI 安装包

&emsp;&emsp;由于 SpringBoot CLI 是 Spring 官方提供的，所以我们可以去对应的网址上下载到合适自己的版本，地址是：[spring-boot-cli](https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/)，访问后即到达如下界面：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587821678899.png)

选择适合自己的版本进行下载即可，我这里选择的是最新版 [2.2.6.RELEASE](https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.2.6.RELEASE/)，然后会到达对应版本的具体下载页面，例如，2.2.6.RELEASE 版本的下载页面是这样的：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587821919761.png)

选择 [spring-boot-cli-2.2.6.RELEASE-bin.tar.gz](https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.2.6.RELEASE/spring-boot-cli-2.2.6.RELEASE-bin.tar.gz) 或者 [spring-boot-cli-2.2.6.RELEASE-bin.zip](https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.2.6.RELEASE/spring-boot-cli-2.2.6.RELEASE-bin.zip)下载即可，用 Windows 的同学建议下载 *.zip 压缩包版本，用 Mac 的同学建议下载 *.tar.gz 压缩包版本。

下载完后，解压即可看到有一个文档，叫 “INSTALL.txt” 这就是安装说明了。查看安装说明里面说 JDK 版本必须得 1.8 及以上，所以各位看官如果想尝试这种方式创建 SpringBoot 应用则必须升级自己本地的 JDK 环境到 1.8 及以上版本。

#### 2、安装 SpringBoot CLI

&emsp;&emsp;按照按照说明文档咋们即可按步骤一步一步完成安装了。一般而言，在 *.zip(或者 *.tar.gz) 解压后的文件夹的 bin/ 目录下存在一个 spring 脚本(Windows 环境下对应的是 spring.bat )，或者使用 java -jar 来运行一个 .jar 文件(该脚本会帮你确定 classpath 是否被正确设置)。

由于我这里使用的是 Mac ，所以我下载的是 *.tar.gz 安装包，解压之后的目录应该都是一致的。

首先，采用控制台模式，进入到刚才下载的 CLI 工具的 bin 目录，如图：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587823789586.png)

可以看到，有两个文件，一个是 spring 另外一个是 spring.bat，我这里使用 spring ，spring.bat 是给 Windows 用户使用的。

执行命令，进入 shell 模式：

```
hy@HYdeMacBook-Pro bin % sh spring shell 
```

初始化一个 Web 项目：

```
init --dependencies=web springboot-web
```

这里我添加的依赖是 web ，你们可以根据自己需求进行添加，执行完成后会输出创建成功的 SpringBoot 项目地址，如图：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587824142190.png)

这时，我们就可以找到刚才创建成功的项目文件夹，导入到自己平常使用的 IDE 工具中即可使用。我平时使用的就是 IDEA ，所以我这里导入到 IDEA 中进行测试。

导入 IDE 工具之后对应的项目结构就是这样啦：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587824456497.png)

咋们运行下试试效果吧：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587824502822.png)

可以看到启动成功，访问端口是 8080 ，咋们本地访问试一下：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587824542367.png)

出现这个页面表示能访问，只是暂时没有页面，我们可以在 resources/static 目录下添加一个 index.html 静态页面即可看正常访问效果了(添加完成后记得重启)，如图：

![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587824651370.png)

到此，咋们使用官方的 SpingBoot CLI 工具创建 SpringBoot 应用程序已经完成了。

有的同学可能会说，借助 SpringBoot CLI 还有其他方式可以创建 SpringBoot 应用程序，这里，咋们就不过多的去采用 CLI 的方式啦，有兴趣的同学可以自己研究研究。

## 三、总结

&emsp;&emsp;本篇讲述只适合 SpringBoot 新用户，作为一个入门的版本教程，可以先按照上面的某种方式创建一个 SpringBoot 应用程序玩玩儿，尝尝鲜即可。如果是作为一个 SpringBoot 初学者，那么你的态度应该是：不要问我为什么，反正我就是可以这样运行！
&emsp;&emsp;现在这个时代，IT技术更新很快，我们去学习一个新技术，一开始不一定需要去弄懂它的原理、流程以及组织代码什么的。我们可以直接从官网或者博客里面仿写一个小 demo ，自己亲自跑一遍，能正常运行起来之后再去一点一点深入的解其中的原理以及用法等等。

**后续：我会继续记录下如何对 SpringBoot 项目进行配置，常见的组件如何集成到一起，常见的问题该如何去解决......咋们一起努力吧💪，奥利给！！**
