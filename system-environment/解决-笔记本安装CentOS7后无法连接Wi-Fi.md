# 解决-笔记本安装CentOS 7 后无法连接Wi-Fi

&emsp;&emsp;今天闲来无事，把一台5年前买的游戏本安装成了 CentOS 7 系统，准备把它当作一台测试服务器来使用，但是安装的过程真是一波三折... ... 网络问题最让人头疼，于是我将我的解决过程记录了下来，以便大家参考。

### 问题现象

本来我在安装的时候在图形界面下选择了我使用的 Wi-Fi 并输入了密码成功链接上了，但是安装完成后我尝试 `ping www.baidu.com` 却发现没成功，提示如下：

```shell
ping www.baidu.com: Name or service not known
```

最开始我以为是我安装的时候选择了【最小化安装】没有安装上对应的附带组件造成的，于是我选择了重新安装，安装的时候选择了【基础设施服务器】进行安装，也勾选了一些组件，想着这次联网应该没问题，但是事实却是没啥变化，还是连不上网，`ping www.baidu.com` 依然提示上面的错误。于是开始在网页里遨游(寻找解决方案)~

### 解决方案

经过对多个网页内容的人眼👀扫描，最终选定了一组方案，进行测试后效果甚佳，具体步骤如下：

#### 检查网卡配置

进入如下文件夹内：

```shell
[root@localhost /]# cd /etc/sysconfig/network-scripts
```

由于我这里链接的是 Wi-Fi ，且安装的时候链接过，所以有一个文件叫 `ifcfg-ChinaNet-txgV`，后面的 `ChinaNet-txgV` 就是我的 Wi-Fi 名称，我看了下里面的配置内容，基本上没啥大问题(该有的都有↖(^ω^)↗)。

这个目录下还有很多其他的以 `ifcfg-` 开头的文件，基本都是些配置文件，默认的配置文件叫 `ifcfg-eth0` 默认使用的就是这个，所以如果没看到其他和你所用网络一致的后缀名称的配置文件，多半是使用了 `ifcfg-eth0` 作为网卡配置文件了。主要是看看里面配置的 `NAME` 是否匹配，`ONBOOT` 是否等于 `yes`，这点儿比较关键哈。
(PS：网上有的教程说调整 `ONBOOT=yes` ，然后重启网络 `service network restart` 就可以了，那估计他电脑插着网线，和咋们情况不一致。我实际尝试了并没啥卵用。。。。)

#### 激活无线网卡

**1、查看无线网卡信息**

执行命令：

```shell
[root@localhost /]# iw dev
phy#0
    Interface wlp2s0
        ifindex 3
        wdev 0x1
        addr 74:df:bf:bf:05:44
        type managed
[root@localhost /]# 
```

从上面的信息中可以得到 Interface 后面的网卡的口号标识是 `wlp2s0` ，一定要记着这个口号，等会下面要使用哦～

**2、激活无线网卡**

执行命令：

```shell
[root@localhost /]# ip link set wlp2s0 up
```

**3、查看激活状态**

执行命令：

```shell
[root@localhost /]# ip link show wlp2s0
3: wlp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DORMANT group default qlen 1000
    link/ether 74:df:bf:bf:05:44 brd ff:ff:ff:ff:ff:ff
```

从返回的结果可以看到里面含有关键字 `UP` 可以看出是激活成功的。

**4、检查是否有连接**

执行命令：

```shell
[root@localhost /]# iw wlp2s0 link
not connect
```

通常来说是没有链接的。

**5、扫描可连接的 WI-FI**

执行命令：

```shell
[root@localhost /]# iw wlp2s0 scan | grep SSID
```

执行完成后会列出能被网卡搜索到的 Wi-Fi 信息，如果你一个也搜索不到，则说明你的无线网卡硬件有问题，或者你电脑附近没有无线网络可供你扫描到。

**6、链接 Wi-Fi**

从上面第五步中找到你要链接的 Wi-Fi ，记住它的【全名称】，输入如下命令进行连接：

```shell
[root@localhost /]# wpa_supplicant -B -i wlp2s0 -c <(wpa_passphrase 'wifi-全名称' 'wifi密码')
```

如果你输入的 Wi-Fi 名称和密码没有错误的情况下，你应该已经连接上无线网络了。

**7、分配 IP**

执行如下命令，用于获得 ip 的分配：

```shell
[root@localhost /]# dhclient wlp2s0
```

**8、查看是否成功分配 IP**

执行如下命令：

```shell
[root@localhost /]# ip addr show wlp2s0
```

这一步执行完成后你将会看到你分配到的 IP 地址。至此，大功告成啦！

测试一下是否可以 ping 通外网：

```shell
[root@localhost /]# ping www.baidu.com
64 bytes from 220.181.38.150: icmp_seq=2 ttl=55 time=7.25 ms
64 bytes from 220.181.38.150: icmp_seq=2 ttl=55 time=6.99 ms
64 bytes from 220.181.38.150: icmp_seq=2 ttl=55 time=7.59 ms
64 bytes from 220.181.38.150: icmp_seq=2 ttl=55 time=7.81 ms
```

