[toc]

## 18. FTP

FTP使用Very Secure FTP Daemon (**vsftpd** package)。

### 18.1 理解FTP

So FTP is not good for sharing files privately (use SSH commands such as **sftp**, **scp**, or **rsync** if you need private, encrypted file transfers).

用户登录FTP，他们的用户名和密码按标准Linux用户名密码做验证。此外，还有一个特殊的，不需要身份验证的账户，称为*anonymous*。

身份验证阶段完成后（在TCP端口21），客户端和服务器之间建立第二个连接。FTP支持积极和被动两种连接类型。使用积极的FTP连接，服务器从自己的20端口发送数据到客户端1023以上的一个端口，哪个端口服务器决定。With passive FTP, it is the client that requests the passive connection and requests a random port from the server.

多数浏览器支持被动FTP模式，因此如果客户端有防火墙，不会阻塞主动模式下FT服务器的数据端口。Supporting passive mode requires some extra work on the server’s firewall to allow random connections to ports above 1023 on the server. The section “Opening up your firewall for FTP” later in this chapter describes what you need to do to your Linux firewall to make both passive and active FTP connections work.

客户端与服务器建立连接后，客户端的当前目录取决于用户。对于匿名用户，`/var/ftp`目录会变成用户的主目录。匿名用户不能离开`/var/ftp`目录。对于常规用户，如*joe*，登录后其主目录`/home/joe`会成为它的当前目录。但joe可以切换到文件系统的其他部分，只要它有相应的权限。

Command-oriented FTP clients (such as `lftp` and `ftp` commands) go into an interactive mode after connecting to the server. From the prompt you see, you can run many commands that are similar to those you would use from the shell. You could use `pwd` to see your current directory, `ls` to list directory contents, and `cd` to change directories. Once you see a file you want, you use the `get` and `put` commands to download files from or upload them to the server, respectively.

### 18.2 安装vsftpd服务器

安装vsftpd：

	# yum install vsftpd

You can view the full contents of the vsftpd package (`rpm -ql vsftpd`). Or you can view just the documentation (`-qd`) or configuration files (`-qc`). 例如查看`vsftpd`包中得文档：

    # rpm -qd vsftpd
    /usr/share/doc/vsftpd-2.3.4/EXAMPLE/INTERNET_SITE/README
    ...
    /usr/share/doc/vsftpd-2.3.4/EXAMPLE/PER_IP_CONFIG/README
    ...
    /usr/share/doc/vsftpd-2.3.4/EXAMPLE/VIRTUAL_HOSTS/README
    /usr/share/doc/vsftpd-2.3.4/EXAMPLE/VIRTUAL_USERS/README
    ...
    /usr/share/doc/vsftpd-2.3.4/FAQ
    ...
    /usr/share/doc/vsftpd-2.3.4/vsftpd.xinetd
    /usr/share/man/man5/vsftpd.conf.5.gz
    /usr/share/man/man8/vsftpd.8.gz

In the `/usr/share/doc/vsftpd-*/EXAMPLE` directory structure, there are sample configuration files included to help you configure vsftpd in ways that are appropriate for an Internet site, multiple IP address site, and virtual hosts. The main `/usr/share/doc/vsftpd-*` directory contains an FAQ (frequently asked questions), installation tips, and version information.

列出配置文件：

    # rpm -qc vsftpd
    /etc/logrotate.d/vsftpd
    /etc/pam.d/vsftpd
    /etc/vsftpd/ftpusers
    /etc/vsftpd/user_list
    /etc/vsftpd/vsftpd.conf

主配置文件是`/etc/vsftpd/vsftpd.conf`。The `/etc/pam.d/vsftpd` file sets how authentication is done to the FTP server. The `/etc/logrotate.d/vsftpd` file configures how log files are rotated over time.

### 18.3 启动vsftpd服务

`vsftpd`服务启动`vsftpd` daemon。`vsftpd` daemon监听的默认端口是21。默认连接建立后，数据通过端口20发送给用户。`vsftpd` daemon读取`vsftpd.conf`决定服务使用那些功能。

Linux用户和`anonymous`用户可以访问FTP服务器。(If SELinux is in Enforcing mode, you need to set a Boolean to allow regular users to log in to the FTP server. See the section “Securing Your FTP Server” for details.)

匿名用户可以下载文件，但不能上传。常规用户可以依据其常规Linux权限，上传或下载文件。Log messages detailing file uploads or downloads are written in the `/var/log/xferlogs` file. Those log messages are stored in a standard xferlog format.

