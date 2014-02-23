#启动client-simulator:
http://jpos.org/blog/2013/07/setting-up-the-client-simulator/



#Simulator -- 模拟程序
##1.定义
Server Simulator 是一个极其特别的例子，基于BSH的simulator可以用来测试基于ISO-8583的客户端软件。


监听在默认的端口10000，所有将要进来的请求都是基于一个BeanShell的脚本，这些都是可自定义的来满足的你的需求。
	
<pre>
	What: 参照如上定义.
	When: 适合于jPOS-EE的所有版本.
	Who: jPOS.org项目组.
	How: jPOS-EE项目组发布的.
	Where: GitHub仓库上的modules/server-simulator目录.
	Why: 当你写基于ISO-8583的客户端程序时，它可以很好的模拟一个非常好用的服务器端。
	Status: 稳定的.
	Dependencies: 依赖于jpos模块
	License: GNU Affero General Public License version 3
</pre>
	
这里是默认的配置文件（D:\Java\workspace\jPOS-EE\jPOS-EE\modules\server-simulator\src\main\resources\META-INF\q2\installs\deploy\05_serversimulator.xml）:
```xml
	<server class="org.jpos.q2.iso.QServer" logger="Q2" name="simulator_10000">
	    <attr name="port" type="java.lang.Integer">10000</attr>
	    <channel class="org.jpos.iso.channel.XMLChannel"  logger="Q2" packager="org.jpos.iso.packager.XMLPackager">
	    </channel>
	    <request-listener class="org.jpos.bsh.BSHRequestListener" logger="Q2">
	        <property name="source" value="cfg/serversimulator.bsh" />
	    </request-listener>
	</server>
```

```java
       -------------------------为什么能够启动QServer----------------------------------------------------
       /**
	 * ISO Server wrapper.
	 * @jmx:mbean description="ISOServer wrapper"
	 *                  extends="org.jpos.q2.QBeanSupportMBean"
	 */
	public class QServer extends QBeanSupport
    					implements QServerMBean, SpaceListener, ISORequestListener
```


看源码便知，QServer 继承了QBeanSupport类，是标准的QBean，所以可以在XML中  

看类的注释信息，QServer类是ISO Server的封装，并且JMX描述mbean信息时说明
Qserver也继承QBeanSupportMBean类，因为它的父类QBeanSupport已经继承了QBeanSupportMBean  
	
###Q2特性之一：
Q2通常扫描deploy目录，然后寻找QBean描述符文件,这样的描述符


很简单和灵活，要求如下两点：  
1）最外层元素至少要拥有name这个属性  
```xml
<server class="org.jpos.q2.iso.QServer" logger="Q2" name="simulator_10000">
```

2) 虽然我们QFactory.properties配置文件中得知外层元素是ResourceBundle,
但是最外层元素必须指定class用来确定QBean实现自哪个类
```xml
<server class="org.jpos.q2.iso.QServer" logger="Q2" name="simulator_10000">
```
	
	
----------解析以上XML-------------------


###Q2特性之二：
Q2 是默认的日志名, 经常定义这个名称的00_logger.xml QBean描述符文件.
对应日志描述符文件路径：serversimulator\build\install\serversimulator\deploy\00_logger.xml 

一个QBean实现Loggeable接口，QBean描述符有一个logger属性，Q2会调用setLogger方法。如果QBean描述符有一个logger属性，

QBean的实现类必须有一个void setLogger(Logger)的方法，这样Q2会设置一个logger
realm属性也一样，如果一个QBean描述符有realm属性，QBean实现类必须有一个void setRealm(String)方法，Q2会调用它设置适当的realm


根据如上描述，QServer父类QBeanSupport的构造方法中调用了setLogger方法
```java
	   public QBeanSupport () {
	        super();
	        setLogger (Q2.LOGGER_NAME);
	        state = -1;
	    }
	    public void setLogger (String loggerName) {
       		 log = Log.getLog (loggerName, getClass().getName());
                 setModified (true);
             }
         同样，因为有name属性，QBean父类QBeanSupport类中也有setName方法：
            public void setName (String name) {
	        if (this.name == null) 
	            this.name = name;
	        if (log != null)
	            log.setRealm (name);
	        setModified (true);
	    }
```
	
###Q2特性之三：
如果顶层元素内部元素有<property>这样的标签  

如果一个QBean的实现类实现了Configurable接口，然后它的描述符XML文件有<property>标签，Q2会创建一个Configuration对象然后调用它的setConfiguration方法。  

这样的话你在QBean的实现类中可以这么取到<property中的属性值  

比如： 
```xml
	<qbean name="test3" class="com.wujintao.jpos.test.Test3" logger="Q2" realm="Test3" >
	<property name="mypropA" value="abc" />
	<property name="mypropB" value="123" />
	</qbean>
```


那么在Test3这个QBean中如下：
```java
	public class Test3 extends QBeanSupport implements Runnable{
	    @Override
	    protected void initService() throws Exception {
	        super.initService();
	        new Thread(this).start();
	    }
	
	    @Override
	    public void run() {
	        String pro1 = cfg.get("mypropA");
	        String pro2 = cfg.get("mypropB");
	        System.err.println(pro1+"==========="+pro2);
	    }
	    。。。。。。。
	}
```
		
server simulator是一个简单的QServer带有一个BSHRequestListener，这样可以处理发送进来的消息然后提供稳定的响应。  

默认的配置用了XMLChannel加之一个XMLPackager,但是使用任何channel/packager组合对你来说是免费的。  

---分析channel子标签开始---

以下是QServer的方法
```java
	    private void newChannel () throws ConfigurationException {
	        Element persist = getPersist ();
	        Element e = persist.getChild ("channel");
	        if (e == null) {
	            throw new ConfigurationException ("channel element missing");
	        }
	
	        ChannelAdaptor adaptor = new ChannelAdaptor ();
	        channel = adaptor.newChannel (e, getFactory ());
	    }
```


从上看，这个方法用jdom获取了channel这个子标签，以上adaptor的newChannel方法在ChannelAdaptor类中如下：
```java
	     public ISOChannel newChannel (Element e, QFactory f) 
	        throws ConfigurationException
	    {
	        String channelName  = e.getAttributeValue ("class");
	        String packagerName = e.getAttributeValue ("packager");
	
	        ISOChannel channel   = (ISOChannel) f.newInstance (channelName);
	        ISOPackager packager = null;
	        if (packagerName != null) {
	            packager = (ISOPackager) f.newInstance (packagerName);
	            channel.setPackager (packager);
	            f.setConfiguration (packager, e);
	        }
	        QFactory.invoke (channel, "setHeader", e.getAttributeValue ("header"));
	        f.setLogger        (channel, e);
	        f.setConfiguration (channel, e);
	
	        if (channel instanceof FilteredChannel) {
	            addFilters ((FilteredChannel) channel, e, f);
	        }
	        if (getName () != null)
	            channel.setName (getName ());
	        return channel;
	    }
	    void newChannel ()这个方法在initServer()方法内被调用
	    private void initServer ()  throws ConfigurationException
	    {
	        if (port == 0) {
	            throw new ConfigurationException ("Port value not set");
	        }
	        newChannel();
	        if (channel == null) {
	            throw new ConfigurationException ("ISO Channel is null");
	        }
	        ...
	      }
```
	      
