[toc]

## 线程

### 多线程环境下的方法注释规范

如果一个方法可能在多线程环境下使用，需要标注一下内容，即便方法的访问级别是private。

1. 该方法是否是线程安全的
1. 如果该方法是线程受限的，指出该方法只能在哪个线程中调用
1. 表明该方法是否可能阻塞（特别耗时）、是否可能无限阻塞；如果会阻塞，是否支持中断。
  1. 如果方法内部使用涉及IO
  1. 该方法内部是否涉及阻塞方法
  1/ 该方法内部是否涉及同步或使用锁的方法；如果类继承有父类，特别注意是否会调用父类的同步方法
1. 如果方法内部用到锁、同步，如果可能，标明该方法使用了锁、同步，标明锁名。标明锁名只是让调用者这道该方法可能产生的死锁，一般开发者不应该尝试使用注明的锁。如果允许调用者使用内部锁（client-size lock），请标明允许，甚至标明使用方法。

例子：
JPOS-2014
线程在下面方法某处阻塞了。（该方法进行了删节）。对象实例为蓝色。
该方法涉及了同步方法、IO方法、阻塞方法。
但该方法没有任何关于并发的注释。如：该方法是否会被并发调用，还是仅限于某一线程调用。

    public int readReceiptDetail(int sock, File file) throws Exception {

        boolean bRunningFlag = true;
        logger.debug("write to " + file.getAbsolutePath());
        if (!file.getParentFile().exists()) {
          file.getParentFile().mkdirs();
        }
        FileOutputStream out = new FileOutputStream(file);
        try {
          // 不断获取收据交易记录

          while (bRunningFlag) {

            int iErrorCode = sm100.readReceiptDetail(sock, iRecordNo, rptdetail); // 调用外部方法
            switch (iErrorCode) {
            case 0x00:// 正确
              out.write(rptdetail.serialize().getBytes("UTF-8"));
              out.write("\n".getBytes("UTF-8"));
              setChanged(); // 此方法是同步方法
              notifyObservers(Sm100Consts.ReceiptDetailRead);  // 此方法是同步方法
              ++count;
              if (count % 10 == 0) {
                logger.info("已读取" + count + "条明细数据");
              }
              break;
          }
          out.flush();
          out.getFD().sync(); // 此方法为阻塞方法
          logger.info("共读取" + count + "条明细数据");
          return count;
        } finally {
          out.close();
        }
      }

该方法的另一个教训是，因该方法阻塞导致其他使用该线程调用的任务不能完成。若非要求顺序执行或任务间有依赖，任何可能阻塞的任务在单独线程里调用为安全。或使用异步-回调模拟带超时的顺序操作：若一个方法调用不支持超时，当需要调用该方法时，新开线程执行该方法，原线程挂起指定时间，等待被新线程唤醒或超时，原线程超时若新线程仍没有结果，将新线程标记为中断，继续原线程的执行。