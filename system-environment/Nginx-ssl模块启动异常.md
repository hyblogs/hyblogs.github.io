# Nginx: [emerg] the "ssl" parameter requires ngx_http_ssl_module in /usr/local/nginx/conf/nginx.conf



&emsp;&emsp;出现上面标题中的问题是因为我当初在安装Nginx的时候没有安装SSL模块，但是现在我在Nginx配置文件(nginx.cnf)中配置了SSL的相关配置信息。当我再次启动Nginx的时候就提示我`nginx: [emerg] the "ssl" parameter requires ngx_http_ssl_module in /usr/local/nginx/conf/nginx.conf:111` 了。下面我将我的解决方案给记录了下来，供大家参考哦！

### 解决问题

> 首先需要申明一下，我安装Nginx的目录是 `/usr/local/nginx/` ，当时安装的时候是通过源码安装的，所以源码也在这个目录下面。接下来我们会利用源码加载 SSL 模块重新编译，如果你没有源码则可以选择重新安装新版 Nginx 进行解决。

先来看看 Nginx 的版本吧：

```shell
[root@xxx ~]# /usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.16.1
...
configure arguments: 
```

可以看到，我这里的`configure arguments: `后面没有任何配置参数。

我们现在切换到源码包目录下面，增加SSL配置项进行重新编译：

```shell
# 切换到Nginx源码目录下面：
[root@xxx ~]# cd /usr/local/nginx/
# 新增SSL模块进行重新配置：
[root@xxx nginx]# ./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
# 重新编译：
[root@xxx nginx]# make
```

<font style="color:red;">⚠️注意：这里不在需要执行`make install`命令了，如果执行了会覆盖之前的安装，会有一定的风险哦！</font>

然后备份原有已安装好的 nginx :

```shell
[root@xxx nginx]# cp /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.bak
```

停止现有的 Nginx 服务：

```shell
[root@xxx ~]# /usr/local/nginx/sbin/nginx -s stop
```

然后将刚刚编译好的 nginx 覆盖掉原有的 nginx，刚才重新编译好的nginx在`/usr/local/nginx/objs/` 下面。

```shell
[root@xxx ~]# cp ./objs/nginx /usr/local/nginx/sbin/
# 执行的时候会有个询问过程，询问你是否选择覆盖？此时输入 `yes` 然后按`Enter`就行了。
```

上述执行成功后，再执行查看 Nginx 版本就可以看到`configure arguments: `后面有新的参数了，如下：

```shell
[root@xxx nginx]# /usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.16.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
```

通过上面控制台返回的信息可见新增的SSL模块成功了。

重新启动Nginx就可以啦：

```shell
# 启动Nginx服务
[root@xxx nginx]# /usr/local/nginx/sbin/nginx
```

### 其他

&emsp;&emsp;在Linux系统下安装 Nginx 服务及其配置 SSL 证书实现 HTTPS 访问可以参考下面的文档哦：

&emsp;&emsp;[Centos7安装Nginx并配置](https://www.hyblogs.com/archives/centos7%E5%AE%89%E8%A3%85nginx%E5%B9%B6%E9%85%8D%E7%BD%AE)

&emsp;&emsp;[Nginx 配置SSL证书，实现HTTPS访问](https://www.hyblogs.com/archives/nginx%E7%9A%84%E7%AE%80%E5%8D%95%E5%BA%94%E7%94%A8)
