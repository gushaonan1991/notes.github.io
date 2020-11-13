---
title: Shell 技巧
date: 2020-10-26
categories:
    - 开发技术
    - Linux
tags: [shell, linux, 技巧]
---

- [概念释疑](#概念释疑)
  - [变量有几种作用域？](#变量有几种作用域)
- [数据结构](#数据结构)
  - [如何定义表结构？](#如何定义表结构)
  - [如何判断当前目录是哪里？](#如何判断当前目录是哪里)
- [输入输出](#输入输出)
  - [如何解析命令行参数？](#如何解析命令行参数)
  - [如何将 stderr 重定向到 stdout？](#如何将-stderr-重定向到-stdout)

# 概念释疑
## 变量有几种作用域？
* Shell 中变量有三种作用域：
  * 全局：即在脚本内任何位置都可见，变量默认具备全局作用域。
  * 局部：仅在函数范围内可见，函数内用 local 修饰的变量具备局部作用域。
  * 环境：当前进程并其子进程都可见，使用 export 显示抛出的变量具备环境作用域。

* 参考：
  * [Shell变量的作用域：Shell全局变量、环境变量和局部变量](http://c.biancheng.net/view/773.html)

# 数据结构
## 如何定义表结构？
* 方法：
  ```shell
  # table 为表结构变量名
  declare -A table
  ```
* 注意：仅对 Bash 4 有效，因为该版 Bash 能直接声明表结构变量，旧版在语法上就不支持，转而功能上实现又太复杂。
* 补充：
  * `${table[@]}`：按添加顺序正序返回表的值集。
  * `${!table[@]}`：按添加顺序倒序返回表的键集。
* 示例：
  ```shell
  # 方法 1
  declare -A table
  table['word1']='hello'
  table['word2']='world'

  # 方法 2
  declare -A table=( ['word1']='hello' ['word2']='world' )

  # 输出
  echo ${table['word1']} # 结果：hello
  echo ${table['word2']} # 结果：world
  ```

* 参考：
  * [How to define hash tables in Bash?](https://stackoverflow.com/questions/1494178/how-to-define-hash-tables-in-bash)

## 如何判断当前目录是哪里？
* 进入脚本时，当前目录位置是调用该脚本的命令或上级脚本所在的位置，注意不是脚本所在的位置。
* 脚本内执行 cd 命令跳转到某一目录时，当前目录设定为该目录。

# 输入输出
## 如何解析命令行参数？
* 示例：
  ```shell
  while getopts ":ab:" arg; do
    case $arg in
      a) echo "a: no args" ;;
      b) echo "b: $OPTARG" ;;
      *) echo "default"    ;;
    esac
  done
  shift "$(( $OPTIND - 1 ))"
  ```
* 说明：
  * 命令行传入参数的格式类似：`-a -b value`，并不支持由双横杠标记的长参数名，如：`--key value`。
  * 将 getopts 调用放在 while 条件语句中，循环解析各个选项参数，arg 变量将保存选项名，选项值（如果有的话）会放到系统自带的 OPTARG 变量中。
  * 在 while 条件语句中声明要解析哪些选项，如果选项为标记型选项，也就是无需传入变量值的选项，那么选项声明之后不能带冒号，反之一定要跟随冒号。
    * 条件语句中没有罗列的选项，即使 case 中有对应处理逻辑也是无法解析，结果会报出错误信息并执行默认处理逻辑。
    * 在参数列表起始处插入引号，表示忽略解析错误的情况。
  * 星号（*）之后的语句是默认处理逻辑，在参数无法解析时程序会执行该逻辑。
  * 循环结束后立即执行 `shift` 命令来左移参数列表，等同于从参数列表中移除已经解析过的选项参数，这样 $1 对应的就会是选项之后的第一个参数内容。
    * OPTIND 代表 getopts 命令即将操作的参数位序，每次循环结束 getopts 命令会自动更新它的值。

* 参考：
  * [An example of how to use getopts in bash](https://stackoverflow.com/questions/16483119/an-example-of-how-to-use-getopts-in-bash)
  * [Using getopts to read the options/arguments passed to a script](http://shouce.jb51.net/shell/internal.html#EX33)
  * [Explain the shell command: shift $(($optind - 1))](https://unix.stackexchange.com/questions/214141/explain-the-shell-command-shift-optind-1)
  * [Bourne Shell Builtins](https://www.gnu.org/software/bash/manual/bash.html#Bourne-Shell-Builtins)

## 如何将 stderr 重定向到 stdout？
* 方法：
  ```shell
  command 2>&1 >/dev/null
  ``` 
* 说明：
  * 两个数字代表文件描述符，其中 1 同 stderr，2 同 stdout，它们作为重定向的目标时前面需要带 &。
  * 后一个重定向会将标准输出的内容再转入到文件 /dev/null 中。

* 参考：
  * [How can I pipe stderr, and not stdout?](https://stackoverflow.com/questions/2342826/how-can-i-pipe-stderr-and-not-stdout)
  * [Redirections](https://www.gnu.org/software/bash/manual/bash.html#Redirections)
