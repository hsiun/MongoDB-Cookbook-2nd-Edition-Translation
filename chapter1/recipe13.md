# 在shell中连接到分片并执行操作

在本菜谱，我们将命令行提示符连接到分片，看看如何分享一个集合，并在一些测试数据上实际观察数据分片。


### 前提条件
明显的，我们需要一个设置好的mongo分片服务器并运行起来。参考前面的菜谱，_启动一个两个分片的简单分片环境_，获取更多如何设置简单分片的细节信息。在前面菜谱提到的mongos进程，应该正在监听27017端口。我们从`names.js`文件中获取一个名字信息。这个文件需要从Packt网站下载，并保留在本地文件系统。这个文件包含一个称为`names`的变量，它的值是JSON格式数组，数组的每一行表示一个人。内容看起来如下所示：
```
names = [
 {name:'James Smith', age:30},
 {name:'Robert Johnson', age:22},
…
]
```

### 如何做...
1. 启动mongo shell并且连接到本地默认端口。这样做可以保证`names`在当前shell可用：
```
mongo --shell names.js
MongoDB shell version: 3.0.2
connecting to: test
mongos>
```

2. 更换到用于测试分片的数据库；我们将其称为`shardDB`:
```
mongos> use shardDB
```

3. 如下启动数据库层面的分片：
```
mongos> sh.enableSharding("shardDB")
```

4. 分享称之为`person`的的集合：
```
mongos>sh.shardCollection("shardDB.person", {name: "hashed"},
false)
```

5. 添加测试数据到分片集合：
```
mongos> for(i = 1; i <= 300000 ; i++) {
... person = names[Math.round(Math.random() * 100) % 20]
... doc = {_id:i, name:person.name, age:person.age}
... db.person.insert(doc)
}
```

6. 执行下面命令获取查下计划和每个分片文档的数目：
```
mongos> db.person.getShardDistribution()
```

### 如何做...
这个菜谱需要一些解释。我们下载了一个包含20个人数组的JavaScript文件。这个数组的每个元素是一个包含`name`和`age`属性的JSON对象。我们启动shell连接到mongos进程加载这个JavaScript文件。之后我们转换到`shardDB`，这个db是用于分片目的的。

对于一个要分片的集合，创建数据库的时候需要先允许分片。我们使用`sh.enableSharding()`来实现。

下一步就是启动集合分片了。默认情况下，所有数据将保留在一个分片上，不会分散到不同的分片上。思考一下：Mongo如何能有意义的拆分数据？整个目的在于有意义的拆分数据，并尽可能均匀，这样我们能基于分片键查询，且Mongo能很简单的确定查询那个分片。如果一个查询不包括分片键，这个查询将执行在所有的分片上，并且数据将会由mongos进程汇总到一起，然后返回给客户端。因此，选择正确的分片键是分成关键的。

现在，让我们看下如何分片集合。我们通过调用`sh.shardCollection("shardDB.person", {name: "hashed"}, false)`实现。它有三个参数：

- 集合`<db name>.<collection name>`格式完限定名是`shardCollection`方法的第一个参数。

- 第二个参数是集合中要分片的域的名字。这个域是将要被用于拆分到分片上的文档。一旦好的分片键这个要求被满足，分片键应该会有一个很高的基数。（这个数值可能的值应该是高的。）在我们的测试数据库中，`name`值的基数非常低，因此它不是作为分片键的好选择。当使用它作为分片键的时候我们会哈希它。我们通过将键标记为`{name: "hashed"}`来实现。

- 最后一个参数指明这个被作为分片键的值是否是唯一的。`name`这个域被定义为不唯一，因此它将被设为false。如果这个域是唯一的，如，个人的身份证号，它将被设置为true。另外，由于SSN的基数高，因此是一个分片键的好选择。记住分片键的出现是为了让查询更有效。

下一步是看下执行计划查找所有数据的部分。这个操作的目的是查看数据是怎样被划分到两个分片上的。对于300000份文档，我们预计每个分片大概会有150000份文档。然而，通过分布统计，我们可以观察到`shard0000`有1,49,715份文档，而`shard0001`有150285份。

```
Shard shard0000 at localhost:27000
 data : 15.99MiB docs : 149715 chunks : 2
 estimated data per chunk : 7.99MiB
 estimated docs per chunk : 74857
Shard shard0001 at localhost:27001
 data : 16.05MiB docs : 150285 chunks : 2
 estimated data per chunk : 8.02MiB
 estimated docs per chunk : 75142
Totals
 data : 32.04MiB docs : 300000 chunks : 4
 Shard shard0000 contains 49.9% data, 49.9% docs in cluster, avg obj size
on shard : 112B
 Shard shard0001 contains 50.09% data, 50.09% docs in cluster, avg obj
size on shard : 112B
```

有两个额外的建议，我建议你这样做。

从mongo shell单独连接到分片，并执行在person集合上执行查询。看到这些集合中的计数与我们在上一个计划中看到的相似。另外，一个可以查找到的文档不会同时存在两个分片上。

我们简短的讨论一下基数如何影响数据拆分到分片的方式。我们做个简单测试。我们先删除person集合，并且再执行分片集合操作一遍，但是这个时候我们使用`{name: 1}`代替`{name: "hashed"}`。这保证了分片建没有被哈希且被存储为他的真实值。现在，使用我们在前面第五步使用的JavaScript函数加载数据，一旦数据被加载然后在集合上执行`explain()`命令。观察数据现在是如何被拆分（或者没有）到不同分片的。


### 更多...
现在有大量的问题必须被提出来，比如说什么是最佳实践？有没有一些小方法，小技巧？MongoDB在幕后如何通过对最终用户透明的方式删除（pull off）分片？

这里这个菜谱只接受了基础知识，在管理员部分，所有的答案将会得到回答。

