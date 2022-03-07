  不知道咋滴，用着Gitlab突然给我退出到了登录界面，可能是登录会话过时了吧，又或者是我更改了用户名吧，因为用户名和代码路径强相关，我为了让代码路径看着更顺眼，所以更改了用户名。但是退出到登录界面后我再也无法登录进入主页了，于是想着更改管理员账户密码，折腾了半天终于改好了～

> 注：本次的Gitlab版本为：v_13.10.3，此版本已亲测有效，其他版本仅供参考哦。

## 方式一：发送邮件更改管理员密码

1. 在登录界面，点击“++Forgot your password?++”，如下所示：

![login](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1620048033145.png)

2. 点击后切换到了如下界面，输入你绑定的电子邮件，然后点击“Reset password”按钮就会由Gitlab服务器发送的重置密码的邮件到你的邮箱了，登录你的邮箱点击重置密码的链接进行密码重置吧。

![sendemail](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1620048114062.png)

如下就是电子邮箱收到的重置密码的邮件信息：

![email](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1620048880310.png)

3. 点击邮件中的“Reset password”按钮即可跳转到重置密码的界面了，如下所示：

![changepassword](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1620048948678.png)

4. 此时，你只需要输入你需要设置的新密码，然后重复输入一次新密码，再点击“Change your password”按钮提交即可修改密码成功！

重置密码成功后你就可以使用新设置的密码登录进入主页啦！ 如果你不知道你的登录账户，更不知道你设置的电子邮件地址，或者你的Gitlab服务暂时还无法发送电子邮件，又或者你根本收不到电子邮件的话，你可以看看下面的方式二哦！

## 方式二：进入Gitlab控制台更改管理员密码

前提条件：需要保证Gitlab、Redis同时处于启动状态。可以运行`gitlab-ctl start`或者`gitlab-ctl restart`命令进行启动或者重启。

1. 切换到Gitlab的bin目录下，如果你配置了系统变量PATH则可以不用切换。(如果你在服务器上安装Gitlab时使用的指定服务器用户，则需要你切换用户到当初安装Gitlab的账户上去)
   准备就绪后，我们可以执行如下命令进入Gitlab控制的了，命令：

```shell
gitlab-rails console -e production
# 低版本可以尝试使用下面一句命令：
gitlab-rails console production
```

如果使用上述命令入法进入Gitlab控制台，建议前往Gitlab官网查询进入Gitlab控制台的方式进行进入哦。

进入控制台后如下：

```shell
[root@hy ~]# gitlab-rails console -e production
--------------------------------------------------------------------------------
 Ruby:         ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-linux]
 GitLab:       13.10.3 (b1774ad36a9) FOSS
 GitLab Shell: 13.17.0
 PostgreSQL:   12.6
--------------------------------------------------------------------------------
Loading production environment (Rails 6.0.3.4)
irb(main):001:0> 
```

2. 查看所有用户，在Gitlab控制台输入`User.all`即可看到所有的用户，如下：

```shell
[root@hy ~]# gitlab-rails console -e production
--------------------------------------------------------------------------------
 Ruby:         ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-linux]
 GitLab:       13.10.3 (b1774ad36a9) FOSS
 GitLab Shell: 13.17.0
 PostgreSQL:   12.6
--------------------------------------------------------------------------------
Loading production environment (Rails 6.0.3.4)
irb(main):001:0> User.all
=> #<ActiveRecord::Relation [#<User id:2 @alert-bot>, #<User id:3 @support-bot>, #<User id:1 @root>, #<User id:4 @code>, #<User id:5 @hy>]>
irb(main):002:0> 
```

3. 找到自己需要重置的用户id号，管理员账户通常id为1，在Gitlab控制台执行如下命令即可获取到用户(如下：定位到id=1的用户)：

