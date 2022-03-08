# Nginx 配置 SSL 证书，实现 HTTPS 访问



&emsp;&emsp;咱自己搞一个个人博客，平时没事儿记录一下学习过程，其实蛮好的。至于建站肯定缺少不了域名解析和服务器配置，今儿个咋们记录一下如何在Linux服务器上面配置Nginx转发来实现咋们的网站能域名访问吧。下面我将用本站的域名配置信息进行记录。

&emsp;&emsp;本站的域名是`hyblogs.com`，由于个人更喜欢`https`访问，所以去阿里云申请了免费的`SSL`证书，感兴趣的小伙伴们可以去试试。申请过程相对简单，有效期一年。申请到`SSL`证书后咱下载下来上传到服务器指定目录(我这里使用的是/data/目录下面)，然后将压缩包解压到指定目录(我这里就解压到当前目录了)，然后在Nginx的配置文件中配置对应的端口(https对应的端口是443)监听，然后重点是配置咋们的`ssl_certificate`和`ssl_certificate_key`对应的分别是咱解压后路径下的`*.pem`和`.key`文件，然后还可以配置缓存以及超时时间等等，下面有对应每项的注释说明可以看看：

```
server { # 每个server代表的是一个服务实例的配置

    listen 443 ssl; # 这个服务实例监听的端口

    server_name hyblogs.com www.hyblogs.com; # 此server块对外提供的虚拟主机名称，可以理解为域名(此值可以有1个或多个，由空格分隔，默认第1个为主要名称。也可以使用通配符，正则表达式等方式)

    ssl_certificate      /data/cert/hyblogs.com.pem; # 配置证书crt（pem）保存路径
    ssl_certificate_key  /data/cert/hyblogs.com.key; # 配置证书key保存路径

    ssl_session_cache    shared:SSL:1m; # 配置加密session重用
    ssl_session_timeout  5m; # session超时时间

    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4; # 指定可用的密码算法，必须是openssl库中指定的方式
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;   #使用该协议进行配置。
    ssl_prefer_server_ciphers  on; # 在使用sslv3或tls协议时，指定服务器密码优先于客户端密码

    client_max_body_size 1024m; # #允许客户端请求的最大单文件字节数

    location / {
        proxy_set_header HOST $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_pass http://127.0.0.1:8090/; # 设置代理，将请求转发到对应的地址(http://ip:port)
    }
}

server {

    listen 80;

    server_name hyblogs.com www.hyblogs.com;

    rewrite ^(.*)$ https://$host$1 permanent;
}
```

因为如果我们需要在浏览器中默认访问域名`hyblogs.com`就直接访问到咱的博客网站的话，还需要在域名解析中配置通配符解析项，如：`*.hyblogs.com`只要符合这个规则的访问都会被定义到咱配置的IPV4地址(你也可以设置其他落地地址项)，如下所示：

![*.hyblogs.com](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1600007882075.png)

配置完这个咋就可以直接访问域名定位到咱的服务器80端口了，但是我想要的是访问到https的443端口啊。别急，咱可以在Nginx上面配置一个转发，将80端口的访问转发到443端口。非常简单，按照上面的示例添加一个`server`配置对应的监听端口，重点是配置一下转发(`rewrite ^(.*)$ https://$host$1 permanent;
}`)就OK了。是不是很简单啊！
