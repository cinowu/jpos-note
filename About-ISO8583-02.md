有关ISO-8583
##讨论ISO-8583之前先看下以下概念的不同：
###1）message format (its binary representation),
> messages是由很多的field域构成的，不同的域有不同的作用。  
>【声明】：以下所说的位置，大都指代bitmap的64个或者128个位的位置，按照数字从小到大，通常，这些位置也是域的位置，是等同的。不过域0位置表示MTI.
    
####1.1)message结构如下：
<pre>
    field           description
    0 - MTI         MESSAGE TYPE INDICATOR                                       位置0  是一个四位的数字
    1 - Bitmap      64 (or 128) bits indicating presence/absence of other fields 位置1  类型：B16（二进制16字节，16*8=128bit）
    2 .. 128        Other fields as specified in bitmap
</pre>
    
####1.2) 简单的例子：0800 message

<pre>
    #           Name                   			 Value                     		Hex Value
    0           MTI                    			 0800                      		08 00
    1           PRIMARY BITMAP                   Indicates presence of fields 3, 11 and 41       20 20 00 00 00 80 00 00                                                                
    3           PROCESSING CODE                          000000                    	   00 00 00
    11          SYSTEM TRACE AUDIT  NUMBER               000001                   	   00 00 01
    41          TERMINAL ID                              29110001                  	   32 39 31 31 30 30 30 31
</pre>
    
>以上0800 message的二进制结果是：080020200000008000000000000000013239313130303031
>至于29110001为何得出十六进制是后面的值，是通过字符串转为字节数组，字节数组转换为十六进制
        
> 在上面的例子中0800就是message type indicator (MTI)消息类型标识，发现MTI由四个数字组成，这四个数字定义如下：
#####   (mti 1.2.1）第一个数字表示ISO-8583版本编号
<pre>
            具体数字含义如下：
        0 for version 1987
        1 for version 1993
        2 for version 2003
        3-7 reserved for ISO use
        8 is reserved for national use
        9 is reserved for private use
            由这个得知0表示1987版本
</pre>
            
#####   (mti 1.2.2）第二个数字表示ISO-8583消息类
<pre>
            具体数字含义如下： 
        0 is reserved for ISO use
        1 authorization
        2 financial
        3 file update
        4 reversals and chargebacks
        5 reconciliation
        6 administrative
        7 fee collection
        8 network management
        9 reserved for ISO use            
            由这个得知8表示网络管理
</pre>
            
#####   (mti 1.2.3）第二个数字表示ISO-8583消息功能
<pre>
            具体数字含义如下：                         
        0 request
        1 request response
        2 advice
        3 advice response
        4 notification
        5-9 reserved for ISO use
            由这个得知0表示请求   
</pre>   

#####   (mti 1.2.4）第二个数字表示交易源
<pre>
            具体数字含义如下：
        0 acquirer
        1 acquirer repeat
        2 card issuer
        3 card issuer repeat
        4 other
        5 other repeat
        6-9 reserved for ISO use            
            由这个得知0表示收购方
            
     So "0800" is a version 1987 network management request.
</pre>         
 
   1.3)Bitmap 位图   类型：B16（二进制16个字节，16*8=128bit），一个字节由十六进制的两个数字表示。
       读取位图的方式：
           先读取8个字节，64位，也就是2020000000800000，每两个数字转换为二进制位，某一位为1则存在该域
           如果第一位为1，则读取完这个8个字节，紧接着还要读取8个字节64位
   B16表示16字节  【占用16个字节2020000000800000】
   byte   hex value     bit value     field #
    0       20          0010 0000       3   --  可以看到位图第一位，这里为指的是bit，第一位为0则否则表示只使用基本位图（64个域），为1则表示使用扩展位图（128个域）
    1       20          0010 0000       11
    2       00          0000 0000
    3       00          0000 0000
    4       00          0000 0000
    5       80          1000 0000       41
    6       00          0000 0000
    7       00          0000 0000    
   
   
####1.4)ISO-8583 FIELD域说明
#####1.4.1)域类型：  

> Fixed length定长的类型如下：
<pre>
Numeric          数字的
Alphanumeric     含有字母和数字的
Binary           二进制的
这三种类型也可以是变长的长度99、999或者9999
</pre>


>注意：ISO-8583并没有指定
>
>个域代表着什么，你可以使用一个Numeric类型的域代表一个ASCII字符串、EBCDIC字符串或者BCD.     


> 网上说的不一定准确：银联8583并不是ISO8583, 只是参考ISO8583, 银联的字符编码是ASCII码,而ISO8583是BCD码,两者是有差别的.
> 
> 【注】变长的域都会有一个前缀来确定它的长度，但是这个域代表什么，也没有明确规定。不同的公司会有不同的表述，
> 可以是BCD编码的值,可以是EBCDIC编码的值, 也可以是二进制的值
         
> 在上面的例子中Field #3用到了一个BCD编码的表示，所以值“000000”仅仅代表三个字节，十六进制内容“00 00 00”.
> Filed #11 "000001"代表就是 "00 00 01".在我们的例子中Field #41是八个字节，代表八个ASCII字符


#####域的读取原则
>1）根据数据长度确定读取几个字节，长度是20，读取10个字节，长度是3读取两个字节，以此类推。
>
>2）根据倒数第三个数字确定数据类型是字符串、数字还是二进制
<pre>
a.如果是数字，那么把读取到的字节内容全部当做实际数字
   如果是定长的数字，那么与域长度是几，那么就读取几/2个字节。如果与长度是3，读取2个字节，长度是6读取3个字节。

