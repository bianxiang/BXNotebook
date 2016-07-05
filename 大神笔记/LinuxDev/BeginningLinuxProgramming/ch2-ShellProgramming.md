[toc]

## 2. Shell编程

### 什么是Shell

Linux中标准shell（安装在/bin/sh）称为bash (the GNU Bourne-Again SHell)。本书使用bash，第三版，主要使用POSIX兼容的Shell功能。We assume that the shell has been installed as /bin/sh and that it is the default shell for your login.

检查bash版本：

    $ /bin/bash --version
    GNU bash, version 3.2.9(1)-release (i686-pc-linux-gnu)
    Copyright (C) 2005 Free Software Foundation, Inc.

若bash不是你机器的默认shell，只要执行/bin/bash进入新的bash的Shell。

### 管道与重定向

#### 输出重定向

如

	$ ls -l > lsoutput.txt

But for now all you need to know is that file descriptor 0 is the stan- dard input to a program, file descriptor 1 is the standard output, and file descriptor 2 is the standard error output. You can redirect each of these independently. In fact, you can also redirect other file descriptors, but it’s unusual to want to redirect any other than the standard ones: 0, 1, and 2.

The preceding example redirects the standard output into a file by using the > operator. By default, if the file already exists, then it will be overwritten. If you want to change the default behavior, you can use thecommandset -o noclobber (or set-C),whichsetsthenoclobberoptiontopreventafilefrom beingoverwrittenusingredirection.Youcancancelthisoptionusingset +o noclobber.You’llsee more options for the set command later in the chapter.

To append to the file, use the >> operator. For example,

	$ ps >> lsoutput.txt

To redirect the standard error output, preface the > operator with the number of the file descriptor you wish to redirect. Because the standard error is on file descriptor 2, use the `2> operator`.

例子：

	$ kill -HUP 1234 >killout.txt 2>killerr.txt

If you prefer to capture both sets of output into a single file, you can use the >& operator to combine the two outputs. Therefore,

	$ kill -1 1234 >killouterr.txt 2>&1

will put both the output and error outputs into the same file. Notice the order of the operators. This reads as “redirect standard output to the file killouterr.txt, and then direct standard error to the same place as the standard output.” If you get the order wrong, the redirect won’t work as you expect.

Because you can discover the result of the kill command using the return code (discussed in more detail later in this chapter), you don’t often want to save either standard output or standard error. You can use the Linux universal “bit bucket” of /dev/null to efficiently discard the entire output, like this:

	$ kill -1 1234 >/dev/null 2>&1

#### 重定向输入

例子：

	$ more < killout.txt

#### 管道

You can connect processes using the pipe operator ( | ). In Linux, unlike in MS-DOS, processes connected by pipes can run simultaneously and are automatically rescheduled as data flows between them. As a simple example, you could use the sort command to sort the output from ps.

	$ ps | sort > pssort.out

通过管道连接的进程的数量是不限的：

	$ ps –xo comm | sort | uniq | grep -v sh | more

### Shell作为编程语言

两种方式：直接在命令行写，保存到文件中（脚本）。

#### 直接在命令行

例如，在一堆文件中搜索`POSIX`：

    $ for file in *
    > do
    > if grep -l POSIX $file
    > then
    > more $file
    > fi
    > done
    posix
    This is a file with POSIX in it - treat it well

The shell also performs wildcard expansion (often referred to as globbing). You are almost certainly aware of the use of ‘*‘ as a wildcard to match a string of characters. What you may not know is that you can request single-character wildcards using ?, while [set] allows any of a number of single char- acters to be checked. [^set] negates the set — that is, it includes anything but the set you’ve specified. Brace expansion using {} (available on some shells, including bash) allows you to group arbitrary strings together in a set that the shell will expand. For example,

	$ ls my_{finger,toe}s

Experienced Linux users would probably perform this simple operation in a much more efficient way, perhaps with a command such as

	$ more `grep -l POSIX *`

or the synonymous construction

	$ more $(grep -l POSIX *)

In addition,

	$ grep -l POSIX * | more

#### 创建脚本

创建一个叫first的文件。内容：

    #!/bin/sh
    # first
    # This file looks through all the files in the current
    # directory for the string POSIX, and then prints the names of
    # those files to the standard output.
    for file in *
    do
    	if grep -q POSIX $file
        then
    		echo $file fi
    done
    exit 0

注释以`#`开头。第一行，`#!/bin/sh`，是一个特殊形式的注释，指出执行该脚本的程序。

Since the script is essentially treated as standard input to the shell, it can contain any Linux commands referenced by your `PATH` environment variable.

The exit command ensures that the script returns a sensible exit code (more on this later in the chapter). This is rarely checked when programs are run interactively, but if you want to invoke this script from another script and check whether it succeeded, returning an appropriate exit code is very important. Even if you never intend to allow your script to be invoked from another, you should still exit with a reasonable code. Have faith in the usefulness of your script: Assume it may need to be reused as part of another script someday.

