# 重命名集合

你是否遇到这样的场景，当你在关系数据库中命名了一个表，而之后的某一时间点感觉到可以命名的更好？或者是你工作的组织在稍后意识到表的命令变得很混乱，因此在命名上执行一些强制标准？关系型数据库有一些其自己的方法来重命名一个表，数据库管理员可以替你完成。

我们在这里提出了一个问题。在Mongo的世界中，集合是类似于表的，有办法在创建集合之后在重命名它吗？在本菜谱中，我们将通过重命名一个含有数据的已存在集合来了解Mongo的这个特征。

### 准备 ###
为了执行这个集合重命名的是要，我们需要一个运行的MongoDB实例。有关如何启动服务器的说明，请参阅第一章_安装和启动服务器_中的_安装单节点MongoDB_菜谱。具体的操作我们将会在mongo shell中执行。

### 实现 ###
1. 一旦服务器启动，保证它处于监听默认27017端口等待客户端连接的状态，在shell中执行下面的命令：
```
> mongo
```

2. 连接上后，使用默认的测试数据库。让我们使用一些测试数据创建一个测试集合。这个集合将被命名为：`sloppyNamedCollection`。
```
> for(i = 0 ; i < 10 ; i++) { db.sloppyNamedCollection.
insert({'i':i}) };
```

3. 测试数据现在穿件好了（我们可以通过查询集合`sloppyNamedCollection`确认）。

4. 重命名集合为`neatNamedCollection`:
```
> db.sloppyNamedCollection.renameCollection('neatNamedCollection')
{ "ok" : 1 }
```

5. 执行下面命令确认集合`neatNamedCollection`不再出现在结果中：
```
> show collections
```

6. 最后，查询集合`neatNamedCollection`验证集合`sloppyNamedCollection`中的数据确实是在的。在mongo shell中执行下面命令：
```
> db.neatNamedCollection.find()
```

### 工作原理 ###
重命名一个集合是非常简单的。它是通过`renameCollection
`方法实现的，这个方法接受两个参数。一般来说，函数的格式如下所示：
```
> db.<collection to rename>.renameCollection('<target name of the
collection>', <drop target if exists>)
```

第一个参数是要重命名集合的名字。

第二个参数，我们没有使用，它是一个布尔值，它指示在集合已存在的情况下是否删除它。这个值默认是FALSE，这意味着在目标存在的情况下不删除目标，而是给出一个错误。这种默认情况和明智，否则如果我们偶然的提供了一个已存在的集合，并且我们不希望删除它，那么结果就很恐怖了。然而，如果在重命名集合的时候你知道你正在做什么并且希望目标被删除，以true为值传递第二个参数就行。第二个参数的名字叫做`dropTarget`。在我们这里，调用例子如下：
```
> db.sloppyNamedCollection.renameCollection('neatNamedCollection', true)
```

作为联系，尝试在创建一下`sloppyNamedCollection`并且不带第二个参数（或者值为false）重命名它。你应该会看到mongo提示目标命名空间已存在。然后，第二个参数为true来重命名一遍，并且现在重命名操作执行成功。


注意重命名操作将会保存原来的数据库，和新的重命名集合在同一个数据库中。`renameCollection`方法不能跨数据库移动/重命名集合。实现这种情况，需要如下运行`renameCollection`函数：
```
> db.runCommand({ renameCollection: "<source_namespace>", to: "<target_
namespace>", dropTarget: <true|false> });
```

假设我们将要重命名集合`sloppyNamedCollection`为`neatNamedCollection`并从`test`数据库移动它到`newDatabase`，我们可以通过执行下面命令来实现。注意开关` dropTarget: true`被用来删除已存在的目标集合 (newDatabase.neatNamedCollection) 。
```
> db.runCommand({ renameCollection: "test.sloppyNamedCollection ", to: "
newDatabase.neatNamedCollection", dropTarget: true });
```

另外，重命名集合的操作无法运行在分片集合上。