b.如果是字符串，那么把读取到的字节内容全部当做字符串。
   如果是变长字符串，那么先要读取第一位字节当做数字，数字是几就表示接下来还要继续读取多少/2个字节，比如第一个数字是37，表示接下来还要读取19个字节的字符串。
   如果是定长字符串，那么与域长度是几，就读取几个字节，比如域长度是13，就读取13个字节。

c.如果是二进制，那么把读取到的字节内容全部当做十六进制，然后转换为二进制，二进制*8就是具体位数。

d.如果是ans类型的，那么定长是几就读取几个字节，如果定长是8就读取8个字节【一个字节由十六进制的两个数字表示】也就是16个十六位数字
</pre>



<pre>
        
Message: 08002020 00000080 00000000 00000001
32393131 30303031
MTI: 0800
Bitmap: 20200000 00800000
Field 03: 000000
Field 11: 000001
field 41: 3239313130303031 (ASCII for "29110001")       
        
解析：　
/* FLD 03 */ {0,"PROCESSING CODE", 6, 0, 0, 0, NULL,0}, 
这里倒数第三个数字0表示数据类型是字符串，倒数第四个数字是0，表示不变长，就是定长。也就是实际长度就是6.也就是三个字节。
field 3, 处理码, n6, 定长, 用3字节BCD表示.        
  ......
  
  
Message: 0800A020 00000080 00100400 00000000
00000000 00000001 32393131 30303031
00106A50 4F532031 2E392E31 0301
MTI: 0800
Primary bitmap: A0200000 00800010
Secondary bitmap: 04000000 00000000
Field 03: 000000
Field 11: 000001
Field 41: 3239313130303031 (ASCII for "29110001")  将16进制转换为ASCII的八位
Field 60: 0010 6A504F5320312E342E31 (length=10, value="jPOS 1.4.1")
Field 70: 0301      


Primary BitMap分析：
byte     hex value       bit value           field #
0           A0           1010 0000 secondary bitmap present plus #3       ---------------看到这里位图第一位是1，表示使用扩展位图，然后再读取8个字节的扩展位图
1           20           0010 0000            11
2           00           0000 0000
3           00           0000 0000
4           00           0000 0000
5           80           1000 0000            41
6           00           0000 0000
7           10           0001 0000            60

Secondary BitMap分析：
byte     hex value       bit value          field #
0           04           0000 0100            70
1           00           0000 0000
2           00           0000 0000
3           00           0000 0000
4           00           0000 0000
5           00           0000 0000
6           00           0000 0000
7           00           0000 0000




ans 域类型描述： ans Alphabetic, numeric & Special Characters  含有数字字母和特殊字符的

/* FLD 41 */ {0,"CARD ACCEPTOR TERMINAL ID. ", 8, 0, 0, 0, NULL,0},  
-----------------------------------------------------------------------        
域(BitMap的bit位置)41 受卡机终端标识码
Card Acceptor Terminal Identification
变量属性
ans8，8位定长的字母、数字和特殊字符
域描述
受卡机的终端标识码。该标识码在代理机构的网络中必须唯一标识一个终端。
用法
如果终端标识码少于八位，则按左靠，右边补空格。
终端标识码由代理机构分发。所有卡交易请求中必须带上终端标识码，且在整个交易周期中保持不变。        
-----------------------------------------------------------------------         
      
-----------------------------------------------------------------------    
/* FLD 60 */ {0,"NO USE ", 5, 0, 3, 0, NULL,0}, 
-----------------------------------------------------------------------

-----------------------------------------------------------------------
/* FLD 70 */ {0,"SYSTEM MANAGEMENT INformATION CODE ", 3, 0, 0, 0, NULL,0}
Bit70管理信息码(System Management Indormation Code)  
位图位置：70  
格式：定长  
类型：N3  指代3位长数字  长度为3，读取两个字节，四位十六进制数字 0301,最终结果是301
描述：  
用于定义和维护银行电子服务系统内部通讯网络状态和应用工作状态。  
网络管理信息代码用于管理清算日期"cutoff"，通讯"sign on/sign off"，"key exchange"等。  
支持以下一些网络管理信息码  
NMIC网络管理信息码动作  
001签到(Sign on)  
002签退(Sign off)  
101交换密钥(Key exchange)  
201结帐日期切换(Cutoff)  
202结帐日期切换完成  
301测试(Echo test)  
-----------------------------------------------------------------------
</pre>

#####[麻烦概述]
为了使得开发者的开发难度加大，不同的厂商会采取不同的方式来填充odd-length BCD域，所以为了表述003，有的厂商可能会用两个字节的值：
如 "00 03"，同时其他的厂商可能用"00 30"，甚至是"00 3F"
变长的域也同样是这样：域的长度可以填充到域的左边或者右边去。当然ISO-8583不会这么做，这样是实现者代码的麻烦而已
如果我们有了嵌套的字段，许多实现者会用“保留为了私有使用”字段去抓取其他的ISO-8583消息。这些消息通常被打包成可变长的二进制域

你可以看到JPSO处理这些问题用了非常简单的方式，因此你不需要担心这些低级的麻烦和废话了
      
###2）wire protocol (how a message is transmitted over the wire), 数据包协议
> 一旦我们有了一个二进制包能够代表已经给出的ISO-8583消息，我们不得不把这些数据利用一些交流协议传输出去，类似
> TCP/IP, UDP, X.25, SDLC, SNA,ASYNC, QTP, SSL, HTTP或者你自定义的一些协议！
> 
> 交流协议不是ISO-8583标准定义的一部分，不同的厂商可能会采取不同的协议
> 
> 许多实现的厂商，尤其是老一点的厂商，需要不同的支持信息。比如CICS传输名称，因此他们用各种不同的协议头。
> 很少有厂商，尤其是基于流式的厂商，需要各种不同的追踪器


