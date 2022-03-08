# scp 文件传输下载

&emsp;&emsp;scp 命令是用于通过 SSH 协议安全地将文件复制到远程系统和从远程系统复制文件到本地的命令。使用 SSH 意味着它享有与 SSH 相同级别的数据加密，因此被认为是跨两个远程主机传输文件的安全方式。

## 基本语法

```shell
scp [option] /path/source/file username@server-ip:/path/destination/directory
```

- `/path/source/file` 这是准备复制到远程主机的源文件全路径；
- `username@server-ip:` 这是远程主机的登录账户名和IP地址，注意账户名和IP之间以`@`服务连接，`ip`地址后面紧跟冒号`:`；
- `/path/destination/directory` 这是远程主机接收本次复制的目标目录。

由上面的基本语法可以看出，scp 命令是支持可选参数的，以下便是 scp 命令常用的几个选项(⚠️注意大小写哦)：

```
-C ： 这会在复制过程中压缩文件或目录。
-P ： 如果默认 SSH 端口不是 22，则使用此选项指定 SSH 端口。
-r ： 此选项递归复制目录及其内容。
-p ： 保留文件的访问和修改时间。
```

## 常规使用

### 从远程主机下载文件到本地

**命令语法**：

```shell
scp username@server-ip:/path/source/file /local/destination/directory
```

**示例**：
将远程主机(ip=115.53.103.100)上面`data`目录下的`macdata64bit.dmg`文件下载到我本地(mac环境)的`Downloads`目录下，本地接收的文件名称叫`macdata64bit.dmg`，具体命令如下：

```shell
scp root@115.53.103.100:/data/macdata64bit.dmg ~/Downloads/macdata64bit.dmg
```

注：命令中可以指定接收远程主机文件复制到本地时**本地的文件名称**，如果不指定则默认同名。

### 从远程主机下载文件夹到本地

**命令语法**：

```shell
scp -r username@server-ip:/path/source/directory /local/destination/directory
```

基本上和上面下载文件的方式一致，只是把文件换成文件夹，且需要指定递归复制的选项`-r`即可。

**示例**：
将远程主机(ip=115.53.103.100)上面`data`目录下的`test`文件夹下载到我本地(mac环境)的`Downloads`目录下，本地接收的文件名称叫`test2`，具体命令如下：

```shell
scp -r root@115.53.103.100:/data/test ~/Downloads/test2
```

注：命令中可以指定接收远程主机文件夹复制到本地时**本地的文件夹的名称**，如果不指定则默认同名。

### 上传本地文件到远程主机

**命令语法**：

```shell
scp /local/source/file username@server-ip:/path/destination/directory
```

**示例**：
将本地(mac环境)的文件`~/Downloads/test2/hello.txt` 复制到远程主机(ip=115.53.103.100)上面`data`目录下，接收名称叫`test.txt`，具体命令如下：

```shell
scp ~/Downloads/test2/hello.txt root@115.53.103.100:/data/test.txt
```

注：命令中可以指定远程主机**接收的文件名称**，如果不指定则默认同名。

### 上传本地文件夹到远程主机

**命令语法**：

```shell
scp -r /local/source/directory username@server-ip:/path/destination/directory
```

**示例**：
将本地(mac环境)的文件夹`~/Downloads/test` 复制到远程主机(ip=115.53.103.100)上面`data`目录下，接收的文件夹名称叫`test3`，具体命令如下：

```shell
scp ~/Downloads/test2 root@115.53.103.100:/data/test3
```

注：命令中可以指定远程主机**接收的文件夹名称**，如果不指定则默认同名。

## 总结

&emsp;&emsp;在上文中，我们使用 scp 命令便可以让文件(夹)游走于本地机器和远程主机之间，只是在复制**文件夹**的时候需要注意加上`-r`选项参数哦，否则会复制失败。
&emsp;&emsp;scp 命令是一种在两个远程节点之间传输文件的非常**便捷且安全**的方式，且无需担心攻击者窥探你的数据。
