---
title: Bash脚本语法1
date: 2020-08-18T11:37:00+08:00
lastmod: 2020-08-18
tags: [linux]
categories: [爱编程爱技术的孩子]
slug: Bash script grammer 1 
---

很多时候都要求能使用 Shell 脚本进行编程，本文是根据阮一峰大神的 [系列教程](https://wangdoc.com/bash/intro.html) 总结的基本知识。

<!--more-->

## 1. Shell 和 Bash

Shell 是一个程序，为用户提供和内核的交互界面，一般由用户从键盘输入命令，然后送给操作系统指向，执行完毕将结果返回给用户，因此又称为命令行环境。

只要能给用户提供命令行界面用于和内核交互，就叫 Shell，因此 Shell 有很多种，比如

- Bourne Shell（sh）
- Bourne Again shell（bash）
- C Shell（csh）

Bash 是目前最常用的 Shell，一般也是默认的 Shell。

具有图形界面的 Linux 打开的命令行一般是终端模拟器，顾名思义，就是模拟命令行环境的，不是真正的命令行，但使用和直接使用非图形界面的命令行没有区别。因此我们随后不区分终端和命令行，全部使用命令行这个称呼。

图形界面下使用 `Ctrl + Alt + T` 打开命令行，默认位于用户主目录 `~`，提示符为美元符号 `$`，输入 `exit` 命令或使用快捷键 `Ctrl + D` 退出当前命令行。

## 2. 预备知识

使用 `echo` 命令可以在屏幕上输入文本，添加引号可以输出多行文本，添加 `-e`  参数会解释引号中的特殊字符。该命令经常使用。

```bash
$ echo hello world
hello world

$ echo "hello
world"
hello
world

$ echo -e "hello\nworld"
hello
world
```

Bash 单个命令一般都是一行，按下回车开始执行，比较长的命令可以写成多行，只需要在每一行的末尾加上反斜杠即可

```bash
$ echo foo \
bar
foo bar
```

`&&` 和 `||` 用于连接两个命令，前者的含义是如果第一个命令执行成功，执行第二个命令；后者的含义是如果第一个命令执行失败，执行第二个命令，第一个命令执行成功就不会执行第二个命令了。

```bash
$ mkdir foo && ls foo
$ mkdir foo || mkdir bar
mkdir: 无法创建目录 “foo”: 文件已存在
$ ls 
foo bar
```

Bash 只有一种数据类型，就是字符串。不管用户输入什么数据，Bash 都视为字符串。

还有一些实用的快捷键

- `Ctrl + L`：清除屏幕并将当前行移到页面顶部。
- `Ctrl + C`：中止当前正在执行的命令。
- `Shift + PageUp`：向上滚动。
- `Shift + PageDown`：向下滚动。
- `Ctrl + U`：从光标位置删除到行首。
- `Ctrl + K`：从光标位置删除到行尾。
- `↑`，`↓`：浏览已执行命令的历史记录。
- `Tab`：补全未完全输入的命令或路径

## 3. 模式扩展

用户输入的命令中有时包括一些特殊的字符，Shell 会先对这些字符进行替换，然后才开始执行，这个过程叫做模式扩展。Shell 提供 8 种模式扩展。

1. `~`：用户主目录

2. `?`：单个字符，但不包括空字符

3. `*`：任意数量的任意字符，包括 0 个

4. `[...]`：匹配方括号中出现的任意字符，如果前面加了 `^` 或 `!` 表示匹配方括号中没有出现的字符

   ```bash
   # 只存在文件 a.txt
   $ ls [ab].txt
   a.txt
   
   # 存在 aaa、bbb、aba 三个文件
   $ ls ?[!a]?
   aba bbb
   ```

   `[start-end]`：匹配一个连续的范围，同样，实用 `!` 可以匹配不属于该范围的字符

   ```bash
   # 存在文件 a.txt、b.txt 和 c.txt
   $ ls [a-c].txt
   a.txt
   b.txt
   c.txt
   ```

5. `{...}`：匹配大括号中的所有字符

   ```bash
   $ echo d{a,e,i,u,o}g
   dag deg dig dug dog
   ```

   `{start...end}`：扩展成一个连续序列

   ```bash
   $ echo {1..4}
   1 2 3 4
   ```

6. `$`：将 `$` 开头的变量替换为变量的值

   ```bash
   $ echo $SHELL
   /bin/bash
   ```

7. `$(...)`：扩展成另一个命令的运行结果，该命令所有输出都作为返回值

   ```bash
   $ echo $(date)
   Tue Jan 28 00:01:13 CST 2020
   ```

8. `$((...))`：扩展成整数运算的结果

   ```bash
   $ echo $((2 + 2))
   4
   ```

模式扩展先于正则表达式出现，可以看作原始的正则表达式，但没有正则表达式灵活。

## 4. 变量

用`env`命令显示所有环境变量，`echo`命令查看单个环境变量的值

```bash
$ env
$ echo $PATH
```

BASH 的变量名区分大小写，由于环境变量一般使用全大写，用户自定义变量一般也使用全大写。

### 4.1 创建变量

变量声明语法如下，等号两边不可有空格

```bash
variable=value
```

如果变量的值中有空格，使用引号包围

```bash
myvar="hello world"
```

BASH 没有数据类型的概念，所有变量都是字符串。

下面是一些自定义变量的例子。

```bash
a=z                     # 变量 a 赋值为字符串 z
b="a string"            # 变量值包含空格，就必须放在引号里面
c="a string and $b"     # 变量值可以引用其他变量的值
d="\t\ta string\n"      # 变量值可以使用转义字符，echo 输出要加 -e 参数
e=$(ls -l foo.txt)      # 变量值可以是命令的执行结果
f=$((5 * 7))            # 变量值可以是数学运算的结果
```

变量可以重复赋值，后面的赋值会覆盖前面的赋值。

```bash
$ foo=1
$ foo=2
$ echo $foo
2
```

上面例子中，变量`foo`的第二次赋值会覆盖第一次赋值。

### 4.2 读取变量

读取变量的时候，直接在变量名前加上`$`就可以了。

```bash
$ foo=bar
$ echo $foo
bar
```

如果变量不存在，Bash 不会报错，而会输出空字符。

读取变量的时候，变量名也可以使用花括号`{}`包围，比如`$a`也可以写成`${a}`。这种写法可以用于变量名与其他字符连用的情况。

```bash
$ a=foo
$ echo $a_file

$ echo ${a}_file
foo_file
```

### 4.3 删除变量

`unset`命令用来删除一个变量。

```bash
unset NAME
```

这个命令不是很有用。因为不存在的 Bash 变量一律等于空字符串，所以即使`unset`命令删除了变量，还是可以读取这个变量，值为空字符串。

所以，删除一个变量，也可以将这个变量设成空字符串。

```bash
$ foo=''
$ foo=
```

### 4.4 输出变量

用户创建的变量仅可用于当前 Shell，子 Shell 默认读取不到父 Shell 定义的变量。为了把变量传递给子 Shell，需要使用`export`命令。这样输出的变量，对于子 Shell 来说就是环境变量。

`export`命令用来向子 Shell 输出变量。

```bash
NAME=foo
export NAME
```

上面命令输出了变量`NAME`。变量的赋值和输出也可以在一个步骤中完成。

```bash
export NAME=value
```

上面命令执行后，当前 Shell 及随后新建的子 Shell，都可以读取变量`$NAME`。

子 Shell 如果修改继承的变量，不会影响父 Shell。

```bash
# 输出变量 $foo
$ export foo=bar

# 新建子 Shell
$ bash

# 读取 $foo
$ echo $foo
bar

# 修改继承的变量
$ foo=baz

# 退出子 Shell
$ exit

# 读取 $foo
$ echo $foo
bar
```

## 5. 脚本

脚本（script）就是包含一系列命令的一个文本文件。Shell 读取这个文件，依次执行里面的所有命令，就好像这些命令直接输入到命令行一样。所有能够在命令行完成的任务，都能够用脚本完成。

Bash 脚本使用 `#` 声明注释

### 5.1 Shebang行

脚本的第一行通常是指定解释器，即这个脚本必须通过什么解释器执行。这一行以`#!`字符开头，这个字符称为 Shebang，所以这一行就叫做 Shebang 行。

`#!`后面就是脚本解释器的位置，Bash 脚本的解释器一般是`/bin/sh`或`/bin/bash`。

```bash
#!/bin/sh
# 或者
#!/bin/bash
```

### 5.2 执行权限和路径

指定了 Shebang 行的脚本，需要赋予执行权限才可以直接执行。

权限的设定有两种格式，一种是字母格式，（r, w, x）分别表示读、写、执行，（u, g, o）分别表示脚本所有者、同组用户、其它用户，（+, -, =）分别表示增加权限、取消权限和设置唯一权限。所有的权限设定命令都是这些参数的组合，举例

```bash
# 给所有用户执行权限
$ chmod +x script.sh

# 只给脚本拥有者读权限和执行权限
$ chmod u+rx script.sh
```

权限还可以以数字方式设定，我们令 r=4, w=2, x=1，权限的组合就是这些操作权重的组合，权限授予的用户就是数字的位置，举例

```bash
# 给脚本所有者全部权限，给同组和其它用户读权限和执行权限
$ chmod 755 script.sh
```

### 5.3 脚本参数

调用脚本的时候，脚本文件名后面可以带有参数。

```
$ script.sh word1 word2 word3
```

脚本文件内部，可以使用特殊变量，引用这些参数。

- `$0`：脚本文件名，即`script.sh`。
- `$1`~`$9`：对应脚本的第一个参数到第九个参数。
- `$#`：参数的总数。
- `$@`：全部的参数，参数之间使用空格分隔。
- `$*`：全部的参数，参数之间使用变量`$IFS`值的第一个字符分隔，默认为空格，但是可以自定义。

如果脚本的参数多于9个，那么第10个参数可以用`${10}`的形式引用，以此类推。

用户可以输入任意数量的参数，利用`for`循环，可以读取每一个参数。

```bash
#!/bin/bash

for i in "$@"; do
  echo $i
done
```

注1，如果命令是`command -o foo bar`，那么`-o`是`$1`，`foo`是`$2`，`bar`是`$3`。

注2，如果命令是`./script.sh "a b"`，即多个参数放在双引号里面，视为一个参数。

### 5.4 命令执行

`exit`命令用于终止当前脚本的执行，并向 Shell 返回一个退出值。

```bash
$ exit
```

上面命令中止当前脚本，将最后一条命令的退出状态，作为整个脚本的退出状态。

`exit`命令后面可以跟参数，该参数就是退出状态。

```bash
# 退出值为0（成功）
$ exit 0

# 退出值为1（失败）
$ exit 1
```

可以使用 `$?` 判断上一条命令的执行结果控制脚本的执行

```bash
cd $some_directory
if [ "$?" = "0" ]; then
  rm *
else
  echo "无法切换目录！" 1>&2
  exit 1
fi
```

使用 `if` 命令可以直接判断

```bash
if cd $some_directory; then
  rm *
else
  echo "Could not change directory! Aborting." 1>&2
  exit 1
fi
```

最简便的方法是使用逻辑连接符

```bash
# 第一步执行成功，才会执行第二步
cd $some_directory && rm *

# 第一步执行失败，才会执行第二步
cd $some_directory || exit 1
```

`source`命令用于执行一个脚本，通常用于重新加载一个配置文件。

```bash
$ source .bashrc
```

`source`命令最大的特点是在当前 Shell 执行脚本，不像直接执行脚本时，会新建一个子 Shell。所以，`source`命令执行脚本时，不需要`export`变量。

```bash
#!/bin/bash
# test.sh
echo $foo
```

`source`有一个简写形式，可以使用一个点（`.`）来表示。

```bash
$ . script.sh
```

注意，要区分 `. script.sh` 和 `./script.sh`

### 5.5 别名

`alias`命令用来为一个命令指定别名，这样更便于记忆。下面是`alias`的格式。

```bash
alias NAME=DEFINITION
```

上面命令中，`NAME`是别名的名称，`DEFINITION`是别名对应的原始命令。注意，等号两侧不能有空格，否则会报错。

一个常见的例子是为`grep`命令起一个`search`的别名。

```bash
alias search=grep
```

`alias`也可以用来为长命令指定一个更短的别名。下面是通过别名定义一个`today`的命令。

```bash
$ alias today='date +"%A, %B %-d, %Y"'
$ today
星期一, 一月 6, 2020
```

有时为了防止误删除文件，可以指定`rm`命令的别名。

```bash
$ alias rm='rm -i'
```

上面命令指定`rm`命令是`rm -i`，每次删除文件之前，都会让用户确认。

## 6. 用户输入

有时，脚本需要在执行过程中，由用户提供一部分数据，这时可以使用`read`命令。它将用户的输入存入一个变量，方便后面的代码使用。用户按下回车键，就表示输入结束。

`read`命令的格式如下。

```bash
read [-options] [variable...]
```

上面语法中，`options`是参数选项，`variable`是用来保存输入数值的一个或多个变量名。如果没有提供变量名，环境变量`REPLY`会包含用户输入的一整行数据。

下面是一个例子`demo.sh`。

```bash
#!/bin/bash

echo -n "输入一些文本 > "
read text
echo "你的输入：$text"
```

上面例子中，先显示一行提示文本，然后会等待用户输入文本。用户输入的文本，存入变量`text`，在下一行显示出来。

```bash
$ bash demo.sh
输入一些文本 > 你好，世界
你的输入：你好，世界
```

`read`可以接受用户输入的多个值。

```bash
#!/bin/bash
echo Please, enter your firstname and lastname
read FN LN
echo "Hi! $LN, $FN !"
```

上面例子中，`read`根据用户的输入，同时为两个变量赋值。

如果用户的输入项少于`read`命令给出的变量数目，那么额外的变量值为空。如果用户的输入项多于定义的变量，那么多余的输入项会包含到最后一个变量中。

如果`read`命令之后没有定义变量名，那么环境变量`REPLY`会包含所有的输入。

```bash
#!/bin/bash
# read-single: read multiple values into default variable
echo -n "Enter one or more values > "
read
echo "REPLY = '$REPLY'"
```

上面脚本的运行结果如下。

```bash
$ read-single
Enter one or more values > a b c d
REPLY = 'a b c d'
```