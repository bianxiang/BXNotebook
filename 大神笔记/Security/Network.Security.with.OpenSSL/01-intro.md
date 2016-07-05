[toc]

## 1 介绍

The primary goals of cryptography, data confidentiality, data integrity, authentication, and non-repudiation (accountability) can be used to thwart numerous types of network-based attacks, 包括窃听、IP欺骗、连接劫持。

安全可靠的使用加密算法比想象中的难很多。算法只是加密协议的一部分，加密协议也很难用对。例如，开发者保护网络连接，有时只加密数据，而不保证数据完整性。使得攻击者可以篡改数据。即使协议设计良好，实现错误也很常见。Most cryptographic protocols have limited applicability, such as secure online voting. However, protocols for securely communicating over an insecure medium have ubiquitous applicability. That's the basic purpose of the SSL protocol and its successor, TLS: 为任意基于TCP的网络连接提供最常见的安全服务，最小化需要的专业知识。

SSL不需要你知道加密算法工作原理，只需要知道算法的一些特性。开发者也不必担心加密协议；SSL不需要开发者立即其内部原理。

For that, we recommend **Applied Cryptography**, by Bruce Schneier (John Wiley & Sons). For those interested in a more technical introduction to cryptography, we recommend Menezes, van Oorschot, and Vanstone's **Handbook of Applied Cryptography** (CRC Press). Similarly, we do not attempt to document the SSL protocol itself, just its application. If you're interested in the protocol details, we recommend Eric Rescorla's **SSL and TLS** (Addison-Wesley).

### 1.1 密码学基础

#### 1.1.1 密码学目标

密码学提供的服务：

- **机密性（Confidentiality）**。在不安全的信道中传送，经过加密，攻击者可以看到加密后的内容，但解不开。古典密码学加密算法是秘密的。但现代密码学，算法是公开的，密钥用于加密和解密过程。唯一需要保密的是密钥。但不是所有密钥都需要保密。
- **完整性（Integrity）**。如保证传输过程中数据不被修改。如checksums就是这样一种机制。注意，加密并不难保证数据完整性。攻击者可以通过直接修改加密后数据修改原来的数据。
- **身份验证（Authentication）**。Cryptography can help establish identity for authentication purposes.
- **抗否认**。Cryptography can enable Bob to prove that a message he received from Alice actually came from Alice. Alice can essentially be held accountable when she sends Bob such a message, as she cannot deny (repudiate) that she sent it. In the real world, you have to assume that an attacker does not compromise particular cryptographic keys. SSL协议本身不支持抗否认，但可以通过数字签名实现。

这些服务可以阻止一系列网络攻击，包括：

- **探听（Snooping）**。被动的偷取。攻击器监听网络通信。
- **篡改（Tampering）**。恶意修改传输中的数据。
- **欺骗（Spoofing）**。攻击者伪造网络数据，是他看起来像来自另一个网络地址。这种攻击用于身份验证基于主机信息的场景，如基于IP地址。
- **劫持（Hijacking）**。Once a legitimate user authenticates, a spoofing attack can be used to "hijack" the connection.
- **捕获并重放**。For example, say that you sell a single share of stock while the price is high. If the network protocol is not properly designed and secured, an attacker could record that transaction, then replay it later when the stock price has dropped, and do so repeatedly until all your stock is gone.

#### 1.1.2 密码学算法

尽量不要直接使用加密算法，使用成熟的、已有的加密协议。

已有协议很多也是不安全的，如 WEP（用于IEEE 802.11）。

本书讨论五类加密算法：对称密钥加密、公钥加密、哈希函数、message authentication codes和数字签名。

##### 1.1.2.1 对称密钥加密

加密解密使用一个密钥。

这种方法一个主要缺陷是，密钥要一直保密。交互密钥可能很难。

一种境界方法是使用密钥交换协议。OpenSSL provides the **Diffie-Hellman** protocol for this purpose, which allows for key agreement without actually divulging the key on the network. However, Diffie-Hellman does not guarantee the identity of the party with whom you are exchanging keys. 需要一些身份验证机制保证你不是与攻击者交换密钥。

