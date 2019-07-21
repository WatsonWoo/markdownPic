## 一些shell中常用技巧(三)高级篇
***
转自: 团子的小窝
http://kodango.com/simple-bash-programming-skills-3


###### >团子的博文已经写得很不错了，分享了他自己的经验。本专题以团子的博文为主，部分地方做了我的解释。
>高级篇我现阶段用到的比较少，基本没注释，此处只是先mark上。
---
#### 1. 替换语法${parameter/pattern/string}的妙用


`${parameter/pattern/string}`将parameter中匹配pattern的部分替换成string，例如下面的例子将字符串中的e替换成x：


```
$ str="three"
$ echo "${str/e/x}"   # thrxe
```

如果pattern部分以/开头，表示替换parameter中所有匹配的内容，例如：


```
$ str="three"
$ echo "${str//e/x}"  # thrxx
```

如果pattern部分以#开头，表示仅当parameter开始处匹配pattern的时候替换，例如：


```
str="three"
$ echo "${str/#e/x}" # three
$ echo "${str/#t/x}" # xhree
```

与此对应地是，如果pattern部分以%开头，表示仅当parameter结尾处匹配pattern的时候替换，例如：


```
$ str="three"
$ echo "${str/%e/x}" # threx
```

如果string部分为空，匹配pattern的部分被删除（替换为空），例如：


```
$ str="three"
$ echo "${str/h/}"  # tree
```

这个时候第二个斜杠可以删除，即：`echo "${str/h}"`

如果parameter是一个数组会怎么样呢？有兴趣的可以看看Bash的man手册说明：


```
man -P 'less -p "\\$\{parameter/pattern/string}"' bash
```

#### 2. +=运算符
有一天，我看到这样一个用法：


```
$ arr=(1 2 3)
$ arr+=(4 5)
```