我终于 ping 通了，哈哈哈，可以愉快的玩耍啦～

### 解决重启后网络连接失效的问题

按照上面的步骤配置完成后成功链接上了 Wi-Fi 但是重启后发现又无法联网了，还得继续安装上述步骤又执行一遍，这谁能忍受，所以找了个法子，防止重启后网络无法自动连接，具体操作如下：

**1、安装 NetworkManager-wifi**

执行命令：

```shell
[root@localhost /]# yum -y install NetworkManager-wifi
```

**2、开启wifi**

执行命令：

```shell
[root@localhost /]# nmcli r wifi on
```

**3、测试（扫描信号）**

执行命令：

```shell
[root@localhost /]# nmcli dev wifi
```

**4、查看网络连接**

执行命令：

```shell
[root@localhost /]# nmcli connection
NAME           UUID                                  TYPE      DEVICE 
ChinaNet-txgV  24a2afc3-d111-4f13-9291-4c8bd1ee5712  wifi      wlp2s0 
virbr0         ddd1cb25-9568-4638-8d71-1a079e409f30  bridge    virbr0 
enp3s0f1       8f4e32c0-0d39-4037-99b0-072b6cdb081d  ethernet  --   
```

⚠️注意：如果你看到你的 DEVICE 这一列下面找不到你的网卡口号标识（我这里是 wlp2s0），那么你继续往下执行 Wi-Fi 链接的时候可能会报错：

```shell
Unknown parameter: wlp2s0
Error: No Wi-Fi device found.
```

如果出现了这些现象，那么建议你重启电脑，重新执行 Wi-Fi 链接即可。

**5、删除上图所有的 TYPE=wifi 的连接（根据UUID删除）**

执行命令：

```shell
[root@localhost /]# nmcli c delete 24a2afc3-d111-4f13-9291-4c8bd1ee5712
```

**6、重新连接 Wi-Fi**

执行命令：

```shell
# 例如链接 Wi-Fi 名为 ChinaNet-txgV，密码为 123456 的无线网
[root@localhost /]# nmcli d wifi connect "ChinaNet-txgV" password "123456" wlp2s0
Unknown parameter: wlp2s0
Device 'wlp2s0' successfully activated with '24a2afc3-d111-4f13-9291-4c8bd1ee5712'.
```

注意这里可能会失败，出现上述第 4 步中说的问题，请按照上述方式解决吧。

**7、测试网络连接**

执行命令：

```shell
[root@localhost /]# ping www.baidu.com
PING www.a.shifen.com (220.181.38.150) 56(84) bytes of data.
64 bytes from 220.181.38.150 (220.181.38.150): icmp_seq=1 ttl=55 time=21.0 ms
64 bytes from 220.181.38.150 (220.181.38.150): icmp_seq=2 ttl=55 time=10.0 ms
64 bytes from 220.181.38.150 (220.181.38.150): icmp_seq=3 ttl=55 time=6.51 ms
64 bytes from 220.181.38.150 (220.181.38.150): icmp_seq=4 ttl=55 time=6.71 ms
```

可以 ping 成功，则网络连接成功了。
现在重启下电脑(`reboot`)查看网络是否自动连接上了吧。（PS：我这里成功自动连接上了）

### 将 Wi-Fi改为静态IP

从我的计划使用场景来看，我需要让系统保持一个固定的 IP ，这样我每次远程局域网登录的时候就不需要上主机上查看当前的 IP 地址了，而且其他服务也不用改 IP 就可以链接上，这样就会方便许多。

**1、运行 ifconfig 查看 IPADDR 和 NETMASK，如下：**

```shell
[root@localhost network-scripts]# ifconfig
enp3s0f1: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether 70:8b:cd:00:69:a9  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 68  bytes 5908 (5.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 68  bytes 5908 (5.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
        ether 52:54:00:1b:d2:af  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlp2s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.19  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 240e:304:2880:3433:234b:c95c:3c98:3618  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::ebd7:536d:d12:51aa  prefixlen 64  scopeid 0x20<link>
        ether 74:df:bf:b0:05:46  txqueuelen 1000  (Ethernet)
        RX packets 2323  bytes 163515 (159.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2091  bytes 190109 (185.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

其中 wlp2s0 后面的 `inet 192.168.1.19  netmask 255.255.255.0 ` 就是我们要记下的信息。
(PS: 如果执行 ifconfig 出现找不到命令的情况请安装`net-tools`，命令：`yum -y install net-tools`)

**2、查看DNS，执行命令如下：**

```shell
[root@localhost network-scripts]# cat /etc/resolv.conf
# Generated by NetworkManager
nameserver 192.168.1.1
nameserver 240e:40:8000::10
nameserver 240e:40:8000::11
```

**3、查看网关 GATEWAY，如下：**

```shell
[root@localhost network-scripts]# netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG        0 0          0 wlp2s0
192.168.1.0     0.0.0.0         255.255.255.0   U         0 0          0 wlp2s0
192.168.122.0   0.0.0.0         255.255.255.0   U         0 0          0 virbr0
```

**4、修改网络配置文件，如下：**

编辑 /etc/sysconfig/network-scripts 目录下以 ifcfg-开头，后面是你 Wi-Fi 名称的配置文件，比如我的是：ifcfg-ChinaNet-txgV，如下：

```shell
ESSID=ChinaNet-txgV
MODE=Managed
KEY_MGMT=WPA-PSK
SECURITYMODE=open
MAC_ADDRESS_RANDOMIZATION=default
TYPE=Wireless
PROXY_METHOD=none
BROWSER_ONLY=no
#BOOTPROTO=dhcp

