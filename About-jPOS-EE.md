#有关jPOS-EE
##1）前言
我们决定构建我们自己的环境用Q2，或者给予JMX的IOC微内核。  


我们把jPOS-EE当做一个Meta-Project，这个项目包含许多其他能力。开源项目比如：
Hibernate、Jetty、Velocity、XStream、JDOM、JDBM、ApacheCommons、Etcetera


为了很容易的装配不同类型的jPOS-EE应用程序，我们开发一个很简单的、基于模块化的jPOS-EE软件开发工具包。
jPOS-EE-SDK，现在放置在gradle上


>Q2 is QSP version 2,described in jPOS's Programmer's Guide.


我们名字叫做jPOS-EE,但是我们替换掉了第一个E从企业的意思改变成扩展的意思。所以我们这个文档主要是描述jPOS扩展版本的。

我们并不要求使用jPOS-E框架，写一个jPOS应用的最佳路径就是支撑项目的选择，人们常常会认为JEE就是要走的路，或者Spring是要走的路，或者Pico/Nano容器可能作为更好的IoC,或者我们应该用JBoss/Jeonimo.
Guice,OSGI,QI4J,你自己命名。用同样的方式，我们选择用Jetty但是有人会说Tomcat会更好。


jPOS-EE刚好是我们在jPOS.org写我们自己的程序是的选择，我们分享这项技术作为一个as-is basis.你可以用它，或者你可以写你自己的jPOS程序用到任意的技术/框架你感觉合适的。

##2）关于这个文档

这是官方的jPOS扩展版本的文档，汇集着jPOS-EE相关的每一件事情，包含但是不限制：它的目的，规范，时间表，优先级，组织报表，线路图，更换日志，代码擦参与者，基础模块，可选的模块，license信息等。


它是对jPOS Programmer's Guide的补充。它不是一个用户指南或者一个向导。它是开发者的主要workbook，是开发者写给开发者的。


你现在读到的是修订版本2.0.1


最新版的可以经常查看： jpos.org/doc/jPOS-EE.pdf



##3）它的目的
我们的主要目的是阻止重复造轮子，记住：dont't repeat yourself(DRY)是很好的名言。


我们在jPOS.org有一个很小的公司和有限的开发资源和一大批客户。大部分的我们最终消费者程序是非常规范的，他们构成了很不一样特征。


jPOS-EE基本都是关于代码可持续的。我们开发了一个简单的SDK基于一个连接模块结构，这样它可以绑定在一起在编译的时候去创建，尽量的快，很好的稳定性的jPOS程序。


我们越来越多的使用jPOS-EE来为我们的客户创建应用，我们趋向于把模块做的越来越小，最好是他们互相之间尽可能的很少产生依赖。



我们希望开发人员利用jPOS-EE去创建他们自己的模块基于我们公开的代码，推动这个jPOS-EE社区为了。。。。



> 【注】本地变化到jPOS-EE代码可能看起来很容易的途径去解决一个已经给的环境，但是它迫使你应用同样的改变到一个新的jPOS扩展版本变得可用。


我们建议你走正规的路径，然后发送完整的请求在jPOS-EE中其他jPOS-EE的开发人员，测试在更多的平台，不同的环境，适当的文档。



##4）我们开始了。
###A.环境要求：
####1）JDK1.6或者更高
#####2）一个git客户端比如git-core（命令行）或者SmartGIT

###B.使用的IDE
jPOS-EE在eclipse、NetBeans、IDEA都能工作，eclipse装了gradle插件可以创建配置用gradle eclipse命令。
  
  
##5）使用一个应用服务器
jPOS-EE是一个独立的程序运行于 jPOS's 的Q2容器中。  

我们注意到有很多公司和机构标准的用一个已知的应用服务器或者JEE容器。非常好，但是jPOS-EE是一个独立的程序。  


如果你要控制你的应用服务器，你可能会整合jPOS-EE作为一个客户端EJB程序或者一个资源适配器，或者一个WAR，或者一个EAR等等。另外，许多应用服务器有许多扩展促进了这样的继承。


