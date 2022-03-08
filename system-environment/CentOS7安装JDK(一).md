# CentOS7 安装 JDK (一)



&emsp;&emsp;最近各大云平台都有开年季活动，找个自己没注册过的平台进行注册成为新用户便可以用比较低的价格购买到心仪的云服务产品，作为一个平时爱折腾的小伙子自然不会放过它！

&emsp;&emsp;作为一名Java-Developer，买来服务器肯定得装上JDK啊，说干就干，下面咋们开始给一台全新的云服务器安装JDK1.8(PS：公司项目用的1.8，平时自己玩儿自然也是1.8咯)。

### 一、下载JDK安装包

&emsp;&emsp;由于是全新安装，第一步就可以去下载对应符合自己版本的JDK安装包了，如果是非全新安装，还需要按需卸载掉之前旧的已安装的JDK。

&emsp;&emsp;这里提供1.8的官方下载地址：https://www.oracle.com/java/technologies/javase-jdk8-downloads.html
&emsp;&emsp;历史版本可以移步这个地址：http://www.oracle.com/technetwork/java/javase/archive-139210.html 

&emsp;&emsp;本篇将采用1.8的版本进行记录，访问1.8的链接打开页面找到符合的平台版本进行下载：
![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1585814566155.png)

<font style="color:green">
(温馨提示：请提前准备好Oracle账户，下载的时候会自动跳转到Oracle登录界面，登录成功后会自动下载！)</font>

下载时可以复制下载链接到服务器上通过 wget 命令进行下载，这样就不用因为下载到本地需要再次传入服务器了；但是这样下载的JDK安装包后面会带上小尾巴。
比如：
<font style="color:#33FF00;">

> jdk-8u241-linux-x64.tar.gz\?AuthParam\=1585814799_5d315930a3af6341eb81ab6cf06040cd</font> 
>  需要重命名为： <font style="color:#33FF00;">jdk-8u241-linux-x64.tar.gz </font> &nbsp;这样看着就舒服多了。

我是下载到本地再通过 scp 命令传入服务器指定目录的(我是放data目录下的)命令行：
<font style="color:#33FF00;">

> scp /Users/Downloads/jdk-8u241-linux-x64.tar.gz root@192.168.1.23:/data/
> </font>

### 二、安装JDK

1、进入刚才下载的 JDK 源码包所在服务器上的目录，命令行：

> cd /data/

2、解压 JDK 源码包到 /usr/local/ 目录下，命令行：

> tar -zxvf jdk-8u241-linux-x64.tar.gz -C /usr/local/

3、切换到 /usr/local/ 目录下，命令行：

> cd /usr/local/

4、重命名 jdk1.8.0_241 为 java ，命令行：

> mv jdk1.8.0_241/ java

### 三、配置Java环境变量

1、编辑服务器系统全局变量文件 /etc/profile ,命令行：

> vim /etc/profile

2、按 i/insert 进入编辑模式，在文件内容末尾添加 Java 配置项，配置项：

```
#JAVA Environment
export JAVA_HOME=/usr/local/java
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin

注意：Linux环境中环境变量采用英文冒号(:)，不同于Windows环境下所采用的英文分号(;)作间隔。
```

3、按 ESC 退出编辑模式，输入 :wq 关闭，命令行：

> :wq

4、让刚刚设置的环境变量生效，命令行：

> source /etc/profile
> 如果环境变量设置得有问题，那么执行此命令会出现错误信息提示出错的地方。

### 四、检查环境变量配置是否生效 & 安装是否成功

1、输入 java -version 查看是否有 刚才安装的 JDK 版本信息，命令行：

> java -version

出现如下信息则表示安装成功：

```
java version "1.8.0_241"
Java(TM) SE Runtime Environment (build 1.8.0_241-b07)
Java HotSpot(TM) 64-Bit Server VM (build 25.241-b07, mixed mode)
```

2、分别输入 java 和 javac 查看是否配置成功，命令行：

> java

出现如下信息则表示配置成功：

```
Usage: java [-options] class [args...]
           (to execute a class)
   or  java [-options] -jar jarfile [args...]
           (to execute a jar file)
where options include:
    -d32      use a 32-bit data model if available
    -d64      use a 64-bit data model if available
    -server      to select the "server" VM
                  The default VM is server,
                  because you are running on a server-class machine.

...

java.lang.instrument
    -splash:<imagepath>
                  show splash screen with specified image
See http://www.oracle.com/technetwork/java/javase/documentation/index.html for more details.
```

> javac

出现如下信息则表示配置成功：

```
Usage: javac <options> <source files>
where possible options include:
  -g                         Generate all debugging info
  -g:none                    Generate no debugging info
  -g:{lines,vars,source}     Generate only some debugging info
  -nowarn                    Generate no warnings
  -verbose                   Output messages about what the compiler is doing
  -deprecation               Output source locations where deprecated APIs are used
  -classpath <path>          Specify where to find user class files and annotation processors
  -cp <path>                 Specify where to find user class files and annotation processors
  -sourcepath <path>         Specify where to find input source files
  -bootclasspath <path>      Override location of bootstrap class files
  -extdirs <dirs>            Override location of installed extensions
  -endorseddirs <dirs>       Override location of endorsed standards path
  -proc:{none,only}          Control whether annotation processing and/or compilation is done.
  -processor <class1>[,<class2>,<class3>...] Names of the annotation processors to run; bypasses default discovery process
  -processorpath <path>      Specify where to find annotation processors
  -parameters                Generate metadata for reflection on method parameters
  -d <directory>             Specify where to place generated class files
  -s <directory>             Specify where to place generated source files
  -h <directory>             Specify where to place generated native header files
  -implicit:{none,class}     Specify whether or not to generate class files for implicitly referenced files
  -encoding <encoding>       Specify character encoding used by source files
  -source <release>          Provide source compatibility with specified release
  -target <release>          Generate class files for specific VM version
  -profile <profile>         Check that API used is available in the specified profile
  -version                   Version information
  -help                      Print a synopsis of standard options
  -Akey[=value]              Options to pass to annotation processors
  -X                         Print a synopsis of nonstandard options
  -J<flag>                   Pass <flag> directly to the runtime system
  -Werror                    Terminate compilation if warnings occur
  @<filename>                Read options and filenames from file
```

至此，JDK安装完毕！
其他JDK版本的安装和上面的方法类似，一步一步操作即可！

### 后续

> 环境：
> CentOS7，自己笔记本用镜像安装的；
> JDK 8u321

我在配置完环境变量后发现执行 `javac` 出现了如下错误信息：

```shell
-bash: /usr/local/java/bin/javac: /lib/ld-linux.so.2: bad ELF interpreter: 没有那个文件或目录
```

解决办法就是在服务器上面安装一下 `glibc.i686` 组件，如下：

```shell
 sudo yum install glibc.i686
```

即可解决问题！
