# 终止服务器

按下Ctrl-C终止服务器时发生的事情：

1. JVM进程收到了终止信号之后 ，首先会经过一个 hook 线程的处理，然后再终止。这个hook线程负责的就是安全终止各服务器。

   ```java
   // CorfuServer.main()
   Thread shutdownThread = new Thread(CorfuServer::cleanShutdown);
           shutdownThread.setName("ShutdownThread");
           Runtime.getRuntime().addShutdownHook(shutdownThread);
   ```

   我们可以看到```shutdownThread```这个线程被注册为收到shutdown信号时的hook，它要做的事是```CorfuServer::cleanShutdown```。

2. ```java
   // CorfuServer.cleanShutdown
   private static void cleanShutdown() {
           log.info("CleanShutdown: Starting Cleanup.");
           shutdownServer = true;
           activeServer.close();
       }
   ```

   上述activeServer就是一个```CorfuServerNode```，调用其close函数依次关闭

   1. serverContext
      - clientGroup
      - bossGroup
      - workerGroup
   2. channel
   3. 生成对应数量（5个）的线程来各自关闭下列server
      - BaseServer
      - SequencerServer
      - LayoutServer
      - LogUnitServer
        - logCleaner
        - batchWriter
      - ManagementServer
        - Orchestrator
        - Fault Detection MonitoringService
        - Local Metrics Polling Task
        - Management Agent

3. 等上述全部成功shutdown后，```shutdownThread任务结束。