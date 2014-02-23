#第一部分：各种重要类解析

##1、ISOChannel类
allows the transmision and reception of ISO 8583 Messages
ISOChannel用来完成底层的package和interchange，是应用层与底层细节的一个联系纽带。
这个类负责完成报文的发送和接收，接收的参数包括：ISOPackager，ServerSocket或者host和port。
	
接口的方法签名：
```java
public void setPackager(ISOPackager p);
public void connect () throws IOException;
public void disconnect () throws IOException;
public void reconnect() throws IOException;
public boolean isConnected();
public ISOMsg receive() throws IOException, ISOException;
public void send (ISOMsg m) throws IOException, ISOException;
public void setUsable(boolean b);
```
	
从上面可以看出，ISOChannel的职责包括：
<pre>
1） 通过调用相应的Packager完成Field的padder、intercpet和prefix或者反向操作；
2） 创建并维护connnection
3） 发送ISOMsg（包括发送请求报文和发送应答报文）
4）根据传输过来的报文，创建ISOMsg（包括接收请求报文和接收应答报文）
</pre>


ISOChannel类直接的子类有四个（一个实现类和另外三个接口）：  

- ChannelPool实现类
- ClientChannel接口
- ServerChannel接口
- FilteredChannel接口
	
org.jpos.BaseChanel同时实现了FilteredChannel接口、ClientChannel接口和ServerChannel接口


所以说ISOChannel大部分的实现逻辑位于org.jpos.BaseChanel.java 这个类中：
```java
public void send(ISOMsg m) throws IOException, ISOException, VetoException
public ISOMsg receive() throws IOException, ISOException

public abstract class BaseChannel extends Observable implements
       FilteredChannel, ClientChannel, ServerChannel, FactoryChannel,
       LogSource, ReConfigurable, BaseChannelMBean {
```
       
        
ServerChannel类.服务器方接收接口,使用了java socket编程接口；需要区别于客户端的接收应答的实现方式  
```java
public interface ServerChannel extends ISOChannel {
	public void accept(ServerSocket s) throws IOException;
}
```


服务器端实现方法在BaseChannel类中:  
```java
  public void accept(ServerSocket s) throws IOException {
    Socket ss = s.accept();
    this.name = ss.getInetAddress().getHostAddress()+":"+ss.getPort();
    connect(ss);
  }
    	  
    	  protected void connect (Socket socket) 
         throws IOException
	    {
	        this.socket = socket;
	        applyTimeout();
	        setLogger(getLogger(), getOriginalRealm() + 
	            "/" + socket.getInetAddress().getHostAddress() + ":" 
	            + socket.getPort()
	        );
	        synchronized (serverInLock) {
	            serverIn = new DataInputStream (
	                new BufferedInputStream (socket.getInputStream ())
	            );
	        }
	        synchronized (serverOutLock) {
	            serverOut = new DataOutputStream(
	                new BufferedOutputStream(socket.getOutputStream(), 2048)
	            );
	        }
	        postConnectHook();
	        usable = true;
	        cnt[CONNECT]++;
	        setChanged();
	        notifyObservers();
	    }
```

这样接收到的数据可以从serverIn中获取，发送应答可以通过serverOut


##2、ISOComponet类（参考：composite模式）

ISOField和ISOBitMap都是实现了ISOComponent的叶子节点。  

ISOMsg则是包含了ISOField的Composite （组合类）：ISOMsg类维护了Hashtable fields，维护了该报文的所有的位元。   

ISOComponent类体系采用composite模式，为的是能够以统一的方式处理基本元素和复合元素

实现Composite pattern相对比较容易,Component是一个接口或者抽象类，定义了叶子节点和Composite节点的公共方法；  

Composite类维护了Component的一个集合，提供了对该集合操作的一些方法，比如addComponent()和removeComponent()，同时也包括了叶子节点的操作方法。
	
##3、ISOBitMap类
ISOBitMap封装了protected BitSet value;并且提供了维护该bitmap的方法
	
##4、ISOMsg类
ISOMsg包括fields，header，direction，消息类型是在第0号位元存放的

##5、ISOFieldPackager类
ISOFieldPackager抽象了iso8583的位元的封装方式，定义了位元的抽象特性和一些模版方法，比如pack和unpack方法  