Q2启动的时候会调用Q2的start方法，QBeanSupport的start方法，start方法内部会调用startService方法。

startService方法在Q2中是空实现，等待QBeanSupport子类去实现。  

QServer正好是QBeanSupport子类，QServer的startService方法如下：
```java
	@Override
	    public void startService () {
	        try {
	            initServer ();
	            initIn();
	            initOut();
	            initWhoToSendTo();
	        } catch (Exception e) {
	            getLog().warn ("error starting service", e);
	        }
	    }
```java

这样的流程，你看懂了吗  
---分析channel子标签结束---      
	
	
---分析request-listener子标签开始--  
看下Q2的如下方法：
```java
	  private void addListeners ()throws ConfigurationException
	    {
	        QFactory factory = getFactory ();
	        Iterator iter = getPersist().getChildren (
	            "request-listener"
	        ).iterator();
	        while (iter.hasNext()) {
	            Element l = (Element) iter.next();
	            ISORequestListener listener = (ISORequestListener)
	                factory.newInstance (l.getAttributeValue ("class"));
	            factory.setLogger        (listener, l);
	            factory.setConfiguration (listener, l);
	            server.addISORequestListener (listener);
	        }
	    }
```

可以看出使用了jdom获取request-listener标签元素  
以上方法在  private void initServer ()方法内调用。  
initServer会在startService中调用  
```java
	    @Override
	    public void startService () {
	        try {
	            initServer ();
	            initIn();
	            initOut();
	            initWhoToSendTo();
	        } catch (Exception e) {
	            getLog().warn ("error starting service", e);
	        }
	    }
```

	    
QFactory的setConfiguration(Object obj, Element e)方法：
```java
	      public void setConfiguration (Object obj, Element e) 
	        throws ConfigurationException 
	    {
	        try {
	            if (obj instanceof Configurable)
	                ((Configurable)obj).setConfiguration (getConfiguration (e));
	            if (obj instanceof XmlConfigurable)
	                ((XmlConfigurable)obj).setConfiguration(e);
	        } catch (ConfigurationException ex) {
	            throw new ConfigurationException (ex);
	        }
	    }
```
	    
QServer的setConfiguration (Configuration cfg)方法：
```java
   public void setConfiguration (Configuration cfg) 
        throws ConfigurationException
    {
        this.cfg = cfg;
        bshSource = cfg.getAll ("source");
        String[] mti = cfg.get ("whitelist", "*").split(",");
        whitelist = new HashSet( Arrays.asList(mti) );
    }
```
这个方法调用了你看懂啦吗？
---分析request-listener子标签结束--
	
BSHRequestListener(参见jPOS programmer’s guide文档)暴露了两个对象：
message (发送进来的ISOMsg) and source (ISOSource).


BSH脚本像这样(D:\Java\workspace\jPOS-EE\jPOS-EE\modules\server-simulator\src\main\resources\META-INF\q2\installs\cfg\serversimulator.bsh文件 )：
```java
	message.setResponseMTI ();                                              -----------1
	Random random = new Random (System.currentTimeMillis());
	message.set (37, Integer.toString(Math.abs(random.nextInt()) % 1000000));
	message.set (38, Integer.toString(Math.abs(random.nextInt()) % 1000000));
	if ("000000009999".equals (message.getString (4)))                      ------------2
	    message.set (39, "01");
	else
	    message.set (39, "00");
	    source.send (message);
```

	    
1--设置响应MTI（比如0800/0810, 1201/1220....）  
2--我们用指定数目值$99.99降低交易  
 
【我们来看执行server端q2的流程】
###1.查找deploy目录
Q2源码类run方法开始 initSystemLogger ():
```java
   getLog().info ("Q2 started, deployDir="+deployDir.getAbsolutePath());
```

	
Q2源码类run方法最后logVersion()方法内调用打印如下两个版本信息：
```java
    	LogEvent evt = getLog().createLogEvent ("version");
        evt.addMessage (getVersionString());
        Logger.log (evt);
```

jPOS 1.9.4 master/3dfc9d5 (2013-12-24 12:44:45 UYST)
这一行，是打印jpos-1.9.4.jar里面org/jpos/q2/buildinfo.properties的版本号和时间，master/3dfc9d5答应的是org/jpos/q2/reversion.properties里面的内容。
请看org/jpos/q2/buildinfo.properties如下： 
<pre>
	 ---------------------------------------------
	    #Revision Properties
        #Tue Dec 24 12:44:45 UYST 2013
        version=1.9.4
        buildTimestamp=2013-12-24 12\:44\:45 UYST
	 ---------------------------------------------
</pre>


请看org/jpos/q2/reversion.properties内容如下：

<pre>
	 ---------------------------------------------
	    #Revision Properties
        #Tue Dec 24 12:34:39 UYST 2013
        revision=3dfc9d5
        branch=master
     ---------------------------------------------
</pre>
     
serversimulator 1.0.0 master/1026057 (2014-01-09 16:40:52 CST)
这一行打印的是serversimulator\build\install\serversimulator\serversimulator-1.0.0.jar里面的buildinfo.properties和reversion.properties的内容. 

buildinfo.properties内容如下：  
<pre>
	  ---------------------------------------------
	    #Revision Properties
        #Thu Jan 09 16:40:53 CST 2014
        version=1.0.0
        projectName=serversimulator
        buildTimestamp=2014-01-09 16\:40\:52 CST
      ---------------------------------------------
</pre>


reversion.properties内容如下：  
<pre>
      ---------------------------------------------	 
	    #Revision Properties
        #Thu Jan 09 16:32:58 CST 2014
        revision=1026057
        branch=master
      ---------------------------------------------    
</pre>    
	
###2.开始PGP标记信息,hash:SHA1
###3.开始PGP认证
###4.结束PGP认证
	
2.3.4打印的一大串内容，实际上是读取jpos-1.9.4.jar里面LICENSEE.asc,我们来看Q2源代码：

```java
	 public static final String LICENSEE            = "/LICENSEE.asc";
	  以下这段代码中调用的getLincense()方法即是：
	  public static String getVersionString() {
        String appVersionString = getAppVersionString();
        if (appVersionString != null) {
            return String.format ("jPOS %s %s/%s (%s)%n%s%s",
                getVersion(), getBranch(), getRevision(), getBuildTimestamp(), appVersionString, getLicensee()
            );
        }
        return String.format ("jPOS %s %s/%s (%s)%s",
            getVersion(), getBranch(), getRevision(), getBuildTimestamp(), getLicensee()
        );
    }
```
    
