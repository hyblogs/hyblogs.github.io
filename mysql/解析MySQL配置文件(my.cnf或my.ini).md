# 解析 MySQL 配置文件(my.cnf或my.ini)



&emsp;&emsp;在我们平时安装 MySQL 数据库服务5.7(5.7.17)以上版本的时候通常会碰到需要创建一个 MySQL 配置文件，这个配置文件就是 my.cnf 或者 my.ini。因为从5.7(5.7.17)开始 MySQL 二进制安装包中就不再提供 my-default.cnf 文件，很多同学肯定是百度一篇安装教程，然后到了要创建或修改这个配置文件时自然就采用了CV，但是其中有些配置项不一定都适合你的安装环境或者你的使用过场景，接下来咋们就一起来认识一下这个配置文件里面的相关配置项。
(PS：上面提到的 CV 实际上就是 Ctrl + C And Ctrl + V )(>^ω^<)

&emsp;&emsp;在 Windows 系统环境下 MySQL 的这个配置文件叫 my.ini ，但是在 CentOS 系统环境下这个配置文件就叫 my.cnf ，他们的后缀不同；但是里面的配置项基本上都是相通的。

### 一、简单认识下 my.cnf

&emsp;&emsp;my.cnf 文件就是把在命令行上启动 MySQL 时后面的参数用cnf 文件的形式配置好，那么启动时就不再需要在命令上加入参数。
&emsp;&emsp;经过简单测试，在 MySQL 5.7.18 版本中，使用 tar 包安装时，已经不再需要 my.cnf 文件也能正常运行了。

&emsp;&emsp;而且 my.cnf 文件可以是自定义位置，也可以使用如下默认的位置，只要放在默认位置，MySQL 自动识别（通过deb或者APT源安装的，初始位置在下方列表）：

| 文件位置                | 选项                                |
| ------------------- | --------------------------------- |
| /etc/my.cnf         | 全局选项                              |
| /etc/mysql/my.cnf   | 全局选项                              |
| SYSCONFDIR/my.cnf   | 全局选项                              |
| $MYSQL_HOME/my.cnf  | 服务器特定选项（仅限服务器）                    |
| defaults-extra-file | 指定的文件 --defaults-extra-file，如果有的话 |
| ~/.my.cnf           | 用户特定选项                            |
| ~/.mylogin.cnf      | 用户特定的登录路径选项（仅限客户端）                |

### 二、my.cnf 常见配置项

下面列举一些比较常见的配置信息，虽然不全，但是可以参考一下：