A zero denotes success in shell programming. Since the script as it stands can’t detect any failures, it always returns success.

判断文件是否是脚本可以用`file`命令。例如`file first`或`file /bin/bash`。

不要把当前目录加入到root的PATH。安全漏洞。脚本前显式加`./`是好习惯。额外的好处是确保执行的是当前目前的脚本，而不是PATH上的同名的命令。

### Shell语法

#### 变量

变量在使用前不必先声明。默认所有的变量都被认为是字符串，即使给它赋数值。The shell and some utilities will convert numeric strings to their values in order to operate on them as required.

访问变量内容使用`$`加变量名。赋值时直接使用变量名。

例子：

    $ salutation=Hello
    $ echo $salutation Hello
    $ salutation=”Yes Dear”
    $ echo $salutation
    Yes Dear
    $ salutation=7+5
    $ echo $salutation 7+5

字符串中又空格必须外围加引号。等号左右不要加任何空格。

You can assign user input to a variable by using the `read` command. 输入回车结束。不需要加引号。

    $ read salutation
    Wie geht’s?
    $ echo $salutation
    Wie geht’s?

Normally, parameters in scripts are separated by whitespace characters (e.g., a space, a tab, or a newline character). If you want a parameter to contain one or more whitespace characters, you must quote the parameter.

引号中的变量（如`$foo`）的解析取决于引号是单引号还是双引号。若是双引号，则执行时替换为变量的值。若是单引号，不替换。You can also remove the special meaning of the `$` symbol by prefacing it with a \.

例子：

    #!/bin/sh
    myvar=”Hi there”

    echo $myvar
    echo “$myvar”
    echo ‘$myvar’
    echo \$myvar
    echo Enter some text
    read myvar
    echo ‘$myvar’ now equals $myvar
    exit 0

执行结果：

    $ ./variable
    Hi there
    Hi there
    $myvar
    $myvar
    Enter some text
    Hello World
    $myvar now equals Hello World

#### 环境变量

- `$HOME`：The home directory of the current user
- `$PATH`：A colon-separated list of directories to search for commands
- `$IFS`：An input field separator. This is a list of characters that are used to separate words when the shell is reading input, usually space, tab, and newline characters.
- `$0`：The name of the shell script
- `$#`：传入参数的数量
- `$$`：The process ID of the shell script, often used inside a script for gener ating unique temporary filenames; for example `/tmp/tmpfile_$$`

#### 参数变量

- `$1`, `$2`, ...：给脚本的参数
- `$*`：A list of all the parameters, in a single variable, separated by the first character in the environment variable `IFS`. If IFS is modified, then the way `$*` separates the command line into parameters will change.
- `$@`：A subtle variation on `$*`; it doesn’t use the `IFS` environment variable, so parameters are not run together even if `IFS` is empty.

例子：

    $ IFS=''
    $ set foo bar bam $ echo “$@“
    foo bar bam
    $ echo “$*“ foobarbam
    $ unset IFS
    $ echo “$*“
    foo bar bam

#### 条件

##### The test or [ Command