jPOS Community Edition, licensed under GNU AGPL v3.0.
This software is probably not suitable for commercial use.
Please see http://jpos.org/license for details.

jPOS社区版本，在GUN通用许可证3.0下面认证。
【GNU Affero 通用公共许可证 (GNU AGPL，gun  General Public License) 基于 GNU GPL，但它添加了一些条款以允许用户获取那些通过网络访问的应用的源代码。
我们建议考虑对于那些通过网络被他人使用的软件采用GNU AGPL。GNU AGPL 的最新版本是3】
   这个软件不适用于商业用途。
	 
参见：https://github.com/jpos/jPOS/blob/v1_9_4/jpos/src/main/resources/LICENSEE.asc
	

###5.realm 寻找各种deploy下各种XML配置文件
Q2源码run方法内下面调用deploy ();方法调用了scan(),scan()内部扫描deploy目录，
然后注册每一个文件调用register(File)方法,并调用Object obj = factory.instantiate (this, rootElement);将class属性值出去来，初始化  
```java
	   String clazz  = e.getAttributeValue ("class");
        if (clazz == null) {
            try {
                clazz = classMapping.getString (e.getName());
            } catch (MissingResourceException ex) {
                // no class attribute, no mapping
                // let MBeanServer do the yelling
            }
        }
        MBeanServer mserver = server.getMBeanServer();
        getExtraPath (server.getLoader (), e);
        return mserver.instantiate (clazz, loaderName);
	
```

deploy方法最后，遍历fileMap,找到每一个文件中涉及到的QBEAN,然后依次通过反射调用其start方法.
```java
        for (ObjectInstance instance : startList)
            start(instance);
    	
    	 private void start (ObjectInstance instance) {
            try {
                factory.startQBean (this, instance.getObjectName());
            } catch (Exception e) {
                getLog().warn ("start", e);
            }
        }
        
        public void startQBean (Q2 server, ObjectName objectName)
        {
            MBeanServer mserver = server.getMBeanServer();
            mserver.invoke (objectName, "start",  null, null);
        }
        
                继承与QBeanSupport的所有QBean默认都继承了父类的start()方法,并且在子类中一般不做覆盖（子类一般延迟实现startService()方法），父类的start()方法如下：
        public synchronized void start() {
            if (state != QBean.DESTROYED && 
                state != QBean.STOPPED   && 
                state != QBean.FAILED)
               return;
    
            this.state = QBean.STARTING;
    
            try {
               startService();
            } catch (Throwable t) {
               state = QBean.FAILED;
               log.warn ("start", t);
               return;
            }
            state = QBean.STARTED;
        }
      
```                  
                
	
--为什么serversimulator\build\install\serversimulator\deploy\99_sysmon.xml这个能够启动Sysmonitor类：
```xml
        	<sysmon logger="Q2">
             <!-- Environment: 'devel' -->
             <attr name="sleepTime" type="java.lang.Long">3600000</attr>
             <attr name="detailRequired" type="java.lang.Boolean">true</attr>
            </sysmon>
```
            
--解释开始-------
QFactory源码构造方法有这么一段：
```java
    ResourceBundle classMapping;
    ConfigurationFactory defaultConfigurationFactory = new SimpleConfigurationFactory();

    public QFactory (ObjectName loaderName, Q2 q2) {
        super ();
        this.loaderName = loaderName;
        this.q2  = q2;
        try {
            classMapping = ResourceBundle.getBundle(this.getClass().getName());
        } catch (MissingResourceException ignored) { }
    }
```

由这个classMapping = ResourceBundle.getBundle(this.getClass().getName());我们得知：

classMapping = ResourceBundle.getBundle("QFactory");也就是读取QFactory.properties.


QFactory.properties（org/jpos/q2/QFactory.properties）内容如下：  
```properties
logger=org.jpos.q2.qbean.LoggerAdaptor
shutdown=org.jpos.q2.qbean.Shutdown
script=org.jpos.q2.qbean.BSH
jython=org.jpos.q2.qbean.Jython
spacelet=org.jpos.q2.qbean.SpaceLet
sysmon=org.jpos.q2.qbean.SystemMonitor
txnmgr=org.jpos.transaction.TransactionManager
transaction-manager=org.jpos.transaction.TransactionManager
qmux=org.jpos.q2.iso.QMUX
channel-adaptor=org.jpos.q2.iso.ChannelAdaptor
qexec=org.jpos.q2.qbean.QExec
```
  
而QFactory类有如下这个方法：
```java
      public Object instantiate (Q2 server, Element e) 
        throws ReflectionException,
               MBeanException,
               InstanceNotFoundException
    {
        String clazz  = e.getAttributeValue ("class");
        if (clazz == null) {
            try {
                clazz = classMapping.getString (e.getName());
            } catch (MissingResourceException ex) {
                // no class attribute, no mapping
                // let MBeanServer do the yelling
            }
        }
        MBeanServer mserver = server.getMBeanServer();
        getExtraPath (server.getLoader (), e);
        return mserver.instantiate (clazz, loaderName);
    } 
```
 
由上得知clazz为空就调用clazz = classMapping.getString (e.getName());这个e.getName就是标签名，所以sysmon得以解析  
--解释结束-------
	
	
	
由于jPOS-EE配置部署了 SystemMonitor QBean，看这个serversimulator\build\install\serversimulator\deploy\99_sysmon.xml.

又因为SystemMonitor是一个标准QBean,所以SystemMonitor会启动，jPOS-EE的Q2启动日志中可以看到

99_sysmon.xml如下：
```xml
	 <sysmon logger="Q2">
		- <!--  Environment: 'devel' 
	 	 --> 
	  	<attr name="sleepTime" type="java.lang.Long">3600000</attr> 
	 	<attr name="detailRequired" type="java.lang.Boolean">true</attr> 
	  </sysmon>
```


可以看到每次执行完SystemMonitor任务输出后，会休息60*60*1000毫秒，也就是一个小时执行输出一次系统信息.默认也是这么多的。  

每次进行系统信息输出时都要进行PGP信息认证。  
	
###6.结束PGP标记信息
```java
private MBeanServer server = MBeanServerFactory.createMBeanServer ("q2");
final ObjectName loaderName = new ObjectName (Q2_CLASS_LOADER);
QFactory factory = new QFactory (loaderName, this);
```
    --------------------------------------------------------------------------------------
    
以上主要分析了Q2是如何启动各个QBean的，真正的服务器端主角类QServer类分析开始：
QServer类启动顺序大致分为如下几个步骤(根据QBean生命周期先调用initService，再调用startService)：

