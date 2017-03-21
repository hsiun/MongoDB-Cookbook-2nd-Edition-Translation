# 查看数据库状态

在之前的菜谱中，我们已经看过在管理的角度如何查看一些集合的重要数据。在本菜谱中，我们将占到更好的层次，（或者其中的大多数）数据是在数据库层次的。

### 前提条件 ###
为了查找数据库的状态，我们需要服务器已经启动并运行。一个单节点应该就可以了。请参阅第一章_安装和启动服务器_中的_安装单节点MongoDB_菜谱获取如何启动服务的信息。我们将操作的数据需要被导入到数据库中。导入数据的步骤已经在菜谱_第二章创建测试数据，命令行操作和索引_给出了。一旦这些步骤完成，对于这个菜谱我们都提前完成好了。如果你需要看看如何查看集合层次的状态，参考前一个菜谱。

### 如何做 ###
1. 出于本菜谱演示的目的，我们将使用`test`数据库。`test`数据库已经有`postCodes`集合在里面了。

2. 在操作系统终端中通过输入下面命令连接到服务器以使用mongo shell。这要求服务器监听在默认端口27017.
```
$ mongo
```

3. 在shell中，执行下面的命令并观察输出：
```
> db.stats()
```

4. 在shell中，再一次执行下面命令，但是这一次，我们添加比例参数。观察输出：
```
> db.stats(1024)
```

### 如何实现 ###

`scale`参数，它是`stats`函数的参数，用给定的`scale`值分割字节数。在本例中，这个值是1024并且因此所有值的单位将是KB。我们分析下面输出：
```
> db.stats(1024)
{
 "db" : "test",
 "collections" : 3,
 "objects" : 39738,
 "avgObjSize" : 143.32699179626553,
 "dataSize" : 5562,
 "storageSize" : 16388,
 "numExtents" : 8,
 "indexes" : 2,
 "indexSize" : 2243,
 "fileSize" : 196608,
 "nsSizeMB" : 16,
"extentFreeList" : {
 "num" : 4,
 "totalSize" : 2696
 },
 "dataFileVersion" : {
 "major" : 4,
 "minor" : 5
 },
 "ok" : 1
}
```

下面的表格暂时了重要值的意义：
<!-- 省略了一个表格 -->

获取更多信息，你可以访问Packet出版社网站参考![Instant MongoDB](http://www.packtpub.com/big-data-and-business-inteliigence/
instant-mongodb-instant)


### 如何实现 ###
我们从将注意力转移到`collections`域开始。如果你仔细观察这个数字，并且在mongo shell执行展示集合命令，您将在统计数据中找到一个额外的集合，与执行命令相比。这个不同的地方对于集合来说是隐藏的。他的名字是`system.namespaces `集合。你可以通过执行`db.system.namespaces.find()`来查看它的内容。

