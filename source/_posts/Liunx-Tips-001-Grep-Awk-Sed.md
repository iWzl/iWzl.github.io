---

title: Liuns系统高效文件处理三剑客-Grep/Awk/Sed
date: 2020-08-14 11:45:29
tag: 
	- Liunx
	- Tools
	- Shell
categories:
    - Tips
---

> **子曰：“工欲善其事，必先利其器。居是邦也，事其大夫之贤者，友其士之仁者。”** --论语·卫灵公

### Grep

[Grep](https://zh.wikipedia.org/wiki/Grep)(global search regular expression(RE) and print out the line)是一款强大的文本搜索工具，支持正则表达式。来自Unix文本编辑器ed类似操作的命令,最初用于Unix操作系统的命令行工具。在给出文件列表或标准输入后，grep会对匹配一个或多个正则表达式的文本进行搜索，并只输出匹配（或者不匹配）的行或文本。

```shell
[root@Leonardo-iWzl-Aliyun-Service ~]# grep --help
用法: grep [选项]... PATTERN [FILE]...
Search for PATTERN in each FILE.
Example: grep -i 'hello world' menu.h main.c
```
<!-- more -->

```shell
Pattern selection and interpretation:
  -E, --extended-regexp     PATTERN is an extended regular expression
  -F, --fixed-strings       PATTERN is a set of newline-separated strings
  -G, --basic-regexp        PATTERN is a basic regular expression (default)
  -P, --perl-regexp         PATTERN is a Perl regular expression
  -e, --regexp=PATTERN      用 PATTERN 来进行匹配操作
  -f, --file=FILE           从 FILE 中取得 PATTERN
  -i, --ignore-case         忽略大小写
  -w, --word-regexp         强制 PATTERN 仅完全匹配字词
  -x, --line-regexp         强制 PATTERN 仅完全匹配一行
  -z, --null-data           一个 0 字节的数据行，但不是空行

杂项:
  -s, --no-messages         不显示错误信息
  -v, --invert-match        选中不匹配的行
  -V, --version             显示版本信息并退出
      --help                显示此帮助并退出

Output control:
  -m, --max-count=NUM       stop after NUM selected lines
  -b, --byte-offset         print the byte offset with output lines
  -n, --line-number         print line number with output lines
      --line-buffered       flush output on every line
  -H, --with-filename       print file name with output lines
  -h, --no-filename         suppress the file name prefix on output
      --label=LABEL         use LABEL as the standard input file name prefix
  -o, --only-matching       只显示匹配PATTERN 部分的行
  -q, --quiet, --silent     不显示所有常规输出
      --binary-files=TYPE   设定二进制文件的TYPE 类型；
                            TYPE 可以是`binary', `text', 或`without-match'
  -a, --text                等同于 --binary-files=text
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
  -L, --files-without-match print only names of FILEs with no selected lines
  -l, --files-with-matches  print only names of FILEs with selected lines
  -c, --count               print only a count of selected lines per FILE

文件控制:
  -B, --before-context=NUM  打印文本及其前面NUM 行
  -A, --after-context=NUM   打印文本及其后面NUM 行
  -C, --context=NUM         打印NUM 行输出文本
  -NUM                      same as --context=NUM
      --group-separator=SEP use SEP as a group separator
      --no-group-separator  use empty string as a group separator
      --color[=WHEN],
      --colour[=WHEN]       use markers to highlight the matching strings;
                            WHEN is 'always', 'never', or 'auto'
  -U, --binary              do not strip CR characters at EOL (MSDOS/Windows)
```

常用参数:

```shell
            -v        取反
            -i        忽略大小写
            -c        符合条件的行数
            -n        输出的同时打印行号
            ^*        以*开头         
            *$         以*结尾 
            ^$         空行 
            
            -a        不忽略二进制数据
            -A<n>     除了显示匹配的行外，还显示之后的n行
            -b        在符合条件的行之前，显示该行第一个字符的编号
```

Demo文案

```shell
[root@Leonardo-iWzl-Aliyun-Service ~]# cat demo.log
I came;
I saw;
i conquered.

我来了，我看到了，我征服了.   ——凯撒大帝
```

#### 查找符合条件的行

```shell
[root@Leonardo-iWzl-Aliyun-Service ~]# cat demo.log |grep 'I'
I came;
I saw;
```

#### 查找符合条件的行数

```shell
[root@Leonardo-iWzl-Aliyun-Service ~]# cat demo.log |grep 'I' -c
2
```

#### 查找不符合条件的行

```shell
[root@Leonardo-iWzl-Aliyun-Service ~]# cat demo.log |grep 'I' -v
i conquered.

我来了，我看到了，我征服了.   ——凯撒大帝
```

#### 忽略大小写查找

```shell
[root@Leonardo-iWzl-Aliyun-Service ~]# cat demo.log |grep 'I' -i
I came;
I saw;
i conquered.
```

#### 查找符合条件的行并输出行号

```shell
[root@Leonardo-iWzl-Aliyun-Service ~]# cat demo.log |grep 'I' -n
1:I came;
2:I saw;
```

#### 以'*'开头的查询

```shell
[root@Leonardo-iWzl-Aliyun-Service ~]# cat demo.log |grep '^I'
I came;
I saw;
```

#### 以'*'结尾的查询

```shell
[root@Leonardo-iWzl-Aliyun-Service ~]# cat demo.log |grep '; $'
I came;
I saw;
```

---

### Awk

Awk不仅仅是一个小工具，也可以算得上一种小型的编程语言了，支持if判断分支和while循环语句还有它的内置函数等，是一个要比grep和sed更强大的文本处理工具，但也就意味着要学习的东西更多了。由 Alfred Aho 、Peter Weinberger 和 Brian Kernighan 创始,并以姓氏的首个字母命名.

#### 基本结构和执行

```shell
awk '{[pattern] action}' {filenames}  
awk 'BEGIN{ commands } pattern{ commands } END{ commands }' {filenames}  
```

Awk脚本通常由：BEGIN语句块、能够使用模式匹配的通用语句块、END语句块3部分组成，这三个部分是可选的。任意一个部分都可以不出现在脚本中，脚本通常是被单引号或双引号中.

- 第一步：执行BEGIN{ commands }语句块中的语句；
- 第二步：从文件或标准输入(stdin)读取一行，然后执行pattern{ commands }语句块，它逐行扫描文件，从第一行到最后一行重复这个过程，直到文件全部被读取完毕。
- 第三步：当读至输入流末尾时，执行END{ commands }语句块。

BEGIN语句块在awk开始从输入流中读取行之前被执行，这是一个可选的语句块，比如变量初始化、打印输出表格的表头等语句通常可以写在BEGIN语句块中。

END语句块在awk从输入流中读取完所有的行之后即被执行，比如打印所有行的分析结果这类信息汇总都是在END语句块中完成，它也是一个可选语句块。

Pattern语句块中的通用命令是最重要的部分，它也是可选的。如果没有提供pattern语句块，则默认执行{ print }，即打印每一个读取到的行，awk读取的每一行都会执行该语句块。

action 在{}内指定，一般用来打印，也可以是一个代码段。也就是commands

```shell
[root@Leonardo-iWzl-Server ~]# awk --help
Usage: awk [POSIX or GNU style options] -f progfile [--] file ...
Usage: awk [POSIX or GNU style options] [--] 'program' file ...
POSIX options:		GNU long options: (standard)
	-f progfile		--file=progfile
	-F fs			--field-separator=fs
	-v var=val		--assign=var=val
Short options:		GNU long options: (extensions)
	-b			--characters-as-bytes
	-c			--traditional
	-C			--copyright
	-d[file]		--dump-variables[=file]
	-e 'program-text'	--source='program-text'
	-E file			--exec=file
	-g			--gen-pot
	-h			--help
	-L [fatal]		--lint[=fatal]
	-n			--non-decimal-data
	-N			--use-lc-numeric
	-O			--optimize
	-p[file]		--profile[=file]
	-P			--posix
	-r			--re-interval
	-S			--sandbox
	-t			--lint-old
	-V			--version
```

#### 内建参数

| 变量        | 描述                                                       |
| :---------- | :--------------------------------------------------------- |
| $n          | 当前记录的第n个字段，字段间由FS分隔                        |
| $0          | 完整的输入记录                                             |
| ARGC        | 命令行参数的数目                                           |
| ARGIND      | 命令行中当前文件的位置(从0开始算)                          |
| ARGV        | 包含命令行参数的数组                                       |
| CONVFMT     | 数字转换格式(默认值为%.6g)ENVIRON环境变量关联数组          |
| ERRNO       | 最后一个系统错误的描述                                     |
| FIELDWIDTHS | 字段宽度列表(用空格键分隔)                                 |
| FILENAME    | 当前文件名                                                 |
| FNR         | 各文件分别计数的行号                                       |
| FS          | 字段分隔符(默认是任何空格)                                 |
| IGNORECASE  | 如果为真，则进行忽略大小写的匹配                           |
| NF          | 一条记录的字段的数目                                       |
| NR          | 已经读出的记录数，就是行号，从1开始                        |
| OFMT        | 数字的输出格式(默认值是%.6g)                               |
| OFS         | 输出记录分隔符（输出换行符），输出时用指定的符号代替换行符 |
| ORS         | 输出记录分隔符(默认值是一个换行符)                         |
| RLENGTH     | 由match函数所匹配的字符串的长度                            |
| RS          | 记录分隔符(默认是一个换行符)                               |
| RSTART      | 由match函数所匹配的字符串的第一个位置                      |
| SUBSEP      | 数组下标分隔符(默认值是/034)                               |

#### 运算支持

| 运算符                  | 描述                             |
| :---------------------- | :------------------------------- |
| = += -= *= /= %= ^= **= | 赋值                             |
| ?:                      | C条件表达式                      |
| \|\|                    | 逻辑或                           |
| &&                      | 逻辑与                           |
| ~ 和 !~                 | 匹配正则表达式和不匹配正则表达式 |
| < <= > >= != ==         | 关系运算符                       |
| 空格                    | 连接                             |
| + -                     | 加，减                           |
| * / %                   | 乘，除与求余                     |
| + - !                   | 一元加，减和逻辑非               |
| ^ ***                   | 求幂                             |
| ++ --                   | 增加或减少，作为前缀或后缀       |
| $                       | 字段引用                         |
| in                      | 数组成员                         |

Demo文案

```shell
[root@Leonardo-iWzl-Server ~]# cat demo.log
小米 20 成都 172 60 女
小张 21 杭州 182 79 男
小文 19 长沙 178 70 男
小紫 22 北京 168 50 女
```

#### 输出指定位置的文档

```shell
[root@Leonardo-iWzl-Server ~]# cat demo.log |awk '{print $1,$3,$5}'
小米 成都 60
小张 杭州 79
小文 长沙 70
小紫 北京 50
```

#### 指定分隔符输出文档

```shell
# 使用"1"分割
[root@Leonardo-iWzl-Server ~]# cat demo.log |awk -F 1 '{print $1}'
小米 20 成都
小张 2
小文
小紫 22 北京

# 或者使用内建变量
[root@Leonardo-iWzl-Server ~]# cat demo.log |awk 'BEGIN{FS='1'} {print $1}'
小米 20 成都
小张 2
小文
小紫 22 北京

# 使用多个分隔符.先使用"1"分割，然后对分割结果再使用"0"分割
[root@Leonardo-iWzl-Server ~]# cat demo.log |awk -F '[10]' '{print $1}'
小米 2
小张 2
小文
小紫 22 北京

# 或者使用内建变量
[root@Leonardo-iWzl-Server ~]# cat demo.log |awk 'BEGIN{FS="[10]"} {print $1}'
小米 2
小张 2
小文
小紫 22 北京
```

#### 设置计算参数并输出文档

```shell
#awk -v 设置变量
[root@Leonardo-iWzl-Server ~]# cat demo.log |awk -v a=100 '{print $1,$2,a-$2}'
小米 20 80
小张 21 79
小文 19 81
小紫 22 78
```

