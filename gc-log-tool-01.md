# 21/32GC 日志解读与分析（番外篇可视化工具）

通过前面的学习，我们发现 GC 日志量很大，人工分析太消耗精力了。由于各种 GC 算法的复杂性，它们的日志格式互相之间不太兼容。

有没有什么工具来减少我们的重复劳动呢? 这种轮子肯定是有现成的。比如 [GCEasy](https://gceasy.io/)、[GCViwer](https://github.com/chewiebug/GCViewer) 等等。

这一节我们就开始介绍一些能让我们事半功倍的工具。

### GCViwer 工具

下面我们介绍一款很好用的开源分析工具：GCViwer。

GCViewer 项目的 GitHub 主页是：

> https://github.com/chewiebug/GCViewer

#### **下载与安装**

然后我们在 Github 项目的 [releases 页面](https://github.com/chewiebug/GCViewer/releases) 中，找到并下载最新的版本，例如：[gcviewer-1.36.jar](http://sourceforge.net/projects/gcviewer/files/gcviewer-1.36.jar/download)。

Mac 系统可以直接下载封装好的应用：[gcviewer-1.36-dist-mac.zip](http://sourceforge.net/projects/gcviewer/files/gcviewer-1.36-dist-mac.zip/download)。下载，解压，安装之后首次打开可能会报安全警告，这时候可能需要到安全设置里面去勾选允许，例如：

![img](https://images.gitbook.cn/889b0b10-68d0-11ea-ae6a-117f7e51795f)

#### **测试案例**

先获取 GC 日志文件，方法同上面的 GCEasy 一样。

#### **启动 GCViewer**

可以通过命令行的方式启动 GCViewer 工具来进行分析：

```shell
java -jar gcviewer_1.3.4.jar
```

新版本支持用 java 命令直接启动。老版本可能需要在后面加上 GC 日志文件的路径。工具启动之后，大致会看到类似下面的图形界面：

![img](https://images.gitbook.cn/7e0f9c60-68d0-11ea-a6a7-51d4d3ff7d7c)

然后在图形界面中点击对应的按钮打开日志文件即可。现在的版本支持单个 GC 日志文件，多个 GC 日志文件，以及网络 URL。

当然，如果不想使用图形界面，或者没法使用图形界面的情况下，也可以在后面加上程序参数，直接将分析结果输出到文件。

例如执行以下命令：

```shell
java -jar gcviewer-1.36.jar /xxxx/gc.demo.log summary.csv chart.png
```

这会将信息汇总到当前目录下的 summary.csv 文件之中，并自动将图形信息保存为 chart.png 文件。

#### **结果报告**

在图形界面中打开某个 GC 日志文件。

![img](https://images.gitbook.cn/71a6bb20-68d0-11ea-a490-d3e65769b9bf)

上图中，Chart 区域是对 GC 事件的图形化展示。包括各个内存池的大小和 GC 事件。其中有 2 个可视化指标：蓝色线条表示堆内存的使用情况，黑色的 Bar 则表示 GC 暂停时间的长短。每个颜色表示什么信息可以参考 View 菜单。

![img](https://images.gitbook.cn/66b7f6c0-68d0-11ea-b5b9-6ffe049fb524)

从前面的图中可以看到，程序启动很短的时间后，堆内存几乎全部被消耗，不能顺利分配新对象，并引发频繁的 Full GC 事件. 这说明程序可能存在内存泄露，或者启动时指定的内存空间不足。

从图中还可以看到 GC 暂停的频率和持续时间。然后发现 GC 几乎不间断地运行。

右边也有三个选项卡可以展示不同的汇总信息：

![img](https://images.gitbook.cn/5c2f2020-68d0-11ea-b561-436a0f360bc4)

“**Summary**（摘要）” 中比较有用的是：

- “Throughput”（吞吐量百分比），吞吐量显示了有效工作的时间比例，剩下的部分就是 GC 的消耗
- “Number of GC pauses”（GC 暂停的次数）
- “Number of full GC pauses”（Full GC 暂停的次数）

以上示例中的吞吐量为 **13.03%**。这意味着有 **86.97%** 的 CPU 时间用在了 GC 上面。很明显系统所面临的情况很糟糕——宝贵的 CPU 时间没有用于执行实际工作，而是在试图清理垃圾。原因也很简单，我们只给程序分配了 512MB 堆内存。

下一个有意思的地方是“**Pause**”（暂停）选项卡：

![img](https://images.gitbook.cn/41f5b750-68d0-11ea-bb52-d9f36e195645)

其中“Pause”展示了 GC 暂停的总时间，平均值，最小值和最大值，并且将 total 与 minor/major 暂停分开统计。如果要优化程序的延迟指标，这些统计可以很快判断出暂停时间是否过长。

另外，我们可以得出明确的信息：累计暂停时间为 26.89 秒，GC 暂停的总次数为 599 次，这在 30 秒的总运行时间里那不是一般的高。

更详细的 GC 暂停汇总信息，请查看主界面中的“**Event details**”选项卡：

![img](https://images.gitbook.cn/2a10c5d0-68d0-11ea-ae6a-117f7e51795f)

从“**Event details**”标签中，可以看到日志中所有重要的GC事件汇总：普通 GC 的停顿次数和 Full GC 停顿次数，以及并发GC 执行数等等。

此示例中，可以看到一个明显的地方：Full GC 暂停严重影响了吞吐量和延迟，依据是 569 次 Full GC，暂停了 26.58 秒（一共执行 30 秒）。

可以看到，GCViewer 能用图形界面快速展现异常的 GC 行为。一般来说，图像化信息能迅速揭示以下症状：

- 低吞吐量。当应用的吞吐量下降到不能容忍的地步时，用于真正的业务处理的有效时间就大量减少。具体有多大的“容忍度”（tolerable）取决于具体场景。按照经验，低于 90% 的有效时间就值得警惕了，可能需要好好优化下 GC。
- 单次 GC 的暂停时间过长。只要有一次 GC 停顿时间过长，就会影响程序的延迟指标。例如，延迟需求规定必须在 1000ms 以内完成交易，那就不能容忍任何一次GC暂停超过 1000 毫秒。
- 堆内存使用率过高。如果老年代空间在 Full GC 之后仍然接近全满，程序性能就会大幅降低，可能是资源不足或者内存泄漏。这种症状会对吞吐量产生严重影响。

真是业界的福音——图形化展示的 GC 日志信息绝对是我们重磅推荐的。不用去阅读和分析冗长而又复杂的 GC 日志，通过图形界面，可以很容易得到同样的信息。不过，虽然图形界面以对用户友好的方式展示了重要信息，但是有时候部分细节也可能需要从日志文件去寻找。
