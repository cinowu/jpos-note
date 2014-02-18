##QBean Descriptors
Q2通常扫描deploy目录，然后寻找QBean描述符文件,这样的描述符
很简单和灵活，要求如下两点：

- 最外层元素至少要拥有name这个属性
```xml
<qbean name="mybean" class="org.jpos.test.MyBean" />
```

- 虽然我们QFactory.properties配置文件中得知外层元素是ResourceBundle,但是最外层元素必须指定class用来确定QBean实现自哪个类  
```xml
<qbean name="mybean" class="org.jpos.test.MyBean" />
```

> If we omit the 'name' attribute, then the root element name (in this case qbean) would be used instead.  
> 如果我们忽略了name属性，root元素的name属性会被拿过来用。
> 切记：从root元素获取的name属性还是本身自己的name属性，必须通过Q2实例获得

除了提供上述所说的name和class属性，Q2还提供了如下两个属性：logger和realm
```xml
<qbean name="mybean" class="org.jpos.test.MyBean" logger="Q2" realm="mybean" />
```


ResourceBundle QFactory.properties文件为很多我们熟知的QBeans提供了快捷称呼，定义入下：  
```properties
	shutdown=org.jpos.q2.qbean.Shutdown
	script=org.jpos.q2.qbean.BSH
	jython=org.jpos.q2.qbean.Jython
	spacelet=org.jpos.q2.qbean.SpaceLet
	sysmon=org.jpos.q2.qbean.SystemMonitor
	txnmgr=org.jpos.transaction.TransactionManager
	transaction-manager=org.jpos.transaction.TransactionManager
	qmux=org.jpos.q2.iso.QMUX
	channel-adaptor=org.jpos.q2.iso.ChannelAdaptor
```
像下面这样使用
```script
<script name="myscript">
...
...
</script> 
```


尽管起别名比较方便并且也是最佳实践，但是容易使得在一个描述符文件定义的简称与另外一个描述符文件定义的简称冲突

shutdownis an useful shortcut, a simple descriptor with a single element called shutdown will initiate a
Q2 clean shutdown, stopping all deployed services. Look at bin/stopfor an example.
shutdown这个配置的简称非常有用，就一行元素会使得Q2停止一切部署的服务。


##Q2生命周期
Q2 为QBean生命周期提供了四个简单的方法如下：
init
start
stop
destroy

##QBean的另外两个方法
int getState(); String
getStateAsString(); 
methods required for housekeeping and JMX-based monitoring.
这两个方法适合用在家务管理和基于JMX的监控系统上


##自定义QBean规范
写一个QBean非常容易，通常我们实现org.jpos.q2.QBean接口就行了

MBean 接口的命名一定要以MBean或MXBean结尾，否则报报如下错：
does not implement DynamicMBean, neither follows the Standard MBean conventions ....
可能的原因是：接口和实现类的名字不符合命名规范: XXXMBean, XXX。

