### 内存 Dump 与分析

内存 Dump 分为 2 种方式：主动 Dump 和被动 Dump。

- 主动 Dump 的工具包括：jcmd、jmap、JVisualVM 等等。具体使用请参考相关工具部分。
- 被动 Dump 主要是：hprof，以及 `-XX:+HeapDumpOnOutOfMemoryError` 等参数。

更多方式请参考：

> https://www.baeldung.com/java-heap-dump-capture

关于 hprof 用户手册和内部格式，请参考 JDK 源码中的说明文档：

> http://hg.openjdk.java.net/jdk8u/jdk8u/jdk/raw-file/beb15266ba1a/src/share/demo/jvmti/hprof/manual.html#mozTocId848088

此外，常用的分析工具有：

- jhat：jhat 用来支持分析 dump 文件，是一个 HTTP/HTML 服务器，能将 dump 文件生成在线的 HTML 文件，通过浏览器查看。
- MAT：MAT 是比较好用的、图形化的 JVM Dump 文件分析工具。

#### **好用的分析工具：MAT**

**1. MAT 介绍**

MAT 全称是 Eclipse Memory Analyzer Tools。

其优势在于，可以从 GC root 进行对象引用分析，计算各个 root 所引用的对象有多少，比较容易定位内存泄露。MAT 是一款独立的产品，100MB 不到，可以从官方下载：[下载地址](https://www.eclipse.org/mat/)。

**2. MAT 示例**

现象描述：系统进行慢 SQL 优化调整之后上线，在测试环境没有发现什么问题，但运行一段时间之后发现 CPU 跑满，下面我们就来分析案例。

先查看本机的 Java 进程：

```
jps -v
```

假设 jps 查看到的 pid 为 3826。

Dump 内存：

```
jmap -dump:format=b,file=3826.hprof 3826
```

导出完成后，dump 文件大约是 3 个 G。所以需要修改 MAT 的配置参数，太小了不行，但也不一定要设置得非常大。

在 MAT 安装目录下，修改配置文件：

> MemoryAnalyzer.ini

默认的内存配置是 1024MB，分析 3GB 的 dump 文件可能会报错，修改如下部分：

```
-vmargs
-Xmx1024m
```

根据 Dump 文件的大小，适当增加最大堆内存设置，要求是 4MB 的倍数，例如改为：

```
-vmargs
-Xmx4g
```

双击打开 MemoryAnalyzer.exe，打开 MAT 分析工具，选择菜单 File –> Open File… 选择对应的 dump 文件。

选择 Leak Suspects Report 并确定，分析内存泄露方面的报告。

![bd3d81d4-d928-4081-a2f7-96c11de76178.png](https://images.gitbook.cn/fed3ce00-6eae-11ea-83c3-675e02f948ee)

**3. 内存报告**

然后等待，分析完成后，汇总信息如下：

![07acbdb7-0c09-40a5-b2c3-e7621a36870f.png](https://images.gitbook.cn/7bc10970-6f1b-11ea-ab1a-832c54b4f266)

分析报告显示，占用内存最大的问题根源 1：

![345818b9-9323-4025-b23a-8f279a99eb84.png](https://images.gitbook.cn/1c41ade0-6eaf-11ea-83c3-675e02f948ee)

占用内存最大的问题根源 2：

![07bbe993-5139-416a-9e6d-980131b649bf.png](https://images.gitbook.cn/249ca800-6eaf-11ea-9d5e-29b50a74a9eb)

占用内存最大的问题根源 3：

![7308f1b5-35aa-43e0-bbb4-05cb2e3131be.png](https://images.gitbook.cn/2dd61b40-6eaf-11ea-a6e5-c1244b77f602)

可以看到，总的内存占用才 2GB 左右。问题根源 1 和根源 2，每个占用 800MB，问题很可能就在他们身上。

当然，根源 3 也有一定的参考价值，表明这时候有很多 JDBC 操作。

查看问题根源 1，其说明信息如下：

```java
The thread org.apache.tomcat.util.threads.TaskThread
  @ 0x6c4276718 http-nio-8086-exec-8
keeps local variables with total size 826,745,896 (37.61%) bytes.

The memory is accumulated in one instance of
"org.apache.tomcat.util.threads.TaskThread"
loaded by "java.net.URLClassLoader @ 0x6c0015a40".
The stacktrace of this Thread is available. See stacktrace.

Keywords
java.net.URLClassLoader @ 0x6c0015a40
org.apache.tomcat.util.threads.TaskThread
```

**4. 解读分析**

大致解读一下，这是一个（运行中的）线程，构造类是 org.apache.tomcat.util.threads.TaskThread，持有了大约 826MB 的对象，占比为 37.61%。

所有运行中的线程（栈）都是 GC-Root。

点开 See stacktrace 链接，查看导出时的线程调用栈。

节选如下：

```java
Thread Stack

http-nio-8086-exec-8
  ...
  at org.mybatis.spring.SqlSessionTemplate.selectOne
  at com.sun.proxy.$Proxy195.countVOBy(Lcom/****/domain/vo/home/residents/ResidentsInfomationVO;)I (Unknown Source)
  at com.****.bi.home.service.residents.impl.ResidentsInfomationServiceImpl.countVOBy(....)Ljava/lang/Integer; (ResidentsInfomationServiceImpl.java:164)
  at com.****.bi.home.service.residents.impl.ResidentsInfomationServiceImpl.selectAllVOByPage(....)Ljava/util/Map; (ResidentsInfomationServiceImpl.java:267)
  at com.****.web.controller.personFocusGroups.DocPersonFocusGroupsController.loadPersonFocusGroups(....)Lcom/****/domain/vo/JSONMessage; (DocPersonFocusGroupsController.java:183)
  at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run()V (TaskThread.java:61)
  at java.lang.Thread.run()V (Thread.java:745)
```

其中比较关键的信息，就是找到我们自己的 package，如：

```
com.****.....ResidentsInfomationServiceImpl.selectAllVOByPage
```

并且其中给出了 Java 源文件所对应的行号。

分析问题根源 2，结果和根源 1 基本上是一样的。

当然，还可以分析这个根源下持有的各个类的对象数量。

点击根源 1 说明信息下面的 `Details »` 链接，进入详情页面。

查看其中的 “Accumulated Objects in Dominator Tree”：

![b5ff6319-a5d9-426f-99ef-19bd100fd80a.png](https://images.gitbook.cn/5ef3a760-6eaf-11ea-a6e5-c1244b77f602)

可以看到占用内存最多的是 2 个 ArrayList 对象。

鼠标左键点击第一个 ArrayList 对象，在弹出的菜单中选择 Show objects by class –> by outgoing references。

![6dbbb72d-ec2b-485f-bc8e-9de044b21b7d.png](https://images.gitbook.cn/6e41a730-6eaf-11ea-9d98-f7fceb2428d3)

打开 class_references 标签页：

![28fe37ed-36df-482a-bc58-231c9552638d.png](https://images.gitbook.cn/767c2100-6eaf-11ea-97af-c3b20af12573)

展开后发现 PO 类对象有 113 万个。加载的确实有点多，直接占用 170MB 内存（每个对象约 150 字节）。

事实上，这是将批处理任务，放到实时的请求中进行计算，导致的问题。

MAT 还提供了其他信息，都可以点开看看，也可以为我们诊断问题提供一些依据。

#### **JDK 内置故障排查工具：jhat**

jhat 是 Java 堆分析工具（Java heap Analyzes Tool）。在 JDK6u7 之后成为 JDK 标配。使用该命令需要有一定的 Java 开发经验，官方不对此工具提供技术支持和客户服务。

**1. jhat 用法**

```
jhat [options] heap-dump-file
```

参数：

- **_options_** 可选命令行参数，请参考下面的 [Options](https://gitbook.cn/writing/editor/5ad61d68ca27190f41b7a950#Options)。
- **_heap-dump-file_** 要查看的二进制 Java 堆转储文件（Java binary heap dump file）。如果某个转储文件中包含了多份 heap dumps，可在文件名之后加上 `#<number>` 的方式指定解析哪一个 dump，如：`myfile.hprof#3`。

**2. jhat 示例**

使用 jmap 工具转储堆内存、可以使用如下方式：

```
jmap -dump:file=DumpFileName.txt,format=b <pid>
```

例如：

```
jmap -dump:file=D:/javaDump.hprof,format=b 3614
Dumping heap to D:\javaDump.hprof ...
Heap dump file created
```

其中，3614 是 java 进程的 ID，一般来说，jmap 需要和目标 JVM 的版本一致或者兼容，才能成功导出。

如果不知道如何使用，直接输入 jmap，或者 `jmap -h` 可看到提示信息。

然后分析时使用 jhat 命令，如下所示：

```
jhat -J-Xmx1024m D:/javaDump.hprof
...... 其他信息 ...
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
```

使用参数 `-J-Xmx1024m` 是因为默认 JVM 的堆内存可能不足以加载整个 dump 文件，可根据需要进行调整。然后我们可以根据提示知道端口号是 7000，接着使用浏览器访问 http://localhost:7000/ 即可看到相关分析结果。

**3. 详细说明**

jhat 命令支持预先设计的查询，比如显示某个类的所有实例。

还支持 **对象查询语言**（OQL，Object Query Language），OQL 有点类似 SQL，专门用来查询堆转储。

OQL 相关的帮助信息可以在 jhat 命令所提供的服务器页面最底部。

如果使用默认端口，则 OQL 帮助信息页面为：

> http://localhost:7000/oqlhelp/

Java 生成堆转储的方式有多种：

- 使用 `jmap -dump` 选项可以在 JVM 运行时获取 heap dump（可以参考上面的示例）详情参见：[jmap(1)](https://docs.oracle.com/javase/jp/8/technotes/tools/unix/jmap.html#CEGBCFBC)。
- 使用 jconsole 选项通过 HotSpotDiagnosticMXBean 从运行时获得堆转储。请参考：[jconsole(1)](https://docs.oracle.com/javase/jp/8/technotes/tools/unix/jconsole.html#CACDDJCH) 以及 HotSpotDiagnosticMXBean 的接口描述：http://docs.oracle.com/javase/8/docs/jre/api/management/extension/com/sun/management/HotSpotDiagnosticMXBean.html。
- 在虚拟机启动时如果指定了 `-XX:+HeapDumpOnOutOfMemoryError` 选项，则抛出 **OutOfMemoryError** 时，会自动执行堆转储。
- 使用 hprof 命令。请参考：性能分析工具——HPROF 简介：[https://github.com/cncounter/translation/blob/master/tiemao*2017/20*hprof/20_hprof.md](https://github.com/cncounter/translation/blob/master/tiemao_2017/20_hprof/20_hprof.md)。

**4. Options 选项介绍**

- `-stack`，值为 false 或 true。关闭对象分配调用栈跟踪（tracking object allocation call stack）。如果分配位置信息在堆转储中不可用，则必须将此标志设置为 false，默认值为 true。
- `-refs`，值为 false 或 true。关闭对象引用跟踪（tracking of references to objects），默认值为 true。默认情况下，返回的指针是指向其他特定对象的对象，如反向链接或输入引用（referrers or incoming references），会统计/计算堆中的所有对象。
- `-port`，即 port-number。设置 jhat HTTP server 的端口号，默认值 7000。
- `-exclude`，即 exclude-file。指定对象查询时需要排除的数据成员列表文件。例如，如果文件列列出了 java.lang.String.value，那么当从某个特定对象 Object o 计算可达的对象列表时，引用路径涉及 java.lang.String.value 的都会被排除。
- `-baseline`：指定一个基准堆转储（baseline heap dump）。在两个 heap dumps 中有相同 object ID 的对象会被标记为不是新的。其他对象被标记为新的（new）。在比较两个不同的堆转储时很有用。
- `-debug`，值为 int 类型。设置 debug 级别，0 表示不输出调试信息，值越大则表示输出更详细的 debug 信息。
- `-version`：启动后只显示版本信息就退出。
- `-h`，即`-help`。显示帮助信息并退出. 同 `-h`。
- `-J <flag>`：因为 jhat 命令实际上会启动一个 JVM 来执行，通过 **-J** 可以在启动 JVM 时传入一些启动参数。例如，`-J-Xmx512m` 则指定运行 jhat 的 Java 虚拟机使用的最大堆内存为 512 MB。如果需要使用多个 JVM 启动参数，则传入多个 `-Jxxxxxx`。

### 参考

- [jmap 官方文档](https://docs.oracle.com/javase/jp/8/technotes/tools/unix/jmap.html#CEGBCFBC)
- [jconsole 官方文档](https://docs.oracle.com/javase/jp/8/technotes/tools/unix/jconsole.html#CACDDJCH)
- [性能分析工具——HPROF 简介](https://github.com/cncounter/translation/blob/master/tiemao_2017/20_hprof/20_hprof.md)
- [JDK 内置故障排查工具：jhat 简介](https://gitbook.cn/writing/editor/[https://github.com/cncounter/translation/blob/master/tiemao_2014/jhat/jhat.md](https://github.com/cncounter/translation/blob/master/tiemao_2014/jhat/jhat.md))