让我们再重复一次，jPOS-EE是一个独立的应用。如果你想知道怎样在XYZ服务器中运行jPOS-EE，那是非常棒的；如果我们能让你的生活更加容易通过做一些事情，使得集成更加简单，可会有jPOS-EE和你的应用服务器提供我们。。。
  
  
##6）准备好你的环境
- A.用GIT下载jPOS-EE SDK源码
- B.编译和安装组件到你本地的系统
 
###A)下载SDK
jPOS-EE仅仅通过Git来发布的，为了获得它的副本，也为了保证你的版本最新，你应该用一个GIT客户端。

 
GIT安装说明在Windows,LINUX或者MACOS

 
jPOS-EE被放置在GitHub上，通过访问这个项目页面你可能发现信息如何检出，浏览和查看系统的历史变化记录。

如果你在LINUX环境上，一个初始化的检出如下：
<pre>
 $ git clone https://github.com/jpos/jPOS-EE.git
Cloning into 'jPOS-EE'...
remote: Counting objects: 627, done.
remote: Compressing objects: 100% (355/355), done.
remote: Total 627 (delta 250), reused 528 (delta 151)
Receiving objects: 100% (627/627), 127.72 KiB, done.
Resolving deltas: 100% (250/250), done.
$
</pre>

这回创建一个目录叫做jPOS-EE包含了GitHub中心仓库当前master上的一个副本
 
###B.安装SDK
 安装这个框架，请参如下命令在SDK的顶级目录下：
 >./gradlew install


 以上执行这个命令会花费一点时间，Gradle下载了所有依赖。如果你已经安装了Gralde到你的系统，你不用在使用gradlew这个包装了。


 你可以仅仅用你的gradle.确保你有了一个最新版本的gradle. 可以通过 build.gradle查看下。

 
 如果你已经安装了Grale，你可能想要加速你的构建通过GRADLE_OPTS=-Dorg.gradle.daemon=true变量或者直接调用gradle --daemon。 也就是启动守护进程。
 

 如果你构建完成的时候输出了“BUILD SUCCESSFUL”，你已经可以准备去处理创建你的第一个项目了！

 
 gradle仅仅是构建和安装它的组件到你本地的Maven仓库，这样你可以在你项目中的任何地方使用了.
     
##7）五分钟教程
###A.初始化，jPOS项目
一旦你安装了jPOS-EE,为了创建你自己的项目，你可以利用我们的伴侣“jPOS Template” 


下载jPOS Template:https://github.com/jpos/jPOStemplate/archive/master.zip
https://github.com/jpos/jPOS-template/archive/master.tar.gz


移动jPOS-template-master目录名称改成你的项目名称"myjposproject"


为了一个初始化完整测试，我们构建一个简单的jPOS应用程序在我们移动到jPOS-EE之前。


调用gradle installApp（如果你没安装gradle的话就调用gradlew installApp）


然后到目录build/install/myjposproject/bin目录中去，你可以查看一个脚本叫做q2（在windows下叫做q2.bat）执行它将会启动jPOS,你可以通过Ctrl+c停止它。