0.调用初始化方法
```java
       public void initService() throws ConfigurationException {
	        Element e = getPersist ();
	        LocalSpace sp   = grabSpace (e.getChild ("space"));    --sp是全局变量
	    }
    
        public void startService () {
            try {
                initServer ();
                initIn();
                initOut();
                initWhoToSendTo();
            } catch (Exception e) {
                getLog().warn ("error starting service", e);
            }
        }
```
 
1.初始化服务器信息  
2.初始化请求进来的信息  
3.初始化发送出去的信息  
4.初始化谁发给谁的信息  
    
####1.初始化服务器信息................................................
包括如下几件事情
#####1.1）初始化全局ISOChannel
```java
    ChannelAdaptor adaptor = new ChannelAdaptor ();
    ISOChannel channel = adaptor.newChannel (e, getFactory ());   --- channel是全局类变量]
```
        
#####1.2）创建一个初始值为1，100的线程池（1表示初始化池大小，100表示池最大的大小）：
```java
       ThreadPool pool  = new ThreadPool (minSessions ,maxSessions);
```
ThreadPool这个类是jpos内带的线程类
               
#####1.3）通过channel和线程池创建全局ISOServer，监听在一个端口
```java
	 	ISOServer server = new ISOServer (port, (ServerChannel) channel, pool);
	        server.setLogger (log.getLogger(), getName() + ".server");
	        server.setName (getName ());        
	        if (socketFactoryString != null) {
	            ISOServerSocketFactory sFac = (ISOServerSocketFactory) getFactory().newInstance(socketFactoryString);
	            if (sFac != null && sFac instanceof LogSource) {
	                ((LogSource) sFac).setLogger(log.getLogger(),getName() + ".socket-factory");
	            }
	            server.setSocketFactory(sFac);
	        }
```
	        
我们来看下ISOServer的构造方法：
```java
	         public ISOServer(int port, ServerChannel clientSide, ThreadPool pool) {
		        super();
		        this.port = port;
		        this.clientSideChannel = clientSide;
		        this.clientPackager = clientSide.getPackager();
		        if (clientSide instanceof FilteredChannel) {
		            FilteredChannel fc = (FilteredChannel) clientSide;
		            this.clientOutgoingFilters = fc.getOutgoingFilters();
		            this.clientIncomingFilters = fc.getIncomingFilters();
		        }
		        this.pool = (pool == null) ?new ThreadPool (1, DEFAULT_MAX_THREADS) : pool;
		        listeners = new Vector();
		        name = "";
		        channels = new HashMap();
		        cnt = new int[SIZEOF_CNT];
		        serverListeners = new ArrayList<ISOServerEventListener>();
		}
```
    
#####1.4）设置工厂配置
```java
    		getFactory().setConfiguration (server, getPersist());
```

#####1.5）添加ServerSocketFactory
```java
	   	QFactory factory = getFactory ();
	        Element persist = getPersist ();
	        Element serverSocketFactoryElement = persist.getChild ("server-socket-factory");
	        if (serverSocketFactoryElement != null) {
	            ISOServerSocketFactory serverSocketFactory = (ISOServerSocketFactory) factory.newInstance (serverSocketFactoryElement.getAttributeValue ("class"));
	            factory.setLogger        (serverSocketFactory, serverSocketFactoryElement);
	            factory.setConfiguration (serverSocketFactory, serverSocketFactoryElement);
	            server.setSocketFactory(serverSocketFactory);
	        }
```
	
#####1.6）添加监听器ISORequestListener
```java
	   	QFactory factory = getFactory ();
	        Iterator iter = getPersist().getChildren ("request-listener").iterator();
	        while (iter.hasNext()) {
	            Element l = (Element) iter.next();
	            ISORequestListener listener = (ISORequestListener)factory.newInstance (l.getAttributeValue ("class"));
	            factory.setLogger(listener, l);
	            factory.setConfiguration (listener, l);
	            server.addISORequestListener (listener);
	        }        
```
	        
######1.7）添加ISOServerConnectionListeners
```java
		  QFactory factory = getFactory ();
	          Iterator iter = getPersist().getChildren ("connection-listener").iterator();
	          while (iter.hasNext()) {
	            Element l = (Element) iter.next();
	            ISOServerEventListener listener = (ISOServerEventListener)
	                factory.newInstance (l.getAttributeValue ("class"));
	            factory.setLogger        (listener, l);
	            factory.setConfiguration (listener, l);
	            server.addServerEventListener(listener);
	         }
```
	      
#####1.8）NameRegistrar注册QServer
```java
	       NameRegistrar.register (getName(), this);
```
	        
#####1.9）启动ISOServer,由于ISOServer本身就是一个Runnable线程，所以当线程启动
```java
	 	new Thread (server).start();
```

我们看下ISOServer的run()方法的第一段关键代码如下
```java
			       ServerChannel  channel;
			        if (socketFactory == null) {
			            socketFactory = this;
			        }
			        serverLoop : while  (!shutdown) {
			            try {
			                serverSocket = socketFactory.createServerSocket(port);
			
			                Logger.log (new LogEvent (this, "iso-server",
			                    "listening on " + (bindAddr != null ? bindAddr + ":" : "port ") + port
			                    + (backlog > 0 ? " backlog="+backlog : "")
			                ));
			                while (!shutdown) { //如果没有shutdown
			                    try {
			                   	 //如果线程池没有可用的
			                        if (pool.getAvailableCount() <= 0) {
			                            try {
			                                serverSocket.close();
			                                fireEvent(new ISOServerShutdownEvent(this));
			                            } catch (IOException e){
			                                Logger.log (new LogEvent (this, "iso-server", e));
			                                relax();
			                            }
			
			                            for (int i=0; pool.getIdleCount() == 0; i++) {
			                                if (shutdown) {
			                                    break serverLoop;
			                                }
			                                if (i % 240 == 0 && cfg.getBoolean("pool-exhaustion-warning", true)) {
			                                    LogEvent evt = new LogEvent (this, "warn");
			                                    evt.addMessage (
			                                        "pool exhausted " + serverSocket.toString()
			                                    );
			                                    evt.addMessage (pool);
			                                    Logger.log (evt);
			                                }
			                                ISOUtil.sleep (250);
			                            }
			                            serverSocket = socketFactory.createServerSocket(port);
			                        }
			                        
			                        //从客户端克隆一个channel
			                        channel = (ServerChannel) clientSideChannel.clone();
			                        //channel接收serversocket
			                        channel.accept (serverSocket);
			
			                        if ((cnt[CONNECT]++) % 100 == 0) {
			                            purgeChannels ();
			                        }
			                        WeakReference wr = new WeakReference (channel);
			                        //channels是一个MAP,Key是channel的名称,wr
			                        channels.put (channel.getName(), wr);
			                        channels.put (LAST, wr);
			                        //通过channel创建会话
			                        pool.execute (createSession(channel));
			                        setChanged ();
			                        notifyObservers (this);
			                        fireEvent(new ISOServerAcceptEvent(this));
			                        if (channel instanceof Observable) {
			                            ((Observable)channel).addObserver (this);
			                        }
			                    } catch (SocketException e) {
			                        if (!shutdown) {
			                            Logger.log (new LogEvent (this, "iso-server", e));
			                            relax();
			                            continue serverLoop;
			                        }
			                    } catch (IOException e) {
			                        Logger.log (new LogEvent (this, "iso-server", e));
			                        relax();
			                    }
			                }
			            } catch (Throwable e) {
			                Logger.log (new LogEvent (this, "iso-server", e));
			                relax();
			            }	 	
	 	
 	//由以上run方法得知首先创建一个serversocket监听在一个端口上，比如下：
	 	public ServerSocket createServerSocket(int port) throws IOException {
		        ServerSocket ss = new ServerSocket();
		        try {
		            ss.setReuseAddress(true);
		            ss.bind(new InetSocketAddress(bindAddr, port), backlog);
		        } catch(SecurityException e) {
		            ss.close();
		            fireEvent(new ISOServerShutdownEvent(this));
		            throw e;
		        } catch(IOException e) {
		            ss.close();
		            fireEvent(new ISOServerShutdownEvent(this));
		            throw e;
		        }
		        return ss;
		    }
```
	 	
