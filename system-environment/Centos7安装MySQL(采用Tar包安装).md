# Centos7 安装 MySQL (采用 Tar 包安装)



&emsp;&emsp;MySQL所使用的 SQL 语言是用于访问数据库的最常用标准化语言。MySQL 由于其体积小、速度快、总体拥有成本低，尤其是开放源码这一特点，一般中小型网站的开发都选择 MySQL 作为网站数据库。

&emsp;&emsp;接下来咋们就从头开始安装 MySQL 数据库服务:
&emsp;&emsp;MySQL 安装目录为：/usr/local 下名称为 mysql57 
&emsp;&emsp;MySQL 数据存储目录为：/data/mysql/

### 一、寻找适合自己的 MySQL 安装包

建议前往 MySQL 官方网站下载，官方下载链接地址：https://dev.mysql.com/downloads/
如图：
![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1586333433743.png)

我们使用开源版本(也叫社区版/免费)，所以选择 “MySQL Community Server”。

默认访问进去看到的是最新版本的安装包下载，如果需要下载历史版本，需要点击旁边的链接进行切换。
如图：
![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1586333729645.png)

这里，我个人倾向于选择 MySQL 5.7 版本，因为很多项目都是用的这个版本开发，避免出现版本差异，就继续用 5.7 这个版本吧！

选择好合适的数据库版本以及系统类型，如图：
![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1586333955823.png)

对于常用的 CentOS 系统，推荐选择图中的选项，除非系统类型不同。

选择好数据库版本及系统类型后再选择要下载的安装包即可，如图：
![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1586334097398.png)

此处推荐选择“Compressed TAR Archive”进行下载即可！
(温馨提示：可以在服务器上通过 wget 命令获取安装包，这样就不必在本地下载后再上传到服务器这么麻烦了。)

### 二、下载并解压缩安装包及创建数据目录

#### 1、下载安装到服务器上存储目录 /data/software_pkg 下，执行命令：

```
[root@hecs-x-large-2-linux-xx software_pkg]# wget https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.29-el7-x86_64.tar.gz
```

#### 2、解压缩安装包到 /usr/local/ 目录下，并重命名为 mysql57，执行命令：

```
[root@hecs-x-large-2-linux-xx software_pkg]# tar -zxvf mysql-5.7.29-el7-x86_64.tar.gz -C /usr/local/
[root@hecs-x-large-2-linux-xx software_pkg]# mv /usr/local/mysql-5.7.29-el7-x86_64 /usr/local/mysql57
```

#### 3、创建 MySQL 的数据存储目录，执行命令：

```
[root@hecs-x-large-2-linux-xx software_pkg]# mkdir /data/mysql
```

### 三、检查系统环境并添加对应的用户组及用户为安装作准备

#### 1、查看系统是否有安装 Mariadb ，执行命令：

```
[root@hecs-x-large-2-linux-xx ~]# rpm -qa|grep mariadb
mariadb-libs-5.5.64-1.el7.x86_64
```

#### 2、若系统自带安装了 Mariadb 则需要进行卸载，执行命令：

```
[root@hecs-x-large-2-linux-xx ~]# rpm -e --nodeps mariadb-libs-5.5.64-1.el7.x86_64
```

至于为什么要卸载掉系统预置的 Mariadb 此处不做过多解释，你只需要明白Centos7 预置的 Mariadb 和你要安装的 MySQL 会存在冲突，所以想要安装 MySQL 则必须要先卸载掉 Mariadb.

#### 3、检查系统是否已经安装了 MySQL ，执行命令：

```
[root@hecs-x-large-2-linux-xx ~]# rpm -qa | grep mysql
```

未返回信息则表示未安装MySQL，因为此处是全新安装，所以按照一台服务器只安装一个 MySQL 的方式进行操作。

#### 4、创建 MySQL 对应的用户组，执行命令：

```
[root@hecs-x-large-2-linux-xx ~]# groupadd mysql
```

未报错则表示执行成功！
如果需要提前检测是否已存在这个用户组，可执行以下命令：

```
[root@hecs-x-large-2-linux-xx ~]# cat /etc/group | grep mysql
```

