http://wiki.centos.org/HowTos/Network/IPTables

[toc]

## 1. 介绍

CentOS内建的防火墙是iptables，更精确的说是iptables/netfilter。Iptables是用户空间的模块，通过命令行编辑规则。Netfilter是内核模块，做实际的过滤。

Iptables将规则放入预定义的链（INPUT, OUTPUT and FORWARD）。根据规则做出操作（actions）。操作也被称为targets。两个预定义的targets是DROP（丢包）或ACCEPT（接受包）。

**链**

有三个预定义的链：

- `INPUT`：所有目标是本机的包。
- `OUTPUT`：所有从本机出去的包。
- `FORWARD`：All packets neither destined for nor originating from the host computer, but passing through (routed by) the host computer. 如果你把电脑稻作路由器会用到这个链。

多数情况下我们需要处理的是`INPUT`链。

规则添加到某个链的列表中。包会被每个规则依次检查，从上到下，如果匹配则对应操作执行（ACCEPT或DROP）。一旦有规则匹配、操作执行，包不再被链中后续规则检查。如果包经过整个列表到最后都没有匹配，则采用链默认的规则。默认规则可以是 ACCEPT 或 DROP 。

对于INPUT链，策略一般是，默认DROP，添加额外规则 ACCEPT 特定包。对于OUTPUT链，策略一般是，默认ACCEPT，添加额外规则 DROP 特定包。

## 2. 入门

检查是否安装了`iptables`（注意结尾s）：

	$ rpm -q iptables
	iptables-1.4.7-5.1.el6_2.x86_64

检查iptables模块是否已加载：

    # lsmod | grep ip_tables
    ip_tables              29288  1 iptable_filter
    x_tables               29192  6 ip6t_REJECT,ip6_tables,ipt_REJECT,xt_state,xt_tcpudp,ip_tables

利用`-L`查看已加载的规则：

    # iptables -L
    Chain INPUT (policy ACCEPT)
    target     prot opt source               destination
    ACCEPT     all  --  anywhere             anywhere            state RELATED,ESTABLISHED
    ACCEPT     icmp --  anywhere             anywhere
    ACCEPT     all  --  anywhere             anywhere
    ACCEPT     tcp  --  anywhere             anywhere            state NEW tcp dpt:ssh
    REJECT     all  --  anywhere             anywhere            reject-with icmp-host-prohibited

    Chain FORWARD (policy ACCEPT)
    target     prot opt source               destination
    REJECT     all  --  anywhere             anywhere            reject-with icmp-host-prohibited

    Chain OUTPUT (policy ACCEPT)
    target     prot opt source               destination


上面是CentOS 6的默认规则。Note that SSH service is permitted by default.

若`iptables`未运行，可以用下面的方式启动：

	# system-config-securitylevel

## 3. 编写简单的规则集

第一个例子：

    # iptables -P INPUT ACCEPT
    # iptables -F
    # iptables -A INPUT -i lo -j ACCEPT
    # iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
    # iptables -A INPUT -p tcp --dport 22 -j ACCEPT
    # iptables -P INPUT DROP
    # iptables -P FORWARD DROP
    # iptables -P OUTPUT ACCEPT
    # iptables -L -v

输出：

    Chain INPUT (policy DROP 0 packets, 0 bytes)
     pkts bytes target     prot opt in     out     source               destination
        0     0 ACCEPT     all  --  lo     any     anywhere             anywhere
        0     0 ACCEPT     all  --  any    any     anywhere             anywhere            state RELATED,ESTABLISHED
        0     0 ACCEPT     tcp  --  any    any     anywhere             anywhere            tcp dpt:ssh
    Chain FORWARD (policy DROP 0 packets, 0 bytes)
     pkts bytes target     prot opt in     out     source               destination
    Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
     pkts bytes target     prot opt in     out     source               destination

下面解释上面8个命令：

`iptables -P INPUT ACCEPT`
若我们是远程连接的，则我们必须先临时设置INPUT链的默认策略是ACCEPT，否则一旦我们刷出当前规则，我们将不能登录。`-P`用于设置某个链的默认策略。

