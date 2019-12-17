# The Linux Command Line
## GNU 和 Linux
* Linux仅仅是一个内核，不包含外围的应用程序，所以现在我们使用的Linux系统，应该完整地称之为GNU/Linux才对
* GNU工程已经开发了一个被称为“GNU”（GNU是“GNU's Not Unix”的缩写）的、对Unix向上兼容的完整的自由软件系统(free software system)

## 第一章 Learning the shell
### 名词解释
* shell: a **program** that takes **keyboard commands** and passes them to the **operating system** to carry out。但也可以加入鼠标操作，如果通过按住鼠标左键并将鼠标拖动到某个文本上（或双击某个单词）来突出显示某个文本，则该文本将被复制到x维护的缓冲区中。
* bash: 几乎所有的Linux发行版都提供了一个来自GNU项目bash的shell程序。“bash”是“Bourne Again SHell”的缩写，它是基于原版Steve Bourne编写的unix shell程序的增强版本。
* terminal: 和shell交互的程序，terminal emulator
* shell prompt：格式 [username@machinename current_directory] \$, 提示就绪可以输入命令。如果\$换成#,表示当前是superuser权限

### 基本命令
```
command -options arguments
```
* history: 一般记录500条历史命令
* date: 显示当前时间和日期
* cal: 显示当前月的日历
* df: 当前硬盘驱动的空闲空间
* free: 当前内存的空闲空间
* exit: 关闭当前终端调试器
### 文件系统导航命令(navigation)
* 文件系统是单一的树结构
* /home/user目录是唯一允许用户对文件进行写入的地方
* 文件名区分大小写， 文件名中的标点符号限制为句点、破折号和下划线，不要有空格
* 文件没有扩展名的意义
* pwd: 打印当前工作目录
* cd: change directory
	* cd -: 回到上次访问的当前目录
	* cd ~user_name: 进入username的home目录
* ls: list directory contents
	* 以句点字符(period character)开头的文件名是隐藏的
- 大多数程序都安装在/usr/bin目录下
- "\." 指的是工作目录，"\.\." 指的是工作目录的父目录
### Exploring the System
* ls
	* ls -lh
* file
	* file 命令会打印出文件内容的简单描述
* less
	* 浏览文本文件
	* /character
	* n
	* q
[http://www.pathname.com/fhs/pub/fhs-2.3.pdf](http://www.pathname.com/fhs/pub/fhs-2.3.pdf)

symbolic link / hard link
symbolic link: 文件名指向另一个文件名
hard link:文件名本身就是一个hard link，ls -li可以看到文件指向同一索引

man whatis info

tee

文件描述符
0标准输入
1标准输出
2标准错误输出

2>&1 将标准错误输出重定向到标准输出

2> /dev/null 处理不需要的输出

- \* 这种通配符工作机制叫做路径名展开
- \~ 当它用在 一个单词的开头时，它会展开成指定用户的家目录名，如果没有指定用户名，则是当前用户的家目录
- 花括号展开可以从一个包含花括号的模式中 创建多个文本字符串
	- ```mkdir {2018..2019}-0{1..9} {2018..2019}-{10..12}```
- printenv
- 变量替换
|格式|描述|
|:-:|:-:|
|\${var}|Substitue the value of _var_.|
|\${var:-word} | If _var_ is null or unset, _word_ is substituted for **var**. The value of _var_ does not change. |
|\${var:=word}|If _var_ is null or unset, _var_ is set to the value of **word**.|
|\${var:?message}|If _var_ is null or unset, _message_ is printed to standard error. This checks that variables are set correctly.
|\${var:+word}|If _var_ is set, _word_ is substituted for var. The value of _var_ does not change.


Original file mode
--- rw- rw- rw-
Mask
000 000 000 010 (umask 0002)
Result
--- rw- rw- r--

Original file mode
--- rw- rw- rw-
Mask
000 000 010 010 (umask 0022)
Result
--- rw- r-- r--

```
su -c 'command'
```
使用这种模式，命令传递到一个新 shell 中执行。把命令用单引号引起来很重要，因为我们不想 命令在我们的 shell 中展开，但需要在新 shell 中展开。

<!--stackedit_data:
eyJoaXN0b3J5IjpbMzk5NDQ3MTI4XX0=
-->