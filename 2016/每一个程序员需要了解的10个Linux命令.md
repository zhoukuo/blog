# 每一个程序员需要了解的10个Linux命令

zhoukuo@2016-10-30

作为一个程序员，在软件开发职业生涯中或多或少会用到Linux系统，并且可能会使用Linux命令来检索需要的信息。本文将为各位开发者分享10个有用的Linux命令，希望对你会有所帮助。

## 1.man命令

第一个你需要知道的Linux命令就是man命令，该命令可以显示指定命令的用法和描述。比如你想知道ls命令的用法和选项，可以在终端执行“man ls”：

```
语法: man <command name>
man ls
```

```bash
root@devopscube:~# man ls
LS(1)                            User Commands                           LS(1)
NAME
       ls - list directory contents
SYNOPSIS
       ls [OPTION]... [FILE]...
DESCRIPTION
       List  information  about  the FILEs (the current directory by default).
       Sort entries alphabetically if none of -cftuvSUX nor --sort  is  speciâ
       fied.
       Mandatory  arguments  to  long  options are mandatory for short options
       too.
       -a, --all
              do not ignore entries starting with .
```

## 2.touch，cat和less命令

touch命令可以在Linux系统中创建大小为0的任意类型文件，作为程序开发者，当你需要在Linux服务器上创建文件时，可以使用touch命令：

```
语法: touch <filename>
touch demo.txt
```

```bash
root@devopscube:~# touch demo.txt
root@devopscube:~# ls
demo.txt
```

cat命令用来查看文件的内容，但是使用cat命令并不能编辑文件的内容，它仅仅是可以浏览文件内容。cat命令不支持键盘上下键翻页。

```
语法: cat <filename>
cat demo.txt
```

同样的less命令也可以让你浏览文件，less命令非常快，并且支持上下键查看文件的开头和末尾。然而more命令和它类似，只是more命令只能用enter键实现文件的向前翻页，不支持回退。

```
语法: less <filename>
      more <filename>
```

```bash
less demo.txt
more demo.txt
```

## 3.sort命令

sort命令用来对文件内容进行排序。创建一个名为test.txt的文件，并且把以下内容拷贝到该文件中：

```
1 mike level intermediate jan
10 lucy level beginer mar
45 Dave level expert dec
4 dennis start beginner jul
7 Megan employee trainee feb
58 Mathew Head CEO nov
```

上面的例子中，第二列是名称，所以如果你想对名称列按字母排序，就可以使用“-k”选项，并标注列号，比如“-k2”：

```
语法: sort
```

```bash
sort -k2 test.txt
```

排序结果

```bash
root@devopscube:~# sort -k2 test.txt
45 Dave level expert dec
4 dennis start beginner jul
10 lucy level beginer mar
58 Mathew Head CEO nov
7 Megan employee trainee feb
1 mike level intermediate jan
```

第一列是数字，如果你想按数字排序，可以使用“-h”选项。如果数字在不同列上，你可以在“-h”选项后使用“-k”选项：

```bash
root@devopscube:~# sort -h test.txt  
1 mike level intermediate jan
4 dennis start beginner jul
7 Megan employee trainee feb
10 lucy level beginer mar
45 Dave level expert dec
58 Mathew Head CEO nov
```

最后一列是月份，你可以使用“-M”选项来让文件内容按月份排序：

```bash
root@devopscube:~# sort -k5 -M test.txt
1 mike level intermediate jan
7 Megan employee trainee feb
10 lucy level beginer mar
4 dennis start beginner jul
58 Mathew Head CEO nov
45 Dave level expert dec
```

注：如果你想消除重复的行，可以在sort命令后使用“-u”选项。

使用“-r”选项，是文件倒序排列：

```bash
root@devopscube:~# sort -h -r test.txt
58 Mathew Head CEO nov
45 Dave level expert dec
10 lucy level beginer mar
7 Megan employee trainee feb
4 dennis start beginner jul
1 mike level intermediate jan
```

## 4.grep命令：

grep命令非常强大，系统管理员经常会用到它。grep命令可以在文件中搜索指定格式的字符串，同时对其进行标准输出。

```
语法: grep "<search string>" <filename> 
      grep "Mathew" test.txt
```

```bash
root@devopscube:~# grep "dennis" test.txt
dennis start beginner jul
```