- 数据包协议由以下几个方面组成:
<pre>
1) 一个可选的数据包头/消息边界分隔符
2）ISO-8583消息数据
3) 一个可选的追踪器（有时候被用作于消息边界分隔符）
</pre>


<pre>
一个基于TCP/IP协议实现的厂商会用一组字节来表示消息【网络传输数据】的长度，因此我们的0800那个例子可以更早的描述成如下样子来发送：
00 46 08 00 A0 20 00 00 00 80 00 10 04 00 00 00
00 00 00 00 00 00 00 00 00 01 32 39 31 31 30 30
30 31 00 10 6A 50 4F 53 20 31 2E 34 2E 31 03 01
0046就是这些网络顺序字节数据的长度表示


如上的仅仅是我们指定网络传输数据长度的一种方式。另外一种方式我们选择传输ASCII字节：
30 30 34 36 08 00 A0 20 00 00 00 80 00 10 04 00
00 00 00 00 00 00 00 00 00 00 00 01 32 39 31 31
30 30 30 31 00 10 6A 50 4F 53 20 31 2E 34 2E 31
30 30 34 36是0046的ASCII表示形式【也就是十六进制转换为ASCII】

许多实现厂商会计算消息长度指示器的大小，在之前的例子中，本来传输"0046"的，他们可能会传输"0050"


【注意】在你的开发过程中，你去早一点读取interchange specification(s)是非常重要的。
jPOS处理数据包协议用了一些列的类叫做channels的，实现了ISOChannel接口而隐藏了数据包协议的细节。
http://jpos.org/doc/javadoc/org/jpos/iso/ISOChannel.html
</pre>


###3）message flow (e.g., send request for authorization, wait for response, retransmit, reversal)消息流

####3.1）认证的例子  

<pre>
    Time        Acquirer        Issuer      Description
    t0          0100 -->                    authorization request
    t1                          <-- 0110    authorization response
    如上是一个典型的请求响应的例子，可能有的时候会失败，也就是你发送请求了然后收不到响应。
    你不得不通过一些途径终止这个请求，恢复之前的交易信息然后重新尝试发一次验证请求.
</pre>
    
####3.2) 认证超时  

<pre>
    Time        Acquirer        Issuer      Description
    t0          0100 -->                    authorization request
    t1                                      no response
    t3          0400 -->                    reverse previous authorization
    t4                          <-- 0410    reverse received
    t5          0120 -->                    authorization advice
    t6                          <-- 0130    advice received

    依赖于你的特别的实现，你可能发送一些重发的信息(e.g., 0101 after an unanswered 0100)，一些实现会用到私有信息
  (比如9600)来进行发送来实现时间到流程的交易。因此你看到了你熟悉你自己的interchange specifications和它期望的数据流是非常重要的。

   jPOS提供了工具来处理消息结构，数据包协议和消息流。你为你的商业逻辑信息流提供的更高级的接口是你的责任。
      下面有一个例子可以帮助你看到验证请求和响应期间到底交换了哪些数据。
</pre>
     
####3.3)  

<pre>
         认证请求的例子数据    ：
    Fld #       Description                 Value                   Comments
    0           MTI                         0100                    Authorization request
    2           Primary Account Number      4321123443211234
    3           Processing Code             000000
    4           Amount transaction          000000012300            i.e., 123.00
    7           Transmission data/time      0304054133              MMYYHHMMSS
    11          System trace audit number   001205
    14          Expiration date             0205                    YYMM
    18          Merchant Type               5399
    22          POS Entry Mode              022                     Swiped Card
    25          POS Condition Code          00 
    35          Track 2                     4321123443211234=0205..
    37          Retrieval Reference Number  206305000014
    41          Terminal ID                 29110001
    42          Merchant ID                 1001001
    49          Currency                    840                     American Dollars

    认证响应数据：
    Fld #       Description                 Value                   Comments
    0           MTI                         0110                    Authorization response
    2           Primary Account Number      4321123443211234
    3           Processing Code             000000
    4           Amount transaction          000000012300            i.e., 123.00
    7           Transmission data/time      0304054133              MMYYHHMMSS
    11          System trace audit number   001205
    14          Expiration date             0205                    YYMM
    18          Merchant Type               5399
    22          POS Entry Mode              022                     Swiped Card
    25          POS Condition Code          00
    35          Track 2                     4321123443211234=0205..
    37          Retrieval Reference  Number 206305000014
    38          Authorization number        010305
    39          Response code               00                      Approved
    41          Terminal ID                 29110001
    42          Merchant ID                 1001001
    49          Currency                    840                     American Dollars


    以上响应多出了Field#38 交易授权机构返回的返回代码
</pre>


##jPOS处理ISO-8583的方式
###以下内容用于展示jPOS是如何处理ISO-8583信息的
###前言：jPOS内部采用ISOMsg或者它的子类代表ISO-8583的Message对象,ISOMsg利用了设计模式中的组合模式
 
