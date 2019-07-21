## sed 篇：00-简单介绍

grep，sed，awk成为shell三大剑客，是处理文本的三大利器。
比较好的讲解网站是:
>**高级Bash脚本编程指南: 一本深入学习shell脚本艺术的书籍**
Appendix C(附录C部分)
>http://www.reddragonfly.org/abscn/sedawk.html

但是‘团子的小窝’讲解的也还不错，这里就用团子的小窝文档
http://kodango.com/sed-and-awk-notes-part-1

本sed专题主要以团子的博文为主，加上我自己理解。
如果有不对的地方，请留言指正。
***

##### Sed是什么
《sed and awk》一书中（1.2 A Stream Editor）是这样解释的：

>Sed is a "non-interactive" stream-oriented editor. It is stream-oriented because, like many UNIX
programs, input flows through the program and is directed to standard output.

Sed本质上是一个编辑器，但是它是非交互式的，这点与VIM不同；同时它又是面向字符流的，输入的字符流经过Sed的处理后输出。这两个特性使得Sed成为命令行下面非常有用的一个处理工具。

如果对sed的历史有兴趣，可以看书中2.1节"Awk, by Sed and Grep, out of Ed"，这里就不多介绍。

##### 基本概念
sed命令的语法如下所示：

```
sed [options] script filename
```
同大多数Linux命令一样，sed也是从stdin中读取输入，并且将输出写到stdout，但是当filename被指定时，则会从指定的文件中获取输入，输出可以重定向到文件中，但是需要注意的是，该文件绝对不能与输入的文件相同。

options是指sed的命令行参数，这一块并不是重点，参数也不多。

script是指需要对输入执行的一个或者多个操作指令(instruction)，sed会依次读取输入文件的每一行到缓存中并应用script中指定的操作指令，因此而带来的变化并不会影响最初的文件（注：如果使用sed时指定-i参数则会影响最初的文件）。如果操作指令很多，为了不影响可读性，可以将其写到文件中，并通过-f参数指定scriptfile:


```
sed -f scriptfile filename
```
这里有一个建议，**在命令行中指定的操作指令最好用单引号引起来，这样可以避免shell对特殊字符的处理（如空格、$等）**。*这个建议同样适用grep/awk等命令，当然如果有时候确实不适合使用单引号时，记得对特殊字符转义。*
无论是将操作指令通过命令行指定，还是写入到文件中作为一个sed脚本，必须包含至少一个指令，否则用sed就没有意义了。一般会同时指定多个操作指令，这时候指令之间的顺序就显得非常重要。而你的脑海中必须有这么一个概念，即每个指令应用后，当前输入的行会变成什么样子。要做到这一点首先必须要了解sed的工作原理，要做到“知其然，且知其所以然”。

