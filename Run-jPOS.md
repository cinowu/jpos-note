#运行jpos

###1)运行gradle installApp创建build/install/jpos目录

###2)进入以上目录发现一个jpos-xxx.jar的文件

###3）执行java -jar jpos-1.9.1-SNAPSHOT.jar 或者执行bin/q2 or bin/q2.bat scripts.
  
###4）看到启动效果如下：
```xml
<log realm="org.jpos.q2.qbean.SystemMonitor" at="Fri Jul 12 11:51:37 UYT 2013.882">
<info>
OS: Mac OS X
host: Macintosh-2.local/192.168.2.20
version: 1.9.1-SNAPSHOT (fb4cc76)
instance: cd5013af-1d38-4a5e-b771-e807904212e1
uptime: 00:00:00.218
processors: 2
drift : 0
memory(t/u/f): 85/7/77
threads: 4
Thread[Reference Handler,10,system]
Thread[Finalizer,8,system]
Thread[Signal Dispatcher,9,system]
Thread[RMI TCP Accept-0,5,system]
Thread[Q2-cd5013af-1d38-4a5e-b771-e807904212e1,5,main]
Thread[DestroyJavaVM,5,main]
Thread[Timer-0,5,main]
Thread[SystemMonitor,5,main]
name-registrar:
logger.Q2.buffered: org.jpos.util.BufferedLogListener
logger.Q2: org.jpos.util.Logger
</info>
</log>
```xml  

发现与这个jpos-xx.jar同级目录下的多出来个deploy目录，这个是从/src/dest目录过来的

