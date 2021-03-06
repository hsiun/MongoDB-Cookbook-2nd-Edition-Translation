# 使用命令行选项启动单节点实例

在本菜谱中，我们将看看如何使用一些命令行选项来启动单节点服务。我们将看一个例子，在例子中我们将做下面这些事：

- 启动服务，监听27000端口
- 日志将会写到`/logs/mongo.log`
- 数据库目录是`/data/mongo/db`

如果服务器已经以开发的目的启动了，我们将不会预分配全量的数据库文件。（不久我们将会知道这是什么意思。）


### 前提条件
如果你已经看过且实践过_单节点MongoDB安装_菜谱，只需要知道这些就可以了。如果所有这些前提条件已经准备好了，那么完成这个食谱就没有问题。


### 如何做
1. 为数据库准备的`/data/mongo/db`目录和为日志准备的`/logs/`应该创建好，并且以适当的写入权限出现在文件系统中。

2. 执行以下命令：
```
> mongod --port 27000 --dbpath /data/mongo/db –logpath /logs/
mongo.log --smallfiles
```


### 如何生效...
好的，这个不是很难，且和上一个菜谱很相像，但是这一次我们有很多另外的命令行选项。MongoDB实际上支持很少的启动时参数，且我们将看到一个在我的观念里最常见和重要的参数列表：

| 选项              | 描述                                      | 
|:----------------- |:------------------------------------------| 
| --help or -h      | 这个选项用来打印各个可用的启动是选项信息  | 
| --config or -f    | 


### 更多信息...
使用`--help`或者`-h`选项，获取有关可用选项的详尽列表。这个选项列表不是详尽无疑的，在后面的菜谱中随着我们需要我们将看到更多参数被提出来。在下一个菜谱中，我们将看看如何使用配置文件来替代命令行参数。

### 看看其他
- _使用配置文件中的选项安装单节点MongoDB_展示了使用配置文件来提供启动选项
- _作为副本集的一部分启动多个实例_启动了一个副本集
- _启动一个两个分片的简单分片环境_设置了一个分片环境
