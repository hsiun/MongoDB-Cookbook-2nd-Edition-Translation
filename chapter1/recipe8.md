# 作为副本集的一部分启动多个实例

在本菜谱中，我们将学习在同一主机上以集群的方式启动多个服务器。对于开发的目的或者是没有关键任务的应用来说，启动单个mongo服务器是够的。对于正式的产品部署，我们需要更高的可用性，当一个服务器实例故障的时候，另一个实例可以取代并且对数据的查询，插入和更新保持可用。集群是一个高级的概念，但我们不会在一个副本集中覆盖整个概念来评判它。在这里，我们只是浅尝辄止，在在本书后面管理员部分其他菜谱中了解更多详情。在本菜谱中，我们在同一台机器上启动多个mongo服务经常，用来测试。在产品环境中，它们将运行在相同甚至是不同数据中心不同的机器上（或者是虚拟机）。


让我们简略的了解下副本集的庐山真面目。如名所示，它是服务器的集合，这些服务器在数据项目上是其他每一个副本。让我们看下他们是如何保持和其他服务器同步的，至于内部实现我们将推迟到后面管理员部分的菜谱中，但是有一件事情要记住，写操作将只发生在一个节点上，这个节点就是primary。默认情况下所有的查询也将发生在primary上，虽然我们也许确实会允许在secondary实例上执行读操作。一个要记住的重要事实是，副本集的目的不是通过跨越副本集中多个节点分布式的读操作来实现扩展性。其唯一目标是保证高可用。


### 前提条件
尽管没有前提条件，回顾一下_使用命令行选项启动单节点实例_菜谱将确实让事情变得简单，防止万一你对这些启动mongo服务的命令行选项和它们的意义不了解。另外，在继续这个菜谱之前，在单个服务器设置中提到的必要的二进制文件和设置必须设置好。让我们一起数数我们需要准备什么。

我们将在我们本地主机（localhost）启动三个mongo进程（mongo服务示例）。

我们将创建三个数据目录，`/data/n1`，` /data/n2`，和`/data/n3`分别为`Node1`，`Node2`和`Node3`。类似的，我们将重定向日志到`/logs/n1.log`，`/logs/n2.log`和`/logs/n3.log`。下面的图片将给你一个集群看起来样子的大概想法：

### 如何做...
让我们看下具体步骤：

1. 为这三个节点分别创建数据和日志目录`/data/n1`，` /data/n2`，`/data/n3`和`logs`。在Windows平台，你可以为数据和日志分别选择`c:\data\n1`，`c:\data\n2`，`c:\data\n3`，和`c:\logs\`目录或者你选择的任意其他目录。保证mongo服务对这些目录有正确的写入权限来写入数据和日志。

2. 如下，启动这三个服务器。Windows平台的用户跳过`--fork`选项，这个选项不支持：
```
$ mongod --replSet repSetTest --dbpath /data/n1 --logpath /logs/
n1.log --port 27000 --smallfiles --oplogSize 128 --fork
$ mongod --replSet repSetTest --dbpath /data/n2 --logpath /logs/
n2.log --port 27001 --smallfiles --oplogSize 128 --fork
$ mongod --replSet repSetTest --dbpath /data/n3 --logpath /logs/
n3.log --port 27002 --smallfiles --oplogSize 128 –fork
```

3. 启动mongo shell，并且连接任意运行的mongo服务器。在我们这，我们连接到第一个（监听27000端口）。执行下面命令：
```
$ mongo localhost:27000
```

4. 连接到它之后，在mongo shell中尝试执行下插入操作：
```
> db.person.insert({name:'Fred', age:35})
```
如果副本集还没有初始化这个操作执行将会失败。在_如何生效..._可以找到更多信息。

5. 下一步开始配置副本集。我们通过在shell中准备JSON配置开始，如下：
```
cfg = {
 '_id':'repSetTest',
 'members':[
 {'_id':0, 'host': 'localhost:27000'},
 {'_id':1, 'host': 'localhost:27001'},
 {'_id':2, 'host': 'localhost:27002'}
 ]
}
```

6. 最后一步是通过准备好的配置初始化副本集，如下：
```
> rs.initiate(cfg)
```

7. 执行`rs.status()`之后几秒中在shell中查看状态。在这几秒中，它们中的一个将会成为primary，并且剩下的两个将会成为secondary。


### 如何生效...
我们已经介绍了使用命令行选项菜谱_单节点MongoDB安装_菜谱中的常见选项，并详细描述所有这些命令行选项。

当我们启动三个独立的mongod服务时，我们在文件系统中有三个独立数据目录。类似的，我们给每个进程一个独立的日志文件。然后我们通过指定的数据库和日志文件路径来启动这三个mongod进程。由于这是为测试设置的，并且启动在同一台机器上，我们使用`--smallfiles`和`--oplogSize`选项。当这三个进程运行在同一台主机上时，我们也选择明确的端口来避免端口冲突。我们这里选择的端口是27000,27001和27002。当我们在不同的主机上启动服务器，我们也行会也许不会选择另外的端口。无论何时我们都可以非常容易的选择使用默认端口。

`--fork`选项需要做一些解释。通过选择这个选项，我们从操作系统的shell中以后台进程启动服务，并在shell中重新获得控制权，在shell中我们可以启动更多这样的mongod进程或者是执行其他操作。在不使用`--fork`选项的情况下，我们不能在每个shell中启动多于一个进程，并且将需要在三个独立的shell中启动三个mongod进程。

如果我们看一眼在日志目录生成的日志，我们将会在其中看到下面行：
```
[rsStart] replSet can't get local.system.replset config from self or any
seed (EMPTYCONFIG)
[rsStart] replSet info you may need to run replSetInitiate --
rs.initiate() in the shell -- if that is not already done
```

尽管我们通过`--replSet`选项启动了三个mongod进程，我们仍能没有将他们和其他服务器一起配置为一个副本集。这个命令行选项只是用来在启动时告诉服务器，这个进程将运行为副本集的一部分。副本集的名字是和通过命令行提示符传入的这个选项的值一样的。这也解释了在副本集被初始化之前为什么在其中的一个节点上执行插入操作会失败。在mongo副本集中，只用有个primary节点可以执行所有的插入和查询操作。如图所示，*N1*节点展示为primary节点，并监听在27000端口等待客户端连接。其他的所以节点都是slave/secondary实例，这些节点从primary同步数据并且因此查询在他们上面默认也是禁止的。仅当primary挂了，secondary之一接手并成为primary节点。然而，正如我们在图片中展示的，从secondary查询数据是可能的；在下一个菜谱中，我们将介绍如何从secondary示例执行查询。


好的，现在剩下的就是通过分组我们开始的三个进程配置副本集。如下所示，这是通过前面定义的JSON对象实现的：
```
cfg = {
 '_id':'repSetTest',
 'members':[
 {'_id':0, 'host': 'localhost:27000'},
 {'_id':1, 'host': 'localhost:27001'},
 {'_id':2, 'host': 'localhost:27002'}
 ]
}
```

有两个区域，`_id`和`_name`，


### 更多...
浏览下_在shell连接到副本集以查询和插入数据_菜谱连接到副本集从shell中执行更多操作。复制不是如我们看到的这么简单的。查看管理员部分获取更多关于复制的高级菜谱。



### 看看其他
如果你在寻找转换单独实例为副本集，然后带有数据的实例需要先成为primary，并且空的secondary实例将被添加副本集中数据将被同步。点击下面的URL了解如何执行这些操作：

<http://docs.mongodb.org/manual/tutorial/convert-standalone-toreplica-set/>