`iptables -F`
`-F`刷出（flush）所有已存在的规则，于是我们可以从一个干净的状态添加新规则。

`iptables -A INPUT -i lo -j ACCEPT`
下面开始添加规则。`-A`表示追加模式，追加到INPUT链。`-i`（interface）指定接口是lo (localhost, 127.0.0.1)。`-j` (jump)指定目标操作，这里是ACCEPT。即这条规则白哦是所有到localhost的包是可接收的。

`iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT`
这里仍然使用追加模式`-A`。`-m`用于加载一个模块（state）。state模块可以检查报的状态，判断是`NEW`、`ESTABLISHED`还是`RELATED`。`NEW` refers to incoming packets that are new incoming connections that weren't initiated by the host system. `ESTABLISHED` and `RELATED` refers to incoming packets that are part of an already established connection or related to and already established connection.

`iptables -A INPUT -p tcp --dport 22 -j ACCEPT`
这条规则允许在TCP 22端口上建立SSH连接。

`iptables -P INPUT DROP`
现在再把INPUT链的默认策略设为DROP。

`iptables -P FORWARD DROP`
我们不想让电脑作为路由器，因此丢掉所有包。

`iptables -P OUTPUT ACCEPT`
OUTPUT链的默认策略设为ACCEPT。

`iptables -L -v`
Finally, we can list (-L) the rules we've just added to check they've been loaded correctly.

最后我们需要保存规则，以便下次启动时规则会被自动加载。

	# /sbin/service iptables save

This executes the iptables init script, which runs `/sbin/iptables-save` and writes the current iptables configuration to `/etc/sysconfig/iptables`. Upon reboot, the iptables init script reapplies the rules saved in `/etc/sysconfig/iptables` by using the `/sbin/iptables-restore` command.

在Shell中输入所有这些命令很烦，不如创建个脚本：

    #!/bin/bash
    #
    # iptables example configuration script
    #
    # Flush all current rules from iptables
    #
     iptables -F
    #
    # Allow SSH connections on tcp port 22
    # This is essential when working on remote servers via SSH to prevent locking yourself out of the system
    #
     iptables -A INPUT -p tcp --dport 22 -j ACCEPT
    #
    # Set default policies for INPUT, FORWARD and OUTPUT chains
    #
     iptables -P INPUT DROP
     iptables -P FORWARD DROP
     iptables -P OUTPUT ACCEPT
    #
    # Set access for localhost
    #
     iptables -A INPUT -i lo -j ACCEPT
    #
    # Accept packets belonging to established and related connections
    #
     iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
    #
    # Save settings
    #
     /sbin/service iptables save
    #
    # List rules
    #
     iptables -L -v

让脚本可执行，然后运行：

	# chmod +x myfirewall
	# ./myfirewall

## 4. 接口

In our previous example, we saw how we could accept all packets incoming on a particular interface, in this case the localhost interface:

	iptables -A INPUT -i lo -j ACCEPT

Suppose we have 2 separate interfaces, `eth0` which is our internal LAN connection and `ppp0` dialup modem (or maybe eth1 for a nic) which is our external internet connection. We may want to allow all incoming packets on our internal LAN but still filter incoming packets on our external internet connection. We could do this as follows:

    iptables -A INPUT -i lo -j ACCEPT
    iptables -A INPUT -i eth0 -j ACCEPT

But be very careful - if we were to allow all packets for our external internet interface (for example, ppp0 dialup modem):

	iptables -A INPUT -i ppp0 -j ACCEPT

we would have effectively just disabled our firewall!

### 5. IP地址

Opening up a whole interface to incoming packets may not be restrictive enough and you may want more control as to what to allow and what to reject. Lets suppose we have a small network of computers that use the 192.168.0.x private subnet. We can open up our firewall to incoming packets from a single trusted IP address (for example, 192.168.0.4):

	# Accept packets from trusted IP addresses
 	iptables -A INPUT -s 192.168.0.4 -j ACCEPT