```
# 客户端设置，即客户端默认的连接参数，在如下节点[client]中进行配置:
[client]
# 连接端口号，默认 3306
port = 3306
# 用于本地连接的 socket 套接字
socket = /usr/local/mysql57/mysql.sock
# 字符编码
default-character-set = utf8mb4

# 服务端基础设置，即 MySQL 数据库服务器端基本设置参数，在如下节点[mysqld]中进行配置：
[mysqld]
# MySQL 服务的唯一编号，每个 MySQL 服务 Id 须唯一
server-id = 1
# 服务端口号，默认 3306
port = 3306
# MySQL 启动用户
user = mysql
# 免密登录(忘记root账户密码后可以配置此项免密登录 MySQL 控制台，然后修改 root 账户密码，切记修改密码后将此项注释掉)
skip-grant-tables

# MySQL 安装根目录(主目录)
basedir = /usr/local/mysql57
# MySQL 数据文件所在位置(储存数据的文件夹)
datadir = /data/mysql
# pid 文件所在目录
pid-file = /usr/local/mysql57/mysqld.pid
# 临时目录(比如 load data infile 会用到)
tmpdir = /tmp

# 设置 socket 文件所在目录(即为 MySQL 客户端程序和服务器之间的本地通讯指定一个套接字文件)
socket = /usr/local/mysql57/mysql.sock
# 主要用于 MyISAM 存储引擎,如果多台服务器连接一个数据库则建议注释下面内容(跳过外部锁定, External-locking 用于多进程条件下为 MyISAM 数据表进行锁定)
skip-external-locking
# 只能用 IP 地址检查客户端的登录，不用主机名
skip_name_resolve = 1
# secure_auth 为了防止低版本的 MySQL 客户端(<4.1)使用旧的密码认证方式访问高版本的服务器。MySQL 5.6.7 开始 secure_auth 默认为启用值 1
secure_auth = 1
# 跳过客户端域名解析；当新的客户连接 mysqld 时，mysqld 创建一个新的线程来处理请求。该线程先检查是否主机名在主机名缓存中。如果不在，线程试图解析主机名。(使用这一选项以消除MySQL进行DNS解析的时间。但需要注意，如果开启该选项，则所有远程主机连接授权都要使用IP地址方式，否则MySQL将无法正常处理连接请求!)
skip-name-resolve

# MySQL绑定IP
bind-address = 127.0.0.1

# 事务隔离级别，默认为 可重复读，MySQL 默认可重复读级别(此级别下可能产生很多间隙锁，影响性能)
transaction_isolation = READ-COMMITTED
# 数据库默认字符集,主流字符集支持一些特殊表情符号(特殊表情符占用4个字节)

# 性能优化的引擎，默认关闭
performance_schema = 0

# 开启全文索引
ft_min_word_len = 1

# 自动修复MySQL的 MyISAM 表
myisam_recover 

character-set-server = utf8mb4
# 数据库字符集对应一些排序等规则，注意要和 character-set-server 对应
collation-server = utf8mb4_general_ci
# 设置client 连接 MySQL 时的字符集,防止乱码
init_connect='SET NAMES utf8mb4'

# 是否对 SQL 语句大小写敏感，1-表示不敏感(即不区分大小写)
lower_case_table_names = 1

# 最大连接数
max_connections = 400
# 最大错误连接数
max_connect_errors = 1000

# TIMESTAMP 如果没有显示声明 NOT NULL，允许 NULL 值
explicit_defaults_for_timestamp = true

# MySQL 连接闲置超过一定时间后(单位：秒)将会被强行关闭。(MySQL默认的wait_timeout  值为8个小时, interactive_timeout 参数需要同时配置才能生效)(interactive_time -- 指的是 MySQL 在关闭一个交互的连接之前所要等待的秒数(交互连接如 mysql gui tool 中的连接)
interactive_timeout = 1800
wait_timeout = 1800

# 内部内存临时表的最大值 ，设置成128M。(比如大数据量的 group by ,order by 时可能用到临时表，超过了这个值将写入磁盘，系统 IO 压力增大)
tmp_table_size = 134217728
max_heap_table_size = 134217728

# 禁用 MySQL 的缓存查询结果集功能。(后期根据业务情况测试决定是否开启,大部分情况下关闭下面两项)
query_cache_size = 0
query_cache_type = 0

# 计划任务(事件调度器)
event_scheduler

# 为了安全起见，复制环境的数据库还是设置--skip-slave-start参数，防止复制随着 MySQL 启动而自动启动
skip-slave-start

#设定是否支持命令 load data local infile。如果指定local关键词，则表明支持从客户主机读文件
local-infile = 0
#指定 MySQL 可能的连接数量。当MySQL主线程在很短的时间内得到非常多的连接请求，该参数就起作用，之后主线程花些时间（尽管很短）检查连接并且启动一个新线程。(back_log参数的值指出在MySQL暂时停止响应新请求之前的短时间内多少个请求可以被存在堆栈中)
back_log = 1024

# sql_mode,定义了 MySQL 应该支持的 SQL 语法，数据校验等!(sql_mode 配置项有：'PIPES_AS_CONCAT,ANSI_QUOTES,IGNORE_SPACE,NO_KEY_OPTIONS,NO_TABLE_OPTIONS,NO_FIELD_OPTIONS,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION')
# NO_ENGINE_SUBSTITUTION 如果需要的存储引擎被禁用或未编译，可以防止自动替换存储引擎；
# NO_AUTO_CREATE_USER：禁止GRANT创建密码为空的用户。
sql_mode = NO_ENGINE_SUBSTITUTION,NO_AUTO_CREATE_USER　　

# 索引块的缓冲区大小，对 MyISAM 表性能影响最大的一个参数，决定索引处理的速度，尤其是索引读的速度。默认值是16M。(通过检查状态值Key_read_requests 和 Key_reads，可以知道 key_buffer_size  设置是否合理)
key_buffer_size = 32M

# 一个查询语句包的最大尺寸。消息缓冲区被初始化为 net_buffer_length 字节，但是可在需要时增加到 max_allowed_packet 个字节。该值太小则会在处理大包时产生错误。如果使用大的 BLOB 列，必须增加该值。这个值来限制server 接受的数据包大小。有时候大的插入和更新会受 max_allowed_packet 参数限制，导致写入或者更新失败。
max_allowed_packet = 512M　　　　　　　　　　

# 线程缓存；主要用来存放每一个线程自身的标识信息，如线程id，线程运行时基本信息等等，我们可以通过 thread_stack 参数来设置为每一个线程栈分配多大的内存。
thread_stack = 256K　　　　　　　　　　　　　

# 是 MySQL 执行排序使用的缓冲大小。如果想要增加 ORDER BY 的速度，首先看是否可以让 MySQL 使用索引而不是额外的排序阶段。如果不能，可以尝试增加 sort_buffer_size 变量的大小。
sort_buffer_size = 16M

# 每个session将会分配参数设置的内存大小，用于表的顺序扫描，读出的数据暂存于read_buffer_size中，当buff满时或读完，将数据返回上层调用者(一般在128kb ~ 256kb,用于MyISAM)。
read_buffer_size = 131072

# 应用程序经常会出现一些两表(或多表) Join 的操作需求，MySQL 在完成某些 Join 需求的时候（all/index join），为了减少参与 Join 的“被驱动表”的读取次数以提高性能，需要使用到 Join Buffer 来协助完成 Join 操作。当 Join Buffer 太小，MySQL 不会将该 Buffer 存入磁盘文件，而是先将 Join Buffer 中的结果集与需要 Join 的表进行 Join 操作，然后清空 Join Buffer 中的数据，继续将剩余的结果集写入此 Buffer 中，如此往复。这势必会造成被驱动表需要被多次读取，成倍增加 IO 访问，降低效率。
join_buffer_size = 16M

# 此项是 MySQL 的随机读缓冲区大小。当按任意顺序读取行时(例如：按照排序顺序)，将分配一个随机读缓存区。进行排序查询时，MySQL 会首先扫描一遍该缓冲，以避免磁盘搜索，提高查询速度，如果需要排序大量数据，可适当调高该值。但 MySQL 会为每个客户连接发放该缓冲空间，所以应尽量适当设置该值，以避免内存开销过大。
read_rnd_buffer_size = 32M

# 通信缓冲区在查询期间被重置到该大小。通常不要改变该参数值，但是如果内存不足，可以将它设置为查询期望的大小。(即，客户发出的SQL语句期望的长度。如果语句超过这个长度，缓冲区自动地被扩大，直到 max_allowed_packet 个字节)
net_buffer_length = 16K

# 当对 MyISAM 表执行 repair table 或创建索引时，用以缓存排序索引；设置太小时可能会遇到” myisam_sort_buffer_size is too small”
myisam_sort_buffer_size = 128M

# 默认8M，当对 MyISAM 非空表执行 insert … select/ insert … values(…),(…) 或者 load data infile 时，使用树状 cache 缓存数据，每个 thread 分配一个。(注：当对 MyISAM 表 load 大文件时，调大bulk_insert_buffer_size/myisam_sort_buffer_size/key_buffer_size 会极大提升速度)
bulk_insert_buffer_size = 32M

# thread_cahe_size 线程池，线程缓存。用来缓存空闲的线程，以至于不被销毁，如果线程缓存在的空闲线程，需要重新建立新连接，则会优先调用线程池中的缓存，很快就能响应连接请求。每建立一个连接，都需要一个线程与之匹配。
thread_cache_size = 384

# 工作原理： 一个 SELECT 查询在 DB 中工作后，DB 会把该语句缓存下来，当同样的一个 SQL 再次来到 DB 里调用时，DB 在该表没发生变化的情况下把结果从缓存中返回给 Client (客户端)。(在数据库写入量或是更新量也比较大的系统，该参数不适合分配过大。而且在高并发，写入量大的系统，建议把该功能禁掉)
query_cache_size = 0

# 决定是否缓存查询结果。这个变量有三个取值：0,1,2，分别代表了 off、on、demand。它规定了内部内存临时表的最大值，每个线程都要分配。（实际起限制作用的是 tmp_table_size 和 max_heap_table_size 的最小值。）如果内存临时表超出了限制，MySQL 就会自动地把它转化为基于磁盘的 MyISAM表，存储在指定的 tmpdir 目录下。
query_cache_type = 0

# 独立的内存表所允许的最大容量。此选项为了防止意外创建一个超大的内存表导致用尽所有的内存资源。
max_heap_table_size = 512M

# 设置 MySQL 打开最大文件数
open_files_limit = 10240

# MySQL 无论如何都会保留一个用于管理员（SUPER）登陆的连接，用于管理员连接数据库进行维护操作，即使当前连接数已经达到了 max_connections。因此 MySQL 的实际最大可连接数为 max_connections+1；这个参数实际起作用的最大值（实际最大可连接数）为16384，即该参数最大值不能超过16384，即使超过也以16384为准；增加 max_connections 参数的值，不会占用太多系统资源。系统资源（CPU、内存）的占用主要取决于查询的密度、效率等；该参数设置过小的最明显特征是出现”Too many connections”错误；
max_connections = 2000

# 此项配置是用来限制用户资源的，0-不限制；对整个服务器的用户限制。
max_user_connections = 0

# max_connect_errors 是一个 MySQL 中与安全有关的计数器值，它负责阻止过多尝试失败的客户端以防止暴力破解密码的情况。max_connect_errors 的值与性能并无太大关系。当此值设置为10时，意味着如果某一客户端尝试连接此 MySQL 服务器，但是失败（如密码错误等等）10次，则 MySQL 会无条件强制阻止此客户端连接。
max_connect_errors = 100000　　　　　　　　 

# 表描述符缓存大小，可减少文件打开/关闭次数；
table_open_cache = 5120

# 二进制日志缓冲大小。(我们知道 InnoDB 存储引擎是支持事务的，实现事务需要依赖于日志技术，为了性能，日志编码采用二进制格式。那么，我们如何记日志呢？有日志的时候，就直接写磁盘？可是磁盘的效率是很低的，如果你用过Nginx，，一般 Nginx 输出 access log 都是要缓冲输出的。因此，记录二进制日志的时候，我们是否也需要考虑 Cache 呢？答案是肯定的，但是 Cache 不是直接持久化，于是面临安全性的问题----因为系统宕机时，Cache 中可能有残余的数据没来得及写入磁盘。因此，Cache 要权衡，要恰到好处：既减少磁盘I/O，满足性能要求；又保证 Cache 无残留，及时持久化，满足安全要求)
binlog_cache_size = 16M

# 开启慢查询
slow_query_log = 1
# 超过的时间为1s；MySQL 能够记录执行时间超过参数 long_query_time 设置值的SQL语句。(默认是不记录的)
long_query_time = 1
# 对于管理型 SQL 可以通过--log-slow-admin-statements 开启记录管理型慢SQL。
log_slow_admin_statements
# 记录管理语句和没有使用index的查询记录
log_queries_not_using_indexes

# 在复制方面的改进就是引进了新的复制技术：基于行的复制。简言之，这种新技术就是关注表中发生变化的记录，而非以前的照抄 binlog 模式。从 MySQL 5.1.12 开始，可以用以下三种模式来实现：基于 SQL 语句的复制(statement-based replication, SBR)，基于行的复制(row-based replication, RBR)，混合模式复制(mixed-based replication, MBR)。相应地，binlog 的格式也有三种：STATEMENT，ROW，MIXED。MBR 模式中，SBR 模式是默认的。
binlog_format = ROW

# 为每个session 最大可分配的内存，在事务过程中用来存储二进制日志的缓存。
max_binlog_cache_size = 102400

# 开启二进制日志功能，binlog 数据位置
log_bin = /data/mysql/binlog/mysql-bin
log_bin_index = /data/mysql/binlog/mysql-bin.index
# relay-log日志记录的是从服务器I/O线程将主服务器的二进制日志读取过来记录到从服务器本地文件，然后SQL线程会读取relay-log日志的内容并应用到从服务器。
relay_log = /data/mysql/relay/mysql-relay-bin

# binlog 传到备机被写到 relaylog 里，备机的 slave sql 线程从relaylog 里读取然后应用到本地。
relay_log_index = /data/mysql/relay/mysql-relay-bin.index 

# log_slave_updates 是将从服务器从主服务器收到的更新记入到从服务器自己的二进制日志文件中。
log_slave_updates = 1
# 二进制日志自动删除的天数。默认值为0,表示“没有自动删除”。启动时和二进制日志循环时可能删除。
expire_logs_days = 15
# 如果二进制日志写入的内容超出给定值，日志就会发生滚动。你不能将该变量设置为大于1GB或小于4096字节。 默认值是1GB。
max_binlog_size = 512M

# replicate_wild_ignore_table 参数能同步所有跨数据库的更新，比如replicate_do_db 或者 replicate_ignore_db 。
replicate_wild_ignore_table = mysql.%　　

# 设定需要复制的Table
replicate_wild_do_table = db_name.%　　 

# 复制时跳过一些错误;不要胡乱使用这些跳过错误的参数，除非你非常确定你在做什么。当你使用这些参数时候，MYSQL 会忽略那些错误，这样会导致你的主从服务器数据不一致。
slave_skip_errors = 1062,1053,1146

# 这两个参数一般用在主主同步中，用来错开自增值, 防止键值冲突。
auto_increment_offset = 1
auto_increment_increment = 2

# 将中继日志的信息写入表:mysql.slave_realy_log_info
relay_log_info_repository = TABLE
# 将 master 的连接信息写入表：mysql.salve_master_info
master_info_repository = TABLE
# 中继日志自我修复；当 slave 从库宕机后，假如 relay-log 损坏了，导致一部分中继日志没有处理，则自动放弃所有未执行的 relay-log，并且重新从master 上获取日志，这样就保证了 relay-log 的完整性。
relay_log_recovery = on

# InnoDB 用来高速缓冲数据和索引内存缓冲大小。 更大的设置可以使访问数据时减少磁盘 I/O。
innodb_buffer_pool_size = 4G

# 单独指定数据文件的路径与大小
innodb_data_file_path = ibdata1:1G:autoextend

# 每次 commit 日志缓存中的数据刷到磁盘中。通常设置为 1，意味着在事务提交前日志已被写入磁盘，事务可以运行更长以及服务崩溃后的修复能力。如果你愿意减弱这个安全，或你运行的是比较小的事务处理，可以将它设置为 0 ，以减少写日志文件的磁盘 I/O。这个选项默认设置为 0。
innodb_flush_log_at_trx_commit = 0

# sync_binlog = n，当每进行 n 次事务提交之后，MySQL 将进行一次fsync 之类的磁盘同步指令来将 binlog_cache 中的数据强制写入磁盘。
sync_binlog = 1000

# 对于多核的CPU机器，可以修改innodb_read_io_threads和innodb_write_io_threads 来增加 I/O 线程，来充分利用多核的性能。
innodb_read_io_threads = 8　
innodb_write_io_threads = 8

# 我们可以通过设置配置参数innodb_thread_concurrency来限制并发线程的数量，一旦执行线程的数量达到这个限制，额外的线程在被放置到对队列中之前，会睡眠数微秒，可以通过设定参数innodb_thread_sleep_delay来配置睡眠时间。(默认值为0，它表示默认情况下不限制线程并发执行的数量)
innodb_thread_concurrency = 128

# 可配置此参数进行并发控制，直到tickets用完，线程进入等待或清理不活跃的线程。(默认值：5000)
innodb_concurrency_tickets = 5000

# Innodb Plugin 引擎开始引入多种格式的行存储机制，目前支持：Antelope、Barracuda 两种。其中 Barracuda 兼容 Antelope 格式。
innodb_file_format = Barracuda

# 限制 InnoDB 能打开的表的数量
innodb_open_files = 65536
# 开始碎片回收线程。这个应该能让碎片回收得更及时而且不影响其他线程的操作。
innodb_purge_threads = 1
# 分布式事务
innodb_support_xa = FALSE
# InnoDB 将日志写入日志磁盘文件前的缓冲大小。理想值为 1M 至 8M。大的日志缓冲允许事务运行时不需要将日志保存入磁盘而直到事务被提交(commit)。因此，如果有大的事务处理，设置大的日志缓冲可以减少磁盘I/O。
innodb_log_buffer_size = 8M

# 日志组中的每个日志文件的大小(单位 MB)。如果 n 是日志组中日志文件的数目，那么理想的数值为 1M 至下面设置的缓冲池(buffer pool)大小的 1/n。较大的值，可以减少刷新缓冲池的次数，从而减少磁盘 I/O。但是大的日志文件意味着在崩溃时需要更长的时间来恢复数据。
innodb_log_file_size = 1G
# 指定有三个日志组
innodb_log_files_in_group = 3　　　　　　
# 在回滚(rooled back)之前，InnoDB 事务将等待超时的时间(单位 秒)
innodb_lock_wait_timeout = 120

# innodb_max_dirty_pages_pct 作用：控制 InnoDB 的脏页在缓冲中在那个百分比之下，值在范围1-100,默认为90。这个参数的另一个用处：当InnoDB 的内存分配过大，致使 swap 占用严重时，可以适当的减小调整这个值，使达到 swap 空间释放出来。建义：这个值最大在90%，最小在15%。太大，缓存中每次更新需要致换数据页太多，太小，放的数据页太小，更新操作太慢。
innodb_max_dirty_pages_pct = 75　　　　　

# innodb_buffer_pool_size 一致 可以开启多个内存缓冲池，把需要缓冲的数据 hash 到不同的缓冲池中，这样可以并行的内存读写。
innodb_buffer_pool_instances = 4

# 这个参数据控制 Innodb checkpoint 时的IO能力
innodb_io_capacity = 500

# 作用：使每个InnoDB 的表，有自已独立的表空间。如删除文件后可以回收那部分空间。分配原则：只有使用不使用。但 DB 还需要有一个公共的表空间。
innodb_file_per_table = 1

# 当更新/插入的非聚集索引的数据所对应的页不在内存中时(对非聚集索引的更新操作通常会带来随机I/O)，会将其放到一个 insert buffer 中，当随后页面被读到内存中时，会将这些变化的记录 merge 到页中。当服务器比较空闲时，后台线程也会做 merge 操作。
innodb_change_buffering = inserts

# 该值影响每秒刷新脏页的操作，开启此配置后，刷新脏页会通过判断产生重做日志的速度来判断最合适的刷新脏页的数量；
innodb_adaptive_flushing = 1

# innodb_flush_method 这个参数控制着 InnoDB 数据文件及 redo log 的打开、刷写模式。InnoDB 使用 O_DIRECT 模式打开数据文件，用 fsync()函数去更新日志和数据文件。
innodb_flush_method = O_DIRECT

# 默认设置值为1。设置为0：表示 InnoDB 使用自带的内存分配程序；设置为1：表示 InnoDB 使用操作系统的内存分配程序。
#innodb_use_sys_malloc = 1

# 将所有发生的死锁错误信息记录到 error.log 中，之前通过命令行只能查看最近一次死锁信息
innodb_print_all_deadlocks = 1

[mysqldump]
# 此项配置会强制 mysqldump 从服务器查询取得记录直接输出而不是取得所有记录后将它们缓存到内存中。
quick
max_allowed_packet = 128M

# 限制 server 接受的数据包大小;指 MySQL 服务器端和客户端在一次传送数据包的过程当中数据包的大小。
max_allowed_packet = 512M
# TCP/IP 和套接字通信缓冲区大小,创建长度达 net_buffer_length 的行。
net_buffer_length = 16384

[mysql]
# auto_rehash 是自动补全的意思
auto_rehash

# isamchk 数据检测恢复工具
[isamchk]
key_buffer = 256M
sort_buffer_size = 256M
read_buffer = 2M
write_buffer = 2M

# 使用myisamchk实用程序来获得有关你的数据库桌表的信息、检查和修复他们或优化他们
[myisamchk]
key_buffer = 256M
sort_buffer_size = 256M
read_buffer = 2M
write_buffer = 2M

# mysqlhotcopy 使用 lock tables、flush tables 和 cp或scp 来快速备份数据库。它是备份数据库或单个表最快的途径,完全属于物理备份,但只能用于备份 MyISAM 存储引擎和运行在数据库目录所在的机器上。与 mysqldump 备份不同,mysqldump 属于逻辑备份,备份时是执行的 SQL 语句.使用mysqlhotcopy 命令前需要要安装相应的软件依赖包.
[mysqlhotcopy]
# 参数含义：服务器关闭交互式连接前等待活动的秒数。交互式客户端定义为在mysql_real_connect() 中使用 CLIENT_INTERACTIVE 选项的客户端。
参数默认值：28800秒（8小时）
interactive_timeout
```