创建会话：
```java
	 	String realm;
	        protected Session(ServerChannel channel) {
	            this.channel = channel;
	            realm = ISOServer.this.getRealm() + ".session";
	        }

	 	//线程池执行会话，会将会话放到线程池类的内部队列中区，线程池类的内部类run方法会不停的从队列中取值.调用它的run方法（因为会话本身就是一个线程）
	 	  pool.execute (createSession(channel));
	 	//通知观察者
	 	notifyObservers
	 	//添加观察者
	 	if (channel instanceof Observable) {
                            ((Observable)channel).addObserver (this);
                 }
	 	
	 	
```


我们来看ISOServer内部类Session会话类的run()方法第一个重要片段如下：
```java
	 	    setChanged ();
	            notifyObservers ();    ---利用观察者模式了
	            if (channel instanceof BaseChannel) {
	                LogEvent ev = new LogEvent (this, "session-start");
	                Socket socket = ((BaseChannel)channel).getSocket ();
	                realm = realm + "/" + socket.getInetAddress().getHostAddress() + ":"
	                        + socket.getPort();
	                try {
	                    //检查权限
	                    checkPermission (socket, ev);
	                } catch (ISOException e) {
	                    try {
	                        int delay = 1000 + new Random().nextInt (4000);
	                        ev.addMessage (e.getMessage());
	                        ev.addMessage ("delay=" + delay);
	                        ISOUtil.sleep (delay);
	                        socket.close ();
	                        fireEvent(new ISOServerShutdownEvent(ISOServer.this));
	                    } catch (IOException ioe) {
	                        ev.addMessage (ioe);
	                    }
	                    return;
	                } finally {
	                    Logger.log (ev);
	                }
	            }
```
	 	
我们来看下ISOServer内部类Session会话类的run()方法中关键的第二段代码如下：
```java
		 for (;;) {
	                    try {
	                        ISOMsg m = channel.receive();
	                        lastTxn = System.currentTimeMillis();
	                        Iterator iter = listeners.iterator();
	                        while (iter.hasNext()) {
	                            if (((ISORequestListener)iter.next()).process
	                                (channel, m)) {
	                                break;
	                            }
	                        }
	                    }
	                    catch (ISOFilter.VetoException e) {
	                        Logger.log (new LogEvent (this, "VetoException", e.getMessage()));
	                    }
	                    catch (ISOException e) {
	                        if (ignoreISOExceptions) {
	                            Logger.log (new LogEvent (this, "ISOException", e.getMessage()));
	                        }
	                        else {
	                            throw e;
	                        }
	      }
```

由上可知道，for(;;)相当于while(true)表示永远循环的执行下去，从循环内容看，所做的事情大致包括如下几件：  

通过channel(也就是XMLChannel)接收客户端请求发过来的ISOMsg，遍历监听器们，调用每个监听器的process方法(通过传入channel和ISOMsg参数)  

BSHRequestListener的process方法主要是做这么一件事情，读取beanshell脚本的代码当做执行：  
```java
    	  public boolean process (ISOSource source, ISOMsg m) {
            try{
                String mti = m.getMTI ();
                if (!whitelist.contains(mti) && !whitelist.contains("*"))
                    mti = "unsupported";
                for (String aBshSource : bshSource) {
                    try {
                        Interpreter bsh = new Interpreter();
                        bsh.set("source", source);
                        bsh.set("message", m);
                        bsh.set("log", this);
                        bsh.set("cfg", cfg);
    
                        int idx = aBshSource.indexOf(MTI_MACRO);
                        String script;
    
                        if (idx >= 0) {
                            // replace $mti with the actual value in script file name
                            script = aBshSource.substring(0, idx) + mti +
                                    aBshSource.substring(idx + MTI_MACRO.length());
                        } else {
                            script = aBshSource;
                        }
    
                        bsh.source(script);
                    } catch (Exception e) {
                        warn(e);
                    }
                }
            }catch (Exception e){
                warn(e);
                return false;
            }
            return true;
        }
```

如上Interpreter换成serversimulator\build\install\serversimulator\cfg\serversimulator.bsh的内容如下：
```java
      public boolean process (ISOSource source, ISOMsg message) {
            try{
                String mti = message.getMTI ();
                if (!whitelist.contains(mti) && !whitelist.contains("*"))
                    mti = "unsupported";
                message.setResponseMTI ();
                
                Random random = new Random (System.currentTimeMillis());
                message.set (37, Integer.toString(Math.abs(random.nextInt()) % 1000000));
                message.set (38, Integer.toString(Math.abs(random.nextInt()) % 1000000));
                
                if ("000000009999".equals (message.getString (4)))
                    message.set (39, "01");
                else
                    message.set (39, "00");
                
                source.send (message);
            }catch (Exception e){
                warn(e);
                return false;
            }
            return true;
        }        
```
	  
由上得知设置了域37、域38、域39后，调用XMLChannel的send返回数据到客户端.
	    
```java
byte[] b = m.pack();    --- 将ISOMsg打包成字节数组
synchronized (serverOutLock) {
    sendMessageLength(b.length + getHeaderLength(m)); //发送信息长度=信息长度+头长度
    sendMessageHeader(m, b.length); //发送信息头
    sendMessage (b, 0, b.length); //发送信息
    sendMessageTrailler(m, b); //发送尾巴
    serverOut.flush ();
}
```
	    
		 	 	
####2.初始化请求进来的信息.................................................
```java
     	Element persist = getPersist();
        String  inQueue = persist.getChildText("in");   -- 全局变量inQueue
        if (inQueue != null) {
            /*
             * We have an 'in' queue to monitor for messages we will
             *  send out through server in our (SpaceListener)notify(Object, Object) method.
             */
            sp.addListener(inQueue, this);
        }
```
        