在启动前可以先检查一下服务是否运行。In Fedora, you would do the following:

	# systemctl status vsftpd.service
    vsftpd.service - Vsftpd ftp daemon
        ￼Loaded: loaded (/lib/systemd/system/vsftpd.service; disabled)
        Active: inactive (dead)
        CGroup: name=systemd:/system/vsftpd.service

In Red Hat Enterprise Linux 6, you need two commands to see the same information:

    # service vsftpd status
    vsftpd is stopped
    # chkconfig --list vsftpd
    vsftpd 0:off 1:off 2:off 3:off 4:off 5:off 6:off

在Fedora中启动并启用`vsftpd`：

    # systemctl start vsftpd.service
    # systemctl enable vsftpd.service
    ln -s '/lib/systemd/system/vsftpd.service'
    	'/etc/systemd/system/multi-user.target.wants/vsftpd.service'
    # systemctl status vsftpd.service
    vsftpd.service - Vsftpd ftp daemon
        Loaded: loaded (/lib/systemd/system/vsftpd.service; enabled)
        Active: active (running) since Thu, 10 May 2012 23:22:08 -0400; 11s ago Main PID: 29787 (vsftpd)
        CGroup: name=systemd:/system/vsftpd.service
        	29787 /usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf

In Red Hat Linux, start and turn on (enable) `vsftpd` (then check the status), as follows:

    # service vsftpd start
    Starting vsftpd for vsftpd: [ OK ]
    # chkconfig vsftpd on ; chkconfig --list vsftpd
    vsftpd 0:off 1:off 2:on 3:on 4:on 5:on 6:off

Now, on either system, you could check that the service is running using the netstat command:

    # netstat -tupln | grep vsftpd
    tcp 0 0 0.0.0.0:21 0.0.0.0:* LISTEN 29787/vsftpd

检查服务器是否就绪的最快方法是向`/var/ftp`添加一个文件，并通过本地浏览器访问：

    # echo "Hello From Your New FTP Server" > /var/ftp/hello.txt

然后打开`ftp://localhost/hello.txt`。

### 18.4 服务器安全

#### 18.4.1 为服务器开放防火墙

In Fedora and Red Hat Enterprise Linux, firewall rules are stored in the `/etc/sysconfig/iptables` file and the service is called `iptables` (RHEL) or `iptables.service` (Fedora). Modules are loaded into your firewall from the `/etc/sysconfig/iptables-config` file.

First, you need to allow your system to accept requests on TCP port 21; then, you need to make sure that the connection tracking module is loaded.

You typically want to add the rule somewhere before the final `DROP` or `REJECT` rule. The following output shows partial contents of the `/etc/sysconfig/iptables`.

    *filter
    :INPUT ACCEPT [0:0]
    :FORWARD ACCEPT [0:0]
    :OUTPUT ACCEPT [0:0]
    -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
    -A INPUT -i lo -j ACCEPT
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
    -A INPUT -m state --state NEW -m tcp -p tcp --dport 21 -j ACCEPT
    ...
    -A INPUT -j REJECT --reject-with icmp-host-prohibited COMMIT

It is important to have the `ESTABLISHED, RELATED` line in your iptables firewall rules. 若没有此行，用户虽然能连接到如21端口，但不能继续通信。即用户可以登录，但不能传输数据。

The next thing you have to do is set up the FTP connection tracking module to be loaded each time the firewall starts up. Edit this line at the beginning of the `/etc/sysconfig/iptables-config` file to appear as follows:

	IPTABLES_MODULES="nf_conntrack_ftp"

现在可以重启防火墙了。使用下面命令中的一个重启`iptables`：

    # service iptables restart or
    # systemctl restart iptables.service

现在可以测试远程访问。

#### （未）18.4.2 Allowing FTP access in TCP wrappers

#### （未）18.4.3 Configuring SELinux for your FTP server

#### 18.4.4 相关Linux文件权限

`vsftpd`服务器依赖标准Linux文件权限控制对文件或目录的访问。若要允许匿名用户下载文件，至少需要文件对“其他人”开放读权限（`------r--`）。要允许访问目录，至少要对“其他人”开放写权限（`--------x`）。

对于普通用户，通用规则是如果它能通过Shell访问一个文件，就能通过FTP访问。因此至少它的主目录是可以访问的。

### 18.5 配置FTP服务器

vsftpd多数配置位于`/etc/vsftpd/vsftpd.conf`。`vsftpd.conf`的样例配置位于`/usr/share/doc/vsftpd-*`。

修改配置后记得重启`vsftpd`服务。

#### 配置用户访问

`vsftpd`服务器可以配置匿名用户和本地Linux用户（列在`/etc/passwd`文件中）访问服务器。在`vsftpd.conf`文件中：

	anonymous_enable=YES
    local_enable=YES

