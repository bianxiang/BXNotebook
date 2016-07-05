[toc]

## 15. Sockets

The Berkeley versions of UNIX introduced a new communication tool, the socket interface, which is an extension of the concept of a pipe, covered in Chapter 13. Socket interfaces are available on Linux. You can use sockets in much the same way as pipes, but they include communication across a network of computers. 可以使不同机器的两个进程之间使用socket，也可以在同一个机器上的两个进程之间使用。

Also, the sockets interface has been made available for Windows via a publicly available specification called Windows Sockets, or WinSock. Windows socket services are provided by a Winsock.dll system file. So, Windows programs can communicate across a network to Linux and UNIX com- puters and vice versa, thus implementing client/server systems. Although the programming interface for WinSock isn't quite the same as UNIX sockets, it still has sockets as its basis.

### 15.1 What Is a Socket?

A socket is a communication mechanism that allows client/server systems to be developed either locally, on a single machine, or across networks. Linux functions such as printing, connecting to databases, and serving web pages as well as network utilities such as ftp for file transfer usually use sockets to communicate.

Sockets are created and used differently from pipes because they make a clear distinction between client and server. 允许多个客户端连接到同一个服务器。

### 15.2 Socket连接

First, a server application creates a socket, which like a file descriptor is a resource assigned to the server process and that process alone. The server creates it using the system call `socket`, and it can't be shared with other processes.

Next, the server process gives the socket a name. Local sockets are given a filename in the Linux file system, often to be found in `/tmp` or `/usr/tmp`. For network sockets, the filename will be a service identifier (port number/access point) relevant to the particular network to which the clients can connect. This identifier allows Linux to route incoming connections specifying a particular port number to the correct server process. For example, a web server typically creates a socket on port 80, an identifier reserved for the purpose. Web browsers know to use port 80 for their HTTP connections to web sites the user wants to read. A socket is named using the system call `bind`. The server process then waits for a client to connect to the named socket. The system call, `listen`, creates a queue for incoming connections. The server can accept them using the system call `accept`.

When the server calls `accept`, a new socket is created that is distinct from the named socket. This new socket is used solely for communication with this particular client. The named socket remains for further connections from other clients. If the server is written appropriately, it can take advantage of multiple connections. A web server will do this so that it can serve pages to many clients at once. For a simple server, further clients wait on the listen queue until the server is ready again.
The client side of a socket-based system is more straightforward. The client creates an unnamed socket by calling `socket`. It then calls `connect` to establish a connection with the server by using the server**'s** named socket as an address.

Once established, sockets can be used like low-level file descriptors, providing two-way data communications.

下面的本地客户端的例子。client1.c. It creates an **unnamed** socket and connects it to a server socket called `server_socket`. We cover the details of the socket system call a little later, after we've discussed some addressing issues.

    #include <sys/types.h>
    #include <sys/socket.h>
    #include <stdio.h>
    #include <sys/un.h>
    #include <unistd.h>
    #include <stdlib.h>
    int main() {
        int sockfd;
        int len;
        struct sockaddr_un address;
        int result;
        char ch = 'A';
        sockfd = socket(AF_UNIX, SOCK_STREAM, 0); // Create a socket for the client
        // Name the socket as agreed with the server:
        address.sun_family = AF_UNIX;
        strcpy(address.sun_path, "server_socket");
        len = sizeof(address);
        // Connect your socket to the server's socket:
        result = connect(sockfd, (struct sockaddr *)&address, len);
        if(result == -1) {
        	perror("oops: client1");
            exit(1);
        }
        // You can now read and write via sockfd:
		write(sockfd, &ch, 1);
		read(sockfd, &ch, 1);
		printf("char from server = %c\n", ch);
        close(sockfd);
    	exit(0);
	}

This program fails when you run it because you haven't yet created the server-side named socket. (The exact error message may differ from system to system.)

    $ ./client1
    oops: client1: No such file or directory $

