# Linux 安装 Redis 并配置

> &emsp;&emsp;在日常开发过程中，难免会用到 Redis 作为数据缓存。一般我们都是在运维同学安装好的环境下直接连接上就可以使用了，殊不知这 Redis 是如何被安装部署到对应环境的服务器上的。而且一般的服务器环境并不像我们平时使用的界面化系统，而是仅有控制台(Console)的 Linux 系统。要在 Linux 环境下安装应用，咱得熟悉常用的 Shell 命令啊，虽然比较简单，但是还是有一定难度的哦，下面我们一起来试试吧～
> 
> &emsp;&emsp;本次演练使用的系统环境是：CentOS Linux release 7.6.1810 (Core) ，对应的 Redis 版本则是：Stable (6.0.5)。

### 下载安装包

> 首先需要下载好对应版本的 Redis 安装包文件，可访问 https://redis.io/download ，然后选择**稳定版**进行下载，如下图：
> 
> ![Redis.Download](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594180468165.png)
> 
> 可以选择在本地下载后上传到服务器指定目录下面进行后续操作，也可以选择在服务器上面直接通过 wget 获取对应的安装包，这里我个人推荐使用第二种方案。
> 
> 选择第二种方案，鼠标右键点击上图方框中的下载按钮，选择复制链接地址，然后打开服务器控制台，切换到指定目录(不切换将默认下载到当前登录账户的个人目录下面)，通过 wget 命令进行下载即可，完整命令行如下：
> 
> ```shell
> [root@hecs redis]# wget http://download.redis.io/releases/redis-6.0.5.tar.gz
> ```
> 
> 输入命令后按下回车键即可开始下载，如下图所示：
> 
> ![wget.redis.download](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594180799591.png)
> 
> 因为安装包很小，所以一般很快就下载完成了。如果下载非常慢，则可以找找国内的下载源进行下载。

### 解压安装 Redis

> &emsp;&emsp;在上面的第一步中我们下载了 Redis 安装包，接下来我们将解压刚下载好的安装包，然后进行编译安装。

#### 解压安装包

> &emsp;&emsp;将刚才下载的安装包进行解压处理，执行如下命令即可解压到当前目录：
> 
> ```shell
> [root@hecs redis]# tar -zxvf redis-6.0.5.tar.gz 
> ```
> 
> 解压完成后用命令输出当前文件夹下面的信息即可看到刚才解压后的文件夹 redis-6.0.5，如下图所示：
> 
> ![redis.ll](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594181305709.png)

#### 移动安装包

> &emsp;&emsp;将刚才解压后的安装包移动到 /usr/local/ 并重命名为 redis ，这样方便后续操作，你也可以在上一步解压的时候直接解压到 /usr/local/ 目录下面。我这里选择先解压再进行转移，执行如下命令：
> 
> ```shell
> [root@hecs redis]# mv redis-6.0.5 /usr/local/redis
> ```
> 
> &emsp;&emsp;用命令查看 /usr/local/ 目录下的文件信息，便可看到刚才转移过来的 redis 文件夹，如下图所示：
> 
> ![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594181646185.png)

#### 编译&安装

> &emsp;&emsp;上面我们将 redis 安装包移动到了 /usr/local/ 目录下，接下来我们将切换到 /usr/local/redis/ 目录下进行编译处理，执行如下命令进行目录切换：
> 
> ```shell
> [root@hecs redis]# cd /usr/local/redis/
> ```
> 
> 执行 make 命令进行编译处理：
> 
> ```shell
> [root@hecs redis]# make
> ```
> 
> 编译完成后将看到如下结果：
> 
> ![redis.make.error](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594182036426.png)
> 
> 很明显编译出错了，遇到这种错误，老司机们知道会升级 gcc ，但是还是有很多人不想折腾。如果不想折腾，那你只能使用低版本的 redis 了，比如：redis-5.0.8 。如果你就是想用最新版本的 redis ，那么你就只能乖乖的升级 gcc 吧，接下来我们一起来试试吧。
> 
> 安装升级 gcc，执行如下命令即可查看当前 gcc 版本：
> 
> ```shell
> [root@hecs redis]# gcc -v
> ```
> 
> 升级 gcc 需要执行如下命令：
> 
> ```shell
> [root@hecs redis]# yum -y install centos-release-scl  # 升级到9.1版本
> [root@hecs redis]# yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
> [root@hecs redis]# scl enable devtoolset-9 bash  # 使用gcc 9.1
> ```
> 
> 如果需要长期使用 gcc 9.1 则需要执行下面的命令：
> 
> ```shell
> [root@hecs redis]# echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
> ```
> 
> 更新完 gcc ，我们再次执行 make 命令就执行成功了，如下结果：
> 
> ![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594182641730.png)
> 
> 编译成功后就可以执行安装了，执行如下命令：
> 
> ```shell
> [root@hecs redis]# make PREFIX=/usr/local/redis install
> ```
> 
> 安装完成效果如下图：
> 
> ![redis.install.result](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594191475475.png)
> 
> &emsp;&emsp;这上面执行的命令中多了一个关键字 PREFIX= 这个关键字的作用是编译的时候用于指定程序存放的路径。比如我们现在就是指定了redis必须存放在 /usr/local/redis 目录。假设不添加该关键字 Linux 会将可执行文件存放在 /usr/local/bin 目录，库文件会存放在 /usr/local/lib 目录。配置文件会存放在 /usr/local/etc 目录。其他的资源文件会存放在 /usr/local/share 目录。这里指定号目录也方便后续的卸载，后续直接 rm -rf /usr/local/redis 即可删除 redis。

