

### 前言
 - 虚拟机内存分堆、程序计数器、方法区、本地方法栈、虚拟机栈五个区域.在运行java程序时,无论以何种方式运行(java启动、借助容器等方式),都涉及到底层的虚拟机内存的分配.
 如果启动过程没有指定虚拟机参数的配置,程序运行时使用各个参数的默认配置.在系统性能满足不了需求的时候,需要修改更改程序的启动配置来优化系统.
 以下通过**java命令启动**、**java启动参数**、**idea运行**等方面说明java运行时虚拟机参数配置
 - 版本: openjdk1.10, mac

### java命令运行
- java命令运行程序方式
	- java 编译后的class文件
	- java -jar app.jar

- 查看默认的虚拟机参数
	- 查看虚拟机参数
	``` 
	java -XX:+PrintFlagsFinal -version
	```
	- 查看默认堆大小
	```
	java -XX:+PrintFlagsFinal -version | grep HeapSize 
	```

- 运行时配置虚拟机运行参数示例
	- 	初始参数: 初始化堆1024M,最大堆1024M,Young区512M,每个线程的栈大小1024K,
	```
	java -Xms1024m -Xmx1024m  -Xmn512m -Xss1024k -Xlog:gc:gc.log Main
	```
	```
	java -jar -Xms1024m -Xmx1024m  -Xmn512m -Xss1024k -Xlog:gc:gc.log  
		contact-webapp.jar
	```

### java启动参数
- java启动参数分三种
	- 标准参数(-)
	- 非标准参数(-X)
	- 非Stable参数(-XX)
- 查看命令: 
	```
	java -help
	java -X -help
	```
- 标准参数:
	- -classpath
	- -cp
		告知jvm搜索目录名、jar文档名、zip文档名，之间用分号;分隔；使用-classpath后jvm将不再使用CLASSPATH中的类搜索路径，如果-classpath和CLASSPATH都没有设置，则jvm使用当前路径(.)作为类搜索路径。
		jvm搜索类的方式和顺序为：Bootstrap，Extension，User。
		Bootstrap中的路径是jvm自带的jar或zip文件，jvm首先搜索这些包文件，用System.getProperty(“sun.boot.class.path”)可得到搜索路径。
		Extension是位于JRE_HOME/lib/ext目录下的jar文件，jvm在搜索完Bootstrap后就搜索该目录下的jar文件，用System.getProperty(“java.ext.dirs”)可得到搜索路径。
		User搜索顺序为当前路径.、CLASSPATH、-classpath，jvm最后搜索这些目录，用System.getProperty(“java.class.path”)可得到搜索路径。

	- -jar   指定以jar包的形式执行一个应用程序, 要这样执行一个应用程序，必须让jar包的manifest文件中声明初始加载的Main-class
	- -verbose
		- -verbose:class
			输出jvm载入类的相关信息，当jvm报告说找不到类或者类冲突时可此进行诊断。
		- -verbose:gc
			输出每次GC的相关情况。
		- -verbose:jni
			输出native方法调用的相关情况，一般用于诊断jni调用错误信息。
- 非标准参数:
	- -Xloggc:file
		与-verbose:gc功能类似，只是将每次GC事件的相关情况记录到一个文件中，文件的位置最好在本地，以避免网络的潜在问题。若与verbose命令同时出现在命令行中，则以-Xloggc为准。
	- -Xms
		指定jvm堆的初始大小，默认为物理内存的1/64，最小为1M；可以指定单位，比如k、m，若不指定，则默认为字节。
	- -Xmx
		指定jvm堆的最大值，默认为物理内存的1/4或者1G，最小为2M；单位与-Xms一致。
	- -Xss
		设置单个线程栈的大小，一般默认为512k。