下面是服务器端。server1.c。It creates the server socket, binds it to a name, creates a listen queue, and accepts connections.

    #include <sys/types.h>
    #include <sys/socket.h>
    #include <stdio.h>
    #include <sys/un.h>
    #include <unistd.h>
    #include <stdlib.h>
    int main() {
        int server_sockfd, client_sockfd;
        int server_len, client_len;
        struct sockaddr_un server_address;
        struct sockaddr_un client_address;
        // Remove any old sockets and create an unnamed socket for the server:
        unlink("server_socket");
        server_sockfd = socket(AF_UNIX, SOCK_STREAM, 0);
        // Name the socket:
        server_address.sun_family = AF_UNIX;
        strcpy(server_address.sun_path, "server_socket");
        server_len = sizeof(server_address);
        bind(server_sockfd, (struct sockaddr *)&server_address, server_len);
        // Create a connection queue and wait for clients:
        listen(server_sockfd, 5);
        while(1) {
        	char ch;
        	printf("server waiting\n");
        	// Accept a connection:
        	client_len = sizeof(client_address);
            client_sockfd = accept(server_sockfd, (struct sockaddr *)&client_address, &client_len);
        	// Read and write to client on client_sockfd:
        	read(client_sockfd, &ch, 1);
            ch++;
        	write(client_sockfd, &ch, 1);
            close(client_sockfd);
        }
    }

The server program in this example can serve only one client at a time. It just reads a character from the client, increments it, and writes it back. In more sophisticated systems, where the server has to perform more work on the client's behalf, this wouldn't be acceptable, because other clients would be unable to connect until the server had finished. You'll see a couple of ways to allow multiple connections later.

When you run the server program, it creates a socket and waits for connections. If you start it in the background so that it runs independently, you can then start clients in the foreground.

    $ ./server1 &
    [1] 1094
    $ server waiting

As it waits for connections, the server prints a message. In the preceding example, the server waits for a file system socket and you can see it with the normal `ls` command.

Remember that it's good practice to remove a socket when you've finished with it, even if the program terminates abnormally via a signal. This keeps the file system from getting cluttered with unused files.

	$ ls -lF server_socket
	srwxr-xr-x 1 neil users 0 2007-06-23 11:41 server_socket=

The device type is "socket," shown by the s at the front of the permissions and the = at the end of the name. The socket has been created just as an ordinary file would be, with permissions modified by the current umask. If you use the `ps` command, you can see the server running in the background. It's shown sleeping (STAT is S) and is therefore not consuming CPU resources.

    $ ps lx
    F UID PID PPID PRI NI VSZ RSS WCHAN STAT TTY TIME COMMAND
    0 1000 23385 10689 17 0 1424 312 361800 S pts/1 0:00 ./server1

Now, when you run the client program, you are successful in connecting to the server. Because the server socket exists, you can connect to it and communicate with the server.

    $ ./client1
    server waiting
    char from server = B

The output from the server and the client get mixed on the terminal, but you can see that the server has received a character from the client, incremented it, and returned it. The server then continues and waits for the next client.

#### 15.2.1 Socket特性

To fully understand the system calls used in this example, you need to learn a little about UNIX networking.

Sockets are characterized by three attributes: domain, type, and protocol. They also have an address used as their name. The formats of the addresses vary depending on the domain, also known as the protocol family. Each protocol family can use one or more address families to define the address format.

##### 15.2.1.1 Socket Domains

Domains指定socket通讯使用的网络介质。最常见的是`AF_INET`，用于局域网和Internet。The underlying protocol, Internet Protocol (IP), which only has one address family, imposes a particular way of specifying computers on a network. This is called the IP address.

IPv6 uses a different socket domain, `AF_INET6`, and a different address format. It is expected to eventually replace IP, but this process will take many years. Although there are implementations of IPv6 for Linux, it is beyond the scope of this book.

Although names almost always refer to networked machines on the Internet, these are translated into lower-level IP addresses. An example IP address is 192.168.1.99.

An IP port is identified by the combination of IP address and port number. The sockets are communication end points that must be bound to ports before communication is possible.

Servers wait for connections on particular ports. Well-known services have allocated port numbers that are used by all Linux and UNIX machines. Examples are ftp (21), and httpd (80). Usually, port numbers less than 1024 are reserved for system services and may only be served by processes with superuser privileges. X/Open defines a constant in `netdb.h`, `IPPORT_RESERVED`, to stand for the highest reserved port number.