上面这段话的注释说的是：我们有一个名字叫做"in"的队列来坚挺我们将要通过server的方法nofity（）发出去的信息，
        
```java
     //这个方法通过我们注册一次的SpaceListener接口调用，我们注意到我们有一个"in"名称的队列
    @Override
    public void notify(Object key, Object value) {
        Object obj = sp.inp(key);
        if (obj instanceof ISOMsg) {
            ISOMsg m = (ISOMsg) obj;
            if ("LAST".equals(sendMethod)) {
                try {
                    ISOChannel c = server.getLastConnectedISOChannel();
                    if (c == null) {
                        throw new ISOException("Server has no active connections");
                    }
                    if (!c.isConnected()) {
                        throw new ISOException("Client disconnected");
                    }
                    c.send(m);
                }
                catch (Exception e) {
                    getLog().warn("notify", e);
                }
            }
            else if ("ALL".equals(sendMethod)) {
                String channelNames = getISOChannelNames();
                if (channelNames!=null) {
                    StringTokenizer tok =new StringTokenizer(channelNames, " ");
                    while (tok.hasMoreTokens()) {
                        try {
                            ISOChannel c = server.getISOChannel(tok.nextToken());
                            if (c == null) {
                                throw new ISOException("Server has no active connections");
                            }
                            if (!c.isConnected()) {
                                throw new ISOException("Client disconnected");
                            }
                            c.send(m);
                        }
                        catch (Exception e) {
                            getLog().warn("notify", e);
                        }

                    }
                }
            }
        }
    }        
```      
        
   
####3.初始化发送出去的信息....................................................
```java
 		Element persist = getPersist();
         String outQueue = persist.getChildText("out");    -- 全局变量outQueue
        if (outQueue != null) {
            /*
             * We have an 'out' queue to send any messages to that are received
             * by the our requestListner(this).
             *
             * Note, if additional ISORequestListeners are registered with the server after
             *  this point, then they won't see anything as our process(ISOSource, ISOMsg)
             *  always return true.
             */
           server.addISORequestListener(this);                 -- ISOServer添加ISORquestListener(this),this指代当前QServer实例
        }
        server.addISORequestListener(this);  这个方法内部实现我们看到
        Collection listeners；
        public void addISORequestListener(ISORequestListener l) {
        	listeners.add (l);  //由于listeners是个集合，所以可以添加多个ISORequestListener
        }

        
        
        以上注释的含义是：
       我们有一个"out"名称的队列用来发送任何信息到被我们的requestListner(this)方法接收的这一方。
       注意，如果附加的ISORequestListeners通过sever在这之后被注册,我们将看不到任何信息，因为我们的process方法总是返回true
       
	     //这个方法通过ISORequestListener接口被调用，如果QServer有一个"out"名称的队列去处理
	    @Override
	    public boolean process(ISOSource source, ISOMsg m) {
	        sp.out(outQueue, m);
	        return true;
	    }
```     
       
        
        
####4.初始化谁发给谁的信息...............................................
```java
        Element persist = getPersist();
        String sendMethod = persist.getChildText("send-request");   ......sendMethod是全局变量
        if (sendMethod==null) {
            sendMethod="LAST";
        }
```

附加：
```java
    //停止服务，ISOServer
    @Override
    public void stopService () {
        if (server != null) {
            server.shutdown ();
            sp.removeListener(inQueue, this);
        }
    }
    ISOServer里的shutdown方法如下：
	  public void shutdown () {
	        shutdown = true;
	        new Thread ("ISOServer-shutdown") {
	            @Override
	            public void run () {
	                shutdownServer ();
	                if (!cfg.getBoolean ("keep-channels")) {
	                    shutdownChannels ();
	                }
	            }
	        }.start();
	    }
	    
	    private void shutdownServer () {
	        try {
	            if (serverSocket != null) {
	                serverSocket.close ();
	                fireEvent(new ISOServerShutdownEvent(this));
	            }
	            if (pool != null) {
	                pool.close();
	            }
	        } catch (IOException e) {
	            fireEvent(new ISOServerShutdownEvent(this));
	            Logger.log (new LogEvent (this, "shutdown", e));
	        }
	    }
	    
	    
	    private void shutdownChannels () {
	        Iterator iter = channels.entrySet().iterator();
	        while (iter.hasNext()) {
	            Map.Entry entry = (Map.Entry) iter.next();
	            WeakReference ref = (WeakReference) entry.getValue();
	            ISOChannel c = (ISOChannel) ref.get ();
	            if (c != null) {
	                try {
	                    c.disconnect ();
	                    fireEvent(new ISOServerClientDisconnectEvent(this));
	                } catch (IOException e) {
	                    Logger.log (new LogEvent (this, "shutdown", e));
	                }
	            }
	        }
	    }
    
    
    //销毁服务,从NameRegistrar中取消注册
    @Override
    public void destroyService () {
        NameRegistrar.unregister (getName());
        NameRegistrar.unregister ("server." + getName());
    }
```java


###2.定义：Client Simulator可以用来开启一组单元测试通过ISO-8583服务器。这些组被定义成一个xml文件的集合

代表着将要发送的信息和他们期望的响应。
	
	
<pre>
	What: 参见如上定义
	When: 使用于jPOS-EE所有版本.
	Who: jPOS.org项目组.
	How: jPOS-EE项目组发布.
	Where: jPOS-EE在 google code上svn的主干目录下opt/clientsimulator
	Why:当需要写一个基于 ISO-8583的服务器端程序时，它有能力模拟一个非常实用的客户端。我们在jPOS.org把它定位很高级来测试我们的程序
	Status: 稳定的.
	Dependencies: 基于jpos模块.
	License: GNU Affero General Public License version 3 The Client Simulator can
	be used to fire a suite of unit tests against an ISO-8583 server. The suite is
	defined by a set of XML files representing messages to be sent and their
	expected responses
	
</pre>

	
为了模拟复杂的ISO-8583交易，client simulator应用了BSH脚本在运行时支持自定义 ISO-8583域的内容。这可以用来指定内容的值，比如终端ID，店家ID,卡号，或者一些动态的值比如跟踪数字，回复数字引用， pinblocks, key exchange等等。

	
让我们看一下模拟程序的QBean配置文件（D:\Java\workspace\jPOS-EE\jPOS-EE\modules\client-simulator\src\main\resources\META-INF\q2\installs\deploy\30_clientsimulator.xml）：
```xml
	<qbean name="clientSimulator" logger="Q2" realm="client-simulator"
	class="org.jpos.simulator.TestRunner">
	<property name="mux" value="clientsimulator-mux" />
	<property name="timeout" value="30000" />
	<property name="sessions" value="1" />
```
	