原来数组还可以这样相加，后来我看了下Bash的手册，确实有一段这么说明的，这里未引用这段文字，有兴趣的可以查看[Bash Reference Manual](http://www.gnu.org/software/bash/manual/bashref.html#Shell-Parameters)。

自然地我们会想到如果一个变量是数字，是否也可以用+=作运算呢？


```
$ i=1
$ i+=1
```

>但是，运行后你会发现i的结果并不为2，而是11，这里bash并不认为i是一个整数，而是作为字符串。

这时,`可以通过declare声明一个变量为整数`，上面的问题就解决了：


```
$ declare -i int=1
$ int+=1
$ echo $int
2
```

#### 3. Here document不为人知的用法
Shell学得越多，越会发现一些神奇的用法，每天都觉得自己实在是一个刚入门的菜鸟。

一般的here document的用法是这样的：


```
$ cat b.sh 
cat<<EOF
hello, $USER
EOF
$ sh b.sh 
hello, kodango
```

here document中的变量都是会被展开的，那能不能不展开呢？答案是可以的，将EOF有引号括起来就可以：


```
$ cat b.sh 
cat<<"EOF"
hello, $USER
EOF
$ sh b.sh 
hello, $USER
```

一般here document用得最多的是在帮助函数(help\usage)函数里面，因为在这里我们要写一大段的脚本用法。

如果你有强迫症（比如我），有时候使用here document的时候会很不爽，因为here document里面每行首部的空格都会被保留，而如果要顶格写，在缩进的地方又会有点打乱结构，例如：


```
$ cat b.sh
# part 1
if :; then
    cat << EOF
    hello, $USER    
EOF
fi

# part 2
if :; then
    if :; then
        cat << EOF
hello, $USER    
EOF
    fi
fi
```

上面的脚本执行的结果为：


```
$ sh b.sh 
    hello, kodango   # part 1 result
hello, kodango       # part 2 result
```

有没有办法既兼顾到缩进又能不保留行首空格呢？

答案也是肯定的，只不过语法又要稍稍变一下，现在在<<的后面加一个短横，这个用法下，行首的Tab字符都会被忽略：


```
$ cat b.sh 
if :; then
    cat <<- EOF
	hello, $USER    
EOF
fi
$ sh b.sh 
hello, kodango
fi
```

一定要是Tab键哦，空格也是不可以的，在vim里面还要注意如果设置了smarttab选项，行首插入的Tab键会替换成相应个数的空格（这里可以按ctrl+v tab插入实际的空格）。

关于这一节的内容，可以进一步参考[Redirection#here_documents [Bash Hackers Wiki]](http://wiki.bash-hackers.org/syntax/redirection#here_documents)。

#### 4. 使用内置命令declare显示脚本中定义的函数
declare的-F选项可以列出脚本中定义的函数名称：


```
$ cat b.sh 
function one()
{
    :
}

function two()
{
    :
}

declare -F | sed 's/declare -f //'
$ sh b.sh 
one
two
```

#### 5. 嵌套函数还可以这么用
Bash中可以嵌套函数定义，即在一个函数中定义另外一个函数，例如：


```
[root@localhost ~]# cat nest.sh
#!/bin/bash

function out()
{
    echo "out"
    
    function inner() {
        echo "inner"
    }
}

inner
out
inner
```

这里out函数里面定义了inner函数，形成嵌套函数。但是，执行上面的例子会出错（nest.sh: line 12: inner: command not found），这是因为这是后inner函数还没定义。一旦out函数执行之后，inner函数就被定义了。整个例子的执行结果是这样的：


```
[root@localhost ~]# sh nest.sh 
nest.sh: line 12: inner: command not found
out
inner
```

看到这里，你可能会想嵌套函数有什么用？事实上，在大多数情况下，我们基本不会用到嵌套函数。但是它并非一无是处，比如下面的例子就向我们展示了嵌套函数的神奇用法。

假设，我们要定义一个调试函数，同时需要一个开关控制该函数是否输出调试日志，最简单的写法是：


```
function log()
{
    if [ "$verbose" = "1" ]; then
        echo "$@"
    fi
}
```

它可以完成任务，但是唯一美中不足的是，每次调用该函数都要判断verbose的值是否为1。这时候可以使用嵌套函数来弥补这个不足：


```
#!/bin/bash

verbose=${1:-1}

function log()
{
    if [ $verbose -eq 1 ]; then
        function log() {
            echo "$@"
        }

        echo "$@"
    else
        function log() {
            :
        }
    fi
}

log what is your name
log my name is kodango
```

上面的例子中，根据verbose的值定义了两个同名的log函数来覆盖之前的旧函数，以后调用的函数就都是后定义的函数了。

#### 6. 删除ps auxf | grep python结果中的grep进程
在shell脚本中，经常需要利用ps和grep命令一起在查找进程相关的信息，尤其是针对python/java/shell等脚本进程，因为pidof本身不大支持查找脚本进程对应的pid。

在用ps auxf | grep python的时候，一个很恼人的事情是，经常会出现多余的grep进程：


```
$ ps auxf | grep python
kodango    18832  0.0  0.0 674192 10444 ?        Sl   23:19   0:00  python test.py
kodango    63860  0.0  0.0  61180   752 pts/2    S+   23:28   0:00  grep python
```

所以我们需要再加一个grep -v grep来排除它。

之前一直弄不明白为什么会这样，今天在看BashPitfalls的时候，终于明白原因了，stackoverflow上也有一个回答解释得很好。

shell在执行以上命令的时候，其实创建了一个管道，并且fork了两个子进程：ps auxf与grep python，并且将管道读的这一端绑定到grep的标准输入，管道写的这一段绑定到ps的标准输出。ps将自己的输出写到管道，grep从管道中读取输入。可能在这个时候，ps与grep是同时执行的，所以ps的结果中也会包含grep进程的信息。

还有一个解决方法是巧用正则表达式：


```
$ ps auxf | grep [p]ython
```

#### 7. Shell如何实现timeout功能
有时候我们不希望某个命令执行太久，所以如果在给定的时间内没有完成，能够杀掉这个命令对应的进程，这就是timeout功能，可惜bash没有提供该功能。所以就得我们自己来实现。

实现代码如下所示：


```
function timeout()
{
    local time cmd pid

    if echo "$1" | grep -Eq '^[0-9]+'; then
        time=$1
        shift && cmd="$@"
    else
        time=5
        cmd="$@"
    fi

    $cmd &
    pid=$!

    while kill -0 $pid &>/dev/null; do
        sleep 1
        let time-=1

        if [ "$time" = "0" ]; then
            kill -9 $pid &>/dev/null
            wait $pid &>/dev/null
        fi
    done
}
```

假设有一个测试脚本sleep.sh：


```
$ cat sleep.sh
echo "sleep $1 seconds"
sleep $1
echo "awake from sleep"
```

现在利用我们写的timeout函数来达到超时kill功能：


```
$ time sh timeout.sh 2 'sh sleep.sh 100'
sleep 100 seconds

real	0m2.005s
user	0m0.002s
sys	0m0.001s
```

看最终执行的时间，差不多就是2秒钟。

上面timeout函数实现的代码中，利用了两个技巧：

kill -0 $pid：发送信号0给进程，可以检查进程是否存活，如果进程不存在或者没有权限，则返回错误，错误码为1；
wait $pid &>/dev/null：等待某个进程退出返回，这样相对比较优雅，同时将错误重定向到黑洞，从而隐藏后台进程被kill的错误输出；
8. 利用/etc/inittab实现watchdog
还在为实现watch dog而头疼吗，其实inittab中已经包含了该功能。可以将自己的脚本或者程序写到inittab文件中：

tt:2345:respawn:/home/kodango/sleep.sh 100
然后执行telinit q使其生效，ps看下该脚本是否已经在运行了，尝试kill后，又会被起起来。

#### 9. 慎用波浪号展开
在shell中对比下面两种用法：


```
$ home1=~kodango
$ home2="~kodango"	
$ echo -e "$home1\n$home2"
/Users/kodango
~kodango
```

第一个变量赋值，波浪号正确展开，所以我们得到了kodango用户的家目录地址；第二个变量，我们使用了双引号，这个时候波波浪号并没有展开。这是一个比较容易出错的地方。

还有一点要注意的地方是，波浪号展开只在:或者=号后面才会执行。所以：


```
$ path=1~kodango
$ echo "$path"
1~kodango

$ path=1:~kodango
$ echo "$path"
1:/Users/kodango
```

为什么要在:后面也可以展开呢？想想PATH的定义吧。

$. 最近淘到的一些实用的shell文章
>BashPitfalls - Greg's Wiki
http://mywiki.wooledge.org/BashPitfalls
>ProcessManagement - Greg's Wiki
http://mywiki.wooledge.org/ProcessManagement
>BashGuide - Greg's Wiki
http://mywiki.wooledge.org/BashGuide
>BashFAQ - Greg's Wiki
http://mywiki.wooledge.org/BashFAQ