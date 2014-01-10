#前提
很多人学习jPOS之后总是云里雾里，不明白client-simulator和server-simulator改如何启动，既然叫做客户端模拟器和服务器端模拟器，肯定得配合使用了，论顺序就是：
<pre>
1.启动服务器端，监听一个端口
2.启动客户端，往指定端口发送消息
</pre>
构建任何一个jPOS应用建议拷贝官网项目模板[jPOS-template-master.zip](https://github.com/jpos/jPOS-template/archive/master.zip)
<br/>
<font color='red'>本文假设读者已经具备git、gradle、maven相关方面知识</font>


# 启动服务器端(server-simulator) #
## 步骤一：安装一个jPOS-template拷贝
<pre>
执行命令：git clone https://github.com/jpos/jPOS-template.git
亦或是直接下载：https://github.com/jpos/jPOS-template/archive/master.zip
</pre>

## 步骤二：重命名下载的jPOS-template目录 ##
<pre>
执行命令：mv jPOS-template serversimulator
进入目录：cd serversimulator
</pre>

## 步骤三：编辑build.gradle文件 ##
<pre>
在build.gradle的dependencies或括号内添加（添加到第二行）：
compile group:'org.jpos.ee', name:'jposee-server-simulator', version:'2.0.2-SNAPSHOT'
</pre>

## 步骤四：执行gradle isntallResources命令 ##
<pre>
执行完成之后src/dist目录下可以查看着两个目录src/dist/deploy and src/dist/cfg,他们下面生成了很多新文件。
</pre>


## 步骤五：执行gradle run命令 ##
<pre>
 启动后，同样地，你可以可以参见这个目录：build/install/serversimulator，直接执行bin/q2 (或者是bin\q2.bat如果在windows上的hauler)也可启动server.
 下一步，重复上面步骤，重新下载jPOS-template命名为clientsimulator,之后将server换成client即可。
</pre>

## BTW:关于端口修改 ##

###服务器端文件###
serversimulator\build\install\serversimulator\deploy\05_serversimulator.xml
##
	<server class="org.jpos.q2.iso.QServer" logger="Q2"
	  name="simulator_10000">
	 <attr name="port" type="java.lang.Integer">10000</attr>
	 <channel class="org.jpos.iso.channel.XMLChannel"
	        logger="Q2" packager="org.jpos.iso.packager.XMLPackager">
	 </channel>
	 <request-listener class="org.jpos.bsh.BSHRequestListener" logger="Q2">
	  <property name="source" value="cfg/serversimulator.bsh" />
	 </request-listener>
	</server>

###客服端文件###
clientsimulator\build\install\clientsimulator\deploy\10_clientsimulator_channel.xml
##
	<channel-adaptor name='clientsimulator-adaptor'
	    class="org.jpos.q2.iso.ChannelAdaptor" logger="Q2">
	 <channel class="org.jpos.iso.channel.XMLChannel" logger="Q2"
	       packager="org.jpos.iso.packager.XMLPackager">
	
	  <property name="host" value="127.0.0.1" />
	  <property name="port" value="10000" />
	 </channel>
	 <in>clientsimulator-send</in>
	 <out>clientsimulator-receive</out>
	 <reconnect-delay>10000</reconnect-delay>
	</channel-adaptor>

以上两个文件都有port属性，可以根据需要进行修改


	     


作者：[@岁月静好--似水流年](http://weibo.com/u/1747720793)<br/>
2014-01-10 星期五 15:44:12 