上面命令的输出结果是包含该子字符串的，如果你想检索完整的单词，你需要添加“-i”选项。同时，也可以用grep命令在多个文件中搜索字符串，命令代码如下：

```bash
grep "dennis" test1.txt test2.txt test3.txt
```

当然你也可以用正则表达式来匹配字符串。

## 5.cut命令

cut命令可以让你用列或者分隔符提取文件中的指定部分。如果你要列出文件中某列的全部内容，可以使用“-c”选项。例如，下面将从test.txt文件中提取第1、2列的全部内容。

```bash
cut -c1-2 test.txt
root@devopscube:~# cut -c1-2 test.txt
1
10
45
4
7
58
```

如果你希望从文件中提取指定的字符串，那么你可以使用分隔符选项“-d”和“-f”选项选中列。例如，我们可以利用cut命令提取names列：

```bash
cut -d' ' -f2 test.txt
root@devopscube:~# cut -d' ' -f2 test.txt
mike
lucy
Dave
dennis
Megan
Mathew
```

下面的例子从/etc/passd file中提取users列：

```bash
cut -d':' -f1 /etc/passwd
```

## 6.sed命令

sed 是一种在线编辑器，它一次处理一行内容。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”（pattern space），接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有 改变，除非你使用重定向存储输出。

如果你想通过搜索替换文件中的指定内容，你可以使用“s”选项来检索到它然后将它替换。

```
语法: sed 's/<old-word>/<new-word>/' test.txt
```

例如，在test.txt文件中用“michael”替换“mike”：

```bash
sed 's/mike/michael/' test.txt
root@devopscube:~# sed 's/mike/michael/' test.txt
1 michael level intermediate jan
10 lucy level beginer mar
45 Dave level expert dec
4 dennis start beginner jul
7 Megan employee trainee feb
58 Mathew Head CEO nov
6、tar命令
```

tar命令用来压缩和解压缩文件，其中经常会用到“-cf”和“-xf”选项。

```
语法: tar <options> <archive-name> <file/folder name>
```

让我们将test.txt文件打包：
```bash
tar -cf test.tar test.txt
root@devopscube:~# tar -cf test.tar test.txt
root@devopscube:~# ls
test.tar  test.txt
```

用“-C”选项将刚才打包好的test.tar文件解压缩至“demo”目录：

```bash
tar -xf test.tar -C /root/demo/
root@devopscube:~# tar -xf test.tar -C /root/demo/
root@devopscube:~# cd demo/
root@devopscube:~/demo# ls
test.txt
```

## 7.find命令

find命令用来检索文件，可以用“-name”选项来检索指定名称的文件：

```bash
find -name  find -name test.txt
root@devopscube:/home/ubuntu# cd ~
root@devopscube:~# find -name test.txt
./demo/test.txt
./test.txt
```

你也可以用“/ -name”来检索指定名称的文件夹：

```bash
find / -name passwd
root@devopscube:~# find / -name passwd
/etc/cron.daily/passwd
/etc/pam.d/passwd
/etc/passwd
/usr/share/lintian/overrides/passwd
```

## 8.diff命令

diff命令用来找出2个文件的不同点。diff命令通过分析文件内容，然后将不同的行打印出来，下面的例子可以找出两个文件test和test1的不同点：

```
语法: diff <filename1> <filename2>
      diff test.txt test1.txt
```

```bash
root@devopscube:~# diff test.txt test1.txt
7c7
< 59 sdfsd
---
> 59 sdfsd  CTO dec
```

## 9.uniq命令

uniq命令用来过滤文件中的重复行：

```
语法: uniq 
```

```bash
uniq test.txt
root@devopscube:~# uniq test.txt
1 mike level intermediate jan
10 lucy level beginer mar
45 Dave level expert dec
4 dennis start beginner jul
7 Megan employee trainee feb
58 Mathew Head CEO nov
```

## 10.chmod命令

chmod命令用来改变文件的读/写/执行权限，权限数值如下所示：

* 4 - read permission
* 2 - write permission
* 1 - execute permission
* 0 - no permission

下面的命令可以给test.txt文件赋最高的权限：
```bash
chmod 755 test.txt
```

如果你对本文有任何的想法和意见，欢迎给出你的点评，我很期待！
