[toc]

## 6. 添加用户

### 6.1 /ETC/PASSWD文件

`/etc/passwd`文件列出系统识别的用户。系统在登录（login）时查看该文件决定用户的UID和主目录等信息。文件中的每一行表示一个用户，7个字段用冒号分隔：

- 登陆名
- 加密的密码或密码占位符
- UID (user ID) number
- Default GID (group ID) number
- “GECOS” information: full name, office, extension, home phone
- Home directory
- Login shell

例如：

    root:lga5FjuGpZ2so:0:0:The System,,x6096,:/:/bin/sh
    jl:x:100:0:Jim Lane,ECT8-3,,:/staff/jl:/bin/sh
    dotty:$1$Ce8QpAQI$L.DvJEWiHlWetKTMLXFZO/:101:20::/home/dotty:/bin/tcsh

现在已经不流行加密密码直接放在上面文件中。很容易被暴力破解。所有的UNIX及LINUX都支持将加密密码放在一个单独的文件中。这被称作shadow password机制，是多数系统的默认选择。后面会详细讨论。

`/etc/passwd`的内容常在系统与数据库（如NIS或LDAP）共享。

**登录名**

登录名（或称用户名）必须唯一，不能超过32个字符。可以包含除冒号和换行之外的任何字符。若使用NIS，登录名限制8个字符。一些老版本的UNIX限制使用字母数字，及最多8个字符。

登录名是大小写敏感的；RFC822 calls for case to be ignored in email addresses. 登录名混合大小写目前没发现什么问题，但传统上用小写字母。

**加密密码**

现在多数系统将加密密码放在**/etc/shadow**，而不是**/etc/passwd**。不过本节的讨论并不依赖于密码的存放位置。

密码必须加密形式。必须通过`passwd`命令设置密码。

若你手工编辑**/etc/passwd**创建新的账户，将加密密码字段设为*或x。星号表示此账户设置密码后才能使用。不要将此字段留空——这将造成安全漏洞，因为不需要密码就能访问该账户。

多数Linux分发默认使用MD5加密。不管密码是什么，加密后的密码长度很定（MD5是34个字符）。密码常进过盐值加密。

**UID (user ID) number**

UIDs是无符号32位整数。由于兼容性问题，推荐限制最大的UID未32767（最大的16位整数）。

root的UID一般为0。多数系统还定义了伪用户如**bin**、**daemon**等。要给非人用户预留，我们推荐给真实用户的UID从500开始。

不要将多个的账户的UID设为0。

尽量不要重用UID（在某个用户被移除后）。

UID应在整个组织唯一。

**默认GID**

与UID一样，一个组ID是一个32位整数。GID 0留给root组。GID 1 is the group “bin” and GID 2 is the group “daemon”.

组定义在**/etc/group**。**/etc/passwd**的GID字段提供登录时默认（或有效的）GID。The default GID is not treated specially when access is determined; 只与创建新文件和目录有关。New files are normally owned by the user’s **effective** group. However, in directories on which the **setgid** bit (02000) has been set and on filesystems mounted with the **grpid** option, new files default to the group of their parent directory.

**GECOS字段**

GECOS字段记录用户的个人信息。它的语法是预定义好的。The GECOS field originally held the login information needed to transfer batch jobs from UNIX systems at Bell Labs to a mainframe running GECOS (the General Electric Comprehensive Operating System); these days, only the name remains. A few programs will expand an ‘&’ in the GECOS field to the user’s login name, which saves a bit of typing. Both finger and sendmail perform this expansion, but many programs do not. It’s best not to rely on this feature.

Although you can use any formatting conventions you like, finger interprets commaseparated GECOS entries in the following order:

• Full name (often the only field used)
• Office number and building
• Office telephone extension
• Home phone number

The `chfn` command lets users change their own GECOS information. `chfn` is useful for keeping things like phone numbers up to date, but it can be misused: a user can change the information to be either obscene or incorrect. Most college campuses disable chfn. GECOS information is the perfect candidate for LDAPification.

**主目录**

Users’ shells are cd’ed to their home directories when they log in. If a user’s home directory is missing at login time, the system prints a message such as “no home directory.” If `DEFAULT_HOME` is set to no in `/etc/login.defs`, the login will not be allowed to continue; otherwise, the user will be placed in the root directory.

Be aware that if home directories are mounted over a network filesystem, they may be unavailable in the event of server or network problems.

**登录shell**

The login shell is normally a command interpreter such as the Bourne shell or the C shell (/bin/sh or /bin/csh), but it can be any program. **bash** is the default and is used if `/etc/passwd` does not specify a login shell. Linux中**sh**和**csh**只是链接到**bash**(the GNU “Bourne again” shell)和**tcsh** (a superset of the C shell)。Many distributions also provide a public-domain version of the Korn shell, **ksh**.

Users can change their shells with the **chsh** command. The file **/etc/shells** contains a list of “valid” shells that **chsh** will permit users to select; SUSE enforces this list, but Red Hat just warns you if the selected shell is not on the list. If you add entries to the shells file, be sure to use absolute paths since chsh and other programs expect them.

### 6.2 /ETC/SHADOW文件
