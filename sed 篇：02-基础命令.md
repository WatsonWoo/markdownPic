## sed 篇：02-基础命令

转自：团子的小窝

http://kodango.com/sed-and-awk-notes-part-3


grep，sed，awk成为shell三大剑客，是处理文本的三大利器。
比较好的讲解网站是:
>**高级Bash脚本编程指南: 一本深入学习shell脚本艺术的书籍**
Appendix C(附录C部分)
>http://www.reddragonfly.org/abscn/sedawk.html

但是‘团子的小窝’讲解的也还不错，这里就用团子的小窝文档

本sed专题主要以团子的博文为主，加上我自己理解。
如果有不对的地方，请留言指正。
---

首先回顾上一篇的重点内容：地址匹配。上一篇中介绍过，地址可以指定0个，1个或者2个。地址的形式可以为斜杠分隔的正则表达式(例如/test/)，行号（例如3,5)或者特殊符号(例如$)。如果没有指定地址，说明sed应用的编辑命令是全局的；如果是1个地址，编辑命令只是应用到匹配的那一行；如果是一对地址，编辑命令则应用到该地址对匹配的行范围。

<sed & awk>书中说，对于sed编辑命令的语法有两种约定，分别是


```
[address]command               # 第一种
[line-address]command          # 第二种
```
第一种[address]是指可以为任意地址包括地址对，第二种[line-address]是指只能为单个地址。文中指出，例如i（后方插入命令）、a（前方追加命令）以及=（打印行号命令）等都属于第二种命令，但是实际上我在ArchLinux上测试，指定地址对也没有报错。不过为了兼容性，建议按作者所说的做。

以下是要介绍的全部基础命令：

**名称** |**命令**|**语法**|**说明**
---|---|---|---|---
替换|	s|	[address]s/pattern/replacement/flags	|替换匹配的内容
删除|	d|	[address]d	|删除匹配的行
插入|	i|	[line-address]i\ text	|在匹配行的前方插入文本
追加|	a|	[line-address]a\ text	|在匹配行的后方插入文本
行替换|	c|	[address]c\ text	|将匹配的行替换成文本text
打印行|	p|	[address]p	|打印在模式空间中的行
打印行号|	=|	[address]=	|打印当前行行号
打印行|	l|	[address]l	|打印在模式空间中的行，同时显示控制字符
转换字符|	y|	[address]y/SET1/SET2/	|将SET1中出现的字符替换成SET2中对应位置的字符
读取下一行|	n|	[address]n	|将下一行的内容读取到模式空间
读文件|	r|	[line-address]r file	|将指定的文件读取到匹配行之后
写文件|	w|	[address]w file	|将匹配地址的所有行输出到指定的文件中
退出|	q|	[line-address]q	|读取到匹配的行之后即退出

##### 替换命令: s
替换命令的语法是：


```
[address]s/pattern/replacement/flags
```
其中[address]是指地址，pattern是替换命令的匹配表达式，replacement则是对应的替换内容，flags是指替换的标志位，它可以包含以下一个或者多个值：

- n: 一个数字（取值范围1-512），表明仅替换前n个被pattern匹配的内容；
- g: 表示全局替换，替换所有被pattern匹配的内容；
- p: 仅当行被pattern匹配时，打印模式空间的内容；
- w file：仅当行被pattern匹配时，将模式空间的内容输出到文件file中；

如果flags为空，则默认替换第一次匹配：

```
$ echo "column1 column2 column3 column4" | sed 's/ /;/'
column1;column2 column3 column4
```

如果flags中包含g，则表示全局匹配：


```
$ echo "column1 column2 column3 column4" | sed 's/ /;/g'
column1;column2;column3;column4
```

如果flags中明确指定替换第n次的匹配，例如n=2：


```
$ echo "column1 column2 column3 column4" | sed 's/ /;/2'
column1 column2;column3 column4
```

当替换命令的pattern与地址部分是一样的时候，比如/regexp/s/regexp/replacement/可以省略替换命令中的pattern部分，这在单个编辑命令的情况下没多大用处，但是在组合命令的场景下还是能省不少功夫的，例如下面是摘自文中的一个段落：
```
The substitute command is applied to the lines matching the address. If no
address is specified, it is applied to all lines that match the pattern, a
regular expression. If a regular expression is supplied as an address, and no
pattern is specified, the substitute command matches what is matched by the
address.  This can be useful when the substitute command is one of multiple
commands applied at the same address. For an example, see the section "Checking
Out Reference Pages" later in this chapter.
```
现在要在substitute command后面增加("s")，同时在被修改的行前面增加+号，以下是使用的sed命令：


