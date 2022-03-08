# Centos7 安装 Nginx 并配置

&emsp;&emsp;当我们将服务部署到服务器上面之后，通常情况下通过服务器公网 IP 地址加上指定的端口号就可以访问到我们发布的服务了。但是这种访问方式需要我们记住服务器的公网 IP 地址以及服务的端口，不是很方便。而且当一个服务部署到多台服务器上面后如何才能方便的通过一个统一的地址进行访问呢，显然需要借助我们常说的反向代理 WEB 服务器了，接下来我们一起在 Centos7 系统上安装常见的反向代理 WEB 服务器 Nginx，并对它进行相应配置，实现上面我们说的场景吧。

> 接下来的演示环境是 Centos 7.6，是全新的未安装任何其他软件环境的系统，安装的 Nginx 版本为 1.18.0。

注⚠️：本文适合小白用户，大神请绕行哦！

## 1、获取安装包

&emsp;&emsp;访问 Nginx 官方下载地址：http://nginx.org/en/download.html，选择稳定版进行下载，下载的时候选择符合 Linux 安装的版本，如下：
![download-nginx.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1618654632424.png)

> 温馨提示：如何你在自己本地环境下载，则需要将安装包通过 ftp 工具传递到服务器上面去。

&emsp;&emsp;大家可以选择复制链接🔗地址，然后到服务器控制台通过 wget 命令进行下载，注意在合适的目录下进行下载哦，避免下载好后找不到安装包放到哪个目录下面了。命令如下：

```shell
# 我将下载包放在了 /tmp/download 目录下，先创建文件夹：
[root@xxx-linux download]# mkdir /tmp/download
# 切换到下载目录下进行安装包的下载：
[root@xxx-linux download]# cd /tmp/download/
# 执行安装包的下载：
[root@xxx-linux download]# wget http://nginx.org/download/nginx-1.18.0.tar.gz
```

到此，安装包准备完毕！

## 2、解压编译安装

### 解压缩包

将我们刚才下载下来的安装包压缩包进行解压处理，可执行如下命令:

```shell
[root@xxx-linux download]# tar -zxvf nginx-1.18.0.tar.gz
```

命令执行完成后便将压缩包解压完毕，在当前目录下会多出来一个和压缩包同名的文件夹，如下所示：

```shell
[root@xxx-linux download]# ll -a
总用量 1028
drwxr-xr-x   3 root root    4096 4月  17 18:25 .
drwxrwxrwt. 10 root root    4096 4月  17 18:18 ..
drwxr-xr-x   8 1001 1001    4096 4月  21 2020 nginx-1.18.0
-rw-r--r--   1 root root 1039530 4月  21 2020 nginx-1.18.0.tar.gz
```

### 编译前配置检查

执行如下命令安装编译工具及库文件：

```shell
[root@xxx-linux download]# yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel
```

由于我需要将 Nginx 安装到指定目录，我的安装目录是 `/usr/local/webserver`，所以我需要保证我的安装目录已存在：

```shell
[root@xxx-linux download]# mkdir /usr/local/webserver
```

切换到解压后的 Nginx 安装包目录下：

```shell
[root@xxx-linux download]# cd nginx-1.18.0
```

执行如下命令进行编译前依赖组件的检查：

```shell
[root@xxx-linux nginx-1.18.0]# ./configure
```

执行此命令检查后会输出依赖的组件是否就绪，如果所有组件名称后面都是 ` ... found` 则表示依赖的组件在系统中已安装，如果出现了 `... not found` 则表示此组件在系统中未找到，我们可以进行手动安装，然后再进行检命令。

我这里检查后有部分组件未找到，如下：

```shell
......
checking for PCRE library ... not found
checking for PCRE library in /usr/local/ ... not found
checking for PCRE library in /usr/include/pcre/ ... not found
checking for PCRE library in /usr/pkg/ ... not found
checking for PCRE library in /opt/local/ ... not found

./configure: error: the HTTP rewrite module requires the PCRE library.
You can either disable the module by using --without-http_rewrite_module
option, or install the PCRE library into the system, or build the PCRE library
statically from the source with nginx by using --with-pcre=<path> option.
```

根据检查后的提示是说让我在系统中安装 PCRE 组件，接下来我们可以执行下面的命令进行安装：

```shell
# 切换到自定义的安装包存储目录：
[root@xxx-linux nginx-1.18.0]# cd /usr/local/src/
# 下载要安装的 PCRE 安装包文件：
[root@xxx-linux src]# wget http://downloads.sourceforge.net/project/pcre/pcre/8.35/pcre-8.35.tar.gz
# 解压安装包：
[root@xxx-linux src]# tar zxvf pcre-8.35.tar.gz
# 进入安装包目录
[root@xxx-linux src]# cd pcre-8.35
# 编译安装 
[root@xxx-linux pcre-8.35]# ./configure
[root@xxx-linux pcre-8.35]# make && make install
# 查看pcre版本
[root@xxx-linux pcre-8.35]# pcre-config --version
8.35
```