ISOMsg, ISOField, ISOBitMapField, ISOBinaryField和一些自定义的域类型均实现自ISOComponent接口,看下ISOComponent接口的方法：
```java
    public abstract class ISOComponent implements Cloneable {
    public void set (ISOComponent c) throws ISOException;
    public void unset (int fldno) throws ISOException;
    public ISOComponent getComposite();
    public Object getKey() throws ISOException;
    public Object getValue() throws ISOException;
    public byte[] getBytes() throws ISOException;
    public int getMaxField();
    public Hashtable getChildren();
    public abstract void setFieldNumber (int fieldNumber);
    public abstract void setValue(Object obj) throws ISOException;
    public abstract byte[] pack() throws ISOException;
    public abstract int unpack(byte[] b) throws ISOException;
    public abstract void dump (PrintStream p, String indent);
    public abstract void pack (OutputStream out) throws IOException, ISOException;
    public abstract void unpack (InputStream in) throws IOException, ISOException;
    }
```
这些方法正好与ISO-8583 Message结构吻合。


<pre>
There are many situations where some methods are not applicable (i.e., getChildren() has no
meaning in a leaf field, same goes for methods such as getMaxField()), but as a general rule,
using the same super-class for ISOMsg and ISOFields has proven to be a good thing. You can
easily assign an ISOMsg as a field of an outer ISOMsg.

有很多问题就是有的方法不是太实用，比如 getChildren()没有说明是子域的意思，居然与getMaxField()是同等意思。通常来说，
超类ISOMsg和ISOFields这两个是非常实用的，你可以轻松为一个outer ISOMsg分配一个ISOMsg成为一个域
</pre>



###1） ISOComponent,ISOMsg, ISOField,ISOBinaryField的关系  

<pre>
    1.1）ISOMsg, ISOField,ISOBinaryField都实现自ISOComponent接口
    1.2）一个ISOMsg拥有多个ISOField和多个ISOBinaryField
</pre>

###2）用于描述0800 message消息的代码（入门）：   
```java  
     m.set (new ISOField (0, "0800"));
     m.set (new ISOField (0, "0800"));
     m.set (new ISOField (3, "000000"));
     m.set (new ISOField (11, "000001"));
     m.set (new ISOField (41, "29110001"));
     m.set (new ISOField (60, "jPOS 6"));
     m.set (new ISOField (70, "301"));
```

为了减少代码量和提高代码可读性，jPOS提供了一些手动静态方法，如：
```java
     ISOMsg.setMTI (String)
```
或者
```java
	ISOMsg.set (int fieldNumber, String fieldValue)
```

以上这两个静态方法的内部实现我们看下：
``java
   public void set (int fldno, String value) throws ISOException {
        set (new ISOField (fldno, value));
    }
    public void setMTI (String mti) throws ISOException {
        if (isInner())
        throw new ISOException ("can't setMTI on inner message");
        set (new ISOField (0, mti));
    }  
```

以上代码可以用如下简写方式：
```java
    ISOMsg m2 = new ISOMsg();
    m2.setMTI ("0800");
    m2.set (3, "000000");
    m2.set (11, "000001");
    m2.set (41, "29110001");
    m2.set (60, "jPOS 6");
    m2.set (70, "301");
```
   
ISOMsg这个类是典型的基于 ISO-8583协议开发的jPOS应用程序中使用最多的类。当然啦，你当你需要的时候可以继承这个类。
如果jPOS里面只有一个类能让你去学习，那么就是这个类了。
       
###3)打包和解包方法：
ISOComponent组件们都提供了以下两个非常有用的方法：
```java
   public abstract byte[] pack() throws ISOException;
   public abstract int unpack(byte[] b) throws ISOException;
```
   
pack方法返回的是字节数组，这些字节数据包含的内容代表的是已经给的组件比如域或者是整个ISOMsg。
unpack方法做了相反的事情，返回的是已经用过的字节的数字。
jPOS利用的相同的模式允许已经知道的ISOComponent可以被打包和解包通过相同的类，阻塞在运行期间。

可以将已知的ISOMsg打包，利用这个方法：
```java
    public void setPackager (ISOPackager p);
    
    ISOPackager customPackager = MyCustomPackager ();
    ISOMsg m = new ISOMsg();
    m.setMTI ("0800");
    m.set (3, "000000");
    m.set (11, "000001");
    m.set (41, "29110001");
    m.set (60, "jPOS 6");
    m.set (70, "301");
    m.setPackager (customPackager);
    byte[] binaryImage = m.pack();
```
    
为了解包这个 binaryImage，你可以利用如下代码：
```java
    ISOPackager customPackager = MyCustomPackager ();
    ISOMsg m = new ISOMsg();
    m.setPackager (customPackager);
    m.unpack (binaryImage);
```
           
利用jPOS进行协议转换是非常容易的，如下代码：
```java
    ISOPackager packagerA = MyCustomPackagerA ();
    ISOPackager packagerB = MyCustomPackagerB ();
    ISOMsg m = new ISOMsg();
    m.setPackager (packagerA);
    m.unpack (binaryImage);
    m.setPackager (packagerB);
    byte[] convertedBinaryImage = m.pack();
```     
    
    
看下pack()方法内部实现：  
pack()代表消息的压缩/加压操作，在如下的实现过程中用同一个ISOPackager：
```java
        public byte[] pack() throws ISOException {
            synchronized (this) {
            recalcBitMap();
            return packager.pack(this);
            }
        }
```
packager.pack(ISOComponent)也同样代表消息的压缩/加压操作。


看下ISOPackager的子类ISOFieldPackager进行打包和解压定长的字符域的内部实现方法：
```java
     public byte[] pack (ISOComponent c) throws ISOException {
        String s = (String) c.getValue();
        if (s.length() > getLength())
        s = s.substring(0, getLength());
        return (ISOUtil.strpad (s, getLength())).getBytes();
     }
     public int unpack (ISOComponent c, byte[] b, int offset)
     throws ISOException
     {
        c.setValue(new String(b, offset, getLength()));
        return getLength();
     }