The domain in the first example is the **UNIX file system domain**, `AF_UNIX`, which can be used by sockets based on a single computer. When this is so, the underlying protocol is file input/output and the addresses are filenames. The address that you used for the server socket was `server_socket`, which you saw appear in the current directory when you ran the server application.

Other domains that might be used include `AF_ISO` for networks based on ISO standard protocols and `AF_XNS` for the Xerox Network System. We won't cover these here.

##### 15.2.1.2 Socket Types

A socket domain may have a number of different ways of communicating, each of which might have different characteristics. This isn't an issue with AF_UNIX domain sockets, which provide a reliable two-way communication path. In networked domains, however, you need to be aware of the characteristics of the underlying network and how different communication mechanisms are affected by them.

Internet protocols provide two communication mechanisms with distinct levels of service: streams and datagrams.

###### Stream Sockets

Stream sockets provide a connection that is a sequenced and reliable two-way byte stream. Thus, data sent is guaranteed not to be lost, duplicated, or reordered without an indication that an error has occurred. Large messages are fragmented, transmitted, and reassembled. This is similar to a file stream, which also accepts large amounts of data and splits it up for writing to the low-level disk in smaller blocks.

Stream sockets, specified by the type `SOCK_STREAM`, are implemented in the `AF_INET` domain by TCP/IP connections. They are also the usual type in the `AF_UNIX` domain. We concentrate on `SOCK_STREAM` sockets in this chapter because they are more commonly used in programming network applications.

###### Datagram Sockets

In contrast, a datagram socket, specified by the type `SOCK_DGRAM`, doesn't establish and maintain a connection. There is also a limit on the **size** of a datagram that can be sent. It's transmitted as a single network message that may get lost, duplicated, or arrive out of sequence — ahead of datagrams sent after it.

Datagram sockets are implemented in the `AF_INET` domain by UDP/IP connections and provide an unsequenced, unreliable service. (UDP stands for User Datagram Protocol.) However, they are relatively inexpensive in terms of resources, because network connections need not be maintained. They're fast because there is no associated connection setup time.

For now, we leave the topic of datagrams; see the "Datagrams" section near the end of this chapter for more information.

##### 15.2.1.3 Socket Protocols

Where the underlying transport mechanism allows for more than one protocol to provide the requested socket type, you can select a specific protocol for a socket. In this chapter, we concentrate on UNIX network and file system sockets, which don't require you to choose a protocol other than the default.

#### 15.2.2 创建一个Socket

The `socket` system call creates a socket and returns a descriptor that can be used for accessing the socket.

	#include <sys/types.h>
    #include <sys/socket.h>
    int socket(int domain, int type, int protocol);

Domains include those in the following table（部分）:

- `AF_UNIX`：UNIX internal (file system sockets)
- `AF_INET`：ARPA Internet protocols (UNIX network sockets)
- `AF_ISO`：ISO standard protocols

The socket parameter `type` specifies the communication characteristics to be used for the new socket. Possible values include `SOCK_STREAM` and `SOCK_DGRAM`.

The protocol used for communication is usually determined by the socket type and domain. There is normally no choice. The `protocol` parameter is used where there is a choice. `0` selects the default protocol, which is used in all the examples in this chapter.

The `socket` system call returns a descriptor that is in many ways similar to a low-level file descriptor. When the socket has been connected to another end-point socket, you can use the `read` and `write` system calls with the descriptor to send and receive data on the socket. The `close` system call is used to end a socket connection.

#### 15.2.3 Socket Addresses

Each socket domain requires its own address format. For an `AF_UNIX` socket, the address is described by a structure, `sockaddr_un`, defined in the `sys/un.h` include file.

    struct sockaddr_un {
	    sa_family_t sun_family; /* AF_UNIX */
        char sun_path[]; /* pathname */
    };

So that addresses of different types may be passed to the socket-handling system calls, each address format is described by a similar structure that begins with a field (in this case, `sun_family`) that specifies the address type (the socket domain). In the `AF_UNIX` domain, the address is specified by a filename in the `sun_path` field of the structure.