Right now, **Triple DES** (usually written 3DES, or sometimes DES3) is the most conservative symmetric cipher available. It is in wide use, but **AES**, the new Advanced Encryption Standard, will eventually replace it as the most widely used cipher. AES is certainly faster than 3DES, but
3DES has been around a lot longer. It is worth mentioning that **RC4** is widely supported by existing clients and servers. It is faster than 3DES, but is difficult to set up properly (don't worry, SSL uses RC4 properly). For purposes of compatibility with existing software in which neither AES nor 3DES are supported, **RC4** is of particular interest. 其他算法我们不推荐使用。

安全性与密码长度有关。密码长度要大于80。AES supports only 128-bit keys and higher, while 3DES has a fixed 112 bits of effective security. Larger keys are probably unnecessary.

##### 1.1.2.2 公钥加密

公钥加密解决了密钥分发问题。多数公钥加密方法都有一堆密钥，一个必须保密（私钥），一个可以随意分发（公钥）。用公钥加密，用私钥解密。

现实中，公钥一般随证书发放；证书由受信任的第三方验证。SSL uses trusted third parties to help address the key distribution problem.

公钥加密的问题是，对于大数据，加密花很久。对称密钥加密解密速度快得多。因此多数系统，包括SSL，都尽量少用公钥加密。公钥加密值用于加密对称加密算法的密钥，然后，后续加密都用对称加密做。

RSA是最常用的公钥加密算法。The **Diffie-Hellman key exchange protocol** is based on public key technology and can be used to achieve the same ends by exchanging a symmetric key, which is used to perform actual data encryption and decryption.

对于密钥长度与安全性，公钥算法与对称加密算法不能直接比较。With public key encryption algorithms, you should use keys of 1,024 bits or more to ensure reasonable security. 512-bit keys are probably too weak. 超过2048位则可能太慢。Recently, there's been some concern that 1,024-bit keys are too weak, but as of this writing, there hasn't been conclusive proof.

1024位应该能经受住短期攻击。如果长时间使用，最好使用2048位。

Requirements for key lengths change if you're using **elliptic curve cryptography (ECC)**, which is a modification of public key cryptography that can provide the same amount of security using faster operations and smaller keys. OpenSSL目前不支持ECC, and there may be some lingering patent issues for those who wish to use it. For developers interested in this topic, we recommend the book Implementing Elliptic Curve Cryptography, by Michael Rosing (Manning).

##### 1.1.2.3 安全（Cryptographic）哈希函数和 Message Authentication Code

Cryptographic hash functions are essentially checksum algorithms with special properties. 传入相同数据的结果总是相同的。几乎不可能两个不同的输入能产生相同的输出。单向函数：拿到输出和算法，并不难倒推出输入。

For general-purpose usage, a minimally secure cryptographic hash algorithm should have a digest twice as large as a minimally secure symmetric key algorithm. **MD5**和**SHA1**是最著名的单向安全哈希函数。MD5的摘要长度只有128位，而SHA1是160位。有些情况下，MD5的长度可能不够。要保证安全，我们推荐只使用产生摘要大于160位的算法。MD5基本上快被攻破了，因此避免使用。

安全哈希算法常用于存储密码。

带密码的哈希，称为Message Authentication Codes (MACs)。MACs常为通用的数据传输提供消息完整性。Indeed, SSL uses MACs for this purpose.

最常用的MAC，及OpenSSL的SSL目前仅支持的，是**HMAC**。

##### 1.1.2.4 数字签名

对于多数应用，MACs不是很有用，因为需要双方约定共享密码。It would be nice to be able to authenticate messages without needing to share a secret. Public key cryptography makes this possible. If Alice signs a message with her secret signing key, then anyone can use her public key to verify that she signed the message. RSA可用于数字签名。Essentially, the public key and private key are interchangeable. If Alice encrypts a message with her private key, anyone can decrypt it. If Alice didn't encrypt the message, using her public key to decrypt the message would result in garbage.

There is also a popular scheme called DSA (the Digital Signature Algorithm), which the SSL protocol and the OpenSSL library both support.

与公钥加密类似，数字签名非常慢。为了加速，算法不会对整个数据签名。Instead, the message is cryptographically hashed, and then the hash of the message is signed. Nonetheless, signature schemes are still expensive. For this reason, MACs are preferable if any sort of secure key exchange has taken place.

数字签名的使用场景之一是证书管理。If Alice is willing to validate Bob's certificate, she can sign it with her private key. Once she's done that, Bob can attach her signature to his certificate. Now, let's say he gives the certificate to Charlie, and Charlie does not know that Bob actually gave him the certificate, but he would believe Alice if she told him the certificate belonged to Bob. In this case, Charlie can validate Alice's signature, thereby demonstrating that the certificate does indeed belong to Bob.

Since digital signatures are a form of public key cryptography, you should be sure to use key lengths of 1,024 bits or higher to ensure security.

### 1.2 SSL概述

SSL是HTTPS背后的协议。SSL实际可以保护TCP之上的任何协议。

一个SSL事务（transaction），开始于客户端与服务器握手，服务器响应它的证书。证书包含公钥、证书所有者信息、过期日期、全限域名（即 www.securesw.com，不能是 securesw.com）等。

在连接过程中，服务器要证明自己的身份，通过用它的私钥解密客户端用服务器公钥签名的挑战。客户端需要受到正确的解密后的数据，然后才能继续。

保证证书的真实性。一种方式是讲证书预先放在客户端。但实际一般通过受信任的第三方机构。这种机构称为 Certification Authority，使用它的私钥对服务器证书进行签名。签名会E币包含进证书，在连接时提供。客户端通过CA的公钥对签名进行检查，若成功，客户端可以确定证书时某家被CA承认的实体拥有的。证书内的信息也是可信的。

少数情况，服务器也会向客户端请求一个证书。在证书校验之前，客户端和服务器先商定采用什么加密算法。证书校验成功后，客户端和服务器商定对称密钥。这些协商都完成后，客户端和服务器开始交换数据。

SSL协议的细节更复杂一些。**Message Authentication Codes** are used extensively to ensure data integrity. Additionally, during certificate validation, a party can go to the Certification Authority for Certificate Revocation Lists (CRLs) to ensure that certificates that appear valid haven't actually been stolen.

### 1.3 SSL的问题

SSL容易被错误使用。

#### 1.3.1 性能

SSL比传统的非安全TCP/IP连接要慢一些。初始的握手需要使用公钥加密算法，是非常慢的。在当前高端PC硬件上，OpenSSL大约每秒能做100个连接。握手成功后，开销就显著小乐，但仍比不安全的TCP/IP连接要大一些。

传输的数据变多了。Data is transmitted in packets, which contain information required by the SSL protocol as well as any padding required by the symmetric cipher that is in use. 当然，仍有加密和解密的开销，但因为使用对称加密撒unfinished，因此不会成为瓶颈。对称加密的性能跟使用的算法及密钥长度密切相关。但即便是最慢的算法一般也不会造成瓶颈。

SSL支持连接恢复机制：客户端在短暂断开后立即重连，不需要经历完整的建立连接的开销。While that is useful for HTTP, it often isn't effective for other protocols. As is HTTP keepalive, which is a protocol option to keep sockets open for a period of time after a request is completed, so that the connection may be reused if another request to the same server follows in short order.

##### 1.3.1.1 加密加速硬件

One common approach for speeding up SSL is to use hardware acceleration. Many vendors provide PCI cards that can unload the burden of cryptographic operations from your processor, and OpenSSL supports most of them. We discuss the specifics of using hardware acceleration in
Chapter 4.

##### 1.3.1.2 负载均衡

Another popular option for managing efficiency concerns with SSL is load balancing, which is simply distributing connections transparently across multiple machines.

Often, however, load balancing requires more work to ensure that persistent data is readily available to all servers on the backend. Another problem with load balancing is that many solutions route new connections to arbitrary machines, which can remove most of the benefit of connection resumption, since few clients will actually connect to the original machine during reconnection.

One simple load balancing mechanism is round-robin DNS, in which multiple IP addresses are assigned to a single DNS name. In response to DNS lookups, the DNS server cycles through all the addresses for that DNS name before giving out the same address twice. This is a popular solution because it is low-cost, requiring no special hardware. Connection resumption generally works well with this solution, since machines tend to keep a short-term memory of DNS results.

One problem with this solution is that the DNS server handles the load management, and takes no account of the actual load on individual servers. Additionally, large ISPs can perform DNS caching, causing an uneven distribution of load. To solve that problem, entries must be set to expire frequently, which increases the load on the DNS server.

Hardware load balancers vary in price and features. Those that can remember outside machines and map them to the same internal machine across multiple connections tend to be more expensive, but also more effective for SSL.

Version 0.9.7 of OpenSSL adds new functionality that allows applications to handle load balancing by way of manipulating session IDs. Sessions are a subset of operating parameters for an SSL connection, which we'll discuss in more detail in Chapter 5.

#### 1.3.2 Keys in the Clear

In a typical SSL installation, the server maintains credentials so that clients can authenticate the server. In addition to a certificate that is presented at connection time, the server also maintains a private key, which is necessary for establishing that the server presenting a certificate is actually presenting its own certificate.

That private key needs to live somewhere on the server. 最安全的方法是使用加密硬件。

In cases in which hardware solutions aren't feasible, there is no absolute way to protect the private key from an attacker who has obtained root access, because, at the very least, the key must be unencrypted in memory when handling a new connection.[4] If an attacker has root, she can generally attach a debugger to the server process, and pull out the unencrypted key.
[4] Some operating systems (particularly "trusted" OSs) can provide protection in such cases, assuming no security problems are in the OS implementation. Linux, Windows, and most of the BSD variants offer no such assurance.

There are two options in these situations. First, you can simply keep the key unencrypted on disk. This is the easiest solution, but it also makes the job of an attacker simple if he has physical access, since he can power off the machine and pull out the disk, or simply reboot to single-user mode. Alternatively, you can keep the key encrypted on disk using a **passphrase**, which an administrator must type when the SSL server starts. In such a situation, the key will only be unencrypted in the address space of the server process, and thus won't be available to someone who can shut the machine off and directly access the disk.

Furthermore, many attackers are looking for low-hanging fruit, and will not likely go after the key even if they have the skills to do so. The downside to this solution is that unattended reboots are not possible, because whenever the machine restarts (or the SSL server process crashes), someone must type in the passphrase, which is often not very practical, especially in a lights-out environment. Storing the key in the clear obviously does not exhibit this problem.

In either case, your best defense is to secure the host and your network with the best available lockdown techniques (including physical lockdown techniques). Such solutions are outside the scope of this book.

What exactly does it mean if the server's private key is compromised? The most obvious result is that the attacker can masquerade as the server, which we discuss in the next section. Another result (which may not be as obvious) is that all past communications that used the key can likely
be decrypted. If an attacker is able to compromise a private key, it is also likely that the attacker could have recorded previous communications. The solution to this problem is to use ephemeral keying. This means a temporary key pair is generated when a new SSL session is created. This is then used for key exchange and is subsequently destroyed. By using ephemeral keying, it is possible to achieve forward secrecy, meaning that if a key is compromised, messages encrypted  with previous keys will not be subject to attack.[5] We discuss ephemeral keying and forward secrecy in more detail in Chapter 5.
[5] Note that if you are implementing a server in particular, it is often not possible to get perfect forward secrecy with SSL, since many clients don't support Diffie-Hellman, and because using cryptographically strong ephemeral RSA keys violates the protocol specification.