ISOFieldPackager的几个子类比如ISOStringFieldPackager分别适用于不同的报文的编码方式，每种不同的编码方式都包括了三个成员类：  
Interpreter、pader和Prefixer，分别用于负责转换位元内容、插入填充内容和插入位元长度。


因此通过继承每一种不同的编码方式的之类，jpos提供了不同的具体的ISO的数据类型类。
Interpreter、Pader、Prefixer接口的具体实现通过构造函数传入ISO的数据类型类。

###5.1）Prefixer（域前缀信息）
其中Prefixer指得是ISOField得域长度信息。有两个主要方法：  
void encodeLength(int length, byte[] b) throws ISOException;  
int decodeLength(byte[] b, int offset) throws ISOException; 

 
第一个方法是根据length产生长度得编码信息，存放到byte b[]中，比如encodeLength(21, byte[] b)，就是把对21编码，生成编码信息存放到b[0]和b[1]当中。  


Length含义的解释：代表该域的实际长度，比如某一个域定义的是LL..99，则length可以是0..99之间的任何数。  

第二个方法是对第一个方法的逆向操作  
参见AsciiPrefixer的源码  
	
###5.2）InterPreter（翻译器）
InterPreter接口有下面三个方法：
<pre>
（1）void interpret(String data, byte[] b, int offset)；
	将String格式的data转换成为byte数组
	Converts the string data into a different interpretation。Standard interpretations are ASCII, EBCDIC, BCD and LITERAL。

（2）String uninterpret(byte[] rawData, int offset, int length)；
	Converts the byte array into a String. This reverses the interpret method.
	将byte数组中的内容转换回String格式

（3）int getPackedLength(int nDataUnits)；
</pre>
	
###5.3） Padder接口（填充器）
An interface for padding and unpadding strings and byte arrays。
根据不同情况有多种填充策略，包括LeftPadder,RightPadder,RightTPadder等  


String pad(String data, int maxLength) throws ISOException;  
用于填充String或者Byte数组在实际长度和定义的长度之间的空白位  


String unpad(String paddedData) throws ISOException; 
 去除实际长度和定义的长度之间填充的空白

##6、ISOPackager类
ISOPackager类为整个iso8583报文，包含了全部128个域的定义。ISO87Apackager和ISO93Apackager等类为一些重要和常用的某个具体的协议硬编码好了完整的位元信息，包括每个位元的代码，描叙和处理类（FieldPackager）。  


在具体使用的时候，可以直接使用这些报文处理类，也可以使用GeneriaPackager类来根据具体读取报文协议信息，比如config/packager/iso87ascii.xml ，可以很方便地切换协议。



##7、ISOFilter类
An ISOFilter has the oportunity to modify an incoming or outgoing ISOMsg that is about to go thru an ISOChannel.   

It also has the chance to Veto by throwing an Exception。  

ISOFilter除了能够修改通过ISOChannel接受和发送的ISOMsg以外，还能够通过抛出异常的方式终止ISOMsg的接受和发送。  

public ISOMsg filter (ISOChannel channel, ISOMsg m, LogEvent evt)  throws VetoException;

	
##8、ISORequestListener类

##9、ISOResponseListener类

##10、ISOServer类	

##11、ISOUtil类
	提供了很多工具类，
	比如Ebcdic与ASCII互相转换的好多方法。
	字符串填充、去空格、字符串左边填充0、字符串和bcd的相互转换、十六进制和字符串转换
	byte、十六进制、十六进制三者的BitSet相互转换等等..



#第二部分: 协议
##发送报文：
```java
m.set(new ISOField(0, "0200"));
m.set(new ISOField(11, "1")); //6位定长数字，前补零
m.set(new ISOField(41, "11"));//8位定长字符，后补空
m.set(new ISOField(44,"nihao")); //25位变长字符
```
 
发送报文串如下：

>0200002000000090000000000111      05nihao

报文类型       Bitmap                         消息内容
```xml
<isomsg direction="incoming">
  <field id="0" value="0200"/>
  <field id="11" value="000001"/>
  <field id="41" value="11      "/>
  <field id="44" value="nihao"/>
</isomsg>
```
 

响应报文串如下：