On current Linux systems, the type `sa_family_t`, defined by X/Open as being declared in sys/un.h, is taken to be a short. Also, the pathname specified in `sun_path` is limited in size (Linux specifies 108 characters; others may use a manifest constant such as `UNIX_MAX_PATH`). Because address structures may vary in size, many `socket` calls require or provide as an output a **length** to be used for copying the particular address structure.

In the `AF_INET` domain, the address is specified using a structure called `sockaddr_in`, defined in `netinet/in.h`, which contains at least these members:

    struct sockaddr_in {
    	short int sin_family; /* AF_INET */
    	unsigned short sin_port;
        int struct in_addr sin_addr; /* Internet address */
    };

The IP address structure, `in_addr`, is defined as follows:

    struct in_addr {
	    unsigned long int s_addr;
    };

The four bytes of an IP address constitute a single 32-bit value. An `AF_INET` socket is fully described by its domain, IP address, and port number. From an application's point of view, all sockets act like file descriptors and are addressed by a unique integer value.

#### 15.2.4 命名一个Socket

To make a socket (as created by a call to `socket`) available for use by other processes, a server program needs to give the socket a name. Thus, `AF_UNIX` sockets are associated with a file system pathname, as you saw in the server1 example. `AF_INET` sockets are associated with an IP port number.

	#include <sys/socket.h>
	int bind(int socket, const struct sockaddr *address, size_t address_len);

The `bind` system call assigns the address specified in the parameter, address, to the unnamed socket associated with the file descriptor socket. The length of the address structure is passed as `address_len`.
The length and format of the address depend on the address family. A particular address structure pointer will need to be cast to the generic address type `(struct sockaddr *)` in the call to bind.

On successful completion, bind returns `0`. If it fails, it returns `-1` and sets `errno` to one of the following values:

- `EBADF`：The file descriptor is invalid.
- `ENOTSOCK`：The file descriptor doesn't refer to a socket.
- `EINVAL`：The file descriptor refers to an already-named socket.
- `EADDRNOTAVAIL`：The address is unavailable.
- `EADDRINUSE`：The address has a socket bound to it already.

There are some more values for `AF_UNIX` sockets:

- `EACCESS`：Can't create the file system name due to permissions.
- `ENOTDIR`, `ENAMETOOLONG`：Indicates a poor choice of filename.

#### 15.2.5 创建一个Socket队列

To accept incoming connections on a socket, a server program must create a queue to store pending requests. It does this using the `listen` system call.

	#include <sys/socket.h>
	int listen(int socket, int backlog);

A Linux system may limit the maximum number of pending connections that may be held in a queue. Subject to this maximum, `listen` sets the queue length to `backlog`. Incoming connections up to this queue length are held pending on the socket; further connections will be refused and the client's connection will fail. This mechanism is provided by listen to allow incoming connections to be held pending while a server program is busy dealing with a previous client.

The listen function will return `0` on success or `-1` on error. Errors include `EBADF`, `EINVAL`, and `ENOTSOCK`, as for the `bind` system call.

#### 15.2.6 接受连接

Once a server program has created and named a socket, it can wait for connections to be made to the socket by using the `accept` system call.

    #include <sys/socket.h>
    int accept(int socket, struct sockaddr *address, size_t *address_len);

The accept system call returns when a client program attempts to connect to the socket specified by the parameter socket. The client is the first pending connection from that socket's queue. The accept function creates a new socket to communicate with the client and returns its descriptor. The new socket will have the same type as the server listen socket.
The socket must have previously been named by a call to `bind` and had a connection queue allocated by listen. The address of the calling client will be placed in the `sockaddr` structure pointed to by address. A null pointer may be used here if the client address isn't of interest.

The `address_len` parameter specifies the length of the client structure. If the client address is longer than this value, it will be truncated. Before calling `accept`, `address_len` must be set to the expected address length. On return, `address_len` will be set to the actual length of the calling client's address structure.

If there are no connections pending on the socket's queue, accept will **block** (so that the program won't continue) until a client does make connection. You can change this behavior by using the `O_NONBLOCK` flag on the socket file descriptor, using the `fcntl` function in your code like this:

	int flags = fcntl(socket, F_GETFL, 0);
    fcntl(socket, F_SETFL, O_NONBLOCK|flags);