若已存在对应的分组则会返回显示在控制台，若未返回分组信息则表示不存在对应的分组。

#### 5、创建 MySQL 对应的用户名并加入到刚创建的 MySQL 用户组中，执行命令：

```
[root@hecs-x-large-2-linux-xx ~]# useradd -r -g mysql mysql
```

未报错则表示执行成功！

#### 6、关联刚才所创建的用户 mysql 到对应的 mysql 用户组中，执行命令：

```
[root@hecs-x-large-2-linux-xx software_pkg]# chown -R mysql:mysql /usr/local/mysql57/
[root@hecs-x-large-2-linux-xx software_pkg]# chown -R mysql:mysql /data/mysql/
[root@hecs-x-large-2-linux-xx software_pkg]# chown -R mysql /usr/local/mysql57/
[root@hecs-x-large-2-linux-xx software_pkg]# chown -R mysql /data/mysql/
```

#### 7、更改 MySQL 安装包文件夹权限，执行命令：

```
[root@hecs-x-large-2-linux-xx software_pkg]# chmod -R 755 /usr/local/mysql57/
```

#### 8、安装 MySQL 依赖包 libaio，(由于我买的华为云服务器 CentOS 系统未自带这个依赖包，所以第一次安装的时候失败了)，执行命令：

```
# 查询是否暗转libaio依赖包
[root@hecs-x-large-2-linux-xx software_pkg]#yum search libaio
# 如果没安装，可以用下面命令安装
[root@hecs-x-large-2-linux-xx software_pkg]#yum install libaio
```

### 四、开始安装 MySQL

#### 1、初始化 MySQL，执行命令：

```
[root@hecs-x-large-2-linux-xx software_pkg]# cd /usr/local/mysql57/bin/
[root@hecs-x-large-2-linux-xx bin]# ./mysqld --user=mysql --basedir=/usr/local/mysql57 --datadir=/data/mysql --initialize
2020-04-18T06:49:15.709902Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2020-04-18T06:49:16.350125Z 0 [Warning] InnoDB: New log files created, LSN=45790
2020-04-18T06:49:16.393822Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2020-04-18T06:49:16.459909Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: b8af2ad0-8140-11ea-a0df-fa163e878024.
2020-04-18T06:49:16.463686Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2020-04-18T06:49:17.137696Z 0 [Warning] CA certificate ca.pem is self signed.
2020-04-18T06:49:17.560585Z 1 [Note] A temporary password is generated for root@localhost: KIwvt/03)yaI
```

注意⚠️：这里需要特别注意最后一行输出的内容：

```
2020-04-18T06:49:17.560585Z 1 [Note] A temporary password is generated for root@localhost: KIwvt/03)yaI
```

其中“KIwvt/03)yaI”就是临时的 MySQL 本地登录密码，后面安装完成后本地登录修改 root 账户密码时会用到，可以记下来备用。

如果初始化时报错如下：

```
error while loading shared libraries: libnuma.so.1: cannot open shared objec
```

是因为libnuma安装的是32位，我们这里需要64位的，执行下面命令就可以解决：

```
[root@hecs-x-large-2-linux-xx bin]# yum install numactl.x86_64
```

执行完后重新初始化 MySQL。

#### 2、启动 MySQL 服务，执行命令：

```
[root@hecs-x-large-2-linux-xx bin]# sh /usr/local/mysql57/support-files/mysql.server start
/usr/local/mysql57/support-files/mysql.server:行239: my_print_defaults: 未找到命令
/usr/local/mysql57/support-files/mysql.server: 第 259 行:cd: /usr/local/mysql: 没有那个文件或目录
Starting MySQLCouldn't find MySQL server (/usr/local/mysql/[失败]sqld_safe)
```

如上所示，启动服务失败，因为没有修改 MySQL 的配置文件。

英文版的报错内容大致如下：

```
/usr/local/mysql57/support-files/mysql.server:line 239: my_print_defaults: command not found
/usr/local/mysql57/support-files/mysql.server: line 259:cd: /usr/local/mysql: No such file or directory
Starting MySQLCouldn't find MySQL server (/usr/local/mysql/bin/mysqld_safe)
```

#### 3、修改 MySQL 配置文件，如下：