以下是这个 session的全部抄送记录：
<pre>
$ cd /tmp
$ wget https://github.com/jpos/jPOS-template/archive/master.tar.gz
...
...
Saving to: `master.tar.gz'
$ tar zxvf master.tar.gz
x jPOS-template-master/
x jPOS-template-master/.gitignore
x jPOS-template-master/COPYRIGHT
x jPOS-template-master/LICENSE
x jPOS-template-master/README.md
x jPOS-template-master/build.gradle
x jPOS-template-master/gradle/
...
...
...
$ mv jPOS-template-master myjposproject
$ cd myjposproject
$ ./gradlew installApp
:compileJava UP-TO-DATE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:jar
:installApp
BUILD SUCCESSFUL
$ build/install/myjposproject/bin/q2
...
...
<log realm="Q2.system" at="Fri Jan 25 18:30:37 UYST 2013.335">
<info>
deploy:/private/tmp/myjposproject/build/install/myjposproject/deploy/99_sysmon.xml
</info>
</log>
...
...        
</pre>
 
jPOS template创建了一个标准的jPOS应用程序，包含有lib,deploy,cfg和log目录，你可以查看build/install/myjposproject目录
  
###B.添加一个jPOS-EE模块[注意，version要改成2.0.2-SNAPSHOT]
现在让我们添加一个jPOS-EE模块。我们会以一个简单的例子开始,Server Simulator


首先编辑build.gradle添加如下依赖：
> compile group:'org.jpos.ee', name:'jposee-server-simulator', version:'2.0.2-SNAPSHOT'


这样，很多依赖如下：
> dependencies {  
> 	compile group:'org.jpos', name:'jpos', version:'1.8.8+'  
> 	compile group:'org.jpos.ee', name:'jposee-server-simulator',   version:'2.0.2-SNAPSHOT'  
> 	testCompile group:'junit', name:'junit', version:'4.8.2'  
> }


如果你再一次通过gradle installApp重新构建一次，你再查看下lib目录，现在build/install/myjposproject/lib这个目录多出来了两个jar
jposee-core-2.0.1-SNAPSHOT.jar和jposee-server-simulator-2.0.1-SNAPSHOT.jar
    
###C.从模块中拉配置
jPOS-EE模块博阿含简单的配置文件在他们的发布jar文件中，他们作为一个引用需要被开发者重新reviewed，但是他们提供了一个很好的开端。


如果你执行以下命令：jar tvf build/install/myjposproject/lib/jposee-server-simulator-2.0.2-SNAPSHOT.jar

你可以看到：
<pre>
META-INF/
META-INF/MANIFEST.MF
META-INF/org/
META-INF/org/jpos/
META-INF/org/jpos/ee/
META-INF/org/jpos/ee/installs/
META-INF/org/jpos/ee/installs/cfg/
META-INF/org/jpos/ee/installs/cfg/serversimulator.bsh                              。。。。。。。。。。。。1
META-INF/org/jpos/ee/installs/deploy/
META-INF/org/jpos/ee/installs/deploy/05_serversimulator.xml                 。。。。。。。。。。。。2
</pre>

- 1，Server Simulator配置脚本
- 2，Server Sunykatir QBean描述符。


 如果你尝试gradle tasks，你可以看到有一个task叫做jposeeSetup，定义在jpos-app.grale如下:

>  task jposeeSetup(dependsOn: 'classes', type: JavaExec) {  
> 	classpath = sourceSets.main.runtimeClasspath  
> 	main = 'org.jpos.ee.support.BasicSetup'  
> 	args = ["src/dist"]  
> }


这个最基本的获取了所有的例子配置文件在jPOS-EE模块中，然后把他们放置在你应用的src/dist目录下，你可以编辑他们，添加你自己的SCM等。
  
因此我们调用gradle jposeeSetup在之前的例子，我们可以在src/dist下以一组新的文件结尾：

- src/dist/cfg/serversimulator.bsh
- src/dist/deploy/05_serversimulator.xml

这些文件会在你下次调用gradle installApp加载到 build/install/myjposproject，或者发布到你 build/distributions当你调用gradle dist
 
从这一点来说，你应该在安装目录中启动q2然后有一个服务监听在10000端口上。这是一个XML服务器，你可以通过telnet localhost 10000连接

可以发送一串信息如下：
```xml
<isomsg>
<field id="0" value="0800"/>
<field id="3" value="000000"/>
<field id="11" value="000001"/>
<field id="41" value="00000001"/>
<field id="70" value="901"/>
</isomsg>
```

如果一切正常，会得到8081响应

###8）关于各个模块
之前介绍的基于gradle的jPOS-EE项目结构，我们说了一个新的模块系统，是基于 Maven type artifacts的。
这样做的好处如下：

- 在你的项目中SDK是独立构建的
- 你的项目用到了依赖于版本的模块.结果是项目的足迹少了，现在仅仅需要跟踪你的代码，不是所有jPOS-EE的依赖都是在你的版本控制系统中的
- 入门的障碍大大减少了，因为一个新的开发者能够在 5分钟内启动jPOS-EE项目
- 不必要关心和跟踪模块的依赖
- 每一个模块都包含有例子配置

一个模块仅仅只是一个JAR包附带一些特性


####8.1)Hibernate映射
 一个模块定义了一个模块描述符，存储在/META-INF/org/jpos/ee/modules目录。这个描述符包含了模块中hibernate持久化映射的是实体。
 下面是个例子【D:\Java\workspace\jPOS-EE\jPOS-EE\modules\status\build\resources\main\META-INF\org\jpos\ee\modules】：
```xml
    <module name="status">
    <mappings>
    <mapping resource="org/jpos/ee/status/Status.hbm.xml" />
    <mapping resource="org/jpos/ee/status/StatusTag.hbm.xml" />
    </mappings>
    </module>