As noted earlier, despite the `local_enable` setting, SELinux will actually prevent `vsftpd` users from logging in and transferring data. Either changing SELinux out of Enforcing mode or setting the correct Boolean allows local accounts to log in and transfer data.

若想禁止用户登录shell，只允许它访问ftp，可以创建一个无默认shell的用户（实际shell是`/sbin/nologin`）。例如，`/etc/passwd`中只能访问FTP的用户账户形如：

	bill:x:1000:1000:Bill Jones:/home/chris:/sbin/nologin

不是所有Linux用户都可以访问FTP服务器。The setting `userlist_enable=YES` in `vsftpd.conf` says to deny access to the FTP server to all accounts listed in the `/etc/vsftpd/user_list` file. That list includes administrative users root, bin, daemon, adm, lp, and others. You can add other users to that list to whom you would like to deny access.

If you change `userlist_enable` to NO, the `user_list` file becomes a list of only those users who do have access to the server. In other words, setting `userlist_enable=NO`, removing all usernames from the `user_list` file, and adding the usernames chris, joe, and mary to that file cause the server to allow only those three users to log in to
the server.

No matter how the value of `userlist_enable` is set, the `/etc/vsftpd/ftpusers` file always includes users who are denied access to the server. Like the `userlist_enable` file, the `ftpusers` file includes a list of administrative users. You can add more users to that file if you want them to be denied FTP access.

One way to limit access to users with regular user accounts on your system is to use `chroot` settings. Here are examples of some `chroot` settings:

    chroot_local_user=YES chroot_list_enable=YES
    chroot_list_file=/etc/vsftpd/chroot_list

With the settings just shown uncommented, you could create a list of local users and add them to the `/etc/vsftpd/chroot_list` file. After one of those users logged in, that user would be prevented from going to places in the system that were outside of that user’s home directory structure.

If uploads to your FTP server are allowed, the directories a user tries to upload to must be writeable by that user. However, uploads can be stored under a username other than that of the user who uploaded the file. This is one of the features discussed next, in the “Allowing uploading” section.

#### 允许上传

任何写操作都需要在`vsftpd.conf`中启用`write_enable=YES`；默认是开的，因此如果允许本地用户登录，则用户登录后立即可以上传文件到它们的主目录。但匿名用户默认禁止上传。

要允许匿名用户上传，必须设置下面两行代码的第一行，第二行代码可选但一般也会设置。（这两行代码都在`vsftpd.conf`中，取消掉注释即可）。第一行允许匿名用户上传文件，第二行允许它们创建文件夹。

	anon_upload_enable=YES
    anon_mkdir_write_enable=YES

下面是创建一个匿名用户可以写入的目录。`/var/ftp`目录之下，任何对`ftp`用户、`ftp`组或其他人开放写权限的目录可以被匿名用户写入。例子：

	# mkdir /var/ftp/uploads
    # chown ftp:ftp /var/ftp/uploads
    # chmod 775 /var/ftp/uploads

As long as the firewall is open and SELinux Booleans are set properly, an anonymous user can `cd` to the `uploads` directory. 在服务器端，文件将被`ftp`用户和`ftp`组拥有。对目录设置的`775`权限，允许匿名用户看到上传上传的文件，但不能修改或覆盖它们。

Because anyone who can find the server can write to this directory, some form of security needs to be in place. You want to prevent an anonymous user from seeing files uploaded by other users, taking files, or deleting files uploaded by other anonymous FTP users. One form of security is the `chown` feature of FTP.

By setting the following two values, you can allow anonymous uploads. The result of these settings is that when an anonymous user uploads a file, that file is immediately assigned ownership of a different user. The following is an example of some chown settings you could put in your `vsftpd.conf` file to use with your anonymous upload directory:

	chown_uploads=YES
    chown_username=joe

If an anonymous user were to upload a file after vsftpd was restarted with these settings, the uploaded file would be owned by the user joe and the ftp group. Permissions would be read/write for the owner and nothing for anyone else (`rw-------`).

So far, you have seen configuration options for individual features on your vsftpd server. There are a few sets of vsftp.conf variables that can work together in ways that are appropriate for certain kinds of FTP sites. The next section contains one of these examples, represented by a sample vsftpd.conf configuration file that come with the vsftpd package. That file can be copied from a directory of sample files to the /etc/vsftpd/vsftpd.conf file, to use for an FTP server that is available on the Internet.

#### （未）Setting up vsftpd for the Internet

### 18.6 Using FTP Clients to Connect to Your Server














