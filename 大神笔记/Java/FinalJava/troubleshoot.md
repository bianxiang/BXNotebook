[toc]

## 远程接口不要抛出JDK之外的异常类，否则可能导致本地抛出ClassNotFoundException

在做远程方法调用时（无论通过Java本身的机制还是远程调用框架），若抛出远程异常，不要把远端异常栈带出去，否则有可能导致本地ClassNotFoundException异常。

即在远端

    try{
       some thing
    } catch(Throwable e){
       throw new RemoteException("message", e);
    }
上面的写法将把远端的异常栈传给本地。

异常类也是类，本地可能没有远端的异常类，如下面的SQLServerException，本地是没有的。于是导致本地抛出ClassNotFoundException。

示例异常：

这是本地的日志。注意到本地的错误是在解析远程异常时，遇到了ClassNotFound。

    2012-02-28 14:51:11,703 [sync] ERROR com.hd123.jpos.server3.impex.EmpPhoneDownloader - 下载员工手机号码列表 -
    com.hd123.jpos.remote.exception.ParseFail: 解析错误; nested exception is:
    java.lang.ClassNotFoundException: com.microsoft.sqlserver.jdbc.SQLServerException
    at com.hd123.jpos.remote.SpringSimpleHttpInvokerRequestExecutor.doExecuteRequest(SpringSimpleHttpInvokerRequestExecutor.java:97)
    at org.springframework.remoting.httpinvoker.AbstractHttpInvokerRequestExecutor.executeRequest(AbstractHttpInvokerRequestExecutor.java:136)
    at org.springframework.remoting.httpinvoker.HttpInvokerClientInterceptor.executeRequest(HttpInvokerClientInterceptor.java:191)
    at org.springframework.remoting.httpinvoker.HttpInvokerClientInterceptor.executeRequest(HttpInvokerClientInterceptor.java:173)
    at com.hd123.jpos.remote.SpringHttpInvokerProxyFactoryBean.invoke(SpringHttpInvokerProxyFactoryBean.java:25)
    at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:171)
    at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:204)
    at $Proxy88.getList(Unknown Source)
    at com.hd123.jpos.server3.impex.EmpPhoneDownloader.run(EmpPhoneDownloader.java:42)
    at com.hd123.jpos.server.impex.Synchronizer$SynchronizerThread.doLoad(Synchronizer.java:239)
    at com.hd123.jpos.server.impex.Synchronizer$SynchronizerThread.doDownload(Synchronizer.java:187)
    at com.hd123.jpos.server.impex.Synchronizer$SynchronizerThread.run(Synchronizer.java:140)
    Caused by: java.lang.ClassNotFoundException: com.microsoft.sqlserver.jdbc.SQLServerException
    at java.net.URLClassLoader$1.run(URLClassLoader.java:202)
    at java.security.AccessController.doPrivileged(Native Method)
    at java.net.URLClassLoader.findClass(URLClassLoader.java:190)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:307)
    at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:301)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:248)
    at org.springframework.util.ClassUtils.forName(ClassUtils.java:211)
    at org.springframework.core.ConfigurableObjectInputStream.resolveClass(ConfigurableObjectInputStream.java:56)
    at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:1575)

上面那段日志实际是下面这段代码产生的：

    try {
          InputStream responseBody = readResponseBody(config, con);
          retVal = readRemoteInvocationResult(responseBody, config.getCodebaseUrl());
        } catch (Throwable ex) {
          if (ex instanceof IOException || ex instanceof SocketException) {
            throw new ServerTimeoutFail("", ex);
          } else if (ex instanceof ClassNotFoundException || ex instanceof NoClassDefFoundError
              || ex instanceof InvalidClassException) {
            throw new ParseFail(ex);
          } else {
            throw new ServerReadFail(null, ex);
          }
        }
因为远端和本地是两个系统，远端的异常细节不应该暴露给本地，因此应该使用单参数版本的RemoteException，message中提供足够多但又不暴露内部细节的信息。

------作为远端，如果想让客户端有效得到错误信息，应该抛一个不带异常栈的、JDK中的异常类。

    try{
       some thing
    } catch(Throwable e){
       throw new RemoteException("message");
    }

另一方面，调用者应该如何处理收到的远程异常？

首先调用者不能假设远端抛出的异常是它可识别的------远端有可能是不受我们自己控制的，因此我们对它要有防御性编码的态度。即便远端是善意的，它也可能不小心抛出整个异常栈。比如如果远端捕获的异常级别是SQLException，但远端抛出RuntimeException；或远端捕获级别是Exception，但由高于Exception级别的异常抛出。

如果异常类在本地是没有的，在由远程调用框架将异常类解序列化时就会报ClassNotFound。此时远程框架应该尝试报给调用者“远端抛出异常，但内容无法解析”，即上面日志中的ParseFail类。

## 精度、标度scale的区别

精度MathContext.setPrecision

如果实际数字标度大于指定标度，必须用舍入

	curLineCopy.getProduct().getWholeSaleUnitPrice()).setScale(2, RoundingMode.HALF_UP)
否则报：

    Exception in thread "AWT-EventQueue-1" java.lang.ArithmeticException: Rounding necessary
          at java.math.BigDecimal.divideAndRound(BigDecimal.java:1439)
          at java.math.BigDecimal.setScale(BigDecimal.java:2390)
          at java.math.BigDecimal.setScale(BigDecimal.java:2437)


