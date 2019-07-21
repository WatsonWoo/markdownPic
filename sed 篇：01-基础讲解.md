## sed 篇：01-基础讲解
### 模式空间与地址匹配
转自：团子的小窝

http://kodango.com/sed-and-awk-notes-part-2

grep，sed，awk成为shell三大剑客，是处理文本的三大利器。
比较好的讲解网站是:
>**高级Bash脚本编程指南: 一本深入学习shell脚本艺术的书籍**
Appendix C(附录C部分)
>http://www.reddragonfly.org/abscn/sedawk.html

但是‘团子的小窝’讲解的也还不错，这里就用团子的小窝文档

本sed专题主要以团子的博文为主，加上我自己理解。
如果有不对的地方，请留言指正。
***

sed并非是将一个编辑命令分别应用到每一行，然后再取下一个编辑命令。恰恰相反，**sed是以行的方式来处理的。另外一方面，每一行都是被读入到一块缓存空间，该空间名为模式空间(pattern space)**，这是一个很重要的概念，在后文中会多次被提及。因此，**sed操作的都是最初行的拷贝，同时后续的编辑命令都是应用到前面的命令编辑后输出的结果，所以编辑命令之间的顺序就显得格外重要**。
##### 简单例子
让我们来看一个非常简单的例子，将一段文本中的pig替换成cow，并且将cow替换成horse：


```
$ sed 's/pig/cow/;s/cow/hores/' input
```

初看起来好像没有问题，但是实际上是错误的，原因是第一个替换命令将所有的pig都替换成cow，紧接着的替换命令是基于前一个结果处理的，将所有的cow都替换成horse，造成的结果是全部的pig/cow都被替换成了horse，这样违背了我们的初衷。

在这种情况下，只需要调换下两个编辑命令的顺序：


```
$ sed 's/cow/hores/;s/pig/cow/' input
```

##### 模式空间的转换
sed只会缓存一行的内容在模式空间，这样的好处是sed可以处理大文件而不会有任何问题，不像一些编辑器因为要一次性载入文件的一大块内容到缓存中而导致内存不足。下面用一个简单的例子来讲解模式空间的转换过程，如下图所示：
[image](http://kodango.oss.aliyuncs.com/2013/05/sed_pattern_space.png)
现在要把一段文本中的Unix System与UNIX System都要统一替换成The UNIX Operating System，因此我们用两句替换命令来完成这个目的：


```
s/Unix /UNIX /
s/UNIX System/UNIX Operating System/
```

对应上图，过程如下：

1. 首先一行内容The Unix System被读入模式空间；
2. 应用第一条替换命令将Unix替换成UNIX；
3. 现在模式空间的内容变成The UNIX System；
4. 应用第二条替换命令将UNIX System替换成UNIX Operating System；
5. 现在模式空间的内容变成The UNIX Operating System；
6. 所有编辑命令执行完毕，默认输出模式空间中的行；

##### 地址匹配
地址匹配在sed中是非常重要的一块内容，因为它限制sed的编辑命令到底应用在哪些行上。默认情况下，sed是全局匹配的，即对所有输入行都应用指定的编辑命令，这是因为sed依次读入每一行，每一行都会成为当前行并被处理，所以s/CA/California/g会将所有输入行的CA替换成California。这一点跟vi/vim是不一样的，众所周知，vim的替换命令默认是替换当前行的内容，除非你指定%s才会作全局替换。

可以通过指定地址来限制sed的处理范围，例如只想将替换包含Sebastopol的行：


```
Sebastopol/s/CA/California/
```
/Sebastopol/是一个正则表达式匹配包含Sebastopol的行，因此像行“San Francisco, CA”则不会被替换。

sed命令中可以包含0个、1个或者2个地址（地址对），地址可以为正则表达式（如/Sebastopol/），行号或者特殊的行符号（如$表示最后一行）：

如果没有指定地址，默认将编辑命令应用到所有行；
如果指定一个地址，只将编辑命令应用到匹配该地址的行；
如果指定一个地址对（addr1,addr2)，则将编辑命令应用到地址对中的所有行（包括起始和结束）；
如果地址后面有一个感叹号（！），则将编辑命令应用到不匹配该地址的所有行；
为了方便理解上述内容，我们以删除命令（d）为例，默认不指定地址将会删除所有行：


