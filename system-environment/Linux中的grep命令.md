# Linux 中的 grep 命令



&emsp;&emsp;Linux系统中的grep命令是一种功能强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。grep全称是: Global Regular Expression Print，表示全局正则表达式版本。它的使用权限是所有用户。

### 主要语法

主要的使用语法可以通过`grep --help`查看，大致信息如下：

```shell
用法: grep [选项]... PATTERN [FILE]...
在每个 FILE 或是标准输入中查找 PATTERN。
默认的 PATTERN 是一个基本正则表达式(缩写为 BRE)。
例如: grep -i 'hello world' menu.h main.c

正则表达式选择与解释:
  -E, --extended-regexp     PATTERN 是一个可扩展的正则表达式(缩写为 ERE)
  -F, --fixed-strings       PATTERN 是一组由断行符分隔的定长字符串。
  -G, --basic-regexp        PATTERN 是一个基本正则表达式(缩写为 BRE)
  -P, --perl-regexp         PATTERN 是一个 Perl 正则表达式
  -e, --regexp=PATTERN      用 PATTERN 来进行匹配操作
  -f, --file=FILE           从 FILE 中取得 PATTERN
  -i, --ignore-case         忽略大小写
  -w, --word-regexp         强制 PATTERN 仅完全匹配字词
  -x, --line-regexp         强制 PATTERN 仅完全匹配一行
  -z, --null-data           一个 0 字节的数据行，但不是空行

输出控制:
  -m, --max-count=NUM       NUM 次匹配后停止
  -b, --byte-offset         输出的同时打印字节偏移
  -n, --line-number         输出的同时打印行号
      --line-buffered       每行输出清空
  -H, --with-filename       为每一匹配项打印文件名
  -h, --no-filename         输出时不显示文件名前缀
      --label=LABEL         将LABEL 作为标准输入文件名前缀
  -o, --only-matching       show only the part of a line matching PATTERN
  -q, --quiet, --silent     suppress all normal output
      --binary-files=TYPE   assume that binary files are TYPE;
                            TYPE is 'binary', 'text', or 'without-match'
  -a, --text                equivalent to --binary-files=text
  -I                        equivalent to --binary-files=without-match
  -d, --directories=ACTION  how to handle directories;
                            ACTION is 'read', 'recurse', or 'skip'
  -D, --devices=ACTION      how to handle devices, FIFOs and sockets;
                            ACTION is 'read' or 'skip'
  -r, --recursive           like --directories=recurse
  -R, --dereference-recursive
                            likewise, but follow all symlinks
      --include=FILE_PATTERN
                            search only files that match FILE_PATTERN
      --exclude=FILE_PATTERN
                            skip files and directories matching FILE_PATTERN
      --exclude-from=FILE   skip files matching any file pattern from FILE
      --exclude-dir=PATTERN directories that match PATTERN will be skipped.
  -L, --files-without-match print only names of FILEs containing no match
  -l, --files-with-matches  print only names of FILEs containing matches
  -c, --count               print only a count of matching lines per FILE
  -T, --initial-tab         make tabs line up (if needed)
  -Z, --null                print 0 byte after FILE name

文件控制:
  -B, --before-context=NUM  打印以文本起始的NUM 行
  -A, --after-context=NUM   打印以文本结尾的NUM 行
  -C, --context=NUM         打印输出文本NUM 行
  -NUM                      same as --context=NUM
      --group-separator=SEP use SEP as a group separator
      --no-group-separator  use empty string as a group separator
      --color[=WHEN],
      --colour[=WHEN]       use markers to highlight the matching strings;
                            WHEN is 'always', 'never', or 'auto'
  -U, --binary              do not strip CR characters at EOL (MSDOS/Windows)
  -u, --unix-byte-offsets   report offsets as if CRs were not there
                            (MSDOS/Windows)
```

上面罗列了部分信息，看着比较多，平时用得比较多的我挑出了几个，如下：

```shell
grep [options]
  [options]主要参数：
  -c：只输出匹配行的计数。
  -i：表示不区分大小写。
  -h：查询多文件时不显示文件名。
  -l：查询多文件时只输出包含匹配字符的文件名。
  -n：显示匹配行及行号。
  -s：不显示不存在或无匹配文本的错误信息。
  -v：显示不包含匹配文本的所有行，表示反向查找。
  --color=auto ：可以将找到的关键词部分加上颜色的显示
```

### 示例用法

- 1）查找包含"mysqld"的行，命令行如下：
  
  ```shell
  grep -n 'mysqld' mysql-slow.log
  ```
- 2）查找不包含"mysqld"的行，命令行如下：
  
  ```shell
  grep -vn 'mysqld' mysql-slow.log
  ```
- 3）查询"mysql"前面不是v的字符串，命令行如下：
  
  ```shell
  grep -n '[^v]mysql' mysql-slow.log
  ```
- 4）查询"mysql"前面不是小写字母的字符串，命令行如下：
  
  ```shell
  grep -n '[^a-z]"mysql"' mysql-slow.log
  ```
- 5）`^` 匹配以某个字符开头的行，查询以"Time"开头的字符串，命令行如下：
  
  ```shell
  grep -n '^Time' mysql-slow.log
  ```
- 6）`[^]` 匹配未包含的一个任意字符，查询不以字母开头的字符串，命令行如下：
  
  ```shell
  grep -n '^[^a-zA-Z]' mysql-slow.log
  ```
- 7）`$` 匹配以某个字符结尾的行。查询以`:`结尾的字符串，命令行如下：
  
  ```shell
  # 其中小数点“:”具有特殊意义，所以需要使用转义字符“\”将具有特殊意义的字符转化为普通字符
  grep -n '\:$' mysql-slow.log
  ```
- 8）`.` 匹配除 \r\n 外的任意一个字符。查询'l'与'k'之间包含两个字符的行，命令行如下：
  
  ```shell
  grep -n 'm..d' mysql-slow.log
  ```
- 9）查询包含连续'm'字母的行，命令行如下：
  
  ```shell
  # “*”表示的是重复零个或多个前面的单字符
  grep -n 'm*' mysql-slow.log
  ```
- 10）查询以'l'开头以'e'结尾中间至少包含一个'x'的行，命令行如下：
  
  ```shell
  grep -n 'lx*e' mysql-slow.log
  ```
- 11）查询以'l'开头'k'结尾，中间的字符可有可无的行，命令行如下：
  
  ```shell
  grep -n 'l.*k' mysql-slow.log
  ```
- 12）`{n}` 匹配确定的'n'次。查询包含两个'm'的行，命令行如下：
  
  ```shell
  # “{}”是特殊字符需要用“\”转义
  grep -n 'm\{2\}' mysql-slow.log 
  ```