```
     
> [注]jPOS定义了很多ISOFieldPackager的实现类，其实你没必须要自己再去写一个实现。
> 
> ISOFieldPackagers通常在org.jpos.iso包下，如果你要自定义自己的ISOFieldPackager类，你的类也一样应该有如下结构：  
> 
<pre>
     Name                   Purpose
    IF_CHAR             Fixed length alphanumeric (ASCII)
    IFE_CHAR            Fixed length alphanumeric (EBCDIC)
    IFA_NUMERIC         Fixed length numeric (ASCII)
    IFE_NUMERIC         Fixed length numeric (EBCDIC)
    IFB_NUMERIC         Fixed length numeric (BCD)
    IFB_LLNUM           Variable length numeric (BCD, maxlength=99)
    IFB_LLLNUM          Variable length numeric (BCD, maxlength=999)
    IFB_LLLLNUM         Variable length numeric (BCD, maxlength=9999)
    …                   …
    …                   …
</pre>
     
####自定义一个ISOFieldPackager：
```java
     public class ISO93APackager extends ISOBasePackager {
    protected ISOFieldPackager fld[] = {
    /*000*/ new IFA_NUMERIC ( 4, "Message Type Indicator"),
    /*001*/ new IFA_BITMAP ( 16, "Bitmap"),
    /*002*/ new IFA_LLNUM ( 19, "Primary Account number"),
    /*003*/ new IFA_NUMERIC ( 6, "Processing Code"),
    /*004*/ new IFA_NUMERIC ( 12, "Amount, Transaction"),
    /*005*/ new IFA_NUMERIC ( 12, "Amount, Reconciliation")
    //...
    //...
    //...
    };
    public ISO93APackager() {
        super();
        setFieldPackager(fld);
    }
    }
```
         
虽然你自定义ISOFieldPackager的不一定非要继承ISOBasePackager类  ，但你继承会更加简单些。
         
         
GenericPackager一个可以配置XML文件的类,这个类的配置文件XML如下：
```xml
   <?xml version="1.0" encoding="UTF-8" standalone="no"?>
    <!DOCTYPE isopackager PUBLIC
    "-//jPOS/jPOS Generic Packager DTD 1.0//EN"
    "http://jpos.org/dtd/generic-packager-1.0.dtd">
    <!-- ISO 8583:1993 (ASCII) field descriptions for GenericPackager -->
    <isopackager>
    <isofield
    id="0"
    length="4"
    name="Message Type Indicator"
    class="org.jpos.iso.IFA_NUMERIC"/>
    <isofield
    id="1"
    length="16"
    name="Bitmap"
    class="org.jpos.iso.IFA_BITMAP"/>
    <isofield
    id="2"
    length="19"
    name="Primary Account number"
    class="org.jpos.iso.IFA_LLNUM"/>
    <isofield
    id="3"
    length="6"
    name="Processing Code"
    class="org.jpos.iso.IFA_NUMERIC"/>
    <isofield
    id="4"
    length="12"
    name="Amount, Transaction"
    class="org.jpos.iso.IFA_NUMERIC"/>
    <isofield
    id="5"
    length="12"
    name="Amount, Reconciliation"
    class="org.jpos.iso.IFA_NUMERIC"/>
    <isofield
    id="6"
    length="12"
    name="Amount, Cardholder billing"
    class="org.jpos.iso.IFA_NUMERIC"/>
    ...
    ...
    ...
    </isopackager>
```
    
org.jpos.iso.packager包有很多可以配置XML文件的packager这些配置文件XML通常在jpos/src/main/resources/packager目录下 
   

[注]：如果你开发过程中要自定义packager,我们强烈建议你用可配置XML的GenericPackager类，这会更加加单。
   
如果你用Q2去配置你的packagers,GenericPackager利用packager-config属性用来决定它的配置文件。

【关于配置文件存放位置】
这些基于XML的packager配置可以放在操作系统路径，或者是在JAR包的classpath中，GenericPackager有能力读取这些文件作为资源。
        
如果要支持嵌套消息，可以查看jpos/src/main/resources/org/jpos/iso/packager/genericpackager.dtd或者jpos/src/main/resources/packager/base1.xml这个文件

###3) 通过ISOChannel管理数据包协议
jPOS用一个叫做ISOChannel的接口来封装数据包协议的细节。  
ISOChannel用来发送和接收ISOMsg对象，它采用相等的设计模式，跟它类似的是ISOPackager实例。  
它拥有send和recevice方法，类似于packager的set和get方法。  
```java
   public voidsend (ISOMsg m) throwsIOException, ISOException;
   publicISOMsg receive() throwsIOException, ISOException;
   public voidsetPackager(ISOPackager p);
   publicISOPackager getPackager();
```
   
ISO有一些连接相关的方法如下：
```java
	 ...
	public voidconnect () throwsIOException;
	public voiddisconnect () throwsIOException;
	public voidreconnect() throwsIOException;
	public voidsetUsable(booleanb);
	public booleanisConnected();
	...
```

在运行时为了应用程序去构建jPOS组件，有一个单例的类叫做org.jpos.util.NameRegistrar，这个类使得你可以注册和获取对象的引用。在运行时期间ISOChannel接口提供了许多便利的方法通过他们的名字来访问ISOChannels.

```java
   ...
	public voidsetName (String name);
	publicString getName();
   ...
