# 使用GDB调试Go程序

zhoukuo#2016-10-16

GDB是FSF(自由软件基金会)发布的一个强大的类UNIX系统下的程序调试工具。使用GDB可以做如下事情：

1. 启动程序，可以按照开发者的自定义要求运行程序。
2. 可让被调试的程序在开发者设定的调置的断点处停住。（断点可以是条件表达式）
3. 当程序被停住时，可以检查此时程序中所发生的事。
4. 动态的改变当前程序的执行环境。

目前支持调试Go程序的GDB版本必须大于7.1

务必保证执行如下操作(保证info goroutines可用)

vim ~/.gdbinit 添加下面一行：
```
add-auto-load-safe-path $GOROOT/src/pkg/runtime/runtime-gdb.py
```

把$GOROOT替换为你自己的路径

## 常用命令

### list

简写命令l,用来显示源代码,默认显示十行代码,后面可以带上参数显示的具体行，例如：list 15,显示十行代码,其中第15行在显示的十行里面的中间

### break

简写命令b,用来设置断点,后面跟上参数设置断点的行数,例如b 10在第十行设置断点

### delete

简写命令d,用来删除断点,后面跟上断点设置的序号,这个序号可以通过info breakpoints获取相应的设置的断点序号,如下是显示的设置断点序号

### backtrace

简写命令bt,用来打印执行的代码过程

### info

info命令用来显示信息,后面有几种参数,我们常用的有如下几种

    #### info locals

    显示当前执行的程序中的变量值

    #### info breakpoints

    显示当前设置的断点列表

    #### info goroutines

    显示当前执行的goroutine列表,带*的表示当前执行的

### print

简写命令p,用来打印变量或者其他信息,后面跟上需要打印的变量名,当然还有一些很有用的函数$len()和$cap(),用来返回当前string,slices或者maps的长度和容量

### whatis

用来显示当前变量的类型,后面跟上变量名

### next

简写命令n,用来单步调试.跳到下一步.当有断点之后.可以输入n跳转到下一步继续执行

### coutinue

简称命令c,用来跳出当前断点处,后面可以跟参数N,跳过多少次断点

### set variable

该命令用来改变运行过程中的变量值，格式如：set variable <var>=<value>

## 调试过程

我们通过下面这个代码来演示如何通过GDB来调试Go程序,下面是将要演示的代码：

```golang
package main

import (
  "fmt"
  "time"
)

func counting(c chan<- int) {
  for i := 1; i <= 10; i++ {
    time.Sleep(1 * time.Second)
    c <- i
  }
  close(c)
}

func main() {
  fmt.Println("main start")
  isexit := 0
  c := make(chan int)
  go counting(c)
  for count := range c {
    if isexit > 0 {
      break
    }
    fmt.Println("c:", count)
  }
  fmt.Println("main end")
}
```

编译文件,生成可执行文件gdb:
```bash
go build gdb.go
```

通过gdb命令启动调试
```bash
gdb gdb
```
启动之后首先看看这个程序是不是可以运行起来,只要输入run命令回车后程序就开始运行,程序正常的话可以看到程序输出如下,和我们在命令行直接执行程序输出是一样的：
```bash
(gdb) run
Starting program: /home/xtgxiso/src/gdb/gdb 
warning: no loadable sections found in added symbol-file system-supplied DSO at 0x2aaaaaaab000
main start
c: 1
c: 2
c: 3
c: 4
c: 5
c: 6
c: 7
c: 8
c: 9
c: 10
main end
[LWP 18714 exited]
[Inferior 1 (process 18714) exited normally]
```

好了,现在我们已经知道怎么让程序跑起来了,接下来开始给代码设置断点：
```bash
(gdb) b 25
```

