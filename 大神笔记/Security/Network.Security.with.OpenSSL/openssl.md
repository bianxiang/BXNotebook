[toc]

## 前言

The SSL (Secure Socket Layer) protocol and its successor TLS (Transport Layer Security) can be used to secure applications that need to communicate over a network. OpenSSL 是实现 SSL 和 TLS 协议的开源库，是目前使用最多的开源实现。OpenSSL跨平台。主要在 C 和 C++ 程序中使用，但可以通过命令行或其他语言使用。

本书介绍如何使用OpenSSL保护应用。We won't just show you how to SSL-enable your applications, we'll be sure to introduce you to the most significant risks involved in doing so, and the methods for mitigating those risks. These methods are important; it takes more work to secure an SSL-enabled application than most people think, especially when code needs to run in multithreaded, highly interoperable environments where efficiency is a concern.

OpenSSL不仅仅是SSL的实现，还是一个通用的的加密库，适用于那些不适合使用SSL的地方。Working with cryptography at such a low level can be dangerous, since there are many pitfalls in applying cryptography of which few developers are fully aware. Nonetheless, we do discuss the available functionality for those that wish to use it. Additionally, OpenSSL provides some highlevel primitives, such as support for the S/MIME email standard.

讨论将围绕例子。We discuss all of the common options OpenSSL users can support, as well as the security implications of each choice.

Depending on your needs, you may end up skipping around in this book. For people who want to use OpenSSL from the command line for administrative tasks, everything they need is in the first three chapters. Developers interested in SSL-enabling an application can probably read Chapter 1, then skip directly to Chapter 5 (though they will have to refer to parts of Chapter 4 to understand all the code).

本书章节内容：

- 第1章：This chapter introduces SSL and the OpenSSL library. We give an overview of the biggest security risks involved with deploying the library and discuss how to mitigate them at a high level. We also look at how to use OpenSSL along with Stunnel to secure third-party software, such as POP servers that don't otherwise have built-in SSL support.
- 第2章：命令行下使用OpenSSL。
- 第3章：解释 Public Key Infrastructure (PKI) 基础。This chapter is primarily concerned with how to go about getting certificates for use in SSL, S/MIME, and other PKI-dependent cryptography. We also discuss how to manage your own PKI using the OpenSSL command line, if you so choose.
- 第4章：In this chapter, we talk about the various low-level APIs that are most important to OpenSSL. Some of these APIs need to be mastered in order to make full use of the OpenSSL library. Particularly, we lay the foundation for enabling multithreaded application support and performing robust error handling with OpenSSL. Additionally, we discuss the OpenSSL IO API, its randomness API, its arbitrary precision math API, and how to use cryptographic acceleration with the library.
- 第5章：Here we discuss the ins and outs of SSL-enabling applications, particularly with SSLv3 and its successor, TLSv1. We not only cover the basics but also go into some of the more obscure features of these protocols, such as session resumption, which is a tool that can help speed up SSL connection times in some circumstances.
- 第6章：This chapter covers everything you need to know to use OpenSSL's interface to secretkey cryptographic algorithms such as Triple DES, RC4, and AES (the new Advanced Encryption Standard). In addition to covering the standard API, we provide guidelines on selecting algorithms that you should support for your applications, and we explain the basics of these algorithms, including different modes of operation, such as counter mode. Additionally, we talk about how to provide some security for UDP-based traffic, and discuss general considerations for securely integrating symmetric cryptography into your applications.
- 第7章：本章介绍如何使用单向加密哈希函数，或称消息摘要算法。We also show how to use Message Authentication Codes (MACs), which can be used to provide data integrity via a shared secret. We show how to apply MACs to ensure that tampering with HTTP cookies will be detected.
- 第8章：Here we talk about the various public key algorithms OpenSSL exports, including Diffie-Hellman key exchange, the Digital Signature Algorithm (DSA), and RSA. Additionally, we discuss how to read and write common storage formats for public keys.
- 第9章：Perl、Python、PHP的使用
- 第10章：In this chapter, we discuss many of the more esoteric parts of the OpenSSL API that are still useful, including the OpenSSL configuration API, creating and using S/MIME email, and performing certificate management programmatically.
- 附录A：Here we provide a reference to the many options in the OpenSSL command-line interface.

Additionally, the book's web site (http://www.opensslbook.com) contains API reference material that supplements this book. We also give pointers to the official OpenSSL documentation.

As we finish this book, OpenSSL is at Version 0.9.6c, and 0.9.7 is in feature freeze, though a final release is not expected until well after this book's publication. Additionally, we expect developers to have to interoperate with 0.9.6 for some time. Therefore, we have gone out of our way to support both versions. Usually, our discussion will apply to both 0.9.6 and 0.9.7 releases unless otherwise noted. If there are features that were experimental in 0.9.6 and changed significantly in 0.9.7 (most notably support for hardware acceleration), we tend to explain only the 0.9.7 solution.

We've set up a web site at www.opensslbook.com. It contains an up-to-date archive of all the example code used in this book. All the examples have been tested with the appropriate version of OpenSSL on Mac OS X, FreeBSD, Linux, and Windows 2000. They're expected to work portably in any environment that supports OpenSSL.



