## 一些shell中常用技巧(二)
转自: 团子的小窝
http://kodango.com/

###### >团子的博文已经写得很不错了，分享了他自己的经验。本专题以团子的博文为主，部分地方做了我的解释。
---

## 1. bash中alias的使用
alias其实是给常用的命令定一个别名，比如很多人会定义一下的一个别名：

```
alias ll='ls -l'
```

以后就可以使用ll，实际展开后执行的是ls -l。现在很多发行版都会带几个默认的别名，比如：

```
alias grep='grep --color=auto' #带颜色显示
alias ls='ls --color=auto' # 同上
alias rm='rm -i' # 删除文件需要确认
```

alias在某些方面确实提高了很大的效率，但是也是有隐患的，这点可以看我以前的一篇文章终端下肉眼看不见的东西。那么如何不要展开alias，而是用本来的意思呢？答案是使用转义：

```
\ls\grep
```

在命令前面加一个反斜杠后就可以了。
## 2. awk打印除第一列之外的其他列
awk用来截取输入行中的某几列很有用，当时如果要排除某几列呢？
例如有如下的一个文件：

```
cat /tmp/test.txt
1 2 3 4 5
10 20 30 40 50
```
可以用下面的代码解决（来源）：


```
$ awk '{$1="";print $0}' /tmp/test.txt
 2 3 4 5
 20 30 40 50
```
但是前面多了一个空格，可以用cut命令稍微调整下：


```
$ awk '{$1="";print $0}' /tmp/test.txt | cut -c2-
2 3 4 5
20 30 40 50
```
## 3. 巧用bash的命令展开功能备份文件
假设要备份文件/your/path/to/file.list为/your/path/to/file.list.20121106，常规的方法是：


```
cp /your/path/to/file.list  /your/path/to/file.list.20121106
```

这样重复写上一长串的路径，是不是很麻烦，这里利用bash的展开特性可以这样做：


```
cp /your/path/to/file.list{,.20121106}
```

/your/path/to/file.list{,.20121106}这一部分会展开为/your/path/to/file.list /your/path/to/file.list.20121106,再将此传给cp命令，就达到了与前面同样的效果。（思路同ls *）。具体可以man bash中的Brace Expansion这一段。

## 4. 命令行下使用ctrl+x ctrl+e来编辑当前命令

这个技巧来自最牛B的 Linux Shell 命令系列连载（二）。使用方法是键入命令之后，再按ctrl+x ctrl+e可以打开一个编辑器来编辑命令，默认是使用emacs。你也可以通过在~/.bashrc中添加以下这一行，将编辑器换成vim:


```
export EDITOR='vim'
```

为什么推荐这一条呢？对于一般的命令（这里指的是长度很短的命令）其实这个技巧没什么用处，我用方向键移一下就OK了，但是有时候（尤其是运维的一些命令）有些命令长度特别长，一堆参数，如果直接在命令行修改其实风险很高的（可以通过在命令的开头加上一个#号来规避这个风险，Bash将当前的命令当成注释不执行），而且方向键一个一个迁移非常不方便（当然有类似ctrl+x ctrl+e这种预设的快捷键来操作，可以看bind -p)。

像使用ctrl+x,ctrl+e打开vim来编辑命令在这种场景有两种好处：
> a. 可以方便的用熟悉的编辑器高效地修改命令；

> b. 有一个确认的过程，无误后，退出vim才执行命令。

不过我不是很推荐最牛B的 Linux Shell 命令 系列连载中的一些对历史命令的技巧，虽然方便，但是风险很高，因为没有一个确认的过程，是执行将历史命令调出就执行了。



## 5. 你知道sed的这个特性吗？

假设一个文件的每一行为一个路径：


```
$ cat /tmp/test.txt
/home/kodango/hello
/home/kodango/hello/world
/home/kodango/good
/home/kodango/good/bye
```
现在要把/home/kodango/good替换成/home/kodango/bye，普通的作法是：


```
[Tue Nov 06 06:35 PM] [kodango@devops] ~ 
$ sed -n 's/\/home\/kodango\/good/\/home\/kodango\/bye/p' /tmp/test.txt 
/home/kodango/bye
/home/kodango/bye/bye
```

因为路径中的分隔符与sed的替换命令的分隔符都是'/'，所以需要转义，非常麻烦。幸运的是，sed可以更改分隔符，例如使用#：