&emsp;&emsp;上面列举的 my.cnf 配置项颇多，但并不是所有人的 MySQL 都需要配置这么多信息，因此需要根据个人情况选择性添加对应的配置项即可！

### 三、常见配置项汇总

&emsp;&emsp;由于配置项很多，但并不是所有的配置项都适合所有人，所以下面会列举几种常用的配置项，可以复制过去稍微修改下即可使用。

#### 1、简化版配置项信息

```
[mysql]
# 设置 MySQL 客户端默认字符集
default-character-set = utf8mb4

[mysqld]
skip-name-resolve
# 设置3306端口
port = 3306 
# 设置 MySQL 的安装目录
basedir = /usr/local/mysql57
# 设置 MySQL 数据库的数据的存放目录
datadir = /usr/local/mysql57/data
# 允许最大连接数
max_connections = 200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server = utf8mb4_general_ci
# 数据库字符集对应一些排序等规则，注意要和 character-set-server 对应
collation-server = utf8mb4_general_ci
# 创建新表时将使用的默认存储引擎
default-storage-engine = INNODB 
# SQL 语句大小写敏感，1-表示不敏感
lower_case_table_names = 1
# 指 MySQL 服务器端和客户端在一次传送数据包的过程当中数据包的大小
max_allowed_packet = 16M
```