```
$ sed '/substitute command/{s//&("s")/;s/^/+ /}' paragraph.txt
```
这里我们用到了组合命令，并且地址匹配的部分和第一个替换命令的匹配部分是一样的，所以后者我们省略了，在replacement部分用到了&这个元字符，它代表之前匹配的内容，这点我们在后面介绍。执行后的结果为：
```
+ The substitute command("s") is applied to the lines matching the address. If no
address is specified, it is applied to all lines that match the pattern, a
regular expression. If a regular expression is supplied as an address, and no
+ pattern is specified, the substitute command("s") matches what is matched by the
+ address.  This can be useful when the substitute command("s") is one of multiple
commands applied at the same address. For an example, see the section "Checking
Out Reference Pages" later in this chapter.
```

替换命令的一个技巧是中间的分隔符是可以更改的（这个技巧我们在简洁的Bash编程技巧续篇 - 5. 你知道sed的这个特性吗？中也曾经介绍过），这个技巧在有些地方非常有用，比如路径替换，下面是采用默认的分隔符和使用感叹号作为分隔符的对比：
```
$ find /usr/local -maxdepth 2 -type d | sed 's/\/usr\/local\/man/\/usr\/share\/man/'
$ find /usr/local -maxdepth 2 -type d | sed 's!/usr/local/man!/usr/share/man!'
```
替换命令中还有一个很重要的部分——replacement（替换内容），即将匹配的部分替换成replacement。在replacemnt部分中也有几个特殊的元字符，它们分别是：

- &: 被pattern匹配的内容；
- \num: 被pattern匹配的第num个分组（正则表达式中的概念，\(..\)括起来的部分称为分组；
- \: 转义符号，用来转义&，\, 回车等符号

&元字符我们已经在上面的例子中介绍过，这里举个例子介绍下\num的用法。在Linux系统中，默认都开了几个tty可以让你切换（ctrl+alt+1, ctrl+alt+2 ...)，例如CentOS 6.0+的系统默认是6个：
```
[root@localhost ~]# grep -B 1 ACTIVE_CONSOLES /etc/sysconfig/init 
# What ttys should gettys be started on?
ACTIVE_CONSOLES=/dev/tty[1-6]
```
可以通过修改上面的配置来减少开机启动的时候创建的tty个数，比如我们只想要2个：


```
[root@localhost ~]# sed -r -i 's!(ACTIVE_CONSOLES=/dev/tty\[1-)6]!\12]!' /etc/sysconfig/init 
ACTIVE_CONSOLES=/dev/tty[1-2]
```
其中-i参数表示直接修改原文件，-r参数是指使用扩展的正则表达式（ERE），扩展的正则表达式中分组的括号不需要用反斜杠转义"(ACTIVE_CONSOLES=/dev/tty\[1-)"，这里[是有特殊含义的（表示字符组），所以需要转义。在替换的内容中使用\1来引用这个匹配的分组内容，1代表分组的编号，表示第一个分组。

##### 删除命令: d
删除命令的语法是：

```
[address]d
```
删除命令可以用于删除多行内容，例如1,3d会删除1到3行。删除命令会将模式空间中的内容全部删除，并且导致后续命令不会执行并且读入新行，因为当前模式空间的内容已经为空。我们可以试试：
```
$ sed '2,${d;=}' list
John Daggett, 341 King Road, Plymouth MA

```
以上命令尝试在删除第2行到最后一行之后，打印当前行号(=)，但是事实上并没有执行该命令。

##### 插入行/追加行/替换行命令: i/a/c
这三个命令的语法如下所示：
```
# Append 追加
[line-address]a\
text
# Insert 插入
line-address]i\
text
# Change 行替换 
[address]c\
text
```
以上三个命令，行替换命令（c)允许地址为多个地址，其余两个都只允许单个地址（注：在ArchLinux上测试表明，追加和插入命令都允许多个地址，sed版本为GNU sed version 4.2.1）

追加命令是指在匹配的行后面插入文本text；相反地，插入命令是指匹配的行前面插入文本text；最后，行替换命令会将匹配的行替换成文本text。文本text并没有被添加到模式空间，而是直接输出到屏幕，因此后续的命令也不会应用到添加的文本上。注意，即使使用-n参数也无法抑制添加的文本的输出。