````

这是最佳实践：命名模块描述符文件的名字和模块名字一样加上“.xml”的扩展名。
代替静态定义他们在hibernate.cfg.xml中，持久化类映射在运行时加从classpath中将所有模块的描述符加载进来


####8.2)安装
有一个非常特殊的资源路径/META-INF/org/jpos/ee/installs，任何资源可以被存储在这个目录下面，当程序启动的时候会被安装到文件系统中

作为一个例子，如果我们把jposee-core模块当做我们的依赖，那这个模块的结构如下：
<pre>
    META-INF
    |-- org
        |-- jpos
            |-- ee
                |-- installs
                    |-- cfg
                    | |-- README.txt
                    |-- deploy
                    | |-- 00_logger.xml
                    | |-- 99_sysmon.xml
                    |-- log
                    |-- q2.log
</pre>

我们执行如下：
>$ java -jar q2.jar -cli
>
>q2> setup .

把如下结构拷贝到当前工作目录下
<pre>
    .
    |-- cfg
    | |-- README.txt
    |-- deploy
    | |-- 00_logger.xml
    | |-- 99_sysmon.xml
    |-- log
    |-- q2.log
</pre>


如果我们现在添加jposee-db-mysql模块作为我们的依赖，会包含如下的结构:
<pre>
    META-INF
        |-- org
            |-- jpos
                |-- ee
                    |-- installs
                        |-- cfg
                            |-- db.properties    
</pre>


在我们的文件系统中将以如下内容结束
</pre>
    |-- cfg
    | |-- README.txt
    | |-- db.properties
    |-- deploy
    | |-- 00_logger.xml
    | |-- 99_sysmon.xml
    |-- log
    |-- q2.log
</pre>

    
####8.3)核心基本模块
#####8.3.1)基础(modules/core)
<pre>
    What        The core module contains all basic jPOS-EE functionality.
    When        Available in all versions of jPOS-EE.
    Who         The jPOS.org team.
    Where       Directory modules/core available in git repository at github.
    Why         This is a core module required in all jPOS-EE applications Status Stable.
    License     GNU Affero General Public License version 3
</pre>


Maven 坐标：
```xml
    <dependency>
        <groupId>org.jpos.ee</groupId>
        <artifactId>jposee-core</artifactId>
        <version>${jposee.version}</version>
    </dependency>    
```
    
这个核心基础模块服务于两个方面：  

- 它包含了 任何 jPOS-EE程序启动所有的基础依赖
- 它包含了所有jPOS-EE程序分享的基础功功能    
    
#####8.3.2)事务支持(modules/txn)
<pre>
    What            The txn module contains Transaction Manager support code as well as
                    common transaction manager participants.
    When            Available in all versions of jPOS-EE.
    Who             The jPOS.org team.
    Where           Directory modules/txn available in git repository at github.
    Why             This module if useful is your jPOS-EE application uses the Transaction Manager.
    Status          Stable.
    License         GNU Affero General Public License version 3
</pre>


Maven坐标：
```xml
    <dependency>
        <groupId>org.jpos.ee</groupId>
        <artifactId>jposee-txn</artifactId>
        <version>${jposee.version}</version>
    </dependency>
```
    
在每一个项目中没有什么比重复造轮子更糟糕的啦。带着这样的心情，jPOS项目组鉴定了一些列的活动，针对我们几乎每个基于jPOS-EE开发项目的企业，创建了一个模块提供了基础的构建伟大的TransactionManager参与者追随了最佳实践模式。
    