- 非stable参数:
	- 用途:
		- 行为参数：用于改变JVM的一些基础行为
		- 性能调优：用于JVM的性能调优
		- 调试参数：一般用于打开跟踪、打印、输出等JVM参数，用于显示JVM更加详细的信息
	- 行为参数:
		```
		-XX:-DisableExplicitGC	禁止调用System.gc()；但jvm的gc仍然有效
		-XX:+MaxFDLimit	最大化文件描述符的数量限制
		-XX:+ScavengeBeforeFullGC	新生代GC优先于Full GC执行
		-XX:+UseGCOverheadLimit	在抛出OOM之前限制jvm耗费在GC上的时间比例
		-XX:-UseConcMarkSweepGC	对老生代采用并发标记交换算法进行GC
		-XX:-UseParallelGC	启用并行GC
		-XX:-UseParallelOldGC	对Full GC启用并行，当-XX:-UseParallelGC启用时该项自动启用
		-XX:-UseSerialGC	启用串行GC
		-XX:+UseThreadPriorities	启用本地线程优先级
		```
	- 性能调优：
		```
		-XX:LargePageSizeInBytes=4m	设置用于Java堆的大页面尺寸
		-XX:MaxHeapFreeRatio=70	GC后java堆中空闲量占的最大比例
		-XX:MaxNewSize=size	新生成对象能占用内存的最大值
		-XX:MaxPermSize=64m	老生代对象能占用内存的最大值
		-XX:MinHeapFreeRatio=40	GC后java堆中空闲量占的最小比例
		-XX:NewRatio=2	新生代内存容量与老生代内存容量的比例
		-XX:NewSize=2.125m	新生代对象生成时占用内存的默认值
		-XX:ReservedCodeCacheSize=32m	保留代码占用的内存容量
		-XX:ThreadStackSize=512	设置线程栈大小，若为0则使用系统默认值
		-XX:+UseLargePages	使用大页面内存
		```
	- 调试参数：
		```
		-XX:-CITime	打印消耗在JIT编译的时间
		-XX:ErrorFile=./hs_err_pid.log	保存错误日志或者数据到文件中
		-XX:-ExtendedDTraceProbes	开启solaris特有的dtrace探针
		-XX:HeapDumpPath=./java_pid.hprof	指定导出堆信息时的路径或文件名
		-XX:-HeapDumpOnOutOfMemoryError	当首次遭遇OOM时导出此时堆中相关信息
		-XX:OnError=”;”	出现致命ERROR之后运行自定义命令
		-XX:OnOutOfMemoryError=”;”	当首次遭遇OOM时执行自定义命令
		-XX:-PrintClassHistogram	遇到Ctrl-Break后打印类实例的柱状信息，与jmap -histo功能相同
		-XX:-PrintConcurrentLocks	遇到Ctrl-Break后打印并发锁的相关信息，与jstack -l功能相同
		-XX:-PrintCommandLineFlags	打印在命令行中出现过的标记
		-XX:-PrintCompilation	当一个方法被编译时打印相关信息
		-XX:-PrintGC	每次GC时打印相关信息
		-XX:-PrintGC Details	每次GC时打印详细信息
		-XX:-PrintGCTimeStamps	打印每次GC的时间戳
		-XX:-TraceClassLoading	跟踪类的加载信息
		-XX:-TraceClassLoadingPreorder	跟踪被引用到的所有类的加载信息
		-XX:-TraceClassResolution	跟踪常量池
		-XX:-TraceClassUnloading	跟踪类的卸载信息
		-XX:-TraceLoaderConstraints	跟踪类加载器约束的相关信息
		```




	参数名称      											|说明
	--|:--:													|
	-Xms128m												|Java Heap初始值，Server端JVM最好将-Xms和-Xmx设为相同值                                                           
	-Xmx1000m												|Java Heap最大值，默认值为物理内存的1/4，最佳设值应该视物理内存大小及计算机内其他内存开销而定
	-XX:ReservedCodeCacheSize=240m							|预留保存代码的内存空间大小
	-XX:+UseCompressedOops									|在JDK 1.6 Update 14之 后，提供了普通对象指针压缩功能(详细可见java虚拟机第二版说明)		
	-Dfile.encoding=UTF-8									|指定编码
	-XX:+-UseConcMarkSweepGC 								|老年代使用CMS垃圾回收策略（并发标记清除）Concurrent Mark Sweep
	-XX:SoftRefLRUPolicyMSPerMB=50							|每兆堆空闲空间中SoftReference的存活时间LRU（Least Recently Used）最近最少使用。意思是最近最少被引用的软引用将在50秒后被JVM清除。单位为秒。
	-ea 													|ea 开启断言 -da 禁止断言
	-Dsun.io.useCanonCaches=false							|是否关闭Canon缓存
	-Djava.net.preferIPv4Stack=true 						|如果在使用ipv4的机器上运行启用了ipv6的系统，那么此参数设为true才能获取机器的完整机器名
	-Djdk.http.auth.tunneling<br>.disabledSchemes="" 		|根据域名自动下载https服务端发送过来的证书并保存成文件
	-XX:+HeapDumpOnOutOfMemoryError 						|虚拟机在OOM异常出现之后自动生成Dump文件
	-XX:-OmitStackTraceInFastThrow 							|省略异常栈信息从而快速抛出
	-Xverify:none											|禁止字节码验证过程
	-XX:ErrorFile=$USER_HOME<br>/java_error_in_idea_%p.log 	|
	-XX:HeapDumpPath=$USER_HOME<br>/java_error_in_idea.hprof|




### 说明:
- 初始化堆和最大堆大小配置:  
	[jdk8文档](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/parallel.html#default_heap_size)  
	[JVM启动参数-Xmx的默认值是多少](https://segmentfault.com/q/1010000007235579)

### 参考文献:
[JVM启动参数大全及默认值](https://www.javatt.com/p/48231)  
[idea.vmoptions参数详解](https://blog.csdn.net/qq_29519041/article/details/88415140)  
Java虚拟机(第二版)-周志明  
[jdk8文档](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/parallel.html#default_heap_size)  
[JVM启动参数-Xmx的默认值是多少](https://segmentfault.com/q/1010000007235579)










