# Linux(CentOS7)时间格式修改为24小时制

先执行如下命令：

```shell
tzselect
```

根据提示选择

5 –> 9–>1–>1

然后执行下面这两条命令

```shell
rm /etc/localtime

ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