每条操作指令由pattern和procedure两部分组成，pattern一般是用'/'分隔的正则表达式（在sed中也有可能是行号，具体参[见Sed命令地址匹配问题总结](http://kodango.com/sed-address-matching-summary)），而procedure则是一连串编辑命令(action)。
sed的处理流程，简化后是这样的：

- 读入新的一行内容到缓存空间；
- 从指定的操作指令中取出第一条指令，判断是否匹配pattern；
- 如果不匹配，则忽略后续的编辑命令，回到第2步继续取出下一条指令；
- 如果匹配，则针对缓存的行执行后续的编辑命令；完成后，回到第2步继续取出下一条指令；
- 当所有指令都应用之后，输出缓存行的内容；回到第1步继续读入下一行内容；
- 当所有行都处理完之后，结束；

具体流程见下图：
![image](http://kodango.oss.aliyuncs.com/2013/05/simple_sed_process_flow_chart.png)
##### 简单例子
概念如果脱离实际的例子就会显得非常枯燥无趣，这本书在这一点上处理得很好，都是配上实际的例子来讲解的。不过有点遗憾的是，书中的例子大多是与troff macro有关的，我们很难有条件来实际测试例子，在笔记中我会尽量使用更加浅显易懂的例子来说明。

下面是摘自书中的一个例子，假设有个文件list：
```
John Daggett, 341 King Road, Plymouth MA
Alice Ford, 22 East Broadway, Richmond VA
Orville Thomas, 11345 Oak Bridge Road, Tulsa OK
Terry Kalkas, 402 Lans Road, Beaver Falls PA
Eric Adams, 20 Post Road, Sudbury MA
Hubert Sims, 328A Brook Road, Roanoke VA
Amy Wilde, 334 Bayshore Pkwy, Mountain View CA
Sal Carpenter, 73 6th Street, Boston MA
```
假如，现在打算将MA替换成Massachusetts，可以使用替换命令s：
```
$ sed -e 's/MA/Massachusetts/' list
John Daggett, 341 King Road, Plymouth Massachusetts
Alice Ford, 22 East Broadway, Richmond VA
Orville Thomas, 11345 Oak Bridge Road, Tulsa OK
Terry Kalkas, 402 Lans Road, Beaver Falls PA
Eric Adams, 20 Post Road, Sudbury Massachusetts
Hubert Sims, 328A Brook Road, Roanoke VA
Amy Wilde, 334 Bayshore Pkwy, Mountain View CA
Sal Carpenter, 73 6th Street, Boston Massachusetts
```
这里面的-e选项是可选的，这个参数只是在命令行中同时指定多个操作指令时才需要用到，如：
```
$ sed -e 's/ MA/, Massachusetts/' -e 's/ PA/, Pennsylvania/' list 
John Daggett, 341 King Road, Plymouth, Massachusetts
Alice Ford, 22 East Broadway, Richmond VA
Orville Thomas, 11345 Oak Bridge Road, Tulsa OK
Terry Kalkas, 402 Lans Road, Beaver Falls, Pennsylvania
Eric Adams, 20 Post Road, Sudbury, Massachusetts
Hubert Sims, 328A Brook Road, Roanoke VA
Amy Wilde, 334 Bayshore Pkwy, Mountain View CA
Sal Carpenter, 73 6th Street, Boston, Massachusetts
```
即使在多个操作指令的情况下，-e参数也不是必需的，我一般不会加-e参数，比如上面的例子可以换成下面的写法：

```
$ sed 's/ MA/, Massachusetts/;s/ PA/, Pennsylvania/' list
```

操作指令之间可以用分号分隔，这点和shell命令可以用分号分隔是一样的。

这里提前说一点使用s替换命令需要小心的地方，不要忘记结尾的斜杠，如果你遇到下面的错误说明你犯了这个错误:


```
$ sed -e 's/ MA/, Massachusetts' list
sed: -e expression #1, char 21: unterminated `s' command
```

假如这时，我只想输出替换命令影响的行，而不想输出文件的所有内容，需要怎么办？这个时候可以利用替换命令s的p选项，它会将替换后的内容打印到标准输出。但是仅仅这样是不够的，否则输出的结果并非我们所想要的：


```
$ sed -e 's/ MA/, Massachusetts/p' list
John Daggett, 341 King Road, Plymouth, Massachusetts
John Daggett, 341 King Road, Plymouth, Massachusetts
Alice Ford, 22 East Broadway, Richmond VA
Orville Thomas, 11345 Oak Bridge Road, Tulsa OK
Terry Kalkas, 402 Lans Road, Beaver Falls PA
Eric Adams, 20 Post Road, Sudbury, Massachusetts
Eric Adams, 20 Post Road, Sudbury, Massachusetts
Hubert Sims, 328A Brook Road, Roanoke VA
Amy Wilde, 334 Bayshore Pkwy, Mountain View CA
Sal Carpenter, 73 6th Street, Boston, Massachusetts
Sal Carpenter, 73 6th Street, Boston, Massachusetts
```

从上面可以看出，文件的所有行都被打印到标准输出，并且被替换命令修改的行输出了两遍。造成这个问题的原因是，sed命令应用完所有指定之后默认会将当前行打印输出，所以就有了上面的结果。解决方法是在使用sed命令是指定-n参数，该参数会抑制sed默认的输出：


```
$ sed -n 's/ MA/, Massachusetts/p' list
John Daggett, 341 King Road, Plymouth, Massachusetts
Eric Adams, 20 Post Road, Sudbury, Massachusetts
Sal Carpenter, 73 6th Street, Boston, Massachusetts
```
##### 正则表达式
书中的第3章的内容都是介绍正则表达式，这是因为sed的很多地方都需要用到正则表达式，比如操作指令的pattern部分以及替换命令中用到的匹配部分。关于正则这部分可以看Linux/Unix工具与正则表达式的POSIX规范，文章中已经讲得非常清楚了。