Breakpoint 1 at 0x400dae: file /home/xtgxiso/src/gdb/gdb.go, line 25.
上面例子b 25表示在第25行设置了断点,之后输入run开始运行程序,现在程序在前面设置断点的地方停住了,我们需要查看断点相应上下文的源码,输入list就可以看到源码显示从当前停止行的前五行开始：
```bash
(gdb) run
Starting program: /home/xtgxiso/src/gdb/gdb 
warning: no loadable sections found in added symbol-file system-supplied DSO at 0x2aaaaaaab000
main start
[New LWP 19761]
[Switching to LWP 19761]

Breakpoint 1, main.main () at /home/xtgxiso/src/gdb/gdb.go:25
25                     fmt.Println("c:", count)
(gdb) list
20              go counting(c)
21              for count := range c {
22                      if isexit > 0 {
23                        break
24                      }
25                      fmt.Println("c:", count)
26              }
27              fmt.Println("main end")
28      }
```

现在GDB在运行当前的程序的环境中已经保留了一些有用的调试信息,我们只需打印出相应的变量,查看相应变量的类型及值：
```bash
(gdb) info locals
isexit = 0
count = 1
c = 0xc210039060
(gdb) p count
$1 = 1
(gdb) p c
$2 = (chan int) 0xc210039060
(gdb) whatis c
type = chan int
```

接下来该让程序继续往下执行,请继续看下面的命令
```gdb
(gdb) c
Continuing.
c: 1

Breakpoint 1, main.main () at /home/xtgxiso/src/gdb/gdb.go:25
25                      fmt.Println("c:", count)
(gdb) c
Continuing.
c: 2

Breakpoint 1, main.main () at /home/xtgxiso/src/gdb/gdb.go:25
25                      fmt.Println("c:", count)
```

每次输入c之后都会执行一次代码,又跳到下一次for循环,继续打印出来相应的信息,设想目前需要改变上下文相关变量的信息,跳过一些过程,得出修改后想要的结果：
```bash
(gdb) info locals
isexit = 0
count = 2
c = 0xf840001a50
(gdb) set variable isexit=1
(gdb) info locals
isexit = 1
count = 3
c = 0xf840001a50
(gdb) c
Continuing.
c: 3
main end
[LWP 11588 exited]
[Inferior 1 (process 11588) exited normally]
```

最后稍微思考一下，前面整个程序运行的过程中到底创建了多少个goroutine，每个goroutine都在做什么：
```bash
(gdb) run
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/xtgxiso/src/gdb/gdb 
warning: no loadable sections found in added symbol-file system-supplied DSO at 0x2aaaaaaab000
main start

Breakpoint 2, main.main () at /home/xtgxiso/src/gdb/gdb.go:25
25                      fmt.Println("c:", count)
(gdb) info goroutines 
* 1  running runtime.park
* 2  syscall runtime.notetsleepg
  3  waiting runtime.park
  4 runnable runtime.park
(gdb) goroutine 3 bt
#0  0x0000000000415586 in runtime.park (unlockf=void, lock=void, reason=void) at /usr/local/go/src/pkg/runtime/proc.c:1342
#1  0x0000000000420084 in runtime.tsleep (ns=void, reason=void) at /usr/local/go/src/pkg/runtime/time.goc:79
#2  0x000000000041ffb1 in time.Sleep (ns=void) at /usr/local/go/src/pkg/runtime/time.goc:31
#3  0x0000000000400c38 in main.counting (c=0xc210039060) at /home/xtgxiso/src/gdb/gdb.go:10
#4  0x0000000000415750 in ?? () at /usr/local/go/src/pkg/runtime/proc.c:1385
#5  0x000000c210039060 in ?? ()
#6  0x0000000000000000 in ?? ()
```

通过查看goroutines的命令我们可以清楚地了解goruntine内部是怎么执行的，每个函数的调用顺序已经明明白白地显示出来了.

本文简单介绍了GDB调试Go程序的一些基本命令，通过上面的例子演示，如果你想获取更多的调试技巧请参考官方网站的GDB调试手册，还有GDB官方网站的手册。

http://www.gnu.org/software/gdb/