TestRunner在github上jPOS-EE下载代码中：jpos\jPOS-EE-2.0.0\jPOS-EE-2.0.0\modules\client-simulator\src\main\java\org\jpos\simulator\TestRunner.java

对照源码TestRunner类：  

源码一：  

```java
	 protected void startService() {
        for (int i=0; i<cfg.getInt("sessions", 1); i++)
            new Thread(this).start();
     }
````
这段代码有一个cfg.getInt("sessions")正好对应<property name="sessions" value="1" />，也就是启动了一个线程
     

因为clientsimulator\build\install\clientsimulator\deploy\20_clientsimulator_mux.xml定义如下：
```xml
	<mux class="org.jpos.q2.iso.QMUX" logger="Q2" name="clientsimulator-mux">
        <in>clientsimulator-receive</in>
        <out>clientsimulator-send</out>
        <unhandled>clientsimulator-unhandled</unhandled>
     </mux>
```


得知q2要启动QMUX,QMUX类startService方法有如下这么一段：  
```java
     public void initService () throws ConfigurationException {
        ......
        NameRegistrar.register ("mux."+getName (), this);
    }
````
    
所以TestRunner源码类中run方法里面的这么一段代码可以好解释：
```java
Interpreter bsh = initBSH ();
mux = (MUX) NameRegistrar.get ("mux." + cfg.get ("mux"));     --- 这个启动QMumx类的时候已经注册了
List suite = initSuite (getPersist().getChild ("test-suite"));   --- test-suite是30_clientsimulator.xml中定义的
runSuite (suite, mux, bsh);        
```kava
        
注意】   
NameRegistrar这个类为了注册jPOS组件  
NameRegistrar.register ("mux."+name, this);   前缀"mux."为了区分不同的类，比如"mux.institutionABC"and"channel.institutionABC"
     
--关于qmux--  
JPOS是实现作为一个金融外汇付款的iso-8583框架协议。isorequestlistener，这个接口是用来建立我们自己的监听器能够与一个多路复用器qmux整合，监听器就会听到所有要求请求和传出的交易和处理，根据需要，在我们的例子中，一个多路复用和发送等待响应，我们可以控制的错误或自定义的回复消息根据我们的需求。
	
--在java程序中调用beanshell 开始--  
可以执行文本然后运行脚本在你的程序中通过创建BeanShell的interpreter实例，然后使用eval()和source()命令。你可以通过使用set()方法在脚本中定义你需要用到的的对象引用，然后通过get()方法取回值。
````java
	
	import bsh.Interpreter;
    Interpreter i = new Interpreter();  // Construct an interpreter
    i.set("foo", 5);                    // Set variables
    i.set("date", new Date() ); 
    
    Date date = (Date)i.get("date");    // retrieve a variable
    // Eval a statement and get the result
    i.eval("bar = foo*10");             
    System.out.println( i.get("bar") );
    
    // Source an external script file
    i.source("somefile.bsh");
```
--在java程序中调用beanshell  结束--	


我们指定了一个mux(QMUX运行在同一个JVM上的名字)和一个响应超时时间。然后我们定义一个初始化的区块：
```java
	<init>
	import org.jpos.space.*;
	int cnt = 1;
	String terminal = "29110001";
	String merchant = "000000001001";
	String pinblk = "0123456789ABCDEF";
	Space sp = SpaceFactory.getSpace();
	</init>
```
	
这个初始化的锁基于BSH脚本，你可以在那些什么都行，比如定义一些可能被后来用到的变量，jPOS对象的引用(比如空间实例，安全模块等等)
	
	
再来看看测试程序，以下这段也是定义在如上xml文件中最下面：
```java
	<test-suite>
	<path>cfg/</path>
	<test file="echo" count="10" continue="yes" name="Simple Echo Test" />
	<test file="echo" count="20" continue="yes" name="Simple Echo Test 2">
	<init>
	// optional init script
	// the variable 'testcase'references _this_ testcase instance
	// the variable 'request' references the ISOMsg that is to be sent
	</init>
	<post>
	// optional post script
	// the variable 'testcase' references _this_ testcase instance
	// the variable 'response' references the received message
	</post>
	</test>
	<path>cfg/anotherpath</path>
	<test file="mytest">MyTest</test>
	...
	...
	</test-suite>
	</qbean>
```
	   
这个程序可以分开在不同的路径，在之前的例子中，我们假设有文件叫做：cfg/echo_s and cfg/echo_r  

cfg/echo_s中的s字母表示发送(也就是请求到服务器端的数据)，cfg/echo_r中的r字母表示接收(也就是期望返回的数据)
	
D:\Java\workspace\jPOS-EE\jPOS-EE\modules\client-simulator\src\main\resources\META-INF\q2\installs\cfg\echo_s:  
```xml
	<isomsg>
	<field id="0" value="1800" />
	<field id="7" value="1025080500" />
	<field id="11" value="000001" />
	<field id="41" value="29110001" />
	</isomsg>
```
	
	
D:\Java\workspace\jPOS-EE\jPOS-EE\modules\client-simulator\src\main\resources\META-INF\q2\installs\cfg\echo_r:
```xml
	<isomsg>
	<field id="0" value="1810" />
	<field id="39" value="00" />
	</isomsg>
```
	
    
TestRunner类源码中，针对如上这个test-suite进行了解析由于ISOPackager packager = new XMLPackager();并且读取如上两个文件echo_s和echo_r，他们的内容正好是XML，所以packager.unpack()可以解析XML文件的内容
    

由如下这段得知：

tc.setRequest (getMessage (prefix + path + "_s"));   ---- echo_s文件内容是请求信息

tc.setExpectedResponse (getMessage (prefix + path + "_r")); -- echo_r文件内容是期望响应信息
	
	
之前的例子中，我们发送一个1800消息和一些混合的数据，我们期望接收一个1810的消息，在域39的内容带有00.用混合内容可能会OK对于大多数的测试用例来说，有很多情况，你可能会想到用动态的内容  

我们的模拟程序支持BSH脚本在域的级别上。凡是开始带上!字符的我们认为是一个脚本，可以这样写：  
```xml
	<isomsg>
	<field id="0" value="1800" />
	<field id="7" value="ISODate.getANSIDate (new Date())" />
	<field id="11" value="! System.currentTimeMillis() % 1000000" />
	<field id="41" value="! terminal" />
	<field id="52" value="@ pinblk" />
	</isomsg>
