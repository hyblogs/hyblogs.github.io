# 解决 MacBook Touch Bar 上面 esc 按钮图标消失不见的问题

&emsp;&emsp;今天在用 mbp 上的终端（Terminal）通过 ssh 连接远程服务器，然后用 vim 编辑器修改 nginx 配置文件，修改好之后顺手点击了一下 TouchBar 的左上角，发现「esc」按钮图标不在了，然后经过一番查找，终于把 TouchBar 上面的 「esc」 按钮找回来了。 

接下来咋们通过本文分享下出现这个问题的解决办法（重新让 「esc」按钮 回到 Touch Bar 上来）。

> MacBook Pro 版本：macOS Catalina Version 10.15.5

解决办法也很简单，直接打开终端（Terminal），杀死 Touch Bar 服务即可，命令如下：

```shell
sudo pkill TouchBarServer
```

大家可以复制上面的命令粘贴到终端直接运行，之后输入密码确认执行，等待 3-5 秒，esc 图标就重新回来了。

如下图示：
![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1609672163633.png)
输入当前登录账户的密码，然后回车即可！