### 启动 Redis

> &emsp;&emsp;在上面的步骤中，我们经历了一波三折，最终完美安装好了 Redis ，接下来我们将会把它启动看看效果。
> 
> 切换到 /usr/local/redis 目录下面，执行如下命令，启动 Redis 服务：
> 
> ```shell
> [root@hecs redis]# ./bin/redis-server & redis.conf
> ```
> 
> 启动结果如下：
> 
> ![redis.run](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594191909417.png)
> 
> 上面的启动方式是采取后台进程方式,下面是采取显示启动方式(如在配置文件设置了 daemonize 属性为 yes 则跟后台进程方式启动其实一样)。
> 
> ```shell
> [root@hecs redis]# ./bin/redis-server redis.conf
> ```
> 
> &emsp;&emsp;两种方式区别无非是有无带符号&的区别。 redis-server 后面是配置文件，目的是根据该配置文件的配置启动redis服务。redis.conf配置文件允许自定义多个配置文件，通过启动时指定读取哪个即可。

### 配置 Redis

> &emsp;&emsp;在上面我们启动的时候用到了 Redis 配置文件 redis.conf ，存放于目录 /usr/local/redis 。我们可以通过 Linux 内置的命令(cat、vim、less等)读取查看该配置文件的内容。通过 cat 命令获取 redis.conf 内容信息如下：
> 
> ![redis.conf](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594192681554.png)
> 
> 也可以通过 redis-cli 命令进入 redis 控制台后通过 CONFIG GET * 的方式读取所有配置项。 如下所示：
> 
> ![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594193110129.png)
> 
> 这里列举下比较重要的配置项进行说明：
> 
> <table border="1px" style="">
>     <tr>
>         <th style="border-right: 1px grey solid;">配置项名称</th>
>         <th style="border-right: 1px grey solid;">配置项值范围</th>
>         <th style="border-right: 1px grey solid;">配置项描述</th>
>     </tr>
>     <tr>
>         <td style="border-right: 1px grey solid;">daemonize</td>
>         <td style="border-right: 1px grey solid;">yes/no</td>
>         <td style="border-right: 1px grey solid;">yes表示启用守护进程，默认是no即不以守护进程方式运行。其中Windows系统下不支持启用守护进程方式运行</td>
>     </tr>
>     <tr>
>         <td style="border-right: 1px grey solid;"> port</td>
>         <td style="border-right: 1px grey solid;"></td>
>         <td style="border-right: 1px grey solid;">指定 Redis 监听端口，默认端口为 6379</td>
>     </tr>
>     <tr>
>         <td style="border-right: 1px grey solid;"> bind </td>
>         <td style="border-right: 1px grey solid;"></td>
>         <td style="border-right: 1px grey solid;">绑定的主机地址,如果需要设置远程访问则直接将这个属性备注下或者改为bind * 即可,这个属性和下面的protected-mode控制了是否可以远程访问</td>
>     </tr>
>     <tr>
>         <td style="border-right: 1px grey solid;">protected-mode</td>
>         <td style="border-right: 1px grey solid;">yes/no</td>
>         <td style="border-right: 1px grey solid;">保护模式，该模式控制外部网是否可以连接redis服务，默认是yes,所以默认我们外网是无法访问的，如需外网连接rendis服务则需要将此属性改为no</td>
>     </tr>
>     <tr>
>         <td style="border-right: 1px grey solid;"> timeout</td>
>         <td style="border-right: 1px grey solid;"> 300 </td>
>         <td style="border-right: 1px grey solid;">当客户端闲置多长时间后关闭连接，如果指定为 0，表示关闭该功能</td>
>     </tr>
>     <tr>
>         <td style="border-right: 1px grey solid;"> loglevel </td>
>         <td style="border-right: 1px grey solid;">debug/verbose/notice/warning</td>
>         <td style="border-right: 1px grey solid;">日志级别，默认为 notice</td>
>     </tr>
>     <tr>
>         <td style="border-right: 1px grey solid;"> databases </td>
>         <td style="border-right: 1px grey solid;"> 16 </td>
>         <td style="border-right: 1px grey solid;">设置数据库的数量，默认的数据库是0。整个通过客户端工具可以看得到</td>
>     </tr>
>     <tr>
>         <td style="border-right: 1px grey solid;"> rdbcompression </td>
>         <td style="border-right: 1px grey solid;">yes/no</td>
>         <td style="border-right: 1px grey solid;">指定存储至本地数据库时是否压缩数据，默认为 yes，Redis 采用 LZF 压缩，如果为了节省 CPU 时间，可以关闭该选项，但会导致数据库文件变的巨大</td>
>     </tr>
>     <tr>
>         <td style="border-right: 1px grey solid;"> dbfilename </td>
>         <td style="border-right: 1px grey solid;">dump.rdb</td>
>         <td style="border-right: 1px grey solid;">指定本地数据库文件名，默认值为 dump.rdb</td>
>     </tr>
>     <tr>
>         <td style="border-right: 1px grey solid;"> dir </td>
>         <td style="border-right: 1px grey solid;"></td>
>         <td style="border-right: 1px grey solid;">指定本地数据库存放目录</td>
>     </tr>
>     <tr>
>         <td style="border-right: 1px grey solid;"> requirepass </td>
>         <td style="border-right: 1px grey solid;"></td>
>         <td style="border-right: 1px grey solid;">设置 Redis 连接密码，如果配置了连接密码，客户端在连接 Redis 时需要通过 AUTH <password> 命令提供密码，默认关闭</td>
>     </tr>
>     <tr>
>         <td style="border-right: 1px grey solid;"> maxclients </td>
>         <td style="border-right: 1px grey solid;">0</td>
>         <td style="border-right: 1px grey solid;">设置同一时间最大客户端连接数，默认无限制，Redis 可以同时打开的客户端连接数为 Redis 进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis 会关闭新的连接并向客户端返回 max number of clients reached 错误信息</td>
>     </tr>
>     <tr>
>         <td style="border-right: 1px grey solid;"> maxmemory </td>
>         <td style="border-right: 1px grey solid;">XXX <bytes></td>
>         <td style="border-right: 1px grey solid;">指定 Redis 最大内存限制，Redis 在启动时会把数据加载到内存中，达到最大内存后，Redis 会先尝试清除已到期或即将到期的 Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis 新的 vm 机制，会把 Key 存放内存，Value 会存放在 swap 区。配置项值范围列里XXX为数值</td>
>     </tr>
> </table>
> 
> &emsp;&emsp;由于默认情况使用 ./bin/redis-server ./redis.conf 每次启动都得在redis-server 命令后面加符号 &，如果不加上这个符号，一旦回到 Linux 控制台则 redis 服务会自动关闭。同时我需要能外网访问，所以将 bind 注释掉，将 protected-mode 设置为 no ，并设置了访问密码，修改了访问端口。这样启动后我就可以在外网通过密码访问了，也相对比较安全了。
> 
> 为了满足我自己的要求，开始更改 redis.conf 文件内容，执行一下命令开始修改：
> 
> ```shell
> [root@hecs redis]# vim redis.conf
> ```
> 
> 你们可以依据自己的需要进行修改哦~

