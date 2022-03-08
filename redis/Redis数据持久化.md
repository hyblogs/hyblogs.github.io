# Redis 数据持久化



> &emsp;&emsp;使用过 Redis 的同学应该都知道，Redis 是将数据存储到内存中的，如果遇到系统故障之类的情况，内存中的数据是会丢失的，这种情况会对现有的业务系统造成很大的影响。因此，很多时候我们需要持久化数据也就是将内存中的数据写入到硬盘里面，当系统故障重启后可以从硬盘里面将备份还原回来继续使用，有效防止数据丢失的风险。

## Redis 是如何支持持久化的

> &emsp;&emsp;Redis 不同于 Memcached 的很重要的一点就是，Redis 支持持久化，而且支持两种不同的持久化操作。Redis的一种持久化方式叫快照(snapshotting，RDB)，另一种方式是追加文件(append-onlyfile,AOF)。这两种方法各有千秋，下面会详细介绍这两种持久化方法是什么，怎么用，如何选择适合自己的持久化方法。

## 采用 RDB(snapshotting) 快照的方式进行持久化操作

> &emsp;&emsp;Redis 可以通过创建快照来获得存储在内存里面的数据在某个时间点上的副本。Redis 创建快照之后，可以对快照进行备份，可以将快照复制到其他服务器从而创建具有相同数据的服务器副本(Redis主从结构，主要用来提高Redis性能)，还可以将快照留在原地以便重启服务器的时候使用。这种方式也是默认的持久化方式，这种方式是就是将内存中数据以快照的方式写入到二进制文件中，默认的文件名为 dump.rdb。

### 默认配置项

> &emsp;&emsp;在 redis.conf 配置文件中默认有如下配置项：
> 
> ```conf
> save    900    1     #在900秒(15分钟)之后，如果至少有1个key发生变化，Redis就会自动触发 BGSAVE 命令创建快照。
> save    300    10    #在300秒(5分钟)之后，如果至少有10个key发生变化，Redis就会自动触发 BGSAVE 命令创建快照。
> save    60    10000    #在60秒(1分钟)之后，如果至少有10000个key发生变化，Redis就会自动触发 BGSAVE 命令创建快照。
> ```
> 
> ![redis.conf](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594361879223.png)
> 
> &emsp;&emsp;根据配置，快照将被写入 dbfilename 选项指定的文件里面，并存储在 dir 选项指定的路径下面。如果在新的快照文件创建完毕之前，Redis、系统或者硬件这三者中的任意一个崩溃了，那么 Redis 将丢失最近一次创建快照写入的所有数据。
> 
> ![dbfilename](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594361927454.png)
> 
> &emsp;&emsp;举个例子：假设 Redis 的上一个快照是 2：35 开始创建的，并且已经创建成功。下午 3：06 时，Redis 又开始创建新的快照，并且在下午 3：08 快照创建完毕之前，有 35 个键进行了更新。如果在下午 3：06 到 3：08 期间，系统发生了崩溃，导致 Redis 无法完成新快照的创建工作，那么 Redis 将丢失下午 2：35 之后写入的所有数据。另一方面，如果系统恰好在新的快照文件创建完毕之后崩溃，那么 Redis 将丢失 35 个键的更新数据。

### 创建快照的方式

> Redis 创建快照的办法有如下几种：
> 
> - BGSAVE 命令：
>   &emsp;&emsp;客户端向 Redis 发送 BGSAVE 命令来创建一个快照。对于支持 BGSAVE 命令的平台来说(基本上除了 Windows 平台外的其他平台都支持)，Redis 会调用 fork 来创建一个子进程，然后子进程负责将快照写入硬盘，而父进程则继续处理命令请求。
>   ![BGSAVE](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594370512038.png)
> 
> - SAVE 命令：
>   &emsp;&emsp;客户端还可以向 Redis 发送 SAVE 命令来创建一个快照，接到 SAVE 命令的 Redis 服务器在快照创建完毕之前不会再响应任何其他命令。SAVE 命令不常用，我们通常只会在没有足够内存去执行 BGSAVE 命令的情况下，又或者即使等待持久化操作执行完毕也无所谓的情况下，才会使用这个命令。
>   ![SAVE](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594370697683.png)
> 
> - save 选项——自动触发：
>   &emsp;&emsp;如果用户设置了 save 选项(如上文讲到的默认配置，一般会默认设置)，比如 save&emsp;60&emsp;10000，那么从 Redis 最近一次创建快照之后开始算起，当“60秒之内有10000次写入”这个条件被满足时，Redis 就会自动触发 BGSAVE 命令。不需要持久化，那么你可以注释掉所有的 save 行来停用保存功能。<br/>
>   **除了需要配置 save 以外其他相关的详细配置如下**：<br/>
>   1、stop-writes-on-bgsave-error ：默认值为 yes。当启用了 RDB 且最后一次后台保存数据失败，Redis 是否停止接收数据。这会让用户意识到数据没有正确持久化到磁盘上，否则没有人会注意到灾难（disaster）发生了。如果Redis 重启了，那么又可以重新开始接收数据了
>   2、rdbcompression：默认值是 yes。对于存储到磁盘中的快照，可以设置是否进行压缩存储。
>   3、rdbchecksum：默认值是 yes。在存储快照后，我们还可以让 redis 使用CRC64 算法来进行数据校验，但是这样做会增加大约 10% 的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能。
>   4、dbfilename：设置快照的文件名，默认是 dump.rdb
>   5、dir：设置快照文件的存放路径，这个配置项一定是个目录，而不能是文件名。
> 
> - SHUTDOWN 命令——自动触发：
>   &emsp;&emsp;当 Redis 通过 SHUTDOWN 命令接收到关闭服务器的请求时，或者接收到标准 TERM 信号时，会执行一个 SAVE 命令，阻塞所有客户端，不再执行客户端发送的任何命令，并在 SAVE 命令执行完毕之后关闭服务器。
> 
> - 一个 Redis 服务器连接到另一个 Redis 服务器——自动触发：
>   &emsp;&emsp;当一个 Redis 服务器连接到另一个 Redis 服务器，并向对方发送 SYNC 命令来开始一次复制操作的时候，如果主服务器目前没有执行 BGSAVE 操作，或者主服务器并非刚刚执行完 BGSAVE 操作，那么主服务器就会执行 BGSAVE 命令。
> 
> &emsp;**BGSAVE 和 SAVE 的对比**：
> ![BGSAVE & SAVE](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594371634461.png)

