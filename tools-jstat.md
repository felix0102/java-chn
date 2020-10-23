### jstat

jstat 用来监控 JVM 内置的各种统计信息，主要是内存和 GC 相关的信息。

查看 jstat 的帮助信息，大致如下：

> $ `jstat -help`

```
Usage: jstat -help|-options
       jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

Definitions:
  <option>      可用的选项，查看详情请使用 -options
  <vmid>        虚拟机标识符，格式：<lvmid>[@<hostname>[:<port>]]
  <lines>       标题行间隔的频率.
  <interval>    采样周期，<n>["ms"|"s"]，默认单位是毫秒 "ms"
  <count>       采用总次数
  -J<flag>      传给jstat底层JVM的 <flag> 参数
```

再来看看 `<option>` 部分支持哪些选项：

> $ `jstat -options`

```
-class
-compiler
-gc
-gccapacity
-gccause
-gcmetacapacity
-gcnew
-gcnewcapacity
-gcold
-gcoldcapacity
-gcutil
-printcompilation
```

简单说明这些选项，不感兴趣可以跳着读。

- `-class`：类加载（Class loader）信息统计。
- `-compiler`：JIT 即时编译器相关的统计信息。
- `-gc`：GC 相关的堆内存信息，用法：`jstat -gc -h 10 -t 864 1s 20`。
- `-gccapacity`：各个内存池分代空间的容量。
- `-gccause`：看上次 GC、本次 GC（如果正在 GC 中）的原因，其他输出和 `-gcutil` 选项一致。
- `-gcnew`：年轻代的统计信息（New = Young = Eden + S0 + S1）。
- `-gcnewcapacity`：年轻代空间大小统计。
- `-gcold`：老年代和元数据区的行为统计。
- `-gcoldcapacity`：old 空间大小统计。
- `-gcmetacapacity`：meta 区大小统计。
- `-gcutil`：GC 相关区域的使用率（utilization）统计。
- `-printcompilation`：打印 JVM 编译统计信息。

实例：

```shell
jstat -gcutil -t 864
```

`-gcutil` 选项是统计 GC 相关区域的使用率（utilization），结果如下：

| Timestamp  | S0   | S1    | E     | O     | M     | CCS   | YGC    | YGCT    | FGC  | FGCT  | GCT     |
| :--------- | :--- | :---- | :---- | :---- | :---- | :---- | :----- | :------ | :--- | :---- | :------ |
| 14251645.5 | 0.00 | 13.50 | 55.05 | 71.91 | 83.84 | 69.52 | 113767 | 206.036 | 4    | 0.122 | 206.158 |

`-t` 选项的位置是固定的，不能在前也不能在后。可以看出是用于显示时间戳，即 JVM 启动到现在的秒数。

简单分析一下：

- Timestamp 列：JVM 启动了 1425 万秒，大约 164 天。
- S0：就是 0 号存活区的百分比使用率。0% 很正常，因为 S0 和 S1 随时有一个是空的。
- S1：就是 1 号存活区的百分比使用率。
- E：就是 Eden 区，新生代的百分比使用率。
- O：就是 Old 区，老年代。百分比使用率。
- M：就是 Meta 区，元数据区百分比使用率。
- CCS：压缩 class 空间（Compressed class space）的百分比使用率。
- YGC（Young GC）：年轻代 GC 的次数。11 万多次，不算少。
- YGCT 年轻代 GC 消耗的总时间。206 秒，占总运行时间的万分之一不到，基本上可忽略。
- FGC：FullGC 的次数，可以看到只发生了 4 次，问题应该不大。
- FGCT：FullGC 的总时间，0.122 秒，平均每次 30ms 左右，大部分系统应该能承受。
- GCT：所有 GC 加起来消耗的总时间，即 YGCT + FGCT。

可以看到，`-gcutil` 这个选项出来的信息不太好用，统计的结果是百分比，不太直观。

再看看 `-gc` 选项，GC 相关的堆内存信息。

```shell
jstat -gc -t 864 1s
jstat -gc -t 864 1s 3
jstat -gc -t -h 10 864 1s 15
```

其中的 `1s` 占了 `<interval>` 这个槽位，表示每 1 秒输出一次信息。

`1s 3` 的意思是每秒输出 1 次，最多 3 次。

如果只指定刷新周期，不指定 `<count>` 部分，则会一直持续输出。 退出输出按 `CTRL+C` 即可。

`-h 10` 的意思是每 10 行输出一次表头。

结果大致如下：

| Timestamp  | S0C    | S1C    | S0U   | S1U  | EC     | EU     | OC      | OU     | MC      | MU      | YGC    | YGCT    | FGC  | FGCT  |
| :--------- | :----- | :----- | :---- | :--- | :----- | :----- | :------ | :----- | :------ | :------ | :----- | :------ | :--- | :---- |
| 14254245.3 | 1152.0 | 1152.0 | 145.6 | 0.0  | 9600.0 | 2312.8 | 11848.0 | 8527.3 | 31616.0 | 26528.6 | 113788 | 206.082 | 4    | 0.122 |
| 14254246.3 | 1152.0 | 1152.0 | 145.6 | 0.0  | 9600.0 | 2313.1 | 11848.0 | 8527.3 | 31616.0 | 26528.6 | 113788 | 206.082 | 4    | 0.122 |
| 14254247.3 | 1152.0 | 1152.0 | 145.6 | 0.0  | 9600.0 | 2313.4 | 11848.0 | 8527.3 | 31616.0 | 26528.6 | 113788 | 206.082 | 4    | 0.122 |

上面的结果是精简过的，为了排版去掉了 GCT、CCSC、CCSU 这三列。看到这些单词可以试着猜一下意思，详细的解读如下：

- Timestamp 列：JVM 启动了 1425 万秒，大约 164 天。
- S0C：0 号存活区的当前容量（capacity），单位 kB。
- S1C：1 号存活区的当前容量，单位 kB。
- S0U：0 号存活区的使用量（utilization），单位 kB。
- S1U：1 号存活区的使用量，单位 kB。
- EC：Eden 区，新生代的当前容量，单位 kB。
- EU：Eden 区，新生代的使用量，单位 kB。
- OC：Old 区，老年代的当前容量，单位 kB。
- OU：Old 区，老年代的使用量，单位 kB。 （需要关注）
- MC：元数据区的容量，单位 kB。
- MU：元数据区的使用量，单位 kB。
- CCSC：压缩的 class 空间容量，单位 kB。
- CCSU：压缩的 class 空间使用量，单位 kB。
- YGC：年轻代 GC 的次数。
- YGCT：年轻代 GC 消耗的总时间。 （重点关注）
- FGC：Full GC 的次数
- FGCT：Full GC 消耗的时间。 （重点关注）
- GCT：垃圾收集消耗的总时间。

最重要的信息是 GC 的次数和总消耗时间，其次是老年代的使用量。

在没有其他监控工具的情况下， jstat 可以简单查看各个内存池和 GC 的信息，可用于判别是否是 GC 问题或者内存溢出。

