&emsp;&emsp;最近在使用 Git 的过程中，需要将本地的分支 B 关联到远程的分支 A，但是我本地分支 B 代码提交的时候只能提交到远程分支 B (远程分支 A 没写入权限)，但是我要能及时的拉取到别人对远程分支 A 的更改 ，大概就是这么个意思。接下来记录一下 git 环境中如何查看本地分支与远程分支的映射，如何解除本地分支与远程分支的映射 以及 如何将本地分支关联到远程的目标分支。

## 1、查看本地分支与远程分支的映射关系
执行如下命令：
```shell
git branch -vv
```
> 注意是两个 `v` .

比如下面的结果：

```shell
huangyong@bogon hyblogs % git branch -vv
* feature/test/hyblogs-v1 4bbf79f3 [origin/feature/test/hyblogs-v1] test测试。
```

可以看到我的**本地分支** `feature/test/hyblogs-v1` 映射的**远程分支**是同名的 `origin/feature/test/hyblogs-v1`，但是现在我需要将我本地分支映射到远程的 `origin/feature/test/hyblogs` 分支。

## 2、撤销本地分支与远程分支的映射关系

基于上面的需求，我需要将本地分支与远程分支的映射关系给解除，方能创建新的映射关系。

执行如下命令解除关联：
```shell
git branch --unset-upstream
```

无报错则表示执行成功了，接下来再查看本地分支与远程分支映射关系(执行上面的 `git branch -vv` 命令)。

如下我本地的执行结果：

```shell
huangyong@bogon hyblogs % git branch --unset-upstream
huangyong@bogon hyblogs % git branch -vv             
* feature/test/hyblogs-v1 4bbf79f3 test测试。
```

可以看到本地分支与远程分支的映射关系已经撤销。接下拉我就可以将本地分支重新映射到远程的目标分支了。

## 3、建立当前本地分支与远程分支的映射关系

执行如下命令：

```shell
git branch -u origin/feature/test/hyblogs
```
> 这里的 `origin/feature/test/hyblogs` 就是你要映射的远程分支完整路径名称。

或者使用命令：

```shell
git branch --set-upstream-to origin/your-branch-name
```

比如我本地的执行结果：

```shell
huangyong@bogon hyblogs % git branch --set-upstream-to origin/feature/test/hyblogs
Branch 'feature/test/hyblogs-v1' set up to track remote branch 'feature/test/hyblogs' from 'origin'.
huangyong@bogon hyblogs % git branch -vv                                                     
* feature/test/hyblogs-v1 4bbf79f3 [origin/feature/test/hyblogs: ahead 1, behind 28] test测试。
```

我按照我的需求将我本地的 `feature/test/hyblogs-v1` 分支映射到了远程分支 `origin/feature/test/hyblogs` 分支。

按照这套流程，我更换了我本地分支映射的远程分支！又可以愉快的玩耍(PS:搬砖🧱)了～～～

## 4、其他

<font color=orange>⚠️重要说明：</font><font color=red>**本地分支可以与远程不同名分支建立映射关系！！！**</font>

在 idea 中，提交代码的时候我们可以指定远程分支进行提交，不一定是用默认分支进行提交。

远程分支之间可以通过 PR （Pull Requests） 的形式进行合并，这样可以进行线上化 Code Review，及时发现代码问题。

～好了～今天的记录就到这里了哦，期待下一次记录吧～