##1）JPOS日志系统
JPOS日志系统和其他日志系统最大的不同是，JPSO 处理live objects。一个日志事件处理live objects以至于可以被LogListener处理。举个例子，为了保护敏感信息（PCI需求）或者按照特殊情况行事。EMAIL操作在异常的时候不得不解析序列信息。

其他日志系统通常"面向于一行",jPOS主要“面向于交易”，一个日志事件可能抓取整个交易过程中的信息，使得它非常时候在查账审核和DEBUG的时候。

为了阻止初始化的时候获取的jPOS Logger而是用你自己的日志系统,你可以把jPOS考虑成一个事件日志或者一个查账审查日志。我们不会用它来添加debug或者是跟踪应用，我们用它来记录商业相关数据。

你仍然可以坚持使用你自己喜爱的日志系统作为你商业逻辑的一部分。

jPOS的日志系统很容易扩展，因此可以很容易移植到其他日志引擎上，比如LOG4J,COMMON LOGGING等，
但是那很少用。

jPOS的益处在于事实上它产生很容易读取的轻量级的可解析的XML输出。LogChannel的例子可以读取一个
jPos的日志文件，可以从中读取ISO-8583信息。如果你移植了其他日志系统在它之上，那么这个输出很有
可能被每行加上一个时间戳以至于渲染到的文件很难被解析。
	
jpos日志的主要实现类。
<pre>
	Class 			 Description
	Logger 			 Main logger class
	LogListener		 Listens to log events
	LogSource 		 A log event producer has to implement LogSource
	LogEvent 		The Log Event
</pre>


Logger内部实现：
```java
	public class Logger {
		public static void log (LogEvent ev);
		...
		public void addListener (LogListener l);
		public void removeListener (LogListener l);
		public boolean hasListeners();
		...
		...
	}
```
	
LogSource内部实现如下：
```java
	public interface LogSource {
		public voidsetLogger (Logger logger, String realm);
		publicString getRealm ();
		publicLogger getLogger ();
	}
```
	
LogEvents实现如下：
```java
	public class LogEvent {
		public LogEvent (LogSource source, String tag);
		...
		...
		public void addMessage (Object msg);
		...
	}
```	
	
参见：http://jpos.org/doc/javadoc/org/jpos/util/LogEvent.html
	
	
如下是简单创建LOGGER的例子：
```java
	Logger logger = new Logger();
	logger.addListener (new SimpleLogListener (System.out))
```
	
现在捏可以很容易的附加上logger到任何的jpos组件实现类中.LogSource比如channel,packagers、multiplexers等你可以很容易的调用如下：
```java
component.setLogger (logger, "some-component-description");
```
	
你可以添加jPOS的日志系统到你自己的日志事件中。在那些情况下，你不得不实现LogSource接口或者继承自
org.jpos.util.SimpleLogSource这个类，或者更好的，直接使用 org.jpos.util.Log类.
你可以如下这么写:
```java
	LogEvent evt = new LogEvent (yourLogSource, "my-event");
	evt.addMessage ("A String message");
	evt.addMessage (anyLoggeableObject);
	Logger.log (evt);
```
	
Loggeable接口提供了非常简单的方式去让一个对象渲染自己。
```java
	public interface Loggeable {
		public voiddump (PrintStream p, String indent);
	}
```
	
大多数jPOS的组件已经实现了Loggeable接口，但是你可以很容易分装任何已经给的对象带上一个Loggeable类
	
LoggerListener几个可用的实现类：
<pre>
	Class			 		Description
	SimpleLogListener 		Dumps log events to a PrintStream (such as System.out)
	RotateLogListener 		Automatically rotate logs based on file size and time window
	DailyLogListener 			Automatically rotate logs daily. Has the ability to compress old log files
	OperatorLogListener 		Applies some filtering and e-mails log-events to an operator
	ProtectedLogListener 		Protect sensitive data from ISOMsgs in LogEvents for PCI compliance
	SysLogListener 			Forward log events to the operating system syslog.
	
	在JPOS-EE代码中可以发现附加的logger实现类，比如IRCLogListener