>021000200000029000000000019911      05nihao

```xml
<isomsg direction="incoming">
  <field id="0" value="0210"/>
  <field id="11" value="000001"/>
  <field id="39" value="99"/>
  <field id="41" value="11      "/>
  <field id="44" value="nihao"/>
</isomsg>
```

 
端口监听获取的传输数据：  
发送报文：00410200002000000090000000000111      05nihao  
响应报文：0043021000200000029000000000019911      05nihao  


解释：前4位为报文长度，为十进制，即发送报文长度为41，接收报文长度为43,第二个4为消息类型MTI后面依次为bitmap和消息内容：

- bitmap使用16进制表示，002表示第11位
- 000001是因为11位元为6位定长数字，前面补了5个零
- 1      是因为是8位定长字符，后补7个空格
- 05nihao 是因为44位元是两位变长，所以05表示实际长度为5


#第三部分：  问题

##1、  写日志的方式：
```java
LogEvent evt = new LogEvent (this, "send");
evt.addMessage (m);
```

这样在每一个需要写日志的方法中都要创建对象，好像不妥，不直到是什么原因，不用静态方法

##2、  打印报文到标准输出的方法：xml格式
###（1）ISOmsg.dump(System.out,"");  
输出结构：
```xml
	<isomsg direction="incoming">
	  <field id="0" value="0200"/>
	  <field id="11" value="000001"/>
	  <field id="41" value="00000001"/>
	</isomsg>
```

###（2）System.out.println(ISOUtil.hexString(m.pack()));
输出16进制的报文：

>30323030303032303030303030303830303030303030303030313030303030303031

###（3）System.out.println(ISOUtil.dumpString(m.pack()));
输出二进制的报文：

>0200002000000080000000000100000001

##3、  问题:jpos如何根据bitmap识别报文内容??
答：

##4、  问题：Jpos如何支持灵活切换多种8583协议？
答：Jpos使用一个通用的packager,这个packager会根据每个协议的配置文件（*.xml）,该xml格式的配置文件定义了该8583协议的每一个位元的信息（位元号，长度，描叙，处理类），根据这个处理类就可以根据不同的协议以不同的方式使用位元。  

这样切换协议的时候，只需要修改下配置文件，或者具体的位元处理类即可，在整体的框架上不需要修改。
 

Does jPOS support ISO8583 - 2003 ?
In ISO8583:2003, for example, the STAN goes from six bytes to 12 bytes in length.   

So, the Generic Packager contains this definition to begin with.   
```xml 
  <isofield 
      id="11" 
      length="6" 
      name="SYSTEM TRACE AUDIT NUMBER" 
      pad="true" 
      class="org.jpos.iso.IFE_NUMERIC"/> 
```

.and you'd change it to look like this:   
```xml
  <isofield 
      id="11" 
      length="12" 
      name="SYSTEM TRACE AUDIT NUMBER" 
      pad="true" 
      class="org.jpos.iso.IFE_NUMERIC"/> 
```



Similarly, you'd make a sweep through the entire packager for all 128 fields 
(or at least the sub-set "in play" for you). 

I'd recommend a 'two-step sweep' exercise: 
1. Go through the generic packager XML file provided with the jPOS download and adjust those items like the STAN as indicated.  [You can find a list of these changes in "Annex F" of the ISO8583:2003 spec provided by the International Standards Organization.]  Also - be careful that you incorporate any changes that came with the 1993 revision.  For example, in the Generic Packager, the Action Code (the all-important Field 39) is two positions in length, but in the 1993 standard it went to three positions. [In case you're wondering: the 1987 implementation is still the dominant standard for most implementations worldwide.] 
2.Secondly, you'll want to go through the spec provided by the authorizing organization (or gateway) to you (that's assuming, of course, that your exercise is to put in place an ISO implementation created by someone else).  When you get into "private use" fields like 62 and 63, the Standards spec gives you only rough guidelines.  You'll want to see what the authorizer/gateway has done in its implementation.  [Unfortunately, for most of those private use fields, the packager definition will only get you so far.  These private use fields typically have esoteric, proprietary table structures inside of an 'IFE_LLLCHAR' (or similar) construction.  In those cases, you'll need some in-line code to complete the packaging and unpackaging tasks.] 