取代展示一个枯燥的表格包含一个所有组件干什么的描述信息这种方式，我认为一个Transaction Manager的实例例子如下：
```xml
    <txnmgr name="txnmgr" logger="Q2" class="org.jpos.transaction.TransactionManager">
        <property name="space" value="transient:default"/>
        <property name="queue" value="TXN"/>
        <property name="max-sessions" value="10"/>
        
        <participant class="org.jpos.transaction.Open" logger="Q2" realm="open-db">        -----------1
            <property name="checkpoint" value="db-open"/>
        </participant>
        
        <participant class="com.mydemo.DemoParticipant"                                     -----------2
        logger="Q2" realm="demo-participant"/>
        
        <participant class="org.jpos.transaction.Close" logger="Q2" realm="close-db">       -----------3
            <property name="checkpoint" value="close"/>
        </participant>    
    </txnmgr>
```
    
1. Open参与者打开了一个新的DB会话和事务
1. 我们的DemoParticipant处理了许多进程
1. Close参与者提交或者是回滚基于overall overcome的交易，然后关闭这个会话


在我们的例子剧本中，事务管理器可能打开一个数据库会话，执行我们DemoParticipant，然后关闭数据库回话（尽管我们的DemoParticipant不需要一个DB会话）


当我们需要添加一些debugging信息的情形时，我们会添加到文件末尾：
```xml
    <participant class="org.jpos.transaction.ProtectDebugInfo"
        logger="Q2" realm="protect-debug">                                                  --------1
        <property name="checkpoint" value="protect-debug-info"/>
        <!-- Wipes entries from context -->
        <property name="wipe-entry" value="PAN"/>
        <property name="wipe-entry" value="EXP"/>
        <!-- Protects contents from any ISOMsg in context -->
        <property name="protect-ISOMsg" value="2"/>
        <property name="protect-ISOMsg" value="35"/>
        <property name="protect-ISOMsg" value="45"/>
        <!-- Wipes contents from any ISOMsg in context -->
        <property name="wipe-ISOMsg" value="48"/>
        <property name="wipe-ISOMsg" value="52"/>
        </participant>
        
        <participant class="org.jpos.transaction.Debug" logger="Q2" realm="debug">          ---------2
        <property name="checkpoint" value="debug"/>
    </participant>
```

1. ProtectDebugInfo参与者从logs中保护敏感的资料
1. Debug参与者dumps了context所有内容到log中


在保护敏感信息的过程中，可能会造成上下文的信息被dumped到日志中    

万一你怀疑的DemoParticipantmight像如下这样：
```java
	public class DemoParticipant extends TxnSupport implements MyConstants 
	{
		protected intdoPrepare(longid, Context ctx) throwsException 
		{
			ISOMsg message = (ISOMsg) ctx.get(REQUEST);
			ISOSource source = (ISOSource) ctx.get(SOURCE);
			assertNotNull(message,"A valid 'REQUEST' is expected in the context"); 
			assertNotNull(source,"A valid 'SOURCE' is expected in the context");
			assertTrue(message.hasField(4), 
			"The message needs to have an amount (ISOMsg:4)");
			message.setResponseMTI();
			Random random = newRandom(System.currentTimeMillis());
			message.set (37, Integer.toString(Math.abs(random.nextInt()) % 1000000));
			message.set (38, Integer.toString(Math.abs(random.nextInt()) % 1000000));
			if("000000009999".equals (message.getString (4)))
			message.set (39, "01");
、
			source.send (message);
			return PREPARED | NO_JOIN | READONLY;
		}
		public void commit(longid, Serializable context) { }
		public void abort(longid, Serializable context) { }
	}
```
	
- 我们的示例参与者继承了TxnSupport类，支持的类由这个模块提供。
- TxnSupport覆盖了prepare方法，指向了doPrepare
- 如你看到的，非空的断言是如此的简单
- 都是布尔的断言。

> 【注意】如果你很认真的对待jPOS-EE开发调用Transaction Manager,我们建议你学习TxnSupportclass这个类。
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    