### 查看 Redis 运行情况并连接

#### 查看 Redis 运行情况

> 1、采取查看进程方式，执行如下命令：
> 
> ```shell
> [root@hecs redis]# ps -aux | grep redis
> ```
> 
> 结果如下图：
> ![ps.aux.grep.redis](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594195949354.png)
> 
> 可以看到我的 redis-server 运行端口是 6380。
> 
> 2、采取端口监听查看方式，执行如下命令(由于我修改了端口，所以大家可以 依据自己的实际端口修改执行下面的命令)：
> 
> ```shell
> [root@hecs redis]# netstat -lanp | grep 6380
> ```
> 
> 结果如下图：
> 
> ![netstat.lanp.grep.redis](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594196117341.png)

#### 连接到 Redis

> &emsp;&emsp;Redis 自带了连接客户端 redis-cli ，采用命令模式进行连接，对应的常见的连接命令语法为：
> 
> ```shell
> # -h 服务器地址 -p 端口号 -a 密码
> [root@hecs redis]# redis-cli -h host -p port -a password
> ```
> 
> 我这里修改了密码，修改了端口，但是属于本机连接(可不输入密码)，对应的连接命令为：
> 
> ```shell
> [root@hecs redis]# ./bin/redis-cli -h 127.0.0.1 -p 6380
> ```
> 
> 连接成功后将进入 Redis 控制台，如下图：
> 
> ![redis-cli](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594196501177.png)
> 
> 如果设置了登录密码，那么需要在 Redis 控制台进行密码认证方可进行其他数据交互操作，在控制台输入: `auth "你的密码"` 进行身份认证，否则会有错误提示：“(error) NOAUTH Authentication required.”
> 如下图所示：
> 
> ![Redis.Auth](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594375331170.png)
> 
> 接下来，尝试进行远程连接，这里，推荐使用 Redis-Desktop-Manager 工具进行远程连接，如下图：
> 
> ![RDM](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594196718946.png)
> 
> 在 Name 处填写自定义名称，在 Host 处填写对应的 Redis 服务器 Ip 地址，在 Port 处填写 Redis 实际的端口号(默认是6379)，在 Auth 处填写设置的密码，如果未设置密码可以不填写(外网访问建议设置密码)，然后点击“Test Connection”进行连接测试，如果设置没问题将会提示“Successful connection to redis-server”，远程连接成功！