### RDB 的优势和劣势

> **1、优势**
> 
> （1）RDB 文件紧凑，全量备份，非常适合用于进行备份和灾难恢复。
> （2）采用 BGSAVE 方式生成 RDB 文件的时候，redis 主进程会 fork() 一个子进程来处理所有保存工作，主进程不需要进行任何磁盘 IO 操作。
> （3）RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快。

> **2、劣势**
> 
> &emsp;&emsp;RDB 快照是一次全量备份，存储的是内存数据的二进制序列化形式，存储上非常紧凑。当进行快照持久化时，会开启一个子进程专门负责快照持久化，子进程会拥有父进程的内存数据，父进程修改内存子进程不会反应出来，所以在快照持久化期间修改的数据不会被保存，可能丢失数据。即：如果系统真的发生崩溃，用户将丢失最近一次生成快照之后更改的所有数据。因此，快照持久化只适用于即使丢失一部分数据也不会造成一些大问题的应用程序。不能接受这个缺点的话，可以考虑 AOF 持久化。

## 采用 AOF(append-onlyfile)机制

> &emsp;&emsp;全量备份总是耗时的，有时候我们会采用一种更加高效的方式：AOF，工作机制很简单，redis 会将每一个收到的写命令都通过 write 函数追加到文件中。通俗的理解就是日志记录。
> 
> &emsp;&emsp;与快照持久化相比，AOF 持久化的实时性更好，因此已成为主流的持久化方案。默认情况下 Redis 没有开启 AOF 方式的持久化，可以通过appendonly 参数开启：`appendonly    yes`。开启 AOF 持久化后每执行一条会更改 Redis 中的数据的命令，Redis 就会将该命令写入硬盘中的 AOF 文件。AOF 文件的保存位置和 RDB 文件的位置相同，都是通过 dir 参数设置的，默认的文件名是 appendonly.aof

### AOF 持久化原理

> 每当有一个写命令过来时，就直接保存在我们的AOF文件中。
> 
> ![AOF](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594372313271.png)

### AOF 文件重写原理

> &emsp;&emsp;AOF 虽然在某个角度可以将数据丢失降低到最小而且对性能影响也很小，但是极端的情况下，体积不断增大的 AOF 文件很可能会用完硬盘空间。另外，如果 AOF 体积过大，那么还原操作执行时间就可能会非常长。为了解决 AOF 体积过大的问题，用户可以向 Redis 发送 BGREWRITEAOF 命令，这个命令会通过移除 AOF 文件中的冗余命令来重写(rewrite) AOF 文件来减小 AOF 文件的体积。BGREWRITEAOF 命令和 BGSAVE 创建快照原理十分相似，所以 AOF 文件重写也需要用到子进程，这样会导致性能问题和内存占用问题，和快照持久化一样。更糟糕的是，如果不加以控制的话，AOF 文件的体积可能会比快照文件大好几倍。
> ![BGREWRITEAOF](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594372559962.png)

### AOF 持久化方式