In practice, most scripts make extensive use of the [ or test command, the shell’s Boolean check. On some systems, the [ and test commands are synonymous, except that when the [ command is used, a trailing ] is also used for readability.

测试命令是否存在：

    if test -f fred.c
    then
    ...
    fi

或写成：

    if [ -f fred.c ]
    then
    ...
    fi

`test`命令的返回码决定条件是否满足。

`[`两边要加括号，因为它是一个命令，如`test`。

If you prefer putting then on the same line as if, you must add a semicolon to separate the test from the `then`:

    if [ -f fred.c ]; then
    ...
    fi

`test`命令可以使用的条件类型有三类：字符串比较，算术比较，文件条件。（还有其他选项，参见`test`命令的文档。）

字符串比较：

- `string1 = string2`：两字符串相等为true
- `string1 != string2`：True if the strings are not equal
- `-n string`：True if the string is not null
- `-z string`：True if the string is null (an empty string)

算术比较：

- `expression1 -eq expression2`：如果表达式相等
- `expression1 -ne expression2`：True if the expressions are not equal
- `expression1 -gt expression2`：True if expression1 is greater than expression2
- `expression1 -ge expression2`：True if expression1 is greater than or equal to expression2
- `expression1 -lt expression2`：True if expression1 is less than expression2
- `expression1 -le expression2`：True if expression1 is less than or equal to expression2
- `! expression`：True if the expression is false, and vice versa

文件条件：

- `-d file`：如果文件是目录
- `-e file`：如果文件存在。Note that historically the `-e` option has not been portable, so `-f` is usually used.
- `-f file`：文件是普通文件
- `-g file`：True if set-group-id is set on file
- `-r file`：True if the file is readable
- `-s file`：True if the file has nonzero size
- `-u file`：True if set-user-id is set on file
- `-w file`：True if the file is writable
- `-x file`：True if the file is executable

例子：


    #!/bin/sh
    if [ -f /bin/bash ]
    then
    	echo “file /bin/bash exists” fi
    if [ -d /bin/bash ]
    then
    	echo “/bin/bash is a directory”
    else
    	echo “/bin/bash is NOT a directory” fi

#### 控制结构

##### if
￼￼￼￼￼￼￼￼￼￼￼￼￼
    if condition
    then
    	statements
    else
	    statements
    fi

##### elif

    if [ $timeofday = “yes” ]
    then
    	echo “Good morning”
    elif [ $timeofday = “no” ]; then
    	echo “Good afternoon”
    else
	    echo “Sorry, $timeofday not recognized. Enter yes or no”
        exit 1
    fi

	exit 0

但上述代码有个问题。执行时，如果直接回车，什么也不输入，会报错：

	[: =: unary operator expected

原因是，如果`timeofday`为空，执行时相当于

	if [ = “yes” ]

解决办法是在变量外加引号：`if [ “$timeofday” = “yes” ]`。于是当变量为空时，相当于：

	if [ “” = “yes” ]

##### for

Use the for construct to loop through a range of values, which can be any set of strings. They could be simply listed in the program or, more commonly, the result of a shell expansion of filenames.

    for variable in values
    do
	    statements
    done

例子，遍历固定的字符串：

    ￼￼￼￼￼￼￼￼#!/bin/sh
    for foo in bar fud 43 do
    	echo $foo
    done
    exit 0

例子，与通配符展开连用。打印当前目录所有以f开头的脚本：

    #!/bin/sh
    for file in $(ls f*.sh); do
    	lpr $file
    done
    exit 0

This illustrates the use of the `$(command)` syntax, which is covered in more detail later (in the section on command execution). Basically, the parameter list for the for command is provided by the output of the command enclosed in the $() sequence.

##### while

    while condition
    do
    	statements
    done

例子：

    ￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼#!/bin/sh
    echo “Enter password”
    read trythis
    while [ “$trythis” != “secret” ]; do
    	echo “Sorry, try again”
    	read trythis
    done
    exit 0

##### until

    until condition
    do
	    statements
    done

As an example of an until loop, you can set up an alarm that is initiated when another user, whose login name you pass on the command line, logs on:

	#!/bin/bash
    until who | grep “$1” > /dev/null
    do
    	sleep 60
    done
    # now ring the bell and announce the expected user.
    echo -e ‘\a’
    echo “**** $1 has just logged in ****“
    exit 0

##### case

    case variable in
    	pattern [ | pattern] ...) statements;;
        pattern [ | pattern] ...) statements;; ...
    esac

Notice that each pattern line is terminated with double semicolons (;;). You can put multiple state- ments between each pattern and the next, so a double semicolon is needed to mark where one statement ends and the next pattern begins.

Be careful with the case construct if you are using wildcards such as ‘*‘ in the pattern. The problem is that the first matching pattern will be taken, even if a later pattern matches more exactly.

例一：

    ￼#!/bin/sh
    echo “Is it morning?
    read timeofday
    case “$timeofday” in
        yes) echo “Good Morning";;
        no ) echo “Good Afternoon";;
        y ) echo “Good Morning";;
        n ) echo “Good Afternoon";;
        * ) echo “Sorry, answer not recognized”;;
    esac exit 0

更好地写法：

    #!/bin/sh
    echo “Is it morning? Please answer yes or no”
    read timeofday
    ￼￼￼￼￼￼￼￼￼￼￼￼case “$timeofday” in
    	yes | y | Yes | YES ) echo “Good Morning";;
        n* | N* ) echo “Good Afternoon";;
    	* ) echo “Sorry, answer not recognized”;;
    esac
    exit 0

例三：

    case “$timeofday” in
    	yes | y | Yes | YES )
    		echo “Good Morning”
    		echo “Up bright and early this morning” ;;
    	[nN]*)
    		echo “Good Afternoon”
    		;;
        *)
	    ￼￼￼￼￼￼￼￼￼￼￼￼￼￼echo “Sorry, answer not recognized”
        echo “Please answer yes or no”
    	exit 1
    	;;
    ￼￼￼￼esac


To make the case matching more powerful, you could use something like this:

	[yY] | [Yy][Ee][Ss] )
￼
##### 列表

AND和OR列表

**AND列表**

	statement1 && statement2 && statement3 && ...

Each statement is executed independently, enabling you to mix many different commands in a single list, as the following script shows. The AND list as a whole succeeds if all commands are executed successfully, but it fails otherwise.

**OR列表**

	statement1 || statement2 || statement3 || ...

例子：

	if [ -f file_one ] || echo “hello” || echo “ there”

##### 语句块

If you want to use multiple statements in a place where only one is allowed, such as in an AND or OR list, you can do so by enclosing them in braces {} to make a statement block. For example, in the application presented later in this chapter, you’ll see the following code:

    get_confirm && {
        grep -v “$cdcatnum” $tracks_file > $temp_file
        ￼cat $temp_file > $tracks_file
        echo
        add_record_tracks
    }

#### 函数