```
[root@hecs-x-large-2-linux-xx bin]# vim ../support-files/mysql.server 
# 修改前的内容如下：

# Set some defaults
mysqld_pid_file_path=
if test -z "$basedir"
then
  basedir=/usr/local/mysql
  bindir=/usr/local/mysql/bin
  if test -z "$datadir"
  then
    datadir=/usr/local/mysql/data
  fi
  sbindir=/usr/local/mysql/bin
  libexecdir=/usr/local/mysql/bin
else
  bindir="$basedir/bin"
  if test -z "$datadir"
  then
    datadir="$basedir/data"
  fi
  sbindir="$basedir/sbin"
  libexecdir="$basedir/libexec"
fi


# 修改后的内容如下：

# Set some defaults
mysqld_pid_file_path=
if test -z "$basedir"
then
  basedir=/usr/local/mysql57
  bindir=/usr/local/mysql57/bin
  if test -z "$datadir"
  then
    datadir=/data/mysql
  fi
  sbindir=/usr/local/mysql57/bin
  libexecdir=/usr/local/mysql57/bin
else
  bindir="$basedir/bin"
  if test -z "$datadir"
  then
    datadir="$basedir/data"
  fi
  sbindir="$basedir/sbin"
  libexecdir="$basedir/libexec"
fi
```

执行万 vim 命令后按‘i’键即可进入编辑模式，编辑完后按‘ESC’键即可退出编辑模式，然后英文状态下输入‘:wq’即可退出 vim 。

修改后再启动 MySQL 就会出现如下信息(此处只作启动验证，启动后记得停止服务再进行后面的操作，避免出现其他不必要的影响)：

```
[root@hecs-x-large-2-linux-xx bin]# sh /usr/local/mysql57/support-files/mysql.server start
Starting MySQL.Logging to '/data/mysql/hecs-x-large-2-linux-20200331153626.err'.
                                                           [  确定  ]
[root@hecs-x-large-2-linux-xx bin]# sh /usr/local/mysql57/support-files/mysql.server stop
Shutting down MySQL..                                      [  确定  ]
```

#### 4、将刚才编辑过的服务启动文件复制到系统目录下并给定权限，执行命令：

```
[root@hecs-x-large-2-linux-xx bin]# cp /usr/local/mysql57/support-files/mysql.server /etc/init.d/mysql.server
[root@hecs-x-large-2-linux-xx bin]# chmod 755 /etc/init.d/mysql.server 
```

#### 5、配置 MySQL 的配置文件 my.cnf，配置内容如下：

```
[client]
no-beep
socket =/usr/local/mysql57/mysql.sock
# pipe
# socket=0.0
port=53306
[mysql]
default-character-set=utf8
[mysqld]
basedir=/usr/local/mysql57
datadir=/data/mysql
port=53306
pid-file=/usr/local/mysql57/mysqld.pid
#skip-grant-tables
skip-name-resolve
socket = /usr/local/mysql57/mysql.sock
character-set-server=utf8
default-storage-engine=INNODB
explicit_defaults_for_timestamp = true
# Server Id.
server-id=1
max_connections=2000
query_cache_size=0
table_open_cache=2000
tmp_table_size=246M
thread_cache_size=300
#限定用于每个数据库线程的栈大小。默认设置足以满足大多数应用
thread_stack = 192k
key_buffer_size=512M
read_buffer_size=4M
read_rnd_buffer_size=32M
innodb_data_home_dir = /data/mysql
innodb_flush_log_at_trx_commit=0
innodb_log_buffer_size=16M
innodb_buffer_pool_size=256M
innodb_log_file_size=128M
innodb_thread_concurrency=128
innodb_autoextend_increment=1000
innodb_buffer_pool_instances=8
innodb_concurrency_tickets=5000
innodb_old_blocks_time=1000
innodb_open_files=300
innodb_stats_on_metadata=0
innodb_file_per_table=1
innodb_checksum_algorithm=0
back_log=80
flush_time=0
join_buffer_size=128M
max_allowed_packet=1024M
max_connect_errors=2000
open_files_limit=4161
query_cache_type=0
sort_buffer_size=32M
table_definition_cache=1400
binlog_row_event_max_size=8K
sync_master_info=10000
sync_relay_log=10000
sync_relay_log_info=10000
#批量插入数据缓存大小，可以有效提高插入效率，默认为8M
bulk_insert_buffer_size = 64M
interactive_timeout = 120
wait_timeout = 120
log-bin-trust-function-creators=1
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
```

