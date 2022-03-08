# Linux 解压报错解决记录



#### 问题场景一

数据库备份压缩包解压时出现如下错误：

```
tar: Skipping to next header
tar: Exiting with failure status due to previous errors
```

执行的解压命令为：

```shell
[root@hecs-x-large-2-linux-xx data]# tar -zxvf mine_field_20200527-0202.sql.gz 
```

因为我第一眼瞅见这是个 *.gz 压缩包，二话不说就使用 tar -zxvf 命令进行解压了，谁知出现了上面的错误信息。

**解决方案：**

++采用 gzip -d *.gz 进行解压。++

如上面解压文件的正确命令行该是：

```shell
[root@hecs-x-large-2-linux-xx data]# gzip -d mine_field_20200527-0202-2.sql.gz 
```

此处暂时记录这些，后面会不定期更新～