```
   

ISOChannel继承自ISOSource接口类如下：

```java
   public interfaceISOSource {
	public voidsend (ISOMsg m) throwsIOException, ISOException, VetoException;
	public booleanisConnected();
   }
```
	   
不同的交换信息采用不同的数据包协议。jPOS通过完全独立的ISOChannels实现类来封装它的功能。
它拥有很多实现，因此你可以很容易写你自己的，也许用BaseChannel作为超类或更加有益处。
   
ISOChannel接口实现类的例子如下：  

<pre>
   	Name 							Description
	ASCIIChannel 					4 bytes message length plus ISO-8583 data
	LogChannel 						Can be used to read jPOS’s logs and inject messages into
									other channels
	LoopbackChannel 					Every message sent gets received (possibly applying filters).
									Very useful for testing purposes.
	PADChannel 						Used to connect to X.25 packet assembler/dissamblers
	XMLChannel						jPOS Internal XML representation for ISO-8583 messages
</pre>  

注：查看org.jpos.iso.channel.\* 可以获得完整实现列表  
 
在所有的ISOChannel实现类之中，PADChannel这个类应该尤其被注意。大多数基于ISO-8583数据协议的
TCP/IP协议实现类利用不同类型的指示器很容易查看到消息。他们中间的大多数用一个packet长度头，因此
接收的实现类可以和已经给出的ISO-8583 packet分离。
......
	
	
	
如下一个ISOChannel类。
```java
	public class ISOChannelTestOne {
		public static void main(String[] args) throws ISOException, IOException {
			Logger logger = new Logger();
			logger.addListener (new SimpleLogListener (System.out));
			ISOChannel channel = new ASCIIChannel ("localhost", 7, new ISO87APackager());
			((LogSource)channel).setLogger (logger, "test-channel");
			channel.connect ();
			ISOMsg m = new ISOMsg ();
			m.setMTI ("0800");
			m.set (3, "000000");
			m.set (41, "00000001");
			m.set (70, "301");
			channel.send (m);
			ISOMsg r = channel.receive ();
			channel.disconnect ();
		}
	}
```

<pre>
在这个文档中我们将来可以看到很多例子更上面一样，都是简单的在mian方法里面实例化和配置jPOS的组件。
接下来我们要介绍Q2了，jPOS的组件装配器。
【注意】我们强烈要求和建议使用Q2来启动jPOS，它会是的你的生活更加容易。
Q2让你定义你自己的基于jPOS的应用程序，通过非常简单的，容易的创建和容易扩展的XML文件配置。
在我们讲Q2之前我们建议你最好像之前一样通过简单的代码来学习jPOS,而不是把它在生产环境运行。
另外，你会经常直接通过send和receive方法来处理channel.你可以采用MUX或者一个服务器ISOServer.

如果你看了ISOChannel的实现类，你会注意到他们大都继承自BaseChannel.
BaseChannel是一个抽象类，提供了很多钩子和一些方法的实现，当你写自己的channels的时候。当然你不是必须要
继承自BaseChannel类来自定义你自己的channels，但是你会发现它很有用。

依赖于你的数据包协议，你很有可能需要继承BaseChannel类仅仅重写如下这几个方法即可：
protected voidsendMessageLength(intlen) throwsIOException;
protected intgetMessageLength() throwsIOException, ISOException;

通过jpos/src/main/java/org/jpos/iso/channel/CSChannel.java就是一个例子。
</pre>
   
   
####3.2）Filtered Channels
许多ISOChannels实现自FilteredChannel,如下：
```java
   public interface FilteredChannel extends ISOChannel {
	public void addIncomingFilter (ISOFilter filter);
	public void addOutgoingFilter (ISOFilter filter);
	public void addFilter (ISOFilter filter);
	public void removeFilter (ISOFilter filter);
	public void removeIncomingFilter (ISOFilter filter);
	public void removeOutgoingFilter (ISOFilter filter);
	public Collection getIncomingFilters();
	public Collection getOutgoingFilters();
	public void setIncomingFilters (Collection filters);
	public void setOutgoingFilters (Collection filters);
    }
```

ISOFilter接口非常简单如下：
```java
   public interface	ISOFilter {
	public ISOMsg filter (ISOChannel channel, ISOMsg m, LogEvent evt) throwsVetoException;
   }
```

无论什么时候你添加一个filter到一个FilteredChannel,所有的发送信息和接收信息都会通过filter传输。
Filters给你机会通过channel利用ISOFilter去停止一个一直信息的发送和接收看下一一个非常简单的Filter：

```java
   public class DelayFilter implements ISOFilter, ReConfigurable {
	longdelay;
	publicDelayFilter() {
	super();
	delay = 0L;
	}
	/**
	* @param delay desired delay, expressed in milliseconds
	*/
	publicDelayFilter(longdelay) {
	super();
	this.delay = delay;
	}
	public voidsetConfiguration (Configuration cfg) {
	delay = cfg.getInt ("delay");
	}
	publicISOMsg filter (ISOChannel channel, ISOMsg m, LogEvent evt)
	{
	evt.addMessage ("<delay-filter delay=\""+delay+"\"/>");
	if(delay > 0L)
	ISOUtil.sleep(delay);
	returnm;
	}
	}
