# 解决 IDEA 导入 Gradle 项目默认下载 Gradle.zip 失败

&emsp;&emsp;这几天打算看看 Spring-Framework 的源码，于是将 Spring-Framework 的源码从 GitHub 拉到了 Gitee ，然后从 Gitee 上面下载到本地，几经波折终于下载到本地了。欣喜若狂的打开 IDEA ，Open 了这个刚下载好的源码项目，结果一打开项目，它便自己开始下载起了 gradle-6.5.1-bin.zip ，下载了老半天都没啥动静，最后还直接给超时失败了。本来以前也不这样啊，我记得之前访问下载 Gradle 的时候挺快的，不知道最近这网络咋抽风了，唉～

## 解决

&emsp;&emsp;遇到上面的问题了，总不能半途而废吧，我好不容易将源码捣腾下来了。(从 GitHub 下载源码那是真真滴慢，找到方法后将对应的源码仓库添加到 Gitee ，然后再由 Gitee 将源码下载到本地就快很多了。题外话，扯远了，哈哈哈～)

### 方案一：手动下载需要版本的 Gradle 并放到指定位置

&emsp;&emsp;有人会说，我这不是网不好么，我咋手动下载啊？其实这种情况可以考虑用方案二，即：将项目使用的版本修改成你本地已有的版本。下载地址：https://services.gradle.org/distributions ，访问这个地址就可以选择你需要下载的版本了，如下：

![Gradle.Download.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1596203823326.png)

选择自己需要下载的版本，点击即可开始下载了。

&emsp;&emsp;如果你本地下载非常缓慢，同时你拥有自己的云服务器，那么你就可以借助云服务器帮你下载到云服务器的磁盘，然后你再将压缩包从自己的服务器上面下载到本地，虽然麻烦了点儿，但是能提速不少。

我自己本地下载 Gradle 就老慢了，我就在我的云服务器上面通过 `wget https://services.gradle.org/distributions/gradle-6.5.1-bin.zip `一句命令就将它下载到我服务器上面了，因为是云服务器，下载这些东西还是挺快的，我下载时候达到了 2.x MB/s 呢。由于我服务器出网带宽是 5M 的，所以从服务器上面下载东西到本地的时候，还是很稳定的700KB/s左右，这样算下来还是比在本地直接下载 Gradle 节约了很多时间的。

### 方案二：修改项目使用的 Gradle 版本

&emsp;&emsp;打开项目里面的 gradle 目录，下面有个 wrapper 文件夹，里面有个 gradle-wrapper.properties 文件，打开这个文件就可以看到需要下载的 gradle 版本及下载地址了，如下所示：

![gradle-wrapper.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1596205996889.png)

如果我们本地有其他版本的 gradle，可以修改这里的版本号和本地一致，这时候再刷新 build 就可以不用下载了。

&emsp;&emsp;默认 gradle 文件的存放目录是在当前用户目录下面的一个叫 .gradle 文件夹里面，因为文件夹前面是点(.)开头的，所以是隐藏文件，一般找不到。你也可以指定一个位置，我这里就单独创建了一个文件夹存放 gradle 的一些文件，对应的各个版本的 gradle 存放在  /Users/hy/gradle/gradle-6.5/wrapper/dists/ 这个路径下面，如下：

![gradle.dists.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1596206211825.png)

可以看到平时项目中常用的到的一些 gradle 版本。如果你是默认的目录，那么你的这个路径可能就是：/Users/当前系统用户名/.gradle/wrapper/dists/ 对应的 Windows 基本上同这个区别就是没有 /Users 这个目录，随之替换的是磁盘分区系统盘： C 盘了。

## 补充

&emsp;&emsp;补充一下，在 Idea 开始下载对应版本的 gradle 的时候，会在你设定的 gradle-home 目录下生成一系列文件夹及文件，我们可以找到对应的 dist 文件夹下面的随机目录文件夹，然后里面有 Idea 生成的 *.lck 和 *.part 文件，我们需要将我们自己下载好的 *.zip 压缩包复制到此目录中并解压，然后到 Idea 项目中刷新 gradle 构建即可，最后我们可以将 *.part 文件删除(因为他是下载时的缓存文件，没啥作用)，Idea 构建后会在刚才的目录下生成 *.ok 文件，最后的目录下文件如下：
![local file](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1607265874571.png)

IDEA中设定 Gradle 的 Home 目录(默认是用户目录下的 .gradle 隐藏文件夹)：
![IDEA-Preferences](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1607266262468.png)

如果我们更改了 gradle-home 那么对应的下载的 jar 包缓存地址也会被调整，个人觉得这样挺好，毕竟属于我自定义的文件夹下面了。

## 总结

&emsp;&emsp;好了，以上就是总结的在我们打开别的 gradle 项目的时候，如果它采用的 gradle 版本你本地未曾使用过，那么它会自动下载对应的版本保存到你的本地，如果此时你的网络不好下载失败或下载缓慢超时了，你就可以采用上面的两种方式任选其一进行解决。推荐采用第一种方案，因为有的项目可能对 gradle 版本有限定，低于某个版本是无法编译或者启动运行项目的，大家平时多注意就是了。