#### 2、复杂版配置项信息

```
[client]
# 连接端口号，默认 3306
port = 3306
# 用于本地连接的 socket 套接字
socket = /usr/local/mysql57/mysql.sock
# 连接客户端字符编码
default-character-set = utf8mb4

[mysql]
# 设置默认字符编码
default-character-set = utf8mb4

[mysqld]
# MySQL 服务的唯一编号，每个 MySQL 服务 Id 须唯一
server-id = 1
# MySQL 安装根目录(主目录)
basedir = /usr/local/mysql57
# MySQL 数据文件所在位置(储存数据的文件夹)
datadir = /data/mysql
# 服务端口号，默认 3306
port = 3306
# pid 文件所在目录
pid-file = /usr/local/mysql57/mysqld.pid
# 跳过客户端域名解析；当新的客户连接 mysqld 时，mysqld 创建一个新的线程来处理请求。该线程先检查是否主机名在主机名缓存中。如果不在，线程试图解析主机名。(使用这一选项以消除MySQL进行DNS解析的时间。但需要注意，如果开启该选项，则所有远程主机连接授权都要使用IP地址方式，否则MySQL将无法正常处理连接请求!)
skip-name-resolve
# 设置 socket 文件所在目录(即为 MySQL 客户端程序和服务器之间的本地通讯指定一个套接字文件)
socket = /usr/local/mysql57/mysql.sock
# 字符编码
character-set-server = utf8mb4
# 数据库字符集对应一些排序等规则，注意要和 character-set-server 对应
collation-server = utf8mb4_general_ci
# 设置client 连接 MySQL 时的字符集,防止乱码
init_connect='SET NAMES utf8mb4'
# 默认存储引擎
default-storage-engine = INNODB
# TIMESTAMP 如果没有显示声明 NOT NULL，允许 NULL 值
explicit_defaults_for_timestamp = true
# 最大连接数
max_connections = 2000
# 禁用 MySQL 的缓存查询结果集功能。(后期根据业务情况测试决定是否开启,大部分情况下关闭下面两项)
query_cache_size = 0
query_cache_type = 0
# 表描述符缓存大小，可减少文件打开/关闭次数；
table_open_cache = 2000
# 内部内存临时表的最大值 ，设置成128M。(比如大数据量的 group by ,order by 时可能用到临时表，超过了这个值将写入磁盘，系统 I/O 压力增大)
tmp_table_size = 128M
# thread_cahe_size 线程池，线程缓存。用来缓存空闲的线程，以至于不被销毁，如果线程缓存在的空闲线程，需要重新建立新连接，则会优先调用线程池中的缓存，很快就能响应连接请求。每建立一个连接，都需要一个线程与之匹配。
thread_cache_size = 300
# 限定用于每个数据库线程的栈大小。默认设置足以满足大多数应用
thread_stack = 192k
# 索引块的缓冲区大小，对 MyISAM 表性能影响最大的一个参数，决定索引处理的速度，尤其是索引读的速度。默认值是16M。(通过检查状态值Key_read_requests 和 Key_reads，可以知道 key_buffer_size  设置是否合理)
key_buffer_size = 512M
# 每个session将会分配参数设置的内存大小，用于表的顺序扫描，读出的数据暂存于read_buffer_size中，当buff满时或读完，将数据返回上层调用者(一般在128kb ~ 256kb,用于MyISAM)。
read_buffer_size = 4M
# 应用程序经常会出现一些两表(或多表) Join 的操作需求，MySQL 在完成某些 Join 需求的时候（all/index join），为了减少参与 Join 的“被驱动表”的读取次数以提高性能，需要使用到 Join Buffer 来协助完成 Join 操作。当 Join Buffer 太小，MySQL 不会将该 Buffer 存入磁盘文件，而是先将 Join Buffer 中的结果集与需要 Join 的表进行 Join 操作，然后清空 Join Buffer 中的数据，继续将剩余的结果集写入此 Buffer 中，如此往复。这势必会造成被驱动表需要被多次读取，成倍增加 I/O 访问，降低效率。
join_buffer_size = 16M
# 此项是 MySQL 的随机读缓冲区大小。当按任意顺序读取行时(例如：按照排序顺序)，将分配一个随机读缓存区。进行排序查询时，MySQL 会首先扫描一遍该缓冲，以避免磁盘搜索，提高查询速度，如果需要排序大量数据，可适当调高该值。但 MySQL 会为每个客户连接发放该缓冲空间，所以应尽量适当设置该值，以避免内存开销过大。
read_rnd_buffer_size = 128M
# 如果想为 innodb tablespace 文件指定不同路径，那么必须在my.cnf文件中指定innodb_data_home_dir。
innodb_data_home_dir = /data/mysql
# 每次 commit 日志缓存中的数据刷到磁盘中。通常设置为 1，意味着在事务提交前日志已被写入磁盘，事务可以运行更长以及服务崩溃后的修复能力。如果你愿意减弱这个安全，或你运行的是比较小的事务处理，可以将它设置为 0 ，以减少写日志文件的磁盘 I/O。这个选项默认设置为 0。
innodb_flush_log_at_trx_commit = 0
# InnoDB 将日志写入日志磁盘文件前的缓冲大小。理想值为 1M 至 8M。大的日志缓冲允许事务运行时不需要将日志保存入磁盘而直到事务被提交(commit)。因此，如果有大的事务处理，设置大的日志缓冲可以减少磁盘I/O。
innodb_log_buffer_size = 16M
# InnoDB 用来高速缓冲数据和索引内存缓冲大小。 更大的设置可以使访问数据时减少磁盘 I/O。
innodb_buffer_pool_size = 256M
# 日志组中的每个日志文件的大小(单位 MB)。如果 n 是日志组中日志文件的数目，那么理想的数值为 1M 至下面设置的缓冲池(buffer pool)大小的 1/n。较大的值，可以减少刷新缓冲池的次数，从而减少磁盘 I/O。但是大的日志文件意味着在崩溃时需要更长的时间来恢复数据。
innodb_log_file_size = 128M
# 我们可以通过设置配置参数innodb_thread_concurrency来限制并发线程的数量，一旦执行线程的数量达到这个限制，额外的线程在被放置到对队列中之前，会睡眠数微秒，可以通过设定参数innodb_thread_sleep_delay来配置睡眠时间。(默认值为0，它表示默认情况下不限制线程并发执行的数量)
innodb_thread_concurrency = 128
# 为表空间定义里的最后一个数据文件指定autoextend属性。然后在文件耗尽空间之时，InnoDB以8MB为增量自动增加该文件的大小。增加的大小可以通过设置innodb_autoextend_increment值来配置,这个值以MB为单位，默认的是8。
innodb_autoextend_increment = 32
# innodb_buffer_pool_size 一致 可以开启多个内存缓冲池，把需要缓冲的数据 hash 到不同的缓冲池中，这样可以并行的内存读写。
innodb_buffer_pool_instances = 8
# 可配置此参数进行并发控制，直到tickets用完，线程进入等待或清理不活跃的线程。(默认值：5000)
innodb_concurrency_tickets = 5000
# 配置此参数让保留在 buffer pool 里面的数据在插入时候没有被改变list位置的时候的保存时间innodb_old_blocks_time，单位是毫秒，这个值的默认值是1000。如果增大这个值的话，就会让buffer pool里面很多页信息变老的速度变快，因为这些数据不会很快被内存中擦除的话，就会变成热数据而挤掉原有缓存的数据。(如果系统中有大量的大查询的话，设置innodb_old_blocks_time 能够较大的提供系统的稳定性)
innodb_old_blocks_time = 1000
# 限制 InnoDB 能打开的表的数量
innodb_open_files = 300
# 把innodb_stats_on_metadata设置成off(即数字0)，像TABLES、TABLE_CONSTRAINTS这些表，都是静态数据，访问时不作索引统计，因此会提高效率。(默认是on)
innodb_stats_on_metadata = 0
# 作用：使每个InnoDB 的表，有自已独立的表空间。如删除文件后可以回收那部分空间。分配原则：只有使用不使用。但 DB 还需要有一个公共的表空间。
innodb_file_per_table = 1
# checksum函数的算法，默认为crc32。可以设置的值有:innodb、crc32、none、strict_innodb、strict_crc32、strict_none
innodb_checksum_algorithm = 0
# 指定 MySQL 可能的连接数量。当MySQL主线程在很短的时间内得到非常多的连接请求，该参数就起作用，之后主线程花些时间（尽管很短）检查连接并且启动一个新线程。(back_log参数的值指出在MySQL暂时停止响应新请求之前的短时间内多少个请求可以被存在堆栈中)
back_log = 128
＃ 默认1800，最小值0。如果设置为一个非零值，所有表关闭每flush_time的的秒钟以释放资源并同步未刷新到磁盘上的数据。
flush_time = 0
# 一个查询语句包的最大尺寸。消息缓冲区被初始化为 net_buffer_length 字节，但是可在需要时增加到 max_allowed_packet 个字节。该值太小则会在处理大包时产生错误。如果使用大的 BLOB 列，必须增加该值。这个值来限制server 接受的数据包大小。有时候大的插入和更新会受 max_allowed_packet 参数限制，导致写入或者更新失败。
max_allowed_packet = 512M　　
# max_connect_errors 是一个 MySQL 中与安全有关的计数器值，它负责阻止过多尝试失败的客户端以防止暴力破解密码的情况。max_connect_errors 的值与性能并无太大关系。当此值设置为10时，意味着如果某一客户端尝试连接此 MySQL 服务器，但是失败（如密码错误等等）10次，则 MySQL 会无条件强制阻止此客户端连接。
max_connect_errors = 2000
# 设置 MySQL 打开最大文件数
open_files_limit = 10240
# 是 MySQL 执行排序使用的缓冲大小。如果想要增加 ORDER BY 的速度，首先看是否可以让 MySQL 使用索引而不是额外的排序阶段。如果不能，可以尝试增加 sort_buffer_size 变量的大小。
sort_buffer_size = 32M
＃默认400，范围400-524288;可以缓存表定义，它不使用文件描述符。
table_definition_cache = 1400
# 批量插入数据缓存大小，可以有效提高插入效率，默认为8M
bulk_insert_buffer_size = 64M
# MySQL 连接闲置超过一定时间后(单位：秒)将会被强行关闭。(MySQL默认的wait_timeout  值为8个小时, interactive_timeout 参数需要同时配置才能生效)(interactive_time -- 指的是 MySQL 在关闭一个交互的连接之前所要等待的秒数(交互连接如 mysql gui tool 中的连接)
interactive_timeout = 1800
wait_timeout = 1800
# 当二进制日志启用后，这个变量就会启用。它控制是否可以信任存储函数创建者，不会创建写入二进制日志引起不安全事件的存储函数。如果设置为0（默认值），用户不得创建或修改存储函数，除非它们具有除CREATE ROUTINE或ALTER ROUTINE特权之外的SUPER权限。 设置为0还强制使用DETERMINISTIC特性或READS SQL DATA或NO SQL特性声明函数的限制。 如果变量设置为1，MySQL不会对创建存储函数实施这些限制。 此变量也适用于触发器的创建。 
log-bin-trust-function-creators = 1
# sql_mode,定义了 MySQL 应该支持的 SQL 语法，数据校验等!(sql_mode 配置项有：'PIPES_AS_CONCAT,ANSI_QUOTES,IGNORE_SPACE,NO_KEY_OPTIONS,NO_TABLE_OPTIONS,NO_FIELD_OPTIONS,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION')
# NO_ENGINE_SUBSTITUTION 如果需要的存储引擎被禁用或未编译，可以防止自动替换存储引擎；
# NO_AUTO_CREATE_USER：禁止GRANT创建密码为空的用户。
sql_mode = NO_ENGINE_SUBSTITUTION,NO_AUTO_CREATE_USER
```

&emsp;&emsp;以上便是列举的一些在实际运用过程中会用到的配置项，不明白的同学可以简单的看看，或者针对某一个配置项需要深入了解的可以单独搜索相关的资料进行深究 。
&emsp;&emsp;后面有更新我也会继续进行补充！