```

DelayFilter简单的应用了延迟到所有将要被发送和接收的channel。它可以用模仿远程主机延迟，可被测试的好工具。
但是filter方法有能力改变ISOMsgobject或者直接用一个新的替换它。一个便捷的日志事件提供。
之前的代码介绍了少数的类和接口，  至于Configuration和LogEvent，我们将很快讨论这部分非常重要。
   

jPOS带有很多通用计划filters:  

-  MD5Filter 用来验证mssage信息;
-  MacroFilter 用来扩展内部变量和程序装置.
-  XSLTFilter can 用来应用ISO-8583 mssage中的XSLT交易信息。

有一个非常流行的filter叫做BSHFilter,它可以执行BeanShell[http://www.beanshell.org]代码，运行时期间他可以不用重启系统来修改内部文件，提供了一个非常优秀的方式来进行迅速改变。这些特性使得它在测试期间非常受欢迎.BSH代码可以非常容易移植到JAVA.

> 【警告】我们已经看到了完整的实现自基于BSH的fIlter的应用。他们都太难以维护、
> 比在JAVA代码中实现商业逻辑更慢。我们鼓励你应用这些边界的脚本的能力作为一个工具，
> 在一个经常修改和测试的环境中，记住移动这些代码到JAVA中，一定要记住！！！！！！

###4）通过ISOServer接收连接
ISOServer监听一个固定端口为的是带来连接和接收他们、然后传输控制利用一个ISOChannel实现类。
一旦一个新的连接被接收到，一个ISOChannel就被创建了，一个受控的线程池线程会接收信息通过这个连接。
这些信息会通过一个ISORequestListener的实现类。
  
ISOServer的例子：
```java
  public class ISOServerTestOne {
	public static void main(String[] args) throws Exception {
		Logger logger = new Logger();
		logger.addListener(new SimpleLogListener(System.out));
		ServerChannel channel = new XMLChannel(new XMLPackager());
		((LogSource) channel).setLogger(logger, "channel");
		ISOServer server = new ISOServer(8000, channel, null);
		server.setLogger(logger, "server");
		new Thread(server).start();
	}
  }
```
ISOServer的构造方法的第三个参数是一个可选的线程池。你也可以放置一个NULL值在那，一个新的线程池会被创建，默认有100个线程。也就是new ThreadPool (1,100)。  

再重复一次，我们展示这个例子是带有教育意义的。在实际的生活应用中，你应该更想用 Q2的QServer组件替代这个ISOServer

为了测试之前的这个server测试程序（会监听在8000端口上），你可以用一个简单的telnet客户端，那里你可以输入一些XML格式的SO-8583信息。比如如下信息：  
>   $ telnet localhost 8000  
> 	Trying 127.0.0.1...  
> 	Connected to localhost.  
> 	Escape character is '^]'.  
   
现在你可以看一下你启动你的测试程序后打的日志
```xml
   <log realm="server" at="Fri May 17 08:11:34 UYT 2002.824">
	<iso-server>
	listening on port 8000
	</iso-server>
   </log>
```
   
此时，我们可以打开telnet窗口，在telnet窗口中输入如下标准ISO-8583格式的XML信息：
```xml
   <isomsg>
	<field id="0" value="0800"/>
	<field id="3" value="333333"/>
  </isomsg>
```

[注意]XMLChannel期望<isomsg>和</isomsg>出现在文件中的第一个位置
   
这个时候看控制台日志:
```xml
   <log realm="channel/127.0.0.1:1528" at="Sat Jan 04 18:48:47 CST 2014.437" lifespan="14594ms">
	  <receive>
	    <isomsg direction="incoming">
	      <!-- org.jpos.iso.packager.XMLPackager -->
	      <field id="0" value="0800"/>
	      <field id="3" value="333333"/>
	    </isomsg>
	  </receive>
   </log>
```
   
按照上面的说明，捏可以添加一个ISORequestListener到你的ISOServer，它可以帮你处理发送进来的信息。所以我们小小改动下我们测试程序来回答我们的信息。我们的测试类不得不实现ISORequestListener接口：  
```java
   public classTest implementsISORequestListener {
	...
	public booleanprocess (ISOSource source, ISOMsg m) {
	try{
	m.setResponseMTI ();
	m.set (39, "00");
	source.send (m);
	} catch(ISOException e) {
	e.printStackTrace();
	} catch(IOException e) {
	e.printStackTrace();
	}
	returntrue;
	}
	...
	...
	}
```

你必须得注册这个request listener到你的sever,你可以通过如下方式来做：
```java
server.addISORequestListener (newTest ());
```
    
改造后带监听器的完整程序如下：
```java
    public class ISOServerTestWithRequestListener implements ISORequestListener {

	public boolean process (ISOSource source, ISOMsg m) {
			try{
				m.setResponseMTI ();
				m.set (39, "00");
				source.send (m);
			} catch(ISOException e) {
			e.printStackTrace();
			} catch(IOException e) {
			e.printStackTrace();
			}
			return true;
		}
	
		public static void main (String[] args) throws Exception {
			Logger logger = new Logger ();
			logger.addListener (new SimpleLogListener (System.out));
			ServerChannel channel = new XMLChannel (new XMLPackager());
			((LogSource)channel).setLogger (logger, "channel");
			ISOServer server = new ISOServer (8000, channel, null);
			server.setLogger (logger, "server");
			server.addISORequestListener (new ISOServerTestWithRequestListener ());
			new Thread (server).start ();
		}
   }