我们用实际的例子来简单介绍下这三个命令的用法，依然用最初的文本list：
```
$ cat list
John Daggett, 341 King Road, Plymouth MA
Alice Ford, 22 East Broadway, Richmond VA
Orville Thomas, 11345 Oak Bridge Road, Tulsa OK
Terry Kalkas, 402 Lans Road, Beaver Falls PA
Eric Adams, 20 Post Road, Sudbury MA
Hubert Sims, 328A Brook Road, Roanoke VA
Amy Wilde, 334 Bayshore Pkwy, Mountain View CA
Sal Carpenter, 73 6th Street, Boston MA
```
现在，我们要在第2行后面添加'------'：
```
$ sed '2a\
--------------------------------------
' list
John Daggett, 341 King Road, Plymouth MA
Alice Ford, 22 East Broadway, Richmond VA
--------------------------------------
Orville Thomas, 11345 Oak Bridge Road, Tulsa OK
Terry Kalkas, 402 Lans Road, Beaver Falls PA
Eric Adams, 20 Post Road, Sudbury MA
Hubert Sims, 328A Brook Road, Roanoke VA
Amy Wilde, 334 Bayshore Pkwy, Mountain View CA
Sal Carpenter, 73 6th Street, Boston MA
```
或者可以在第3行之前插入：
```
$ sed '3i\
--------------------------------------
' list
John Daggett, 341 King Road, Plymouth MA
Alice Ford, 22 East Broadway, Richmond VA
--------------------------------------
Orville Thomas, 11345 Oak Bridge Road, Tulsa OK
Terry Kalkas, 402 Lans Road, Beaver Falls PA
Eric Adams, 20 Post Road, Sudbury MA
Hubert Sims, 328A Brook Road, Roanoke VA
Amy Wilde, 334 Bayshore Pkwy, Mountain View CA
Sal Carpenter, 73 6th Street, Boston MA
```
我们来测试下文本是否确实没有添加到模式空间，因为模式空间中的内容默认是会打印到屏幕的：


```
$ sed -n '2a\
--------------------------------------
' list
--------------------------------------
```
通过-n参数来抑制输出后发现插入的内容依然被输出，所以可以判定插入的内容没有被添加到模式空间。

使用行替换命令将第2行到最后一行的内容全部替换成'----'：


```
$ sed '2,$c\
--------------------------------------
' list
John Daggett, 341 King Road, Plymouth MA
--------------------------------------
```
##### 打印命令: p/l/=
这里纯粹的打印命令应该是指p，但是因为后两者(l和=)和p差不多，并且相对都比较简单，所以这里放到一起介绍。

这三个命令的语法是：
```
[address]p
[address]=
[address]l
```
p命令用于打印模式空间的内容，例如打印list文件的第一行：


```
$ sed -n '1p' list
John Daggett, 341 King Road, Plymouth MA
```

l命令类似p命令，不过会显示控制字符，这个命令和vim的list命令相似，例如：


```
$ echo "column1 column2 column3^M" | sed -n 'l'
column1\tcolumn2\tcolumn3\r$
```

=命令显示当前行行号，例如：
```
$ sed '=' list
1
John Daggett, 341 King Road, Plymouth MA
2
Alice Ford, 22 East Broadway, Richmond VA
3
Orville Thomas, 11345 Oak Bridge Road, Tulsa OK
4
Terry Kalkas, 402 Lans Road, Beaver Falls PA
5
Eric Adams, 20 Post Road, Sudbury MA
6
Hubert Sims, 328A Brook Road, Roanoke VA
7
Amy Wilde, 334 Bayshore Pkwy, Mountain View CA
8
Sal Carpenter, 73 6th Street, Boston MA
```
##### 转换命令:
 y
转换命令的语法是：

```
[address]y/SET1/SET2/
```
它的作用是在匹配的行上，将SET1中出现的字符替换成SET2中对应位置的字符，例如1,3y/abc/xyz/会将1到3行中出现的a替换成x，b替换成y，c替换成z。是不是觉得这个功能很熟悉，其实这一点和tr命令是一样的。可以通过y命令将小写字符替换成大写字符，不过命令比较长：
```
$ echo "hello, world" | sed 'y/abcdefghijklmnopqrstuvwxyz/ABCDEFGHIJKLMNOPQRSTUVWXYZ/'
HELLO, WORLD
```
使用tr命令来转换成大写：


```
$ echo "hello, world" | tr a-z A-Z                                                    
HELLO, WORLD
```
##### 取下一行命令: n
取下一行命令的语法为：


```
[address]n
```
n命令为将下一行的内容提前读入，并且将之前读入的行（在模式空间中的行）输出到屏幕，然后后续的命令会应用到新读入的行上。因此n命令也会同d命令一样改变sed的控制流程。