The accept function returns a new socket file descriptor when there is a client connection pending or `-1` on error. Possible errors are similar to those for bind and listen, with the addition of `EWOULDBLOCK`, where `O_NONBLOCK` has been specified and there are no pending connections. The error `EINTR` will occur if the process is interrupted while blocked in accept.

#### 15.2.7 请求连接

Client programs connect to servers by establishing a connection between an unnamed socket and the server listen socket. They do this by calling `connect`.

    #include <sys/socket.h>
    int connect(int socket, const struct sockaddr *address, size_t address_len);

The socket specified by the parameter socket is connected to the server socket specified by the parameter address, which is of length address_len. The `socket` must be a valid file descriptor obtained by a call to `socket`.

If it succeeds, connect returns 0, and -1 is returned on error. Possible errors this time include the following:

- EBADF：An invalid file descriptor was passed in socket.
- EALREADY：A connection is already in progress for this socket.
- ETIMEDOUT：A connection timeout has occurred.
- ECONNREFUSED：The requested connection was refused by the server.

If the connection can't be set up immediately, connect will block for an unspecified timeout period. Once this timeout has expired, the connection will be aborted and connect will fail. However, if the call to connect is interrupted by a signal that is handled, the connect call will fail (with `errno` set to `EINTR`), but the connection attempt won't be aborted — it will be set up asynchronously, and the program will have to check later to see if the connection was successful.

As with `accept`, the blocking nature of connect can be altered by setting the `O_NONBLOCK` flag on the file descriptor. In this case, if the connection can't be made immediately, connect will fail with `errno` set to `EINPROGRESS` and the connection will be made asynchronously.
Though asynchronous connections can be tricky to handle, you can use a call to `select` on the socket file descriptor to check that the socket is ready for writing. We cover `select` later in this chapter.

#### 15.2.8 关闭Socket

You can terminate a socket connection at the server and client by calling `close`, just as you would for low-level file descriptors. You should always close the socket at both ends. For the server, you should do this when read returns zero. Note that the close call may **block** if the socket has untransmitted data, is of a connection-oriented type, and has the `SOCK_LINGER` option set. You learn about setting socket options later in this chapter.

#### 15.2.9 Socket通讯

Now that we have covered the basic system calls associated with sockets, let's take a closer look at the example programs. You'll try to convert them to use a network socket rather than a file system socket. The file system socket has the disadvantage that, unless the author uses an absolute pathname, it's created in the server program's current directory. To make it more generally useful, you need to create it in a globally accessible directory (such as `/tmp`) that is agreed between the server and its clients. For network sockets, you need only choose an unused port number.