> 建议：如果系统里面已有 PCRE 组件可以直接使用，但是最好找到对应的目录位置为下一步作准备，如果系统里面没有 PCRE 组件则必须进行上述安装。

### 编译安装

此时我们可以执行下面的命令行进行编译并安装，如下：

```shell
# 切换目录到 Nginx 解压后的安装包目录下面：
[root@xxx-linux pcre-8.35]# cd /tmp/download/nginx-1.18.0/
# 配置检查 （指定安装后的目录为：/usr/local/webserver/nginx，可自由更改）
[root@xxx-linux nginx-1.18.0]# ./configure --prefix=/usr/local/webserver/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre=/usr/local/src/pcre-8.35
# 执行编译
[root@xxx-linux nginx-1.18.0]# make
# 安装
[root@xxx-linux nginx-1.18.0]# make install
```

### 查看版本

执行下面的命令查看刚才安装的 Nginx 版本：

```shell
[root@xxx-linux nginx-1.18.0]# /usr/local/webserver/nginx/sbin/nginx -v
nginx version: nginx/1.18.0
```

执行运行成功后则代表安装成功了，接下来我们将进行 Nginx 配置。

## 3、配置 Nginx

切换到 Nginx 配置目录下，如下：

```shell
[root@xxx-linux nginx-1.18.0]# cd /usr/local/webserver/nginx/conf/
```

编辑 Nginx 配置文件 nginx.conf ，可参考如下配置结合具体实际情况进行配置：

```shell
#user  nobody;
worker_processes  2;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

pid        /usr/local/webserver/nginx/nginx.pid;

worker_rlimit_nofile 65535;

events {
    use epoll;
    worker_connections  65535;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /usr/local/webserver/nginx/logs/access.log  main;

    sendfile        on;
    tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    gzip  on;

    server {
        listen       80;
        server_name  localhost;

        client_max_body_size 1024m;
        client_body_buffer_size 1024m;
        sendfile on;
        keepalive_timeout 1800;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

> 注意：在配置文件中，注释一行代码请在对应那一行代码前加上 `#` 符号。
> 配置完成后检查配置文件 nginx.conf 的正确性，执行如下命令：

```shell
[root@xxx-linux conf]# /usr/local/webserver/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/webserver/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/webserver/nginx/conf/nginx.conf test is successful
```

如上所示，执行检查命令后返回成功则表示配置信息正确。

## 4、启动 Nginx

Nginx 启动命令如下：

```shell
[root@xxx-linux conf]# /usr/local/webserver/nginx/sbin/nginx
```

执行后无任何返回，表示成功执行无异常！

现在我们可以通过浏览器访问我们安装了 Nginx 服务的机器了，如本机访问：127.0.0.1，会出现如下界面：
![nginx-welcome.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1618660058615.png)

表示我们安装并启动成功！！

## 其他

在我们日常使用中，可能还会使用到如下命令：

```shell
# 重新载入配置文件
[root@xxx-linux conf]# /usr/local/webserver/nginx/sbin/nginx -s reload
# 重启 Nginx
[root@xxx-linux conf]# /usr/local/webserver/nginx/sbin/nginx -s reopen
# 停止 Nginx
[root@xxx-linux conf]# /usr/local/webserver/nginx/sbin/nginx -s stop
```

一般执行命令的时候喜欢直接使用 nginx ，而不是加上一连串的目录，这时候我们可以将 nginx 配置到 PATH 环境变量中，下面我们就一起来将 Nginx 配置到系统的环境变量中，方便我们日后的操作。

### 配置 Nginx 环境变量

切换到 etc 目录，如下：

```shell
[root@xxx-linux conf]# cd /etc/
```

编辑配置文件 profile，如下：

```shell
[root@xxx-linux etc]# vim profile
```

在文件的末尾加上如下字符：

```shell
export NGINX_HOME=/usr/local/webserver/nginx/
export PATH=$PATH:$NGINX_HOME/bin:$NGINX_HOME/sbin:.
```

执行如下命令让配置文件立即生效：

```shell
[root@xxx-linux etc]# source profile
```

执行如下命令查看刚才配置环境变量：

```shell
[root@xxx-linux etc]# echo $NGINX_HOME
/usr/local/webserver/nginx/
[root@xxx-linux etc]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin:/usr/local/webserver/nginx//bin:.:/usr/local/webserver/nginx//bin:/usr/local/webserver/nginx//sbin:.
```

可见，环境变量配置成功！
测试直接使用 nginx 进行命令访问：

```shell
[root@xxx-linux etc]# nginx -v
nginx version: nginx/1.18.0
```

由上面得到的结果可以看到，直接使用 nginx 处理命令成功！以后使用起来将更加方便了吧，哈哈！

### 后续

如果需要在 Nginx 中配置 SSL 证书，让域名访问更安全，则可以参考 [Nginx 配置SSL证书，实现HTTPS访问](https://www.hyblogs.com/archives/nginx%E7%9A%84%E7%AE%80%E5%8D%95%E5%BA%94%E7%94%A8) 进行配置即可哦！