Breaking this command down, we first append (-A) a rule to the INPUT chain for the source (-s) IP address 192.168.0.4 to ACCEPT all packets.

Obviously if we want to allow incoming packets from a range of IP addresses, we could simply add a rule for each trusted IP address and that would work fine. But if we have a lot of them, it may be easier to add a range of IP addresses in one go. To do this, we can use a netmask or standard slash notation to specify a range of IP address. For example, if we wanted to open our firewall to all incoming packets from the complete 192.168.0.x (where x=1 to 254) range, we could use either of the following methods:

    # Accept packets from trusted IP addresses
    iptables -A INPUT -s 192.168.0.0/24 -j ACCEPT
    iptables -A INPUT -s 192.168.0.0/255.255.255.0 -j ACCEPT

Finally, as well as filtering against a single IP address, we can also match against the MAC address for the given device. To do this, we need to load a module (the `mac` module) that allows filtering against mac addresses. Earlier we saw another example of using modules to extend the functionality of iptables when we used the state module to match for `ESTABLISHED` and `RELATED` packets. Here we use the `mac` module to check the mac address of the source of the packet in addition to it's IP address:

	# Accept packets from trusted IP addresses
 	iptables -A INPUT -s 192.168.0.4 -m mac --mac-source 00:50:8D:FD:E6:32 -j ACCEPT

First we use `-m mac` to load the mac module and then we use `--mac-source` to specify the mac address of the source IP address (192.168.0.4). You will need to find out the mac address of each ethernet device you wish to filter against. Running `ifconfig` (or `iwconfig` for wireless devices) as root will provide you with the mac address.

This may be useful for preventing spoofing of the source IP address as it will allow any packets that genuinely originate from 192.168.0.4 (having the mac address 00:50:8D:FD:E6:32) but will block any packets that are spoofed to have come from that address. Note, mac address filtering won't work across the internet but it certainly works fine on a LAN.

## 6. 端口与协议

Before we can begin, we need to know what protocol and port number a given service uses. For a simple example, lets look at bittorrent. Bittorrent uses the **tcp** protocol on port **6881**, so we would need to allow all tcp packets on destination port (the port on which they arrive at our machine) 6881:

	# Accept tcp packets on destination port 6881 (bittorrent)
	iptables -A INPUT -p tcp --dport 6881 -j ACCEPT

In order to use matches such as destination or source ports (`--dport` or `--sport`), you must first specify the protocol (tcp, udp, icmp, all).

We can also extend the above to include a port range, for example, allowing all tcp packets on the range 6881 to 6890:

    # Accept tcp packets on destination ports 6881-6890
    iptables -A INPUT -p tcp --dport 6881:6890 -j ACCEPT

## 7. 总结

Now we've seen the basics, we can start combining these rules.

A popular UNIX/Linux service is the secure shell (SSH) service allowing remote logins. By default SSH uses port 22 and again uses the tcp protocol. So if we want to allow remote logins, we would need to allow tcp connections on port 22:

	# Accept tcp packets on destination port 22 (SSH)
	iptables -A INPUT -p tcp --dport 22 -j ACCEPT

This will open up port 22 (SSH) to all incoming tcp connections which poses a potential security threat as hackers could try brute force cracking on accounts with weak passwords. However, if we know the IP addresses of trusted remote machines that will be used to log on using SSH, we can limit access to only these source IP addresses. For example, if we just wanted to open up SSH access on our private lan (192.168.0.x), we can limit access to just this source IP address range:

	# Accept tcp packets on destination port 22 (SSH) from private LAN
	iptables -A INPUT -p tcp -s 192.168.0.0/24 --dport 22 -j ACCEPT

Using source IP filtering allows us to securely open up SSH access on port 22 to only trusted IP addresses. For example, we could use this method to allow remote logins between work and home machines. To all other IP addresses, the port (and service) would appear closed as if the service were disabled so hackers using port scanning methods are likely to pass us by.








