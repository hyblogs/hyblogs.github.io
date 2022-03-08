# CentOS7安装GitLab私服



&emsp;&emsp;在此，可能有的同学会问：Gitlab 是什么东东呢？大家可能对Github 和 Gitee比较熟悉，但是对 Gitlab 不太了解，没关系，接下来我们一起来了解下什么是 Gitlab，以及它有何特点，我们该如何去使用它等等。

## 什么是 Gitlab ？

&emsp;&emsp;GitLab 是 Ruby 开发的自托管的 Git 项目仓库，可通过 Web 界面进行访问公开的或者私人项目。GitLab 更是开放的 DevOps 平台。

> 对于代码私有性要求较高的用户，可以选择搭建私有的 Git 服务进行自有代码的托管，再也不用担心把自己的私有代码放到别人服务器上面存在的安全隐患问题了。

## Gitlab 的特点

1. 版本控制与协作：源代码管理可在整个软件开发团队中进行协调，共享和协作。跟踪和合并分支机构，审核更改并启用并发工作，以加快软件交付。
2. 远程工作：GitLab是一种协作工具，旨在帮助人们在同一个地点或分布在多个时区更好地协作。
3. 更多的特点在大家使用过程中可以去感受发掘哦... ...

## 开始安装 Gitlab

> 可以参考 [GitLab 官方快速安装文档](https://about.gitlab.com/install/#centos-7) 以及 [GitLab 官方手动安装文档](https://docs.gitlab.com/omnibus/manual_install.html) ，按照步骤进行安装。

### 安装 Gitlab 前注意事项

⚠️注意：GitLab 对服务器配置有一定要求，配置过低将无法正常使用，建议双核8GB以上配置，最低配置要求双核4GB。

### 安装SSH协议

执行如下命令：

```shell
sudo yum install -y curl policycoreutils-python openssh-server perl
```

当出现下面这样，则说明安装成功：

```shell
已安装:
  policycoreutils-python.x86_64 0:2.5-34.el7                                                                                    

作为依赖被安装:
  audit-libs-python.x86_64 0:2.8.5-4.el7        checkpolicy.x86_64 0:2.5-8.el7        libcgroup.x86_64 0:0.41-21.el7          
  libsemanage-python.x86_64 0:2.5-14.el7        python-IPy.noarch 0:0.75-6.el7        setools-libs.x86_64 0:3.3.8-4.el7       

完毕！
```

### 设置SSH服务开机自启动

执行如下命令：

```shell
sudo systemctl enable sshd
```

设置成功后控制台不会返回任何说明。

### 启动SSH服务

执行如下命令：

```shell
sudo systemctl start sshd
```

设置成功后控制台不会返回任何说明。

### 添加 HTTP 服务到 firewalld

执行如下命令：

```shell
sudo firewall-cmd --permanent --add-service=http
```

当返回了 `FirewallD is not running` 则说明防火墙未启用。

启动防火墙，执行如下命令：

```shell
service firewalld start
```

启动时会输出 `Redirecting to /bin/systemctl start firewalld.service`，属于正常现象。

然后，重新执行上面的添加 Http 服务到 firewalld，此时会返回：

```shell
success
```

### 添加 HTTPS 服务到 firewalld

```shell
sudo firewall-cmd --permanent --add-service=https
```

添加成功会返回 `success` 字样。

### 重启防火墙

```shell
sudo systemctl reload firewalld
```

执行重启命令不会有任何返回字样。

### 安装Postfix以发送通知邮件

> 接下来，将安装 Postfix 以发送通知电子邮件。如果要使用其他解决方案发送电子邮件，请跳过此步骤并在安装 GitLab 之后配置外部 SMTP 服务器。

执行如下命令安装Postfix以发送通知邮件，并进行相关设置：

```shell
#安装Postfix以发送通知邮件：
sudo yum install postfix
#将postfix服务设置成开机自启动
sudo systemctl enable postfix
#启动postfix
sudo systemctl start postfix
```

### 添加GitLab软件包存储库

执行如下命令：

```shell
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash
```

出现如下字样表示安装完毕：

```shell
......
完毕！
Generating yum cache for gitlab_gitlab-ee...
导入 GPG key 0x51312F3F:
 用户ID     : "GitLab B.V. (package repository signing key) <packages@gitlab.com>"
 指纹       : f640 3f65 44a3 8863 daa0 b6e0 3f01 618a 5131 2f3e
 来自       : https://packages.gitlab.com/gitlab/gitlab-ee/gpgkey

......

The repository is setup! You can now install packages.
```

### 下载安装包

切换到我们的下载目录下，执行如下命令进行下载即可：

```shell
#切换到我的下载目录（你们随意）
cd /tmp/download/
#执行如下命令开始下载（安装包比较大，约 850MB 左右，不过在云服务器上面下载还是挺快的）
wget --content-disposition https://packages.gitlab.com/gitlab/gitlab-ce/packages/el/7/gitlab-ce-13.10.3-ce.0.el7.x86_64.rpm/download.rpm
```

### 开始安装

执行下面的命令安装 GitLab 安装包：

```shell
rpm -Uvh gitlab-ce-13.10.3-ce.0.el7.x86_64.rpm
```

执行后会出现如下界面进行安装完成：
![install-gitlab.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1618757356644.png)

### 配置 GitLab (重要步骤)

切换到 `/etc/gitlab/` 目录下：

```shell
cd /etc/gitlab/
```

会发现此目录下有一个配置文件 `gitlab.rb`，我们需要编辑此配置文件进行相关的配置。

打开编辑 `gitlab.rb` 文件，找到如下参数进行调整：

```shell
#打开文件编辑：
vim gitlab.rb

#找到如下参数后可以将域名地址修改为实际的访问地址，可以是 IP+Port 或者域名地址皆可哦！建议先使用IP+Port的方式进行测试可行后切换到域名方式。
external_url 'http://gitlab.example.com'

#找到如下参数后将前面的`#`移除并修改值为`Asia/Shanghai`，如下第一行是调整前的样子，第二行是调整后的样子：
# gitlab_rails['time_zone'] = 'UTC'
gitlab_rails['time_zone'] = 'Asia/Shanghai'

#设置超时时间
unicorn['worker_timeout'] = 60

#设置进程数（默认为CPU核心数+1）
unicorn['worker_processes'] = 2

#设置监听地址及端口（⚠️注意：此端口要和上面设置的URL的端口一致）
unicorn['listen'] = '127.0.0.1'
unicorn['port'] = 8087

#设置内存控制：128MB ～ 256MB （可依据自己机器配置进行设置哦）
unicorn['worker_memory_limit_min'] = "128 * 1 << 20"
unicorn['worker_memory_limit_max'] = "256 * 1 << 20"

#设置sidekiq并发数控制
sidekiq['max_concurrency'] = 16
sidekiq['min_concurrency'] = 2

#设置数据库缓存为256MB
postgresql['shared_buffers'] = "256MB"
#设置数据库最大并发数为8
postgresql['max_worker_processes'] = 8
```

### 配置防火墙

由于前面我们打开了防火墙，所以现在需要针对上面配置的访问端口进行打开：

```shell
#开放8080端口
firewall-cmd --zone=public --add-port=8080/tcp --permanent   
#重启防火墙
firewall-cmd --reload
#查看端口号是否开启
firewall-cmd --query-port=8080/tcp
```

### 重置 GitLab 并启动

```shell
#重置Gitlab
gitlab-ctl reconfigure
#启动Gitlab
gitlab-ctl restart
```

重置时会显示如下字样：

```shell
......
Notes:
It seems you haven't specified an initial root password while configuring the GitLab instance.
On your first visit to  your GitLab instance, you will be presented with a screen to set a
password for the default admin account with username `root`.

gitlab Reconfigured!
```

记住打印出来的 password 哦！！！

启动的时候会出现如下字样：

```shell
ok: run: alertmanager: (pid 15449) 0s
ok: run: gitaly: (pid 15461) 1s
ok: run: gitlab-exporter: (pid 15484) 0s
ok: run: gitlab-workhorse: (pid 15486) 1s
ok: run: grafana: (pid 15494) 0s
ok: run: logrotate: (pid 15508) 1s
ok: run: nginx: (pid 15514) 0s
ok: run: node-exporter: (pid 15558) 0s
ok: run: postgres-exporter: (pid 15602) 1s
ok: run: postgresql: (pid 15611) 0s
ok: run: prometheus: (pid 15621) 1s
ok: run: puma: (pid 15631) 0s
ok: run: redis: (pid 15639) 1s
ok: run: redis-exporter: (pid 15645) 0s
ok: run: sidekiq: (pid 15652) 0s
```

启动成功，我们可以通过配置好的URL进行访问试试了。

首次启动后访问可能会出现 **502 - Whoops, GitLab is taking too much time to respond.** 的情况，如下：
![GitLab-502.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1618931414812.png)

这是正常情况，等服务端启动完成后刷新即可进入重置密码的界面了。
我这里首次启动安装也出现了 502 的情况，后来检查发现是 `unicorn` 的端口配置和 `external_url` 不一致导致的，调整后重启服务大概过了十多二十秒就可以正常访问了。

正常访问后重置完密码使用 root 账户登录后效果如下：
![gitlab.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1618931576857.png)

## 其他

### 访问出现 502 的情况说明

&emsp;&emsp;启动完成后出现 **502 - Whoops, GitLab is taking too much time to respond.** 的情况有如下可能性：

#### 1、配置文件设置不合法

> 可能是 `gitlab.rb` 配置文件设置项设置不合法，比如下面的情况：

- `unicorn` 的端口配置和 `external_url` 不一致；
- 配置的端口（默认8080）已经被占用。<检查端口是否被占用>

#### 2、服务器资源不足

> 通常是服务器配置过低，比如下面的情况：

- 机器配置较低，达不到2核CPU+4GB内存配置的最低要求；
- 使用过程中内存占用过大，导致服务器崩溃。<可尝试添加 Swap 分区>

### 汉化说明

安装成功后访问登录进入主界面，点击右上角的头像并找到 preferences 项进行本土化设置，如下图找到 preferences：

<img src="https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1619879400752.png" style="zoom:25%;" alt="Preferences"/>
<p>

更改本土化设置，找到‘Localization’，在‘Language’处选择“简体中文”，如下图：

![Localization.Language](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1619879611140.png)

选择好之后，点击“Save changes”进行保存，然后筛选界面就可以看到中文版面啦，如下效果：

![中文简体](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1619879945557.png)
