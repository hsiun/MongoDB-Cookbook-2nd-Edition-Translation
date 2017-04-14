# mongostat和mongotop的用法

你可能会发现他们的名字和两个常用的Unix命令相识，`iostat`和`top`。在MongoDB中，`mongostat`和`mongotop`所做的工作和这两个Unix命令是非常相似的，另外不用猜，这两个命令就是用来监控mongo实例的。


### 准备 ###
在本菜谱中，我们将在单独的mongo实例上模拟一些操作，通过运行脚本来尝试保持你服务器处于忙碌状态，然后哎其他终端运行这些工具来监控db实例。

你需要启动一个监听任意端口等待客户端连接的单独服务器；在本例中，我们将继续使用默认27017端口。如果还不知道如何启动单独服务，参考第一章_安装和启动服务器_中的_安装单节点MongoDB_菜谱。另外我们还需要从Packt站点下载`KeepServeBusy.js`脚本，并将它保存在本地磁盘中方便执行的地方。另外，假设你的mongo安装的bin目录存在于操作系统的路径变量中（ it is assumed that the bin directory of your mongo installation
is present in the path variable of your operating system）。如果不是这样，那么这些命令需要在shell中使用绝对路径执行。这两个工具`mongostat`和`mongotop`是mongo安装的标准配置。


### 实现 ###
1. 启动MongoDB服务，并让他监听在默认端口等待连接。

2. 在其他终端中如下所示执行我们提供的JavaScript脚本`KeepServerBusy.js`：
```
$ mongo KeepServerBusy.js –quiet
```

3. 打开一个新的操作系统终端，并执行下面命令：
```
$ mongostat
```

4. 截下某一时刻的输出内容，然后按下Ctrtl+C停止避免捕获太多状态。保持终端打开，或者复制状态到另外一个文件。

5. 现在，执行下面命令关闭终端：
```
$ mongotop
```

6. 捕获某时刻输出的内容，并按下Ctrl+C停止命令，停止捕获更多状态。保持终端打开或者复制状态到其他文件。

7. 在执行JavaScript脚本`KeepServerBusy.js `保持服务器忙碌状态的shell中按下Ctrl+C来停止这个脚本。


### 工作原理 ###
让我们一起看下我们在这两个命令下的截图。

我们从分析`mongstat`开始。在我的笔记本中，使用`mongostat`的截图如下所示：
```
mongostat
connected to: 127.0.0.1
insert query update delete getmore command flushes mapped vsize res
faults idx miss % qr|qw ar|aw netIn netOut conn time
 1000 1 950 1000 1 1|0 0 624.0M 1.4G 50.0M
0 0 0|0 0|1 431k 238k 2 08:59:21
 1000 1 1159 1000 1 1|0 0 624.0M 1.4G 50.0M
0 0 0|0 0|0 468k 252k 2 08:59:22
 1000 1 984 1000 1 1|0 0 624.0M 1.4G 50.0M
0 0 0|0 0|1 437k 240k 2 08:59:23
 1000 1 1066 1000 1 1|0 0 624.0M 1.4G 50.0M
0 0 0|0 0|1 452k 246k 2 08:59:24
 1000 1 944 1000 1 2|0 0 624.0M 1.4G 50.0M
0 0 0|0 0|1 431k 237k 2 08:59:25
 1000 1 1149 1000 1 1|0 0 624.0M 1.4G 50.0M
0 0 0|0 0|1 466k 252k 2 08:59:26
 1000 2 1015 1053 2 1|0 0 624.0M 1.4G 50.0M
0 0 0|0 0|0 450k 293k 2 08:59:27
```

你也许会选择查看下脚本`KeepServerBusy.js `做了什么来保持服务器忙碌。这个脚本所做的就是在集合`monitoringTest`中插入1000个文档，然后一个个的给他们设置新键更新它们，对所有文档执行查询和遍历，最后一个个的删除他们并且基本上是一个写密集的操作。

这个内容封装的输出看起来确实比较丑，但是让我们一个个的分析下这些区域，并且看下那些是要关注的。


<!-- 省略表格 -->

如果`mongostat`连接到副本集中的primary或者secondary节点将会由更多区域。作为一个作业，一旦收集统计信息或独立实例，启动副本集服务器并执行相同的脚本来保持服务器的繁忙。 使用mongostat连接到主实例和辅助实例，并查看捕获的不同统计信息。


作为`mongostat`的一部分，我们也使用了`mongotop'工具来捕获状态。我们一起看下它的输出并且了解下它们的意义：
```
$>mongotop
connected to: 127.0.0.1
 ns total read
write
2014-01-15T17:55:13
 test.monitoringTest 899ms 1ms
898ms
 test.system.users 0ms 0ms
0ms
 test.system.namespaces 0ms 0ms
0ms
 test.system.js 0ms 0ms 0ms
 test.system.indexes 0ms 0ms
0ms
 ns total read
write
2014-01-15T17:55:14
 test.monitoringTest 959ms 0ms
959ms
 test.system.users 0ms 0ms 0ms
 test.system.namespaces 0ms 0ms
0ms
 test.system.js 0ms 0ms 0ms
 test.system.indexes 0ms 0ms
0ms
 ns total read
write
2014-01-15T17:55:15
 test.monitoringTest 954ms 1ms
953ms
 test.system.users 0ms 0ms
0ms
 test.system.namespaces 0ms 0ms
0ms
 test.system.js 0ms 0ms 0ms
 test.system.indexes 0ms 0ms
0ms
```

<!-- 省略了部分内容 -->
### 学习更多 ###
+ 在菜谱_获取当前执行的操作并杀死它们_，我们将学习如何重shell
中获取当前正在执行的操作，并且如果需要的话终止它们。

+ 在菜谱_使用profiler剖析操作_中，我们将看到如何使用Mongodb的内置分析功能来记录操作执行时间。