```
[Tue Nov 06 06:34 PM] [kodango@devops] ~ 
$ sed -n 's#/home/kodango/good#/home/kodango/bad#p' /tmp/test.txt 
/home/kodango/bad
/home/kodango/bad/bye
```

这样就清爽多了。

补充，如果是在地址对中使用，首个分隔符前面要加反斜杠：


```
$ sed -n '\#/home/kodango/#p' /tmp/test.txt 
/home/kodango/hello
/home/kodango/hello/world
/home/kodango/good
/home/kodango/good/bye
```
## 6. 合并连续重复的字符（即squeeze操作）
例如要合并一个字符串中连续的多个空格，假设字符串为'print hello, world'。

第一种方法，使用sed命令，扫描整个字符串，替换2个以上的空格为1格：
```shell
$ echo 'print  hello,   world  ' | sed -r 's/ {2,}/ /g'
print hello, world 
```
第二种方法，使用tr命令的-s选项，专门就是为了合并连续重复的字符：
```
$ echo 'print  hello,   world  ' | tr -s ' '
print hello, world 
```
第三种方法，使用awk的域赋值来完成该目的：(我个人对awk不熟，有待进一步理解)
```
$ echo 'print  hello,   world  ' | awk '$1=$1'
print hello, world
```

对已经存在的域例如$1,$2..进行赋值，会导致awk重新使用OFS输出分隔符重组$0，关于这一点的详细说明见sosodream同学的博文[Awk里的域赋值操作和部分源码解析](http://blog.csdn.net/sosodream/article/details/6425192)（$1=$1,$0=$0,FS,OFS)

## 7. 将文本中某列相同的行输出到不同的文件中
标题有点绕口，我们以实际例子来讲解，假设我们有以下的一个文件：
```
$ cat /tmp/test.txt
a char
1 int
2 int
b char
abc string
```
我们的目标是将该文本中的行按第二列的值归类，并且输出到相应的文件中，文件名为第二列的名称。例如第2行、第3行会输出到int.txt文件中，而第1行、第4行则输出到char.txt，以此类推。

我没有找到其它简单的方法，只找到一种用awk来处理的方法：
```
[Wed Nov 07 07:31 PM] [kodango@devops] ~/workspace 
$ awk '{print $1 > $2 ".txt"}' /tmp/test.txt

[Wed Nov 07 07:34 PM] [kodango@devops] ~/workspace/output 
$ grep -nH . *
char.txt:1:a
char.txt:2:b
int.txt:1:1
int.txt:2:2
string.txt:1:abc
```
## 8. 用exec命令来完成重定向
以一个简单的例子开始，现在需要一个脚本，它可以接受一个文件名作为参数，然后按行读取该文件的内容并打印到标准输出。如果不指定文件名，则默认从标准输入读。首先按上面的功能需求写出一个可以完成功能的脚本：
```
[Sat Nov 10 12:16 AM] [kodango@devops] ~/workspace 
$ cat test.sh 

filename=$1

if [ -z "$filename" ]; then
    while read line; do
        echo $line
    done
else
    while read line; do
        echo $line
    done < $filename
fi
```
如果换exec来实现重定向，可以把脚本写得更优雅：
```
$ cat test1.sh 

filename=$1

if [ -n "$filename" ]; then
    exec 0< $filename
fi

while read line; do
    echo $line
done
```
这里的关键在第5行代码，exec命令不仅可以用于执行命令，还可以用于打开、关闭或者复制文件描述符，这里就是利用exec将指定的文件名打开重定向到标准输入。类似地可以用exec >$filename将文件重定向到标准输出。我们可以在命令行上做一个试验：
```
[Sat Nov 10 12:26 AM] [kodango@devops] ~ 
$ exec 3>&1                   # 首先将fd 3重定向到标准输出，作为标准输出的一个备份

$ ls /proc/629/fd/{1,3} -l    # 现在fd 3和fd 1指向同一个设备文件
lrwx------ 1 kodango kodango 64 Nov 10 00:26 /proc/629/fd/1 -> /dev/pts/1
lrwx------ 1 kodango kodango 64 Nov 10 00:26 /proc/629/fd/3 -> /dev/pts/1

$ exec >stdout               # 现在把标准输出重定向到stdout这个文件中

$ ls /proc/629/fd/1 -l        # 如果你此刻在同一个终端下执行本命令是没有返回的

$ ls /proc/629/fd/1 -l        # 现在重新打开一个终端看看，确实已经重定向到stdout这个文件
l-wx------ 1 kodango kodango 64 Nov 10 00:26 /proc/629/fd/1 -> /home/kodango/stdout

$ exec 1>&3                   # 现在重新把标准输出重定向到之前备份的fd 3上
$ ls /proc/629/fd/{1,3} -l  # 现在屏幕可以看到输出了，但是fd 3这个描述符还打开，需要关闭
lrwx------ 1 kodango kodango 64 Nov 10 00:26 /proc/629/fd/1 -> /dev/pts/1
lrwx------ 1 kodango kodango 64 Nov 10 00:26 /proc/629/fd/3 -> /dev/pts/1

$ exec 3>&-                   # 关闭fd 3
$ ls /proc/629/fd/3 -l
ls: cannot access /proc/629/fd/3: No such file or directory

$ cat stdout                  # 检查stdout文件，确实有之前被吃掉的输出
l-wx------ 1 kodango kodango 64 Nov 10 00:26 /proc/629/fd/1 -> /home/kodango/stdout
```
关于I/O重定向的更详细的说明，可以看[I/O Redirection](http://tldp.org/LDP/abs/html/io-redirection.html)，这里有很多例子讲解了各种I/O重定向的用法，包括exec来改变重定向。

这一点在while read; do xxx; done < file内部仍需要从标准输入读取内容时非常有用，此时必须要将循环外部的重定向和内部的剥离开来。
## 9. 引号之间的区别
Shell中比较让人抓狂的是各种引号的处理，其中，**反引号(&apos;cmd&apos;)是最容易掌握的，它其实和$(cmd)是差不多的**。

引号的作用有几点，***一个是为了将多个因为空格或者回车等分隔符隔开的字符串合在一起，避免被命令行解析分开***，例如"one two three"就是一整个字符串，而不是像one two three会被解析成三个单独的字符串；***另外一方面，引号可以让一些特殊符号保持原义***。

其中，单引号的处理是比较简单的，***被单引号包括的所有字符都保留原有的意思***，例如'$a'不会被展开, ' &apos; cmd &apos;'也不会执行命令；***而双引号，则相对比较松，在双引号中，以下几个字符*** $, &apos;, \依然有其特殊的含义，比如$可以用于变量展开, 反引号&apos;可以执行命令，反斜杠\可以用于转义。但是，在双引号包围的字符串里，反斜杠的转义也是有限的，它只能转义$, `, ", \或者newline（回车）这几个字符，后面如果跟着的不是这几个字符，只不会被黑底，反斜杠会被保留，例如：
```
$ echo "\$,\",\`,\',\t"
$,",`,\',\t
```
双引号内可以直接包含单引号，而且单引号也没有如上据说的特殊含义，所以像"var='$var'"中$var还是会被展开的，而不要以为简单地认为在单引号内部就不会展开了。如果双引号内部包含感叹号！就比较头痛了，感叹号是用于命令行历史展开，例如!!展开为上一次执行的命令。你可以试试双引号中包含!：
```
[Sat Nov 10 07:39 PM] [kodango@devops] ~ 
$ echo "!"
-bash: !: event not found
$ echo "\!"
\!
```
可见，即使你用反斜杠也没办法转义，除非你把历史展开功能关闭（在脚本里面是没有问题的，默认是关闭的）。
```
[Sat Nov 10 07:50 PM] [kodango@devops] ~ 
$ set +o histexpand 

[Sat Nov 10 07:50 PM] [kodango@devops] ~ 
$ echo "!"
!
```
到此为止，其实双引号和单引号的区别已经说得差不多了。不过还可以再说几个特殊的用法，前面说过可以在双引号内部使用单引号，你有想过在单引号里面使用单引号吗？
```
$ echo '\''  
>   #这个例子有点难理解
```
是不是发现不能用，因为单引号中反斜杠是没有转义的效果的，任何字符都没有特殊的含义。那就没有办法了吗？方法总是有的，可以在第一个单引号前面加个$符号：

```
$ echo $'\''
'
```
这又是另外一种神奇的用法了
关于这一点的内容，具体可以看以下两份资料：
a. http://www.gnu.org/software/bash/manual/html_node/Quoting.html#Quoting
b. http://tldp.org/LDP/abs/html/quoting.html

## 10. 特殊用法$'string'
前面一点中已经介绍了 $'string'这种用法，比如 $'\''，之所以可以这样用，通俗地讲，就是在这种语法里一些转义字符串是被认可的，事实上有效地的转义底字符串列表可以看[这里](http://www.gnu.org/software/bash/manual/html_node/ANSI_002dC-Quoting.html#ANSI_002dC-Quoting)，例如\b，\',\n,\f,\nnn,\xhh等等，是不是很熟悉。

$'string'的这个特性，其实为我们提供了一种很有用的技巧：

```
$ echo $'\x41'
A  #他可以将ASCII对应的字符赋值给某个变量或者输出。
```
## 11. 用双引号比不用更加安全
双引号除了前面第10点讲到的去除特殊涵义的作用外，还可以避免字符串被分隔解析，例如：
```
$ echo `ls -l`
total 4.0K -rw-r--r-- 1 kodango kodango 4 Nov 10 20:09 1 -rw-r--r-- 1 kodango kodango 0 Nov 10 20:09 2
$ echo "`ls -l`"
total 4.0K
-rw-r--r-- 1 kodango kodango 4 Nov 10 20:09 1
-rw-r--r-- 1 kodango kodango 0 Nov 10 20:09 2
```
前者没有加双引号，ls -l输出行之间的回车就被吃掉了。原因是，当ls -l返回的结果传递给echo之前，会先被shell进行参数解析，而shell是用IFS定义的分隔符来分隔字符串的，一般包括\n，所以它把解析后的结果再传递给echo，就成为echo "line 1...." "line 2..."这种形式了，结果就像上面一样。

而用双引号包括起来可以避开字符串被拆开解析，因为shell认为它是一个单独的字符串。所以一般情况下，多用引号包括变量是好的，"$var"比$var更安全。
## 12. 显示一个文件并且在每行开头添加行号
有两种做法，第一种借助cat和nl命令来完成：
```
$ cat test.txt | nl
     1	line 1 
     2	line 2
```
另外一种做法是用sed命令:


```
$ sed '=' test.txt | sed 'N;s/\n/\t/'
```
 还有一种方法是通过cat -n或者cat -b命令，两者的区别是后者不会给空行增加行号
## 13. 命令行键映射，编辑模式
命令行下默认是emacs的keymap，对于不会emacs的人来说真是灾难，完全不知道各种ctrl+x键是做什么的，可以通过执行以下命令切换到vi模式：


```
set -o vi
```
在这种模式下，就可以用熟悉的vi命令了，默认输入命令的是在insert模式，按ESC键可以切换到命令模式，这点和vim是一样的，熟悉vim的人很快就可以上手。

之前介绍过命令行下使用ctrl+x ctrl+e来编辑当前命令，而在vi模式下，可以在命令模式下直接键入v。还有，如果不想执行当前输入的命令，可以在命令模式下按#号键，它会在当前命令当作注释而不执行（在命令开头添加#号）。

更多vi模式的介绍可以参见[Working Productively in Bash's Vi Command Line Editing Mode](http://www.catonmat.net/blog/bash-vi-editing-mode-cheat-sheet/)，作者还给了一份[Vi Editing Mode Cheat Sheet](http://www.catonmat.net/download/bash-vi-editing-mode-cheat-sheet.pdf)留作参考。

如果你想将vi模式作为默认的编辑模式，可以将set -o vi写入到~/.bashrc文件中。当然，在运维的线上生产环境这样做是不合适的，你只能手动输入切换了。不过，如果你选择的ssh管理客户端比较高级的话，应该可以避免每次手动输出。比如我用的是xshell，可以通过设置Login script在每次登录的时候自动执行命令，或者将命令添加到quick command set，然后调出quick command set toolbar，手动点击按钮切换。这两种方法结合起来就几乎同写入到~/.bashrc一样的方便了。
## 14. 分别输出两个文件相同的行和不同的行
```
$ echo test{,2}.txt;paste test{,2}.txt
test.txt test2.txt
line 1 	line 11
line 2	line 2
```
如果要输出两个文件之间相同的行，只有test.txt拥有的行以及只有test2.txt拥有的行，怎么做？首先可以使用grep -f：
```
$ grep -f test{,2}.txt
line 2
$ grep -vf test{,2}.txt
line 11
$ grep -vf test{2,}.txt
line 1 
```
还有一种选择是comm命令，这个命令就是专门用于比较文件的：
>`comm - compare two sorted files line by line`。

使用方法也很简单，comm比较两个排序好的文件返回的结果有三列，第一列是只有在文件A中有的行，第二列是只有在文件B中有的行，第三列则是两个文件共有的行：
```
$ comm test.txt test2.txt                
line 1 
	line 11
		line 2
```
要得到最初要求的结果，则只需要取相应的列就可以了。comm命令非常人性化地考虑到这个需求：
```
$ comm test.txt test2.txt -1 -2
line 2
$ comm test.txt test2.txt -2 -3
line 1 
$ comm test.txt test2.txt -1 -3
line 11
#其中，-1, -2与-3这个参数分别表示不输出第1、2或者3列。
```
## 15. 获取被source的脚本的名称
一般的情况下，脚本的名称可以通过$0获取，但是这在被source导入的脚本中却不成立。假设A脚本中source了B脚本，那么它是把B的代码导入到A的环境中直接执行的，因此A和B的代码其实是在同一个执行环境下分不开的，B的代码中访问到的$0，甚至$1, $2等位置参数都是与A脚本是一致的。

因此$0并非是被导入的脚本的名称，实际上，Bash将被source的脚本名称保存在一个叫BASH_SOURCE的数组中，该数组的第一个元素正是当前被source的脚本的名称。
当一个脚本被source时，它的名称就被压入到这个数组的第一个位置上，举个实际的例子，假设有三个脚本a.sh,b.sh,c.sh，它们的内容如下所示：
```
$ cat a.sh 
. ./b.sh
echo "\$0=$0"
echo "\${BASH_SOURCE[0]}=${BASH_SOURCE[0]}"
echo "\$BASH_SOURCE=(${BASH_SOURCE[@]})"

$ cat b.sh 
. ./c.sh
. ./c.sh
echo "\$0=$0"
echo "\${BASH_SOURCE[0]}=${BASH_SOURCE[0]}"
echo "\$BASH_SOURCE=(${BASH_SOURCE[@]})"

$ cat c.sh 
$ cat c.sh 
echo "\$0=$0"
echo "\${BASH_SOURCE[0]}=${BASH_SOURCE[0]}"
echo "\$BASH_SOURCE=(${BASH_SOURCE[@]})"
```
现在执行a.sh这个脚本，实际的输出是（为了方便理解，我在实际的输出中加了一些注释和空行）：

```
$ bash a.sh
# c.sh的输出
$0=a.sh
${BASH_SOURCE[0]}=./c.sh
$BASH_SOURCE=(./c.sh ./b.sh a.sh)

# b.sh的输出
$0=a.sh
${BASH_SOURCE[0]}=./b.sh
$BASH_SOURCE=(./b.sh a.sh)

# a.sh的输出
$0=a.sh
${BASH_SOURCE[0]}=a.sh
$BASH_SOURCE=(a.sh)
```
此外，我们还可以利用BASH_SOURCE的值，在脚本中判断是被直接执行还是被导入：
```
if [ -n "$BASH_SOURCE" -a "$BASH_SOURCE" != "$0" ]
then
    echo "be sourced by other scripts"
else
    echo "be run in shell"
fi
```
## 16. ${}参数展开
我们知道${parameter}是展开变量parameter这个值，在上一篇[简洁的bash编程技巧](http://kodango.com/simple-bash-programming-skills)中也曾经介绍过${parameter:-word}这种用法，用于给变量赋一个默认值。

事实上除此之外，参数展开还有许多形式，在此之前，首先要说明一下变量的几种值的形式：
1. unset： 变量未设置，即变量从未声明，或者被unset命令重置；
2. null: 变量声明但未被赋值（var=）或者被赋值成空（var=""）；
3. not null： 变量被赋值；

unset和null在参数展开的时候还是有很大的区别的，以下是参数展开的各种形式：
1. ${parameter:-word}：假如parameter为unset或者null，则展开后返回word的值；
2. ${parameter-word}：假如parameter为unset时，则展开后返回word的值；
3. ${parameter:=word}：假如parameter为unset或者null，将word赋值给parameter；
4. ${parameter=word}：假如parameter为unset，将word赋值给parameter；
5. ${parameter:?word}：假如parameter为unset或者null，则将word作为错误输出到标准输出；
6. ${parameter?word}：假如parameter为unset，则将word作为错误输出到标准输出；
7. ${parameter:+word}：假如parameter为unset或者null，则不做展开，返回为空；（刚好与:-相反）
8. ${parameter:word}：假如parameter为unset，则不做展开，返回为空；（刚好与-相反）

上面其实准确地应该是分成2组，一组带:，一组不带:，不带:的这组更加严格，只检查unset这种情况。以:+为例子, unset的情况均无返回：

```
$ unset var && echo ${var:+hello}

$ unset var && echo ${var+hello}
```
当var为空时
```
$ var= && echo "${var:+hello}"

$ var= && echo "${var+hello}"
hello
```
当var为非空时：
```
$ var=1 && echo "${var:+hello}"
hello
$ var=1 && echo "${var+hello}"
hello
```
关于参数展开的具体内容可以参考Bash Man手册中的Parameter Expansion这一节。
## 17. 冒号的多种使用场景
冒号是一个比较奇怪的符号，它的用途有很多，这里介绍几种常用的：

1. 内置命令null command：nop，表示什么都不做，也可以被当作true值使用；
```
$ :
$ echo $?    # return 0
```
它也可以在循环中当作true值，例如:
```
while :; do   # 等价于 while true; do
    take-some-action
done

if condition
then :
else 
    take-some-action
fi
```
2. 占位符
冒号可以在很多场景下充当占位符，例如之前介绍的${parameter=var}，如果直接执行会报错，表示找不到命令；这时可以借用冒号来完成赋值：
```
: ${parameter=var}
```
同样地，可以来判断变量是否赋值：
```
: ${parameter1?} ${parameter2?}
```
更多其它用法可以看ABS的[Special Characters](http://www.tldp.org/LDP/abs/html/special-chars.html#COLON0REF)这一节。
## 18. 扩展的括号展开功能
这个功能不能说鸡肋，也可以了解下：
```
$ echo {0..3}
0 1 2 3
$ echo {z..a}
z y x w v u t s r q p o n m l k j i h g f e d c b a
$ echo {a..z}
a b c d e f g h i j k l m n o p q r s t u v w x y z
```
## 19. [[]]比[]作条件测试更安全
[[]]的功能比[]更加多，使用起来也更加安全。

1. 首先[[]]内部不会发生文件名展开和单词分隔。

例如：
```
$ touch hello\ world
$ [[ -f $file ]] && echo yes
yes
$ [ -f $file ] && echo yes
-bash: [: hello: binary operator expected
```
2. 进制之间自动转化

当一个十进制与八进制做比较时，会自动计算两个数的值，统一后做比较：


```
$ o=017
$ h=0x0f
$ [[ $o -eq $h ]] && echo yes
yes
$ [[ $o -eq 15 ]] && echo yes
yes
```
3. [[]]支持&&，||等运算符


```
$ a=1;b=3
$ [[ $a > 0 && $b < 4 ]] && echo yes
yes
```

## 20. 获取Bash脚本的最后一个参数

我们都知道可以用$0，$1等来获取传递给脚本或者函数的参数，也可以用$*或者$@获取所有的参数，但是如果我只想要获取最后一个参数呢？

首先，你可能想到用遍历地方法（这里为了方便，我们使用set命令来设置位置参数）：

```
$ set -- arg1 arg2 arg3
$ for i in $@; do :; done #注意这里do后面的冒号占位用法
$ echo $i
arg3
```
这里的循环什么事情都没做，我用冒号（:）完成这个任务；循环结束后, $i就是保存着最后一个参数的值。

下面是两种更加简单的方法的：

```
$ echo ${@: -1}
$ echo ${!#}
```
上面的第一种方法事实上就是Parameter Expansion中的${parameter:offset:length}这种形式，只不过offset为-1表示最后一个元素，忽略length表明是从offset开始往后直到最后一个元素，即只取最后一个元素。这里要注意的一点是，在冒号和短横之间的空格不能少，否则就变成15. ${}参数展开中介绍的${parameter:-var}这种用法了。

而第二种方法则是indirect referencing的一种表现，#这个特殊的变量存放参数的个数，!#则是对最后一个变量的引用。

## 21. Bash中的引用(indirect referencing)
有没有想法在Bash中也可以达到C++引用的效果？你可能不知道，但是你可能曾经有这种需求，我就有过：

>有时候，我想要一个变量存放另外一个变量的名称，然后在后面我想通过这个变量的名称引用它的值

例子是这样的：
```
$ a=b
$ b=1
$ echo $a
b
$ eval "echo \$$a"
1
```
但是利用indirect referencing的用法，你可以这样获取b的值:


```
$ echo ${!a}
1
$ b=2
$ echo ${!a}
2
```
很奇怪的一种用法，关于indirect referencing你可以查看这里或者这里。