请根据自己的实际情况更改对应的文件目录及端口号等信息。

6、启动 MySQL 服务，执行命令：

```
[root@hecs-x-large-2-linux-xx bin]# /etc/init.d/mysql.server start
Starting MySQL..                                           [  确定  ]
```

如果启动出现如下错误信息：

```
Starting MySQL.Logging to '/data/mysql/SZY.err'.
2018-07-02T10:09:03.779928Z mysqld_safe The file /usr/local/mysql/bin/mysqld
does not exist or is not executable. Please cd to the mysql installation
directory and restart this script from there as follows:
./bin/mysqld_safe&
See http://dev.mysql.com/doc/mysql/en/mysqld-safe.html for more information
ERROR! The server quit without updating PID file (/software/mysql/mysqld.pid).
```

这是因为新版本的 MySQL 安全启动安装包只认 /usr/local/mysql 这个路径。

**解决办法：**

##### 方法1、建立软连接，如下所示执行命令：

```
[root@hecs-x-large-2-linux-xx bin]# cd /usr/local/mysql
[root@hecs-x-large-2-linux-xx bin]# ln -s /usr/local/mysql57/bin/myslqd mysqld
```

##### 方法2、修改 mysqld_safe 文件（有强迫症的同学建议这种）

```
[root@hecs-x-large-2-linux-xx bin]# vim /usr/local/mysql57/bin/mysqld_safe
```

将所有出现的“/usr/local/mysql” 更改为 “/usr/local/mysql57”

保存退出。（可以将这个文件拷出来修改然后再进行替换）

### 五、登录 MySQL 更改 root 账户密码

#### 1、本地登录 MySQL ，执行命令：

```
[root@hecs-x-large-2-linux-xx bin]# /usr/local/mysql57/bin/mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.29

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

注意⚠️：这里需要输入密码，此处的密码就是上面初始化数据库时让大家特别留意的临时密码。

此时，执行 MySQL 语句是会出现如下错误信息的，大概意思就是说必须要重置当前登录的 root 账户的密码：

```
mysql> show databases;
ERROR 2006 (HY000): MySQL server has gone away
No connection. Trying to reconnect...
Connection id:    4
Current database: *** NONE ***

ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
mysql> 
```

#### 2、重置 root 账户密码，执行SQL语句如下：

```
mysql> set password=password('123456');
ERROR 2006 (HY000): MySQL server has gone away
No connection. Trying to reconnect...
Connection id:    5
Current database: *** NONE ***

Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> 
```

如上所示，发现执行失败，大概意思是说没有得到有效的连接。

此时我本以为是端口问题，当我退出 MySQL 控制台后，重新指定端口登录链接则告知我密码错误：

```
[root@hecs-x-large-2-linux-xx bin]# /usr/local/mysql57/bin/mysql -P53306 -u root -p
Enter password: 
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
```

这是，我只好跳过密码登录了，对应的修改 /etc/my.cnf 配置文件，执行命令：

```
[root@hecs-x-large-2-linux-xx bin]# vi /etc/my.cnf 
```

修改配置项：skip-grant-tables 将前面的符号(#)删除，即放开注释。
修改完成后按‘ESC’退出编辑模式，输入英文状态下的‘:wq’退出编辑文档。

修改完成后需要重新启动 MySQL 服务，执行命令：

```
[root@hecs-x-large-2-linux-xx bin]# /etc/init.d/mysql.server restart
Shutting down MySQL..                                      [  确定  ]
Starting MySQL.                                            [  确定  ]
```

重启完成后再次登入 MySQL，执行命令：

```
[root@hecs-x-large-2-linux-xx bin]# /usr/local/mysql57/bin/mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.29 MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

这一次，由于配置跳过了密码验证，所以不需要输入密码即可登录了。

此时，咋们切换当前数据库为 ‘mysql’，然后修改 root 账户密码，执行SQL语句如下：

