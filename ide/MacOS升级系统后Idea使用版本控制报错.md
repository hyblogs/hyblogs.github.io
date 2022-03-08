# Mac OS 升级系统后 Idea 集成的SVN使用报错：The subversion command line tools are no longer provided by Xcode.



&emsp;&emsp;昨晚上升级了 XCode ,升级完成后看系统版本有更新，版本 10.15.5，推送过来已经有一段时间了，只是迟迟未确认进行更新。昨晚一时兴起便立即更新了，提示需要几分钟时间，但是我等了很长时间(实际多久我就不知道了，等了十多分钟就躺下睡了，哈哈哈)，今早起来发现系统已经更新完成了，变试了试平时用的 IDE 工具 idea 能否正常使用，当我使用 SVN 拉取最新代码的时候发现不对劲，SVN 提示异常了，然后便有了这篇文档。嘿嘿😁

### Mac OS 升级后使用 SVN 提示异常

 &emsp;&emsp;使用 SVN 的时候会提示如下异常：

```
svn: error: The subversion command line tools are no longer provided by Xcode.
```

### 解决方案一

&emsp;&emsp;本以为很简单，因为之前遇到过，按照之前的方式，卸载掉 CommandLineTools 然后重新安装就好了，因为之前系统版本是 10.15.1 所以没什么问题。大致步骤如下：

#### 1、卸载掉 CommandLineTools

```shell
sudo rm -rf /Library/Developer/CommandLineTools
```

卸载的时候会提示输入密码，这时候输入本机当前账户的密码就可以了，输入的密码不会直接展示出来，是隐藏状态的，不要以为没有输入成功～

#### 2、重新安装对应的 XCode 组件

```shell
xcode-select --install
```

安装完成后，本应该在 Idea 中就能正常使用 SVN 了。但是实际测试了还是报出上面的异常，纹丝不动，老法子不管用了？其实不然，因为系统更新了，这个方法失效了，但是对于 10.15.4 及之前的版本是有效的，只是我系统更新到了 10.15.5 所以这个方法暂时不管用了，需要切换其他方案了。

### 解决方案二

&emsp;&emsp;使用上面的方案解决不了异常，是因为系统大于 10.15.4 时，用上面的方案不管用了，需要用下面的新方案。

#### 1、下载安装 homebrew，国内镜像库：

```shell
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

可以直接复制执行即可，执行的过程中有中文描述及提示输入 y/N ,输入 y 即可。
如下图开始安装：
![Homebrew](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1592891369481.png)
我这里选择的下载源是：中科大，感觉还是比较稳定的，你们也可以尝试选择清华大学等其他的下载源，如果不稳定或下载太慢，可以 ctrl + C 结束掉后重新执行后选择前面两个比较稳定的下载源。

#### 2、安装 Subversion

```shell
brew install svn
```

安装中途如果遇到下载卡死的情况，可以 ctrl + C 结束掉后重新执行。

### 总结

&emsp;&emsp;如果遇到上述异常错误信息，记得先瞅瞅当前的系统版本号，如果当前系统版本号小于 10.15.5 则可以尝试选择方案一进行解决；如果当前系统版本号大于 10.15.4 则需要采用方案二进行解决，其中重要环节是安装 Homebrew ，默认使用国外下载源很容易卡死安装失败，除非你能翻墙，所以推荐大家使用国内的下载源，使用上面的方案执行选择‘中科大下载源’或者‘清华大学下载源’均可，相对比较稳定，如果卡死了可以重新进行或切换下载源试试。大家可以看看这篇文章，对与安装 Homebrew 失败的很多情况都有说明解释，地址：[Homebrew国内如何自动安装（国内地址）](https://zhuanlan.zhihu.com/p/111014448)。

&emsp;&emsp;好了，如果你也遇到了上面的问题，欢迎参考进行解决哦～