```
	
请注意在我们的例子中终端是一个运行脚本变量我们已经定义在我们的区块中了。@字符和字符!操作了同样的方式，结果值建议处理成一个hexadecimal(十六进制)字符串，  

ISOUtil.hex2byte(String)用这个方法转换成byte[]为了产生一个ISOBinaryField.  
	
在接收的时候发生了同样的事情，当我们试图去模拟voids,reversals，我们经常需要之前的交易事务中信息，比如数字引用的回复，审查数字等。  
因此我们可以保存那些信息为了方便后来用 receive-time脚本使用：  
```xml
	<isomsg>
	    <field id="0" value="1810" />
	    <field id="11" value="! previousTrace=value" />
	    <field id="37" value="! rrn=value" />
	    <field id="39" value="00" />
	</isomsg>
```

有一个特殊的变量名叫做value，我们可以把接收的内容放进去，所以在之前的例子中，接收的回复数字引用（域37），保存在变量命名为rrn为了后来使用

接收的脚本可能可选择的返回true或者false,因此我们可以像以下这么写：
```xml
	<isomsg>
	    <field id='39' value='! return value.equals(EXPECTED_RETVALUE)' />
	</isomsg>
```
	
这里面的EXPECTED_RETVALUE就是定义在之前的初始化区块中的。
事实上，之前的例子等同于下面的:
```xml
	<isomsg>
	<field id='39' value='! EXPECTED_RETVALUE' />
	</isomsg>
```

EXPECTED_RETVALUE的字符串值已经用了（除非它是一个boolean值）

有一个特殊的字符串*E用来测试输出echo.为了确定接收一个域的内容和我们发送的相同，我们可以这么写：
```xml
	<isomsg>
	<field id='4' value='*E' />
	</isomsg>
```
	
注意：特殊字符串M*可以用来检查mandatory field presence，不会检查它的内容。
同样的，E*可以用来检查 mandatory echo，*O可以用来检查optional echo.
	
	
测试用例的支持计数属性可用于开启相同测试N次。  

它也支持一个continue属性，如果continue="yes"，那么test runner可能记录一个异常当发生错误的时候，然后它会继续记下一个测试。  
默认的超时时间是60秒，但是我们可以指定不同的超时时间，用timeout属性在每一个测试用例的元素中  
	
最后，你获取到了一个测试结果：  
```xml
	<log realm="org.jpos.simulator.TestRunner" at="Mon Oct 25 13:46:13 UYT 2004.581">
	<results>
	Simple Echo Test [OK] 58ms.
	Simple Echo Test [OK] 38ms.
	Simple Echo Test [OK] 70ms.
	Simple Echo Test [OK] 23ms.
	Simple Echo Test [OK] 56ms.
	Simple Echo Test [OK] 24ms.
	Simple Echo Test [OK] 73ms.
	Simple Echo Test [OK] 107ms.
	Simple Echo Test [OK] 20ms.
	Simple Echo Test [OK] 50ms.
	Simple Echo Test [OK] 23ms.
	Simple Echo Test [OK] 24ms.
	Simple Echo Test [OK] 86ms.
	Simple Echo Test [OK] 24ms.
	Simple Echo Test [OK] 24ms.
	Simple Echo Test [OK] 23ms.
	Simple Echo Test [OK] 26ms.
	Simple Echo Test [OK] 21ms.
	Simple Echo Test [OK] 22ms.
	Simple Echo Test [OK] 79ms.
	Simple Echo Test 2 [OK] 22ms.
	elapsed server=893ms(62%), simulator=526ms(37%), total=1419ms
	</results>
	</log>
```
	
	
【我们来看执行client端q2的流程】  
1.查找deploy目录  
2.开始PGP标记信息,hash:SHA1  
3.开始PGP认证  
4.结束PGP认证  
5.realm 寻找各种deploy下各种XML配置文件  
	

客户端和服务器端Q2流程基本一致，客户主要看TestRunner这个类的流程：
1）从XML获取sessions的个数，有多少个sessions，启动多少个线程.  

2）初始化XML中的init区块中的beanshell代码：  
```java
	    import java.util.Date;
	    import org.jpos.iso.ISODate;
	    int cnt = 1;
	    String terminal = "29110001";
	    String previousTrace = "000000";
	
	    String get_date() {
	        return ISODate.getDateTime(new Date());
	    }
	    String get_date (String format) {
	        return ISODate.formatDate (new Date(), format);
	    }
```

3）NameRegistrar获取mux这个已经注册的类MUX mux = org.jpos.q2.iso.QMUX  --全局变量  

4）组装测试代码块（通过如下可以读取echo_s和echo_r的ISOMsg信息）：  
```xml
	<test-suite>
	   <path>cfg/</path>
	   <test file="echo" count="20" continue="yes" name="Simple Echo Test">
	    <init>// print ("Init Script");</init>
	    <post>// print ("Post Script");</post>
	   </test>
	   <test file="echo" count="1" continue="yes" name="Simple Echo Test 2" />
	  </test-suite>	
```
	
因为<test>标签有两个，所以就等于测试两次
	
循环TestCase列表，调用mux全局变量的request方法进行请求消息，这个方法返回值就是响应值，如下：
```java
	 public ISOMsg request (ISOMsg m, long timeout) throws ISOException {
	        String key = getKey (m);
	        String req = key + ".req";
	        if (sp.rdp (req) != null)
	            throw new ISOException ("Duplicate key '" + req + "' detected");
	        sp.out (req, m);
	        m.setDirection(0);
	        if (timeout > 0)
	            sp.out (out, m, timeout);
	        else
	            sp.out (out, m);
	
	        ISOMsg resp = null;
	        try {
	            synchronized (this) { tx++; rxPending++; }
	
	            for (;;) {
	                resp = (ISOMsg) sp.rd (key, timeout);
	                if (shouldIgnore (resp)) 
	                    continue;
	                sp.inp (key);
	                break;
	            } 
	            if (resp == null && sp.inp (req) == null) {
	                // possible race condition, retry for a few extra seconds
	                resp = (ISOMsg) sp.in (key, 10000);
	            }
	            synchronized (this) {
	                if (resp != null) 
	                {
	                    rx++;
	                    lastTxn = System.currentTimeMillis();
	                }else {
	                    rxExpired++;
	                    if (m.getDirection() != ISOMsg.OUTGOING)
	                        txExpired++;
	                }
	            }
	        } finally {
	            synchronized (this) { rxPending--; }
	        }
	        return resp;
	    }
	    
```
疑问？sp.inp,sp.in,sp.xx方法表示请求响应？？？？
	

5）将期望返回的ISOMsg与实际请求返回响应值的ISOMsg进行断言比较：
<pre>
	jPOS is quite Q2-centric. Starting Q2 from your application is quite simple, you just need to instantiate Q2 and call its 'start' method.
	You can study Q2 to see how it configures the components, and simulate its life cycle from your application.
	Regarding jPOS-EE, interesting enough, there's a 'doc' directory with the ASCIIDOC sources for the documentation
	Many of the components depend on Q2's MBean service lifecycle, and won't operate correctly outside of Q2.

</pre>
	    
