### jmap

jmap 主要用来 Dump 堆内存。当然也支持输出统计信息。

官方推荐使用 JDK 8 自带的 jcmd 工具来取代 jmap，但是 jmap 深入人心，jcmd 可能暂时取代不了。

查看 jmap 帮助信息：

> $ `jmap -help`

```
Usage:
    jmap [option] <pid>
        (连接到本地进程)
    jmap [option] <executable <core>
        (连接到 core file)
    jmap [option] [server_id@]<remote-IP-hostname>
        (连接到远程 debug 服务)

where <option> is one of:
    <none>               等同于 Solaris 的 pmap 命令
    -heap                打印 Java 堆内存汇总信息
    -histo[:live]        打印 Java 堆内存对象的直方图统计信息
                         如果指定了 "live" 选项则只统计存活对象，强制触发一次 GC
    -clstats             打印 class loader 统计信息
    -finalizerinfo       打印等待 finalization 的对象信息
    -dump:<dump-options> 将堆内存 dump 为 hprof 二进制格式
                         支持的 dump-options：
                           live         只 dump 存活对象，不指定则导出全部。
                           format=b     二进制格式(binary format)
                           file=<file>  导出文件的路径
                         示例：jmap -dump:live,format=b,file=heap.bin <pid>
    -F                   强制导出，若 jmap 被 hang 住不响应，可断开后使用此选项。
                         其中 "live" 选项不支持强制导出。
    -h | -help           to print this help message
    -J<flag>             to pass <flag> directly to the runtime system
```

常用选项就 3 个：

- `-heap`：打印堆内存（/内存池）的配置和使用信息。
- `-histo`：看哪些类占用的空间最多，直方图。
- `-dump:format=b,file=xxxx.hprof`：Dump 堆内存。

示例：看堆内存统计信息。

> $ `jmap -heap 4524`

输出信息：

```
Attaching to process ID 4524, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.65-b01

using thread-local object allocation.
Parallel GC with 4 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 2069889024 (1974.0MB)
   NewSize                  = 42991616 (41.0MB)
   MaxNewSize               = 689963008 (658.0MB)
   OldSize                  = 87031808 (83.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 24117248 (23.0MB)
   used     = 11005760 (10.49591064453125MB)
   free     = 13111488 (12.50408935546875MB)
   45.63439410665761% used
From Space:
   capacity = 1048576 (1.0MB)
   used     = 65536 (0.0625MB)
   free     = 983040 (0.9375MB)
   6.25% used
To Space:
   capacity = 1048576 (1.0MB)
   used     = 0 (0.0MB)
   free     = 1048576 (1.0MB)
   0.0% used
PS Old Generation
   capacity = 87031808 (83.0MB)
   used     = 22912000 (21.8505859375MB)
   free     = 64119808 (61.1494140625MB)
   26.32600715361446% used

12800 interned Strings occupying 1800664 bytes.
```

- Attached，连着；
- Detached，分离。

可以看到堆内存和内存池的相关信息。当然，这些信息有多种方式可以得到，比如 JMX。

看看直方图：

> $ `jmap -histo 4524`

结果为：

```
 num     #instances         #bytes  class name
----------------------------------------------
   1:         52214       11236072  [C
   2:        126872        5074880  java.util.TreeMap$Entry
   3:          5102        5041568  [B
   4:         17354        2310576  [I
   5:         45258        1086192  java.lang.String
......
```

简单分析，其中 `[C` 占用了 11MB 内存，没占用什么空间。

`[C` 表示 `chat[]`，`[B` 表示 `byte[]`，`[I` 表示 `int[]`，其他类似。这种基础数据类型很难分析出什么问题。

Java 中的大对象、巨无霸对象，一般都是长度很大的数组。

Dump 堆内存：

```shell
cd $CATALINA_BASE
jmap -dump:format=b,file=3826.hprof 3826
```

导出完成后，dump 文件大约和堆内存一样大。可以想办法压缩并传输。

分析 hprof 文件可以使用 jhat 或者 [mat](https://www.eclipse.org/mat/) 工具。