</pre>

##2）NameRegistrar
org.jpos.util.NameRegistrar是一个非常简单的单例类可以用来注册和定位jPOS组件。  
没有什么用但是非常简单，熟悉的MAP可以很容易通过arbitrary name找到组件。  
拥有如下静态方法：  
```java
	public static void register (String key, Object value);
	public static void unregister (String key);
	public static Object get (String key) throws NameRegistrar.NotFoundException;
	public static Object getIfExists (String key);
```java
	
因此你可以这么写：
```java
	...
	...
	ISOMUX mux = new ISOMUX (...);
	NameRegistrar.register ("myMUX", mux);
	...
	...
	
或者在你程序中你可以获得一个引用指向你的MUX：
```java
	try{
	ISOMUX mux = (ISOMUX) NameRegistrar.get ("myMUX");
	} catch(NameRegistrar.NotFoundeException e) {
	...
	...
	}
```
	
或者：
```
	ISOMUX mux = (ISOMUX) NameRegistrar.getIfExists ("myMUX");
	if(mux != null) {
	...
	...
	}
```

尽管我们用NameRegistrar为了注册jPOS组件，有时候我们最好用组件的setName方法。

大多数组件都有setName这个方法如下：
```java
	public classISOMUX {
	...
	...
	public void setName (String name) {
		this.name = name;
		NameRegistrar.register ("mux."+name, this);
	}
	...
	...
```
前缀"mux."为了区分不同的类，比如"mux.institutionABC"and"channel.institutionABC"
	
不同的组件利用不同的前缀如下：
<pre>
	Component 				Prefix 					Getter
	ConnectionPool 		"connection.pool." 				N/A
	ControlPanel 				"panel." 					N/A
	DirPoll 					"qsp.dirpoll." 				N/A
	BaseChannel 				"channel." 				BaseChannel.getChannel
	ISOMUX 				"mux." 					ISOMUX.getMUX
	QMUX 					"mux." 					QMUX.getMUX
	ISOServer 				"server." 					ISOServer.getServer
	KeyStore 				"keystore." 				N/A
	Logger 					"logger." 					Logger.getLogger
	LogListener 				"log-listener." 				N/A
	PersistentEngine 			"persistent.engine." 		N/A
	SMAdapter 				"s-m-adapter." 			BaseSMAdapter.getSMAdapter
</pre>

我们强烈建议你获得组件引用的时候有问题的时候去看代码。
	
利用getter方法如下：
```java
	try{
	ISOMUX mux = ISOMUX.get ("myMUX");
	} catch(NameRegistrar.NotFoundeException e) {
	...
	...
	}
```
这里面实际会调用 NameRegistrar.get ("mux.myMUX")。接下来，我们可以看到
NameRegistrar被广泛应用在jPOS' Q2的应用程序中。Q2处理配置很多jPOS组件，的那是你的代码可能
得通过名字定位他们。

	
> 【注】单例通常是一种错觉，你想到的它紧急那是一个，但是它可能大于一个。如果你有很多个类加载器在你的
> 程序中，你很有可能拥有一个单例的多个拷贝，比如NameRegistrar.如果你用Q2作为一个标准程序的话以上问题是不会存在的。
	
> 【注】NameRegistrar也是一个Loggeable对象，因此它的实例可以被添加到LogEnvent为了帮助在DEBUG过程中的SESSION.当我们运行Q2环境的 时候，我们建议部署一个sysmon服务为了周期性的查看NameRegistrar的内容。


##3)Configuration
org.jpos.core.Configuration接口内部实现如下：
```java
	package org.jpos.core;
	public interface Configuration {
		public voidput (String name, Object value);
		public String get (String propertyName);
		public String get (String propertyName, String defaultValue);
		public String[] getAll (String propertyName);
		public int[] getInts (String propertyName);
		public long[] getLongs (String propertyName);
		public double[] getDoubles (String propertyName);
		public boolean[] getBooleans (String propertyName);
		public int getInt (String propertyName);
		public int  getInt (String propertyName, intdefaultValue);
		public long getLong (String propertyName);
		public long getLong (String propertyName, longdefaultValue);
		public double getDouble (String propertyName);
		public double getDouble (String propertyName, doubledefaultValue);
		public boolean getBoolean (String propertyName);
		public boolean getBoolean (String propertyName, booleandefaultValue);
	}
```
	
当我们看到Q2程序的时候，我们可以看到Q2通过调用setConfiguration方法：
```xml
	<object name="myObject" class="com.mycompany.MyObject">
	<property name="myProperty" value="any Value"/>
	</object>
```
	
com.mycompany.MyObject"这样的类实现Configurable接口的时候，Q2会调用setConfiguration方法
提供访问myProperty属性。
	

非常有趣的是Q2提供了这样的能力：可以有属性的数组在同一个名字下
```xml
	<object name="myObject" class="com.mycompany.MyObject">
	<property name="myProperty" value="Value A"/>
	<property name="myProperty" value="Value B"/>
	<property name="myProperty" value="Value C"/>
	</object>
```

我们可以很方便的调用方法比如String[] getAll(String )  
setConfiguration(Configuration cfg)可以检查Configuration对象异常，这种异常发生在当一个已经需要的
属性不是本文件的或者不是合法的。


##4）SystemMonitor
org.jpos.util.SystemMonitor是一个非常简单的类周期性的记录有用的信息，比如运行时线程的数量、内存使用情况等。
	
它的构造方法：
publicSystemMonitor (intsleepTime, Logger logger, String realm)
参照:http://jpos.org/doc/javadoc/org/jpos/util/SystemMonitor.html
	
使用SystemMonitor是非常容易的，你可以简单的初始化它：
```java
	...
	...
	new SystemMonitor (60*60*1000L, yourLogger, "system-monitor"); // dumps every hour
	...
	...
```

这段代码会每隔一个小时将所有监控信息存储在你的LOG文件中，输出如下：
```xml
	<info>
	OS: Mac OS X
	host: Macintosh-2.local/192.168.2.20
	version: 1.9.3-SNAPSHOT (d3c9ac3)
	instance: 38d512f6-f812-4d85-8520-cb96de2654a0
	uptime: 00:00:00.234
	processors: 2
	drift : 0
	memory(t/u/f): 85/7/78
	threads: 4
	Thread[Reference Handler,10,system]
	Thread[Finalizer,8,system]
	Thread[Signal Dispatcher,9,system]
	Thread[RMI TCP Accept-0,5,system]
	Thread[Q2-38d512f6-f812-4d85-8520-cb96de2654a0,5,main]
	Thread[DestroyJavaVM,5,main]
	Thread[Timer-0,5,main]
	Thread[SystemMonitor,5,main]
	name-registrar:
	logger.Q2.buffered: org.jpos.util.BufferedLogListener
	logger.Q2: org.jpos.util.Logger
	</info>
```

##5）Profiler
org.jpos.util.Profiler非常简单，很容易去使用user-space Profiler. 它使得Logger系统提供了
准确的信息关于处理时间的。

	
内部实现如下：
```java
	public voidreset();
	public voidcheckPoint (String detail);
	public longgetElapsed();
	public longgetParcial();
```
参照：http://www.jpos.org/doc/javadoc/org/jpos/util/Profiler.html
	
Profiler实现了Loggeable接口，因此你可以添加一个Profiler Object到一个LogEvent去产生合适的描述信息。
	

Profiler例子：
```java
	Profiler prof = newProfiler();
	LogEvent evt = newLogEvent (this, "any-transaction", prof);
	// initialize message
	ISOMsg m = newISOMsg ();
	m.setMTI ("1200");
	...
	...
	prof.checkPoint ("initialization");
	// send message to remote host
	...
	...
	ISORequest req = newISORequest (m);
	mux.queue (req);
	ISOMsg response = req.getResponse (60000);
	prof.checkPoint ("authorization");
	// capture data in local database
	...
	...
	prof.checkPoint ("capture");
	...
	...
	Logger.log (evt);
```

checkPoint会自动计算输出时间
	
这个profiler输出如下：
<pre>
	prepare: org.jpos.jcard.PrepareContext [0.2/0.2]                .......... 1
	prepare: org.jpos.jcard.CheckVersion [0.1/0.3]                   .......... 2
	prepare: org.jpos.transaction.Open [1.0/1.3]
	prepare: org.jpos.jcard.Switch [0.1/1.5]
	prepare: org.jpos.jcard.NotSupported [0.1/1.7]
	prepare: org.jpos.jcard.PrepareResponse [11.2/13.0]
	prepare: org.jpos.transaction.Close [0.2/13.2]
	prepare: org.jpos.jcard.SendResponse [0.0/13.3]
	prepare: org.jpos.jcard.ProtectDebugInfo [0.1/13.4]
	prepare: org.jpos.transaction.Debug [0.0/13.5]
	commit: org.jpos.transaction.Close [1.8/15.4]
	commit: org.jpos.jcard.SendResponse [2.2/17.6]
	commit: org.jpos.jcard.ProtectDebugInfo [0.3/17.9]
	commit: org.jpos.transaction.Debug [3.9/21.9]              .............3
	end [1.9/23.9]                                                                   .............4
	1.到目前为止0.2毫秒
	2.CheckVersion花费0.1毫秒，整个到目前为止华为0.3毫秒
	3.总共到目前花费21.9毫秒
	4.在last checkPoint和log time之间是1.9毫秒
</pre>

##6）DirPoll

许多基于jPOS的应用不得不受到第三方遗留软件的影响，比如batch file能获得知识,零售应用等等。大多数时间，
一个可能很足够的幸运去处理遗留应用，发送交易到有分寸的协议，但是有的时候你可能不够幸运，最好事情对你
来说就是获取到一个基于硬盘的交换。比如他们放置了一个请求到一个目录，你处理这个请求然后提供响应。
	

org.jpos.util.DirPoll结构如下：
<pre>
	..../request
	..../respons
	..../tmp
	..../run
	..../bad
</pre>


定义了如下内部接口：
```java
	public interface Processor {
		public byte[] process(String name, byte[] request) throws DirPollException;
	}
	public interface FileProcessor {
		public void process (File name)throws DirPollException;
	}
```
	
你要么创建Processor或者FileProcessor处理进来的交易。

	
无论什么时候，遗留项目放置一个文件在request目录中，你的Processor或者FileProcessor会被调用。
给你一个机会去处理已经给的请求然后提供一个响应。（如果你用了一个Proccessor，这个响应可能会被
放置在response目录中）

	
DirPoll Proccessor例子：
```java
	public class DirPollProcessor implements DirPoll.Processor {
		DirPollProcessor () {
			super();
			DirPoll dp = newDirPoll ();
			dp.setLogger (logger, "dir-poll");
			db.setPath ("/tmp/dirpoll");
			db.createDirs ();
			db.setProcessor (this);
			newThread (dp).start ();
		}
		public byte[] process (String name, byte[] b) {
			return("request: "+ name + " content="+ newString (b)).getBytes();
		}
	}
```


###7）ThreadPool
这个类可能会被过时，在最新的代码中不会被用到了。  


ThreadPool常用在不同的jPOS组件中，比如ISOServer，10年前这是个非常有用的类。我们将会替换掉它通过
JAVA执行框架的组件。
