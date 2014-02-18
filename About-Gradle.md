#关于gradle构建：

> A typical use case would be to set the HTTP proxy in  and the JAVA_OPTS
> memory options in GRALDE_OPTS . 
> 
> Those variables can also be set at the beginning of the  or  GRADLE_OPTS gradle gradlew script.

- 可以共通过swith --no-demon 使得daemon守护进程不起作用
- 可以通过forums.gradle.org论坛进行问题提问和回答


##基本的构建脚本
build.gradle这个文件就是构建脚本

> vi build.grale  

```gradle
task hello {
	doLast {
		println 'Hello world!'
	}
}
```

> gradle -q hello  

如上内容也可以简写成：  
```gradle
task hello >> {
	println 'Hello world!'
}
```

-q这个参数表示打出hello的输出结果  

gradle里面的tasks就相当于ant里面的targets  

如果我们经常讨论task指的就是gradle的task  

我们发现与ant不同的是，gralde的构建脚本中的任务模块内容可以直接编写groovy代码，非常方便。  


##构建任务的依赖
```gradle
task hello << {
	println 'Hello world!'
}
task intro(dependsOn: hello) << {
	println "I'm Gradle"
}
```
> gradle  intro 
> 的时候会出将hello任务中的结果先打出来，其实这样的依赖跟ANT的target依赖效果一样，被依赖的先执行

##延迟依赖
指的依赖的Task在本task定义之后，这样引用的时候要在任务名称两边加上‘’

```gradle
task taskX(dependsOn: 'taskY' ) << {
	println 'taskX'
}
task taskY << {
	println 'taskY'
}
```


##动态任务
也就是任务名称是动态的，是属于groovy执行过程中某个变量产生的名称  

```gradle
4.times { counter ->
    task "task$counter" << {
        println "I'm task number $counter"
    }
}
```
> gradle -q task1  
打印结果是：'m task number task1


##操作已经存在的任务
也就是创建好的任务我们可以通过API的方式调用这些任务[添加依赖]  

```gradle
4.times { counter ->
    task "task$counter" << {
        println "I'm task number $counter"
    }
}
task0.dependsOn task2, task3
``

> gradle -q task0  
> I'm task number 2  
> I'm task number 3  
> I'm task number 0  

##操作已经存在的任务
也就是创建好的任务我们可以通过API的方式调用这些任务[添加行为action]

```gradle
task hello << {
    println 'Hello Earth'
}
hello.doFirst {
    println 'Hello Venus'
}
hello.doLast {
    println 'Hello Mars'
}
hello << {
    println 'Hello Jupiter'
}
```

Output of gradle -q hello
> gradle -q hello  
> Hello Venus  
> Hello Earth  
> Hello Mars  
> Hello Jupiter  

提示： << 这个标志是.doLast这个action的别名，效果一样


##给任务添加属性  

```gradle
task myTask {
    ext.myProperty = "myValue"
}

task printTaskProperties << {
    println myTask.myProperty
}
```

额外的属性仅限于在task中

##在gradle脚本中调用ant脚本任务

##在gradle脚本中使用方法
既然是groovy代码，当然可以添加方法了

```gradle
task checksum << {
    fileList('../antLoadfileResources').each {File file ->
        ant.checksum(file: file, property: "cs_$file.name")
        println "$file.name Checksum: ${ant.properties["cs_$file.name"]}"
    }
}

task loadfile << {
    fileList('../antLoadfileResources').each {File file ->
        ant.loadfile(srcFile: file, property: file.name)
        println "I'm fond of $file.name"
    }
}

File[] fileList(String dir) {
    file(dir).listFiles({file -> file.isFile() } as FileFilter).sort()
}
```


##指定默认的task任务名

```gradle
defaultTasks 'clean', 'run'

task clean << {
    println 'Default Cleaning!'
}

task run << {
    println 'Default Running!'
}

task other << {
    println "I'm not a default task!"
}
```

使用defaultTasks指定任务名称，在脚本第一行，多个任务名称间使用,分割，任务名称加上双引号使用执行输出的时候就不需要加上具体task名称了，因为已经有默认了，内部自己会去找.
Output of gradle -q
> gradle -q  
> Default Cleaning!  
> Default Running!  