# 将dhcp修改为 static
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ChinaNet-txgV
UUID=24a2afc3-d111-4f13-9291-4c8bd1ee5712
ONBOOT=yes

# 主要添加了如下配置内容
IPADDR=192.168.1.19
NETMASK=255.255.255.0
DNS1=192.168.1.1
DNS2=240e:40:8000::10
DNS3=240e:40:8000::11
GATEWAY=192.168.1.1
```

修改保存完毕，reboot 重启，查看网络是否连接。

我这里调整完重启后 Wi-Fi 自动连接正常，`ping www.baidu.com` 正常，执行 `ifconfig` 查看 IP 地址是我设定的静态 IP 地址。所以，一切正常！

### 后记

这次的 CentOS 系统安装到游戏本是第一次成功，颇有一番成就感，但是过程是非常波折的，一开始我选择 `Install CentOS 7` 进行安装时就遇到了 `dracut：/# ... timeout` 等一大堆问题。于是一路百度搜索，按照友友们的说法，是在用 【uitralso软碟通】制作完启动 U 盘后 U 盘名称和正常 CentOS 配置文件中的对应不上，正常的应该是 `CentOS 7 x86_64`，实际我看到是 `CentOS 7 x8` 原因是 Windows 对磁盘名称的限制，不能再输入更多的文字了，于是按照教程，我修改了启动目录，在进入选择 `Install CentOS 7` 目录后，键盘上按 `E` 键进行编辑，修改内容如下：

```shell
vmlinuz initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x86_64.check quiet
```

 把这段改成

```shell
vmlinuz initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x8.check quiet
```

实际就是把 LABEL= 后面的值改成你 U 盘名称，因为默认制作完启动盘后的 U 盘名称就是 `CentOS 7 x8` 其中命令行中的空格是用 `x20` 代替，所以这里只需将 `6_64` 删除后就好了。

但是，绝望的是我修改这些后，进行安装的过程中出现了：

```shell
Not asking for VNC because we don't have a network
```

然后就一直卡死在那儿了，网上一顿狂找答案，看到各种答案都有，但是其中有一个说是镜像问题，我抱着试试看的想法，重新从官方找了 CentOS 7 的镜像，使用 aliyun 地址下载了下来，重新制作了启动盘，按上面的步骤修改了 `CentOS 7 x86_64` 为 `CentOS 7 x8` 然后直接安装，就顺利进入了安装界面了，看到安装界面的那一刻真的非常幸福，哈哈哈哈。。。。 因为我3年前就曾尝试安装过，但是没安装成功，网上说是因为 N 卡显卡驱动问题，当时就放弃了，这次终于成功啦，耶✌️～

### 其他

笔记本安装 CentOS 7 后无法像 Windows 界面化操作设置合盖不睡眠(PS: 因为我打算将这台笔记本当作我平时使用的服务器)，所以需要设置一下合盖不睡眠，说干就干：

```shell
[root@localhost /]# vi /etc/systemd/logind.conf
# 按 【i】键进入编辑模式，将 HandleLidSwitch 设置为 lock (默认是suspend，lock代表仅锁屏)，如下：
HandleLidSwitch=lock
```

设置完后按【esc】键退出编辑模式，然后依次输入 `:wq` (注意英文字符) 退出并保存。
最后重置一下就好了，重置命令如下：

```shell
[root@localhost /]# systemctl restart systemd-logind
```

执行完命令就完事儿啦，完美～

### 相关地址

我下载 CentOS 7 镜像使用的官方地址：
http://isoredirect.centos.org/centos/7/isos/x86_64/

我选择了 aliyun 镜像 ：
http://mirrors.aliyun.com/centos/7.9.2009/isos/x86_64/

下载的是 DVD 版本：[CentOS-7-x86_64-DVD-2009.iso](http://mirrors.aliyun.com/centos/7.9.2009/isos/x86_64/CentOS-7-x86_64-DVD-2009.iso)

你们也可以下载 Everything 版本：[CentOS-7-x86_64-Everything-2009.iso](http://mirrors.aliyun.com/centos/7.9.2009/isos/x86_64/CentOS-7-x86_64-Everything-2009.iso)