```shell
[root@hy ~]# gitlab-rails console -e production
--------------------------------------------------------------------------------
 Ruby:         ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-linux]
 GitLab:       13.10.3 (b1774ad36a9) FOSS
 GitLab Shell: 13.17.0
 PostgreSQL:   12.6
--------------------------------------------------------------------------------
Loading production environment (Rails 6.0.3.4)
irb(main):001:0> User.all
=> #<ActiveRecord::Relation [#<User id:2 @alert-bot>, #<User id:3 @support-bot>, #<User id:1 @root>, #<User id:4 @code>, #<User id:5 @hy>]>
irb(main):002:0> user=User.where(id:1).first
=> #<User id:1 @root>
rb(main):003:0> 
```

4. 在Gitlab控制台执行如下命令修改密码：

```shell
[root@hy ~]# gitlab-rails console -e production
--------------------------------------------------------------------------------
 Ruby:         ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-linux]
 GitLab:       13.10.3 (b1774ad36a9) FOSS
 GitLab Shell: 13.17.0
 PostgreSQL:   12.6
--------------------------------------------------------------------------------
Loading production environment (Rails 6.0.3.4)
irb(main):001:0> User.all
=> #<ActiveRecord::Relation [#<User id:2 @alert-bot>, #<User id:3 @support-bot>, #<User id:1 @root>, #<User id:4 @code>, #<User id:5 @hy>]>
irb(main):002:0> user=User.where(id:1).first
=> #<User id:1 @root>
irb(main):003:0> user.password='hy123456'
=> "hy123456"
irb(main):004:0> user.password_confirmation='hy123456'
=> "hy123456"
irb(main):005:0> 
```

执行`user.password='hy123456'`是设置密码，然后执行`user.password_confirmation='hy123456'`是确认密码，两次密码需要设置成一致的。

注意：密码不能设置过于简单，最好先不要设置特殊字符，会报错，可能需要转义！

5. 在Gitlab控制之下下面的命令保存密码：

```shell
[root@hy ~]# gitlab-rails console -e production
--------------------------------------------------------------------------------
 Ruby:         ruby 2.7.2p137 (2020-10-01 revision 5445e04352) [x86_64-linux]
 GitLab:       13.10.3 (b1774ad36a9) FOSS
 GitLab Shell: 13.17.0
 PostgreSQL:   12.6
--------------------------------------------------------------------------------
Loading production environment (Rails 6.0.3.4)
irb(main):001:0> User.all
=> #<ActiveRecord::Relation [#<User id:2 @alert-bot>, #<User id:3 @support-bot>, #<User id:1 @root>, #<User id:4 @code>, #<User id:5 @hy>]>
irb(main):002:0> user=User.where(id:1).first
=> #<User id:1 @root>
irb(main):003:0> user.password='hy123456'
=> "hy123456"
irb(main):004:0> user.password_confirmation='hy123456'
=> "hy123456"
irb(main):005:0> user.save!
Enqueued ActionMailer::MailDeliveryJob (Job ID: 2222b8da-6863-4909-8e35-c01ee88c9dd5) to Sidekiq(mailers) with arguments: "DeviseMailer", "password_change", "deliver_now", {:args=>[#<GlobalID:0x00007f85c6d40ac8 @uri=#<URI::GID gid://gitlab/User/5>>]}
=> true
irb(main):006:0> 
```

执行`user.save!`就是保存上面我们给用户设置的密码信息，**切记后面有个英文感叹号(!)**。
如上，看到Gitlab控制台保存密码成功后会打印出 `...true`等一堆信息，表示设置成功了！上面可以看到触发了发送邮件的Job，它会发送一封电子邮件到刚才重置密码的账户绑定的邮箱中，内容大致如下：

![ResetPassword](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1620053102105.png)

6. 在Gitlab控制台执行`exit`或者`quit`命令即可退出控制台。

7. 通过Gitlab命令重置完密码后，即可访问Gitlab登陆界面进行登录啦，账户名就是上面通过`User.all`获取到的账户列表里面的用户，密码就是刚才重新设置的密码。登录进入主页如下：

![home](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1620052788735.png)

## 其他

如果你知道用户的电子邮件，想获取用户信息，可以通过Gitlab控制台执行命令进行获取哦，如下：

```shell
user=User.where(email:'jenkins@domian.com').first
```
