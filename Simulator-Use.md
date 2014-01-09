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


作者：[@岁月静好--似水流年](http://weibo.com/u/1747720793)<br/>
2014-01-09 星期四 18:09:37 