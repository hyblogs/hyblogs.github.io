# Gitlab配置邮件发送服务



&emsp;&emsp;上次在自己的服务器上面安装了一个Gitlab服务，折腾半天终于可以正常访问并使用了，但是遇到重置密码的时候突然发现发不出去邮件，气死我了😤。于是便想着把邮件发送服务给修复一下，这样以后用起来也更方便不是。好了，下面我们开始配置Gitlab邮件发送服务吧！

> 注：本次的Gitlab版本为：v_13.10.3，此版本已亲测有效，其他版本仅供参考哦。

## 一、申请电子邮箱，并开通 SMTP 服务

我个人比较喜欢使用 网易邮箱(俗称163邮箱)，下面我将使用网易邮箱进行配置。

1. 申请电子邮箱比较简单，到网易邮箱官网注册一个就好了，注册好了之后登录进入主页，然后找到“设置”按钮，点击打开菜单，找到“POP3/SMTP/IMAP”项，点击进入设置界面，如下：

![设置-SMTP](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1620054119568.png)

2. 打开 SMTP 设置界面，如下：

![SMTP](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1620054223708.png)

3. 点击“开启”按钮，开启SMTP服务，如下：

![SMTP-Start](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1620054422529.png)

注意：开启的时候会校验手机号码，需要发送短信到指定号码进行校验。<font style="color:red;">**手机号校验成功后记得保存授权密码，等会配置文件中需要使用！**</font>

可以将"IMAP/SMTP服务"和"POP3/SMTP服务"都开启。

## 二、修改Gitlab配置文件 gitlab.rb

1. 登录到部署了 Gitlab 服务的服务器上面，切换到Gitlab的配置文件目录下，如下：

```shell
cd /etc/gitlab/
```

2. 编辑 Gitlab 配置文件 gitlab.rb ，如下：

```shell
vim gitlab.rb
```

3. 编辑如下配置信息：

> 执行 `vim gitlab.rb`即可进入文件编辑界面，按键盘上面的 ‘i’ 键进入输入状态，输入完成后按键盘上面的 ‘esc’ 按钮推出编辑，输入 ‘:wq’ (英文冒号哦)即可退出编辑。

```shell
# 启用 smtp 服务
gitlab_rails['smtp_enable'] = true
# 配置 smtp 服务地址，这里需要填写邮件服务里面的“SMTP服务器地址”(如下是网易163邮箱的smtp服务器地址)
gitlab_rails['smtp_address'] = "smtp.163.com"
# 配置 smtp 服务的端口号(默认)
gitlab_rails['smtp_port'] = 465
# 配置发送邮件的电子邮箱名称(即刚才注册的邮箱名称)
gitlab_rails['smtp_user_name'] = "xxx@163.com"
# 配置发送邮件的电子邮箱授权密码，刚才在邮箱里面开启 SMTP 服务的时候弹框提示的那一串【授权密码】（切记：这里不是邮箱的登录密码，是SMTP的授权密码）
gitlab_rails['smtp_password'] = "xxAxxSxxDx"
# 配置 SMTP 服务的域名，和上面的smtp服务器地址一致(如下是网易163邮箱的smtp域名)
gitlab_rails['smtp_domain'] = "smtp.163.com"
# 配置 SMTP 鉴定类别(默认 login 即可)
gitlab_rails['smtp_authentication'] = "login"
# 开启纯文本通信协议扩展
gitlab_rails['smtp_enable_starttls_auto'] = true
# 开启 smtp_tls (传输安全)
gitlab_rails['smtp_tls'] = true

# gitlab 服务邮件发送来源邮箱(即发出邮件的发送方邮箱)，填写刚才注册的邮箱即可
gitlab_rails['gitlab_email_from'] = 'git_xxx@163.com'
```

4. 完成了上面的配置后，进入到 Gitlab 服务的 bin 目录(如果你配置了系统环境变量 PATH 则不需要切换目录)，然后执行如下命令进行更新服务：

```shell
# 重新加载配置信息
gitlab-ctl reconfigure
# 重新启动服务
gitlab-ctl restart
```

5. 验证邮件发送服务是否可用：

&emsp;&emsp;第一步：执行如下命令进入到控制台中：

```shell
gitlab-rails console
```

&emsp;&emsp;第二步：执行发送邮件脚本测试发送邮件是否正常：

```shell
Notify.test_email('xxx@163.com', 'title', 'body').deliver_now
```

&emsp;&emsp;注意：上面的参数‘xxx@163.com’是你要测试接收邮件的邮箱，‘title’是你要测试发送邮件的主题，‘body’是你要测试发送邮件的内容信息。

&emsp;&emsp;**如下是完整的测试操作信息**：

```shell
[root@hy gitlab]# gitlab-rails console
--------------------------------------------------------------------------------
 Ruby:         ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-linux]
 GitLab:       13.10.3 (b1774ad36a9) FOSS
 GitLab Shell: 13.17.0
 PostgreSQL:   12.6
--------------------------------------------------------------------------------
Loading production environment (Rails 6.0.3.4)
irb(main):001:0> Notify.test_email('xxx@163.com', 'title', 'body').deliver_now
Notify#test_email: processed outbound mail in 1.0ms
Delivered mail 609017fc3cd09_2f1158ac570bc@hy.mail (2486.4ms)
Date: Mon, 03 May 2021 23:34:20 +0800
From: GitLab <git_xxx@163.com>
Reply-To: GitLab <noreply@git.xxx.com>
To: xxx@163.com
Message-ID: <609017fc3cd09_2f1158ac570bc@hy.mail>
Subject: title
Mime-Version: 1.0
Content-Type: text/html;
 charset=UTF-8
Content-Transfer-Encoding: 7bit
Auto-Submitted: auto-generated
X-Auto-Response-Suppress: All

<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN" "http://www.w3.org/TR/REC-html40/loose.dtd">
<html><body><p>body</p></body></html>

=> #<Mail::Message:194400, Multipart: false, Headers: <Date: Mon, 03 May 2021 23:34:20 +0800>, <From: GitLab <git_xxx@163.com>>, <Reply-To: GitLab <noreply@git.xxx.com>>, <To: xxx@163.com>, <Message-ID: <609017fc3cd09_2f1158ac570bc@hy.mail>>, <Subject: title>, <Mime-Version: 1.0>, <Content-Type: text/html; charset=UTF-8>, <Content-Transfer-Encoding: 7bit>, <Auto-Submitted: auto-generated>, <X-Auto-Response-Suppress: All>>
irb(main):002:0> 
```

6. 至此，Gitlab 服务的邮件发送服务配置完成了，今后就可以愉快的通过发送电子邮件重置密码啦，哈哈哈哈！

## 注意事项

1. 注意在开启邮箱 SMTP 服务的时候一定要记住授权密码(可以复制到文本中记下来)；
2. 配置 gitlab.rb 的时候 ‘smtp_password’ 一定要填写上面说到的开启 SMTP 服务的时候提示的授权密码，不要填写登录密码；
3. 配置 smtp 服务信息的时候，比如：smtp端口号，smtp服务器地址，smtp域名等信息可以通过邮箱服务商提供的信息进行配置，切记不能随意配置哦。
