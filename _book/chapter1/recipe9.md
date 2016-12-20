# 在shell连接到副本集以查询和插入数据

在前面的菜谱中，我们启动了一个三个mongod进程的副本集。在这个菜谱中，我们将在此基础上工作，通过使用mongo客户端应用连接到它，以执行查询，插入数据，并从客户端的角度了解一些副本集有趣的方面。


### 前提条件
本菜谱的前提条件是副本集应该被设置好并处于运行中。查看前面的菜谱_作为副本集的一部分启动多个实例_，获取如何启动副本集的详细信息。

### 如何做...
1. 这里我们将启动两个shell，一个给`PRIMARY`另一个给`SECONDARY`，在命令行执行下面命令：
```
> mongo localhost:27000
```

2. shell提示符告诉我们我们连接到的服务器是`PRIMARY`还是`SECONDARY`。副本集的名字将会展示在`a:`后面，跟着服务器状态。在这里，如歌副本集已经初始化，启动，并运行，我们将看到`repSetTest:PRIMARY>`或者`repSetTest:SECONDARY>`。

3. 假设我们连接到的一个服务是`secondary`，我们需要找到primary。在shell中执行rs.status()命令并找到`stateStr`区域。这将告诉我们哪个是primary服务器。使用mongo shell连接到这个服务。

4. 到这一步，我们应该会有两个运行的shell，一个连接到primary，另一个连接到secondary。

5. 在shell中连接到primary节点，执行下面插入：
```
repSetTest:PRIMARY> db.replTest.insert({_id:1, value:'abc'})
```

6. 这里没什么特别的。我们仅仅在集合中插入了一个小文档，这个文档将被用于复制测试。

7. 通过在primary上执行下列查询，我们将会货到下面结果：
```
repSetTest:PRIMARY> db.replTest.findOne()
{ "_id" : 1, "value" : "abc" }
```

8. 目前为止，一切都好。现在，我们将在连接到SECONDARY节点的shell，并执行下面命令：
```
repSetTest:SECONDARY> db.replTest.findOne()
```

在执行此命令时，我们将会在控制台看到下面错误：
```
{ "$err" : "not master and slaveOk=false", "code" : 13435 }
```

9. 在控制台执行下面命令：
```
repSetTest:SECONDARY> rs.slaveOk(true)
```

10. 在shell中执行我们第7步执行的查询。这将会获得下面结果：
```
repSetTest:SECONDARY>db.replTest.findOne()
{ "_id" : 1, "value" : "abc" }
```

11. 在secondary节点执行下面插入；这将不会成功并得到下面信息：
```
repSetTest:SECONDARY> db.replTest.insert({_id:1, value:'abc'})
not master
```

### 如何生效...
在这个菜谱中我们做了很多事情，我们将尝试对一些要记住的重要概念抛出一些亮点。

我们从shell连接到primary和secondary节点，并执行（我是说尽量执行）查询和插入操作。Mongo副本集的架构是由一个primary节点（只有一个，不多不少）和多个secondary节点组成的。所有的写入只发生在`PRIMARY`上。注意，复制不是可以扩展系统的分布读请求负载的机制。它的首要目的是保证数据的高可用性。默认情况下，我们不允许从secondary节点读数据。在第六步，我们简单的从primary节点插入数据，然后执行插入获取我们插入的文档。这很简单明了，并且和集群也没有关系。只要记住，我们从priamry中插入文档，然后查询回它。

在下一步，我们执行相同的查询，但是这一次，是从secondary的shell。默认情况下，`SECONDARY`上的查询是不被允许的。在复制数据的时候会有一点延迟，这可能是由于要复制的数据集很大，网络延迟，或者是硬件容量等等几个原因，因此，在secondary上的查下也许不会反应在primary上最后插入或者更新。然而，如果我们对于这种情况没什么问题，并且可以忍受数据复制时的少量延迟，所有我们要做的只是明确的通过执行一条命令，` rs.slaveOk()`或者`rs.slaveOk(true)`来允许在`SECONDARY`节点上对的查询。

最终，我们从节点上尝试插入数据到集合中。在任何情况下允许的，不管我们是否已经做了rs.slaveOk（）。 当调用rs.slaveOk（）时，它只允许从SECONDARY节点查询数据。 所有写操作仍然必须转secondary，然后流向primary。 复制的内部将在管理部分中以不同的菜谱涵盖。


### 看看其他
在下一个菜谱，_通过Java客户端连接到副本集以查询和插入数据_，是关于通过Java客户端连接到副本集的。