```
mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> update user set authentication_string=password('123456') where user='root' and Host='localhost';
Query OK, 0 rows affected, 1 warning (0.00 sec)
Rows matched: 1  Changed: 0  Warnings: 1

mysql> 
```

注意⚠️：MySQL 5.7 版本对应到密码存储字段是‘authentication_string’而不是旧版本中的'password'，而且需要采用 password 函数进行加密后存储，否则修改失败或者修改后无法登录。

如果在执行该步骤的时候出现如下类似错误：

```
ERROR 1290 (HY000): The MySQL server is running with the --skip-grant-tables option so it cannot execute this statement 
```

则执行如下SQL命令即可：

```
mysql> flush privileges;
```

如上所示，修改 root 账户密码成功。这时，我们需要调整刚才的配置项，继续采用密码登录，不然不安全。

即：改回 /etc/my.cnf 配置文件中的"# skip-grant-tables"，将此项配置注释即可！
然后重启 MySQL 服务并采用账户密码登录。

```
[root@hecs-x-large-2-linux-xx bin]# /etc/init.d/mysql.server restart
Shutting down MySQL..                                      [  确定  ]
Starting MySQL.                                            [  确定  ]
[root@hecs-x-large-2-linux-xx bin]# /usr/local/mysql57/bin/mysql -P53306 -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.29 MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

至此，修改 root 账户密码操作完成!

如果想创建一个账户用于远程登录请继续往下看。

### 六、添加 MySQL 登录用户及授权，并配置远程登录

#### 1、创建 MySQL 登录账户，进入 MySQL 控制台，执行SQL语句：

创建用户的SQL语法格式如下：

```
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
```

我需要创建的账户名为：mysqlhy，执行SQL语句如下：

```
mysql> use mysql;
Database changed
mysql> CREATE USER 'mysqlhy'@'%' IDENTIFIED BY '123456';
Query OK, 0 rows affected (0.00 sec)

mysql> 
```

注意：必须将当前数据库切换到 mysql 。
如果创建的账户是只用于本地登录，则 @ 符号后面填写 ‘localhost’即可；
如果需要指定给对应的主机登录，则 @ 符号后面填写对应主机的IP地址即可；
如果需要远程登录，则 @ 符号后面必须填写百分符号，即: % 。

#### 2、删除 MySQL 用户

如果用户创建错了，肯定要支持删除操作，使用的 SQL 语句格式如下：

```
DROP USER 'username'@'host';
```

#### 3、给创建的 MySQL 用户授权

授权有多种场景，下面将列举几个常见的场景：

##### 1、授权 mysqlhy 用户有 testDB 数据库的某一部分权限，执行 MySQL 语法如下：

```
grant select,update on testDB.* to mysqlhy@'%' identified by '123456';
```

上面示例的部分权限：select,update 还可以添加 create,drop,alter等。

##### 2、授权 mysqlhy 用户有 testDB 数据库的所有操作权限，执行 MySQL 语法如下：

```
grant all privileges on testDB.* to 'mysqlhy'@'%' identified by '123456';
```

##### 3、授权 mysqlhy 用户拥有所有数据库的某些权限，执行 MySQL 语法如下：

```
grant select,delete,update,create,drop on *.* to 'mysqlhy'@'%' identified by '123456';
```

##### 4、授权 mysqlhy 用户拥有所有数据库的所有权限，执行 MySQL 语法如下：

```
grant all privileges on *.* to 'mysqlhy'@'%' identified by '123456';
```

温馨提示：identified by 后面即你需要设置的权限密码，最好设置三种以上字符，保证安全性。

操作示例：

```
mysql> use mysql;
Database changed
mysql> GRANT ALL privileges ON *.* TO 'mysqlhy'@'%' IDENTIFIED BY 'Abc1234).' WITH GRANT OPTION;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> 
```

至此，创建 MySQL 用户并授权远程登录就完成了！

#### 4、验证远程登录

结尾咋们来验证一下远程登录效果，此处采用的链接工具是 Navicat Premium
链接参数如下；
![image.png](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1587199232135.png)

提示“连接成功”则说明配置没有问题，成功了！