```
   
现在我们尝试去telnet连接8000，然后发送另外一个ISO-8583格式的XML信息。你会得到一个回复
这个回复结果带上一个“00” field #39:  
```xml
	<log realm="channel/127.0.0.1:1558" at="Sat Jan 04 19:00:08 CST 2014.515" lifespan="6531ms">
	  <receive>
	    <isomsg direction="incoming">
	      <!-- org.jpos.iso.packager.XMLPackager -->
	      <field id="0" value="0800"/>
	      <field id="3" value="333333"/>
	    </isomsg>
	  </receive>
	</log>
```

```xml
	
	<log realm="channel/127.0.0.1:1558" at="Sat Jan 04 19:00:08 CST 2014.515">
	  <send>
	    <isomsg>
	      <!-- org.jpos.iso.packager.XMLPackager -->
	      <field id="0" value="0810"/>
	      <field id="3" value="333333"/>
	      <field id="39" value="00"/>
	    </isomsg>
	  </send>
	</log>
```


<pre>
   
[注：原理解读]ISOServer利用一个线程池，为了同时接收更多的连接。每一个SOCKET连接都是被单独的线程处理。


如果你的请求监听器实现类花费太长时间来回复，新的消息达到的时候，session不得不等待他们的回复。
为了解决这个问题，你的ISORequestListener应该在它自己的县城池里运行，以至于每个线程他的process方法可以处理队列中的请求

在你担心太多处理同时交易前，你应该HAPPY起来，因为jPOS有一个TransactionManager可以处理你以上的
那些问题，很快会讲到他，继续阅读。

ISOServer利用ISOChannel的实现类来发送来自于数据包中的ISOMsg。这些ISOChannels，可以
，应该今早的有一些关联的filters来描述他们。

在现代的jPOS应用程序中，ISOServer常常通过QServer服务来管理。ISORequestListener通常是一个
单一的实现来发送请求到TransactionManager。
</pre>


###5）Multiplexing an ISOChannel with a MUX   

想象一下，在很多POS终端同时接收到很多不同的请求的实现能力，然后通过ISOChannel发送他们到发行机构。
当然你可以为每一个交易创建一个socket连接，它常常用来启动一个socket连接（被ISOChannel实例处理）然后传输它。  
所以MUX是一个最基本的多路channel传输装置。当你初始化一个MUX，你仅仅需要发送一个请求等待响应即可。  
  
MUX接口内部定义如下：
```java
public interfaceMUX {
	publicISOMsg request (ISOMsg m, longtimeout) throwsISOException;
	public booleanisConnected();
} 
```
request方法使得一个通过ISOChannel被发送的请求排队，然后等待响应在指定的超时毫秒数内。
这个方法要么返回一个结果，要么就是NULL.
isConnected方法是一个自我解释方法，当channel连接被连接上的时候总是返回true


> 【注意】MUX接口可以有很多不同的实现。依赖于实现，isConnected返回配置的值可能不是很可靠的。
> （甚至它可以返回true当一个断开的channel的时候）
   
到目前为止，我们已经添加了不能协同的队列请求的能力，最新的MUX接口有另外一个请求的方法，会理解返回
同时会调用一个ISOResponseListener(带上一个可选的handBack Objec)如下：
```java
public interfaceMUX {
	...
	...
	public void request  (ISOMsg m, longtimeout, ISOResponseListener r, Object handBack) throwsISOException;
}
```
   
新的不能协同的方法调用MUX在QMUX实现中是可用的。  
ISOMUX有一个排队方法可以用来实现一个相似的不能协同的行为。  


为了发送响应到适当的发送线程，一个MUX实现用从原始ISOMsg选择的字段请求期望被ISOMsg响应代表。 
尽管不是MUX接口的一部分，实现比如QMUX（新的）和ISOMUX（旧的）有保护的方法疾走做getKey(ISOMsg m)
返回一个匹配的基于ISOMsg内容的匹配的key  


QMUX读取一个XML文件拥有 <key>nn,nn,nn</key>子元素可以轻松用来设置适当的匹配的key.  


默认的实现利用字段比如域41（终端ID）加上域11（连续跟踪号）来创建一个唯一的key.你可以重写getKey()
这样可以使用其他的域。  

###MUX 例子：

```java
...
...
MUX mux = (MUX) NameRegister.get ("mux.mymultiplexer");
...
...   


ISOMsg m = new ISOMsg();
m.setMTI ("0800");
m.set (11, "000001");
m.set (41, "00000001");
ISOMsg response = mux.request (m, 30000);
if(response != null) {
// you've got a response
} else{
// request has timed out
// you may want to reverse or retransmit
}
```
    
当你一个消息到达MUX下的ISOChannel，MUX实现会检查看消息的key是否已经注册了作为一个期间请求。
如果那个key匹配了一个期间请求，响应会被等待的线程处理。如果那个key已经被注册成了一个请求，或者响应
太久太晚，这样响应（依赖于配置）会被忽略的，一个ISORequestListener会被定义在队列空间（参见QMUX）

在很多情况下，同样的频道，客户端应发送很多请求等待响应可能会接收远端服务器的请求。
那些不匹配的来自于远端服务器的请求被代表到ISORequestListener。
     
ISORequestListener接口如下：
```java
public interfaceISORequestListener {
	public booleanprocess (ISOSource source, ISOMsg m); 
}
```
     
想象我们要回答0800请求到达我们的MUX，我们可以像如下这么实现：
```java
public class EchoHandler extends Log implements ISORequestListener {
	public boolean process(ISOSource source, ISOMsg m) {
		try {
			if ("0800".equals(m.getMTI())) {
				m.setResponseMTI();
				m.set(39, "00");
				source.send(m);
			}
		} catch (Exception e) {
			warn("echo-handler", e);
		}
		return true;
	}

}
```
       
       
       