书中给出了一个例子来介绍n的用法，假设有这么一个文本：
```
$ cat text
.H1 "On Egypt"

Napoleon, pointing to the Pyramids, said to his troops:
"Soldiers, forty centuries have their eyes upon you."
```
现在要将.H1后面的空行删除：


```
$ sed '/.H1/{n;/^$/d}' text
.H1 "On Egypt"
Napoleon, pointing to the Pyramids, said to his troops:
"Soldiers, forty centuries have their eyes upon you."
```
##### 读写文件命令: r/w
读写文件命令的语法是：


```
[line-address]r file
[address]w file
```
读命令将指定的文件读取到匹配行之后，并且输出到屏幕，这点类似追加命令(a)。我们以书中的例子来讲解读文件命令。假设有一个文件text：


```
$ cat text
For service, contact any of the following companies:
[Company-list]
Thank you.
```
同时我们有一个包含公司名称列表的文件company.list：


```
$ cat company.list 
Allied
Mayflower
United
```
现在我们要将company.list的内容读取到[Company-list]之后：


```
$ sed '/^\[Company-list]/r company.list' text
For service, contact any of the following companies:
[Company-list]
Allied
Mayflower
United
Thank you.
```

更好的处理应当是用公司名称命令替换[Company-list]，因此我们还需要删除[Company-list]这一行：


```
$ sed '/^\[Company-list]/r company.list;/^\[Company-list]/d;' text
For service, contact any of the following companies:
[Company-list]
Thank you.
```

但是结果我们非但没有删除[Company-list]，而且company.list的内容也不见了？这是怎么回事呢？

下面是我猜测的过程，读文件的命令会将r空格后面的所有内容都当成文件名，即company.list;/^\[Company-list]/d;，然后读取命令的时候发现该文件不存在，但是sed命令读取不存在的文件是不会报错的。所以什么事都没干成。

我们用-e选项将两个命令分开就正常了：

```
$ sed -e '/^\[Company-list]/r company.list' -e '/^\[Company-list]/d;' text
For service, contact any of the following companies:
Allied
Mayflower
United
Thank you.
```
写命令将匹配地址的所有行输出到指定的文件中。假设有一个文件内容如下，前半部分是人名，后半部分是区域名称：


```
$ cat text 
Adams, Henrietta Northeast
Banks, Freda South
Dennis, Jim Midwest
Garvey, Bill Northeast
Jeffries, Jane West
Madison, Sylvia Midwest
Sommes, Tom South
```
现在我们要将不同区域的人名字写到不同的文件中：
```
$ sed '/Northeast$/w region.northeast
> /South$/w region.south
> /Midwest$/w region.midwest
> /West$/w region.west' text
Adams, Henrietta Northeast
Banks, Freda South
Dennis, Jim Midwest
Garvey, Bill Northeast
Jeffries, Jane West
Madison, Sylvia Midwest
Sommes, Tom South
```
查看输出的文件：


```
$ cat region.
region.midwest    region.northeast  region.south      region.west 
$ cat region.midwest     
Dennis, Jim Midwest
Madison, Sylvia Midwest
```

我们也可以试试将所有的w命令放到一起用分号分隔，发现会出下面的错误，它把w空格后面的部分当作文件名，这样就验证了上面的猜测：


```
$ sed '/Northeast$/w region.northeast;/south$/w region.south;' text
sed: couldn't open file region.northeast;/south$/w region.south;: No such file or directory
```
##### 退出命令: q
退出命令的语法是


```
[line-address]q
```

当sed读取到匹配的行之后即退出，不会再读入新的行，并且将当前模式空间的内容输出到屏幕。例如打印前3行内容：

```
$ sed '3q' list
John Daggett, 341 King Road, Plymouth MA
Alice Ford, 22 East Broadway, Richmond VA
Orville Thomas, 11345 Oak Bridge Road, Tulsa OK
```
打印前3行也可以用p命令：


```
$ sed -n '3p' list
John Daggett, 341 King Road, Plymouth MA
Alice Ford, 22 East Broadway, Richmond VA
Orville Thomas, 11345 Oak Bridge Road, Tulsa OK
```

但是对于大文件来说，前者比后者效率更高，因为前者读取到第N行之后就退出了。后者虽然打印了前N行，但是后续的行还是要继续读入，只不会不作处理。

到此为止，sed基础命令的部分就介绍完了。下面一篇的内容会在最近几天更新上来，主要会介绍sed高级命令。不过一般情况下，会使用基础命令已经差不多了，高级命令不一定需要去了解。

![image](http://kodango.oss.aliyuncs.com/2013/05/sed_basic_command.bmp)