> AOF 的持久化方式共有三种，对应的配置项如下：
> 
> 1、`appendfsync  always  #每次有数据修改发生时都会写入 AOF 文件,这样会严重降低 Redis 的速度；`
> 2、`appendfsync  everysec  #每秒钟同步一次，显示地将多个写命令同步到硬盘；`
> 3、`appendfsync  no  #让操作系统决定何时进行同步。`
> 
> **三种持久化方式对应的应用场景**：
> 
> &emsp;&emsp;`appendfsync  always` 可以实现将数据丢失减到最少，不过这种方式需要对硬盘进行大量的写入而且每次只写入一个命令，十分影响 Redis 的速度。另外使用固态硬盘的用户谨慎使用 `appendfsync  always` 选项，因为这会明显降低固态硬盘的使用寿命。
> &emsp;&emsp;为了兼顾数据和写入性能，用户可以考虑`appendfsync 
>  everysec`选项，让 Redis 每秒同步一次 AOF 文件，Redis 性能几乎没受到任何影响。而且这样即使出现系统崩溃，用户最多只会丢失一秒之内产生的数据。当硬盘忙于执行写入操作的时候，Redis 还会优雅的放慢自己的速度以便适应硬盘的最大写入速度。
> &emsp;&emsp;`appendfsync  no` 选项一般不推荐，这种方案会使 Redis 丢失不定量的数据而且如果用户的硬盘处理写入操作的速度不够的话，那么当缓冲区被等待写入的数据填满时，Redis 的写入操作将被阻塞，这会导致 Redis 的请求速度变慢。虽然 AOF 持久化非常灵活地提供了多种不同的选项来满足不同应用程序对数据安全的不同要求，但 AOF 持久化也有缺陷 —— AOF 文件的体积太大。
> 
> ![AOF](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594373562566.png)
> 
> &emsp;&emsp;和快照持久化可以通过设置 save 选项来自动执行 BGSAVE 一样，AOF 持久化也可以通过设置 `auto-aof-rewrite-percentage` 选项和 `auto-aof-rewrite-min-size` 选项自动执行 `BGREWRITEAOF` 命令。
> &emsp;&emsp;举例：假设用户对 Redis 设置了如下配置选项并且启用了 AOF 持久化。那么当 AOF 文件体积大于 64mb，并且 AOF 的体积比上一次重写之后的体积大了至少一倍（100%）的时候，Redis 将执行 BGREWRITEAOF 命令。需要进行如下设置(默认配置)：
> 
> ```
> auto-aof-rewrite-percentage  100
> auto-aof-rewrite-min-size  64mb
> ```
> 
> &emsp;&emsp;无论是 AOF 持久化还是快照持久化，将数据持久化到硬盘上都是非常有必要的，但除了进行持久化外，用户还必须对持久化得到的文件进行备份(最好是备份到不同的地方)，这样才能尽量避免数据丢失事故发生。如果条件允许的话，最好能将快照文件和重新重写的 AOF 文件备份到不同的服务器上面。
> 
> 随着负载量的上升，或者数据的完整性变得越来越重要时，用户可能需要使用到复制特性。

### AOF 的优势和劣势

> **1、优点**
> 
> （1）AOF 可以更好的保护数据不丢失，一般 AOF 会每隔1秒，通过一个后台线程执行一次 fsync 操作，最多丢失1秒钟的数据。
> （2）AOF 日志文件没有任何磁盘寻址的开销，写入性能非常高，文件不容易破损。
> （3）AOF 日志文件即使过大的时候，出现后台重写操作，也不会影响客户端的读写。
> （4）AOF 日志文件的命令通过非常可读的方式进行记录，这个特性非常适合做灾难性的误删除的紧急恢复。比如某人不小心用 flushall 命令清空了所有数据，只要这个时候后台 rewrite 还没有发生，那么就可以立即拷贝AOF文件，将最后一条 flushall 命令给删了，然后再将该 AOF 文件放回去，就可以通过恢复机制，自动恢复所有数据。

> **2、缺点**
> 
> （1）对于同一份数据来说，AOF 日志文件通常比 RDB 数据快照文件更大；
> （2）AOF 开启后，支持的写 QPS 会比 RDB 支持的写 QPS 低，因为 AOF 一般会配置成每秒 fsync 一次日志文件。当然，每秒一次 fsync，性能也还是很高的；
> （3）以前 AOF 发生过 bug，就是通过 AOF 记录的日志，进行数据恢复的时候，没有恢复一模一样的数据出来。

## 该如何选择 RDB 和 AOF 进行持久化操作

> &emsp;&emsp;俗话说得好“小孩子才做选择，大人全都要！”，选择的话，两者加一起才更好。因为两个持久化机制你明白了，剩下的就是看自己的需求了，需求不同选择的也不一定，但是通常都是结合使用。有一张图可供总结：
> 
> ![RDB & AOF](https://hyblogs.oss-cn-beijing.aliyuncs.com/hyblogs/image_1594374248938.png)
> 
> &emsp;&emsp;Redis4.0 开始支持 RDB 和 AOF 的混合持久化(默认关闭，可以通过配置项 `aof-use-rdb-preamble` 开启)。
> 如果把混合持久化打开，AOF 重写的时候就直接把 RDB 的内容写到 AOF 文件开头。这样做的好处是可以结合 RDB 和 AOF 的优点,快速加载同时避免丢失过多的数据。
> <br/><br/>
> 参考资料来源声明：
> https://baijiahao.baidu.com/s?id=1654694618189745916&wfr=spider&for=pc
