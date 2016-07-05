[toc]

## 4. Linux环境

### 4.1 程序参数

主函数：

```c
int main(int argc, char *argv[])
```

`argc`是参数个数。`argv`是参数列表。

Remember that a Linux shell normally performs wild card expansion of filename arguments before argc and argv are set, whereas the MS-DOS shell expects programs to accept arguments with wild cards and perform their own wild card expansion.

For example, if we give the shell the following command,

	$ myprog left right ‘and center’

the program myprog will start at main with parameters:

    argc: 4
    argv: {“myprog”, “left”, “right”, “and center”}

注意到第一个参数是程序名。最后一个参数带空格，在引号内。

例子：

```c
    #include <stdio.h>
    #include <stdlib.h>
    int main(int argc, char *argv[]) {
        int arg;
        for(arg = 0; arg < argc; arg++) {
        	if(argv[arg][0] == ‘-‘)
        		printf(“option: %s\n”, argv[arg]+1);
            else
        		printf(“argument %d: %s\n”, arg, argv[arg]);
        }
        exit(0);
    }
```

The X/Open specification (available at http://opengroup.org/) defines a standard usage for command-line options (the Utility Syntax Guidelines) as well as a standard programming interface for providing command-line switches in C programs: the `getopt` function.

#### getopt

To help us adhere to these guidelines, Linux provides the `getopt` facility, which supports the use of options with and without values and is simple to use.

```c
    #include <unistd.h>
    int getopt(int argc, char *const argv[], const char *optstring);
    extern char *optarg;
    extern int optind, opterr, optopt;
```

The getopt function takes the `argc` and `argv` parameters as passed to the program’s `main` function and an options specifier string that tells `getopt` what options are defined for the program and whether they have associated values. The optstring is simply a list of characters, each representing a single character option. If a character is followed by a colon, it indicates that the option has an associated value that will be taken as the next argument. The `getopts` command in bash performs a very similar function.

例子：

	getopt(argc, argv, “if:lr”);

The return result for `getopt` is the next option character found in the `argv` array (if there is one). Call `getopt` repeatedly to get each option in turn. It has the following behavior:

- If the option takes a value, that value is pointed to by the external variable `optarg`.
- `getopt` returns `-1` when there are no more options to process. A special argument, `--`, will cause `getopt` to stop scanning for options.
- `getopt` returns `?` if there is an unrecognized option, which it stores in the external variable `optopt`.
- If an option requires a value and no value is given, `getopt` normally returns `?`. By placing a colon as the first character of the options string, `getopt` returns `:` instead of `?` when no value is given.

The external variable, `optind`, is set to the index of the next argument to process. `getopt` uses it to remember how far it’s got. Programs would rarely need to set this variable. When all the option arguments have been processed, `optind` indicates where the remaining arguments can be found at the end of the `argv` array.

Some versions of `getopt` will stop at the first non-option argument, returning `-1` and setting `optind`. Others, such as those provided with Linux, can process options wherever they occur in the program arguments. Note that, in this case, `getopt` effectively **rewrites** the `argv` array so that all of the non-option arguments are presented together, starting at `argv[optind]`. For the GNU version of `getopt`, this behavior is controlled by the `POSIXLY_CORRECT` environment variable. If set, `getopt` will stop at the first non-option argument. Additionally, some `getopt` implementations print error messages for unknown options. Note that the POSIX specification says that if the opterr variable is non-zero, `getopt` will print an error message to stderr.

```c
    #include <stdio.h>
    #include <unistd.h>
    #include <stdlib.h>
    int main(int argc, char *argv[]) {
    	int opt;
    	while((opt = getopt(argc, argv, “:if:lr”)) != -1) {
        	switch(opt) {
    			case ‘i’: case ‘l’: case ‘r’:
    				printf(“option: %c\n”, opt);
    				break;
                case ‘f’:
    				printf(“filename: %s\n”, optarg);
    				break;
                case ‘:’:
    				printf(“option needs a value\n”);
                    break;
                case ‘?’:
    				printf(“unknown option: %c\n”, optopt);
    				break;
            }
        }
        for(; optind < argc; optind++)
    		printf(“argument: %s\n”, argv[optind]);
        exit(0);
    }
```

#### getopt_long