For the example, select port number 9734. This is an arbitrary choice that avoids the standard services (you can't use port numbers below 1024 because they are reserved for system use). Other port numbers are often listed, with the services provided on them, in the system file /etc/services. When you're writing socket-based applications, always choose a port number not listed in this configuration file.

You'll run your client and server across a local network, but network sockets are not only useful on a local area network; any machine with an Internet connection (even a modem dial-up) can use network sockets to communicate with others. You can even use a network-based program on a stand-alone UNIX computer because a UNIX computer is usually configured to use a **loopback** network that contains only itself. For illustration purposes, this example uses this loopback network, which can also be useful for debugging network applications because it eliminates any external network problems.

The loopback network consists of a single computer, conventionally called **localhost**, with a standard IP address of **127.0.0.1**. This is the local machine. You'll find its address listed in the network hosts file, **/etc/hosts**, with the names and addresses of other hosts on shared networks.

Here's a modified client program, client2.c, to connect to a network socket via the loopback network. It contains a subtle bug concerned with hardware dependency, but we'll discuss that later in this chapter.

    #include <sys/types.h>
    #include <sys/socket.h>
    #include <stdio.h>
    #include <netinet/in.h>
    #include <arpa/inet.h>
    #include <unistd.h>
    #include <stdlib.h>
    int main() {
        int sockfd;
        int len;
        struct sockaddr_in address;
        int result;
        char ch = 'A';
        sockfd = socket(AF_INET, SOCK_STREAM, 0);
        address.sin_family = AF_INET;
        address.sin_addr.s_addr = inet_addr("127.0.0.1");
        address.sin_port = 9734;
        len = sizeof(address);

The rest of this program is the same as client1.c from earlier in this chapter.

The client program used the `sockaddr_in` structure from the include file `netinet/in.h` to specify an AF_INET address.

You also need to modify the server program to wait for connections on your chosen port number. Here's a modified server: server2.c.

    #include <sys/types.h>
    #include <sys/socket.h>
    #include <stdio.h>
    #include <netinet/in.h>
    #include <arpa/inet.h>
    #include <unistd.h>
    #include <stdlib.h>
    int main() {
        int server_sockfd, client_sockfd;
        int server_len, client_len;
        struct sockaddr_in server_address;
        struct sockaddr_in client_address;
        server_sockfd = socket(AF_INET, SOCK_STREAM, 0);
        server_address.sin_family = AF_INET;
        server_address.sin_addr.s_addr = inet_addr("127.0.0.1");
        server_address.sin_port = 9734;
        server_len = sizeof(server_address);
        bind(server_sockfd, (struct sockaddr *)&server_address, server_len);

From here on, the listing follows server1.c exactly.

If you want to allow the server to communicate with remote clients, you must specify **a set of** IP addresses that you are willing to allow. You can use the special value, `INADDR_ANY`, to specify that you'll accept con nections from all of the interfaces your computer may have. If you chose to, you could distinguish between different network interfaces to separate, for example, internal Local Area Network and external Wide Area Network connections. `INADDR_ANY` is a 32-bit integer value that you can use in the `sin_addr.s_addr` field of the address structure. However, you have a problem to resolve first.

#### 15.2.10 主机与网络的字节顺序

When we run these versions of the server and client programs on an Intel processor–based Linux machine, we can see the network connections by using the **netstat** command. It shows the client/server connection waiting to close down. The connection closes down after a small timeout. (Again, the exact output may vary among different versions of Linux.)

    $ ./server2 & ./client2
    [3] 23770
    server waiting
    server waiting
    char from server = B
    $ netstat –A inet
    Active Internet connections (w/o servers)
    Proto Recv-Q Send-Q Local Address Foreign Address (State) User
    tcp 1 0 localhost:1574 localhost:1174 TIME_WAIT root

You can see the port numbers that have been assigned to the connection between the server and the client. The local address shows the server, and the foreign address is the remote client. (Even though it's on the same machine, it's still connected over a network.) To ensure that all sockets are distinct, these client ports are typically different from the server listen socket and unique to the computer.

However, the local address (the server socket) is given as 1574 (or you may see mvel-lm as a service name), but the port chosen in the example is 9734. Why are they different? The answer is that port numbers and addresses are communicated over socket interfaces as binary numbers. Different computers use different byte ordering for integers. For example, an Intel processor stores the 32-bit integer as four consecutive bytes in memory in the order 1-2-3-4, where 1 is the most significant byte. IBM PowerPC processors would store the integer in the byte order 4-3-2-1. If the memory used for integers were simply copied byte-by-byte, the two different computers would not be able to agree on integer values.
To enable computers of different types to agree on values for multibyte integers transmitted over a network, you need to define a network ordering. Client and server programs must convert their internal integer representation to the network ordering before transmission. They do this by using functions defined in **netinet/in.h**. These are

    #include <netinet/in.h>
    unsigned long int htonl(unsigned long int hostlong);
    unsigned short int htons(unsigned short int hostshort);
    unsigned long int ntohl(unsigned long int netlong);
    unsigned short int ntohs(unsigned short int netshort);

These functions convert 16-bit and 32-bit integers between native host format and the standard network ordering. Their names are abbreviations for conversions — for example, "host to network, long" (htonl) and "host to network, short" (htons). For computers where the native ordering is the same as network ordering, these represent null operations.

To ensure correct byte ordering of the 16-bit port number, your server and client need to apply these functions to the port address. The change to server3.c is

	server_address.sin_addr.s_addr = htonl(INADDR_ANY);
    server_address.sin_port = htons(9734);

You don't need to convert the function call, `inet_addr("127.0.0.1")`, because `inet_addr` is defined to produce a result in network order. The change to client3.c is

	address.sin_port = htons(9734);

The server has also been changed to allow connections from any IP address by using `INADDR_ANY`. Now, when you run server3 and client3, you see the **correct** port being used for the local connection.

    $ netstat
    Active Internet connections
    Proto Recv-Q Send-Q Local Address Foreign Address (State) User tcp 1 0 localhost:9734 localhost:1175 TIME_WAIT root

Remember that if you're using a computer that has the same native and network byte ordering, you won't see any difference. It's still important always to use the conversion functions to allow correct operation with clients and servers on computers with a different architecture.

### 15.3 网络信息

So far, your client and server programs have had addresses and port numbers compiled into them. For a more general server and client program, you can use network information functions to determine addresses and ports to use.

If you have permission to do so, you can add your server to the list of known services in **/etc/services**, which assigns a name to port numbers so that clients can use symbolic services rather than numbers.

Similarly, given a computer's name, you can determine the IP address by calling host database functions that resolve addresses for you. They do this by consulting network configuration files, such as **/etc/hosts**, or network information services, such as NIS (Network Information Services, formerly known as Yellow Pages) and DNS (Domain Name Service).
Host database functions are declared in the interface header file netdb.h.

    #include <netdb.h>
    struct hostent *gethostbyaddr(const void *addr, size_t len, int type);
    struct hostent *gethostbyname(const char *name);

The structure returned by these functions must contain at least these members:

    struct hostent {
    	char *h_name; /* name of the host */
    	char **h_aliases; /* list of aliases (nicknames) */
        int h_addrtype; /* address type */
        int h_length; /* length in bytes of the address */
    	char **h_addr_list /* list of address (network order) */
    };


If there is no database entry for the specified host or address, the information functions return a null pointer.

Similarly, information concerning services and associated port numbers is available through some service information functions.

    #include <netdb.h>
    struct servent *getservbyname(const char *name, const char *proto);
    struct servent *getservbyport(int port, const char *proto);

The `proto` parameter specifies the protocol to be used to connect to the service, either "tcp" for SOCK_STREAM TCP connections or "udp" for SOCK_DGRAM UDP datagrams.

The structure servent contains at least these members:

    struct servent {
    	char *s_name; /* name of the service */
        char **s_aliases; /* list of aliases (alternative names) */
        int s_port; /* The IP port number */
    	char *s_proto; /* The service type, usually "tcp" or "udp" */
    };

You can gather host database information about a computer by calling `gethostbyname` and printing the results. Note that the address list needs to be cast to the appropriate address type and converted from network ordering to a printable string using the `inet_ntoa` conversion, which has the following definition:

    #include <arpa/inet.h>
    char *inet_ntoa(struct in_addr in)

The function converts an Internet host address to a string in dotted quad format. It returns -1 on error, but POSIX doesn't define any specific errors. The other new function you use is `gethostname`.

    #include <unistd.h>
    int gethostname(char *name, int namelength);

This function writes the name of the current host into the string given by `name`. The hostname will be null-terminated. The argument namelength indicates the length of the string name, and the returned hostname will be truncated if it's too long to fit. gethostname returns 0 on success and -1 on error, but again no errors are defined in POSIX.

This program, getname.c, gets information about a host computer.

	#include <netinet/in.h>
    #include <arpa/inet.h>
    #include <unistd.h>
    #include <netdb.h>
    #include <stdio.h>
    include <stdlib.h>
	int main(int argc, char *argv[]) {
		char *host, **names, **addrs;
        struct hostent *hostinfo;
		// Set the host to the argument supplied with the getname call,
        // or by default to the user's machine:
        if(argc == 1) {
        	char myname[256];
            gethostname(myname, 255);
            host = myname;
        } else
        	host = argv[1];
		// Call gethostbyname and report an error if no information is found:
        hostinfo = gethostbyname(host);
        if(!hostinfo) {
        ￼	fprintf(stderr, "cannot get info for host: %s\n", host);
        	exit(1);
        }
		// Display the hostname and any aliases that it may have:
		printf("results for host %s:\n", host);
        printf("Name: %s\n", hostinfo -> h_name);
        printf("Aliases:");
		names = hostinfo -> h_aliases;
        while(*names) {
			printf(" %s", *names);
			names++;
        }
		printf("\n");
        // Warn and exit if the host in question isn't an IP host:
		if(hostinfo -> h_addrtype != AF_INET) {
        	fprintf(stderr, "not an IP host!\n");
            exit(1);
		}
		// Otherwise, display the IP address(es):
		addrs = hostinfo -> h_addr_list;
        while(*addrs) {
			printf(" %s", inet_ntoa(*(struct in_addr *)*addrs));
            addrs++;
        }
	    printf("\n");
        exit(0);
	}

Alternatively, you could use the function `gethostbyaddr` to determine which host has a given IP address. You might use this in a server to find out where the client is calling from.

    $ ./getname tilde
    results for host tilde:
    Name: tilde.localnet
    Aliases: tilde
    192.168.1.1 158.152.x.x

Most UNIX and some Linux systems make their system time and date available as a standard service called `daytime`. Clients may connect to this service to discover the server’s idea of the current time and date. Here’s a client program, getdate.c, that does just that.

    #include <sys/socket.h>
    #include <netinet/in.h>
    #include <netdb.h>
    #include <stdio.h>
    #include <unistd.h>
    #include <stdlib.h>
    int main(int argc, char *argv[]) {
    	char *host;
    	int sockfd;
    	int len, result;
    	struct sockaddr_in address;
        struct hostent *hostinfo;
        struct servent *servinfo;
        char buffer[128];
    	if(argc == 1)
    		host = “localhost”;
    	else
    		host = argv[1];
        // Find the host address and report an error if none is found:
    	hostinfo = gethostbyname(host);
        if(!hostinfo) {
    		fprintf(stderr, “no host: %s\n”, host);
    		exit(1);
        }
		// Check that the daytime service exists on the host:
        servinfo = getservbyname(“daytime”, “tcp”);
        if(!servinfo) {
        	fprintf(stderr,”no daytime service\n”);
        	exit(1);
        }
    	printf(“daytime port is %d\n”, ntohs(servinfo -> s_port));
    	// Create a socket:
	    sockfd = socket(AF_INET, SOCK_STREAM, 0);
     	// Construct the address for use with connect:
    	address.sin_family = AF_INET;
    	address.sin_port = servinfo -> s_port;
    	address.sin_addr = *(struct in_addr *)*hostinfo -> h_addr_list;
        len = sizeof(address);
    	// Then connect and get the information:
    	result = connect(sockfd, (struct sockaddr *)&address, len);
        if(result == -1) {
    		perror(“oops: getdate”);
    		exit(1);
        }
	    result = read(sockfd, buffer, sizeof(buffer));
        buffer[result] = ‘\0’;
    	printf(“read %d bytes: %s”, result, buffer);
    	close(sockfd);
    	exit(0);
    }

You can use getdate to get the time of day from any known host.

    $ ./getdate localhost
    daytime port is 13
    read 26 bytes: 24 JUN 2007 06:03:03 BST

#### （未）15.3.1 The Internet Daemon (xinetd/inetd)

#### 15.3.2 Socket选项

There are many options that you can use to control the behavior of socket connections — too many to detail here. The `setsockopt` function is used to manipulate options.

	#include <sys/socket.h>
	int setsockopt(int socket, int level, int option_name, const void *option_value, size_t option_len);

You can set options at various levels in the protocol hierarchy. To set options at the socket level, you must set level to `SOL_SOCKET`. To set options at the underlying protocol level (TCP, UDP, and so on), set level to the number of the protocol (from either the header file **netinet/in.h** or as obtained by the function `getprotobyname`).

The `option_name` parameter selects an option to set; the `option_value` parameter is an arbitrary value of length `option_len` bytes passed unchanged to the underlying protocol handler.

Socket level options defined in **sys/socket.h** include the following.

- `SO_DEBUG`：Turn on debugging information.
- `SO_KEEPALIVE`：Keep connections active with periodic transmissions.
- `SO_LINGER`：Complete transmission before close.

`SO_DEBUG` and `SO_KEEPALIVE` take an integer option_value used to turn the option on (1) or off (0). `SO_LINGER` requires a linger structure defined in sys/socket.h to define the state of the option and the linger interval.

`setsockopt` returns 0 if successful, -1 otherwise. The manual pages describe further options and errors.

### 15.4 多客户端

