```
$ sed 'd' list 
$
```
指定地址则删除匹配的行，如删除第一行：


```
$ sed '1d' list 
Alice Ford, 22 East Broadway, Richmond VA
Orville Thomas, 11345 Oak Bridge Road, Tulsa OK
Terry Kalkas, 402 Lans Road, Beaver Falls PA
Eric Adams, 20 Post Road, Sudbury MA
Hubert Sims, 328A Brook Road, Roanoke VA
Amy Wilde, 334 Bayshore Pkwy, Mountain View CA
Sal Carpenter, 73 6th Street, Boston MA
```
或者删除最后一行，$符号在这里表示最后一行，这点要下正则表达式中的含义区别开来：

```
$ sed '$d' list 
John Daggett, 341 King Road, Plymouth MA
Alice Ford, 22 East Broadway, Richmond VA
Orville Thomas, 11345 Oak Bridge Road, Tulsa OK
Terry Kalkas, 402 Lans Road, Beaver Falls PA
Eric Adams, 20 Post Road, Sudbury MA
Hubert Sims, 328A Brook Road, Roanoke VA
Amy Wilde, 334 Bayshore Pkwy, Mountain View CA
```

这里通过指定行号删除，行号是sed命令内部维护的一个计数变量，该变量只有一个，并且在多个文件输入的情况下也不会被重置。

前面都是以行号来指定地址，也可以通过正则表达式来指定地址，如删除包含MA的行：

```
$ sed '/MA/d' list 
Alice Ford, 22 East Broadway, Richmond VA
Orville Thomas, 11345 Oak Bridge Road, Tulsa OK
Terry Kalkas, 402 Lans Road, Beaver Falls PA
Hubert Sims, 328A Brook Road, Roanoke VA
Amy Wilde, 334 Bayshore Pkwy, Mountain View CA
```
通过指定地址对可以删除该范围内的所有行，例如删除第3行到最后一行：


```
$ sed '2,$d' list 
John Daggett, 341 King Road, Plymouth MA
```
使用正则匹配，删除从包含Alice的行开始到包含Hubert的行结束的所有行：


```
$ sed '/Alice/,/Hubert/d' list 
John Daggett, 341 King Road, Plymouth MA
Amy Wilde, 334 Bayshore Pkwy, Mountain View CA
Sal Carpenter, 73 6th Street, Boston MA
```
当然，行号和地址对是可以混用的：


```
$ sed '2,/Hubert/d' list 
John Daggett, 341 King Road, Plymouth MA
Amy Wilde, 334 Bayshore Pkwy, Mountain View CA
Sal Carpenter, 73 6th Street, Boston MA
```

如果在地址后面指定感叹号（！），则会将命令应用到不匹配该地址的行：


```
$ sed '2,/Hubert/!d' list 
Alice Ford, 22 East Broadway, Richmond VA
Orville Thomas, 11345 Oak Bridge Road, Tulsa OK
Terry Kalkas, 402 Lans Road, Beaver Falls PA
Eric Adams, 20 Post Road, Sudbury MA
Hubert Sims, 328A Brook Road, Roanoke VA
```

以上介绍的都是最基本的地址匹配形式，GNU Sed（也是我们用的sed版本）基于此添加了几个扩展的形式，具体可以看man手册，或者可以看我之前写的[Sed命令地址匹配问题总结](http://kodango.com/sed-address-matching-summary)一文。

上面说的内容都是对匹配的地址执行单个命令，如果要执行多个编辑命令要怎么办？sed中可以用{}来组合命令，就好比编程语言中的语句块，例如：
```
$ sed -n '1,4{s/ MA/, Massachusetts/;s/ PA/, Pennsylvania/;p}' list 
John Daggett, 341 King Road, Plymouth, Massachusetts
Alice Ford, 22 East Broadway, Richmond VA
Orville Thomas, 11345 Oak Bridge Road, Tulsa OK
Terry Kalkas, 402 Lans Road, Beaver Falls, Pennsylvania
```
