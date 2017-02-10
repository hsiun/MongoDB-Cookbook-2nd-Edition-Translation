# 为查询创建索引和视图

在本菜谱中，我们将了解查询数据，通过分析查询计划分析查询的性能，并通过创建索引优化查询。


### 前提条件
为了创建索引，我们需要启动并运行服务器。一个简单的单节点服务器就是我们需要的。有关如何启动服务器的说明，请参阅第一章_安装和启动服务器_中的_安装单节点MongoDB_菜谱（G）。我们将要操作的数据需要已经导入到数据库中。导入数据的步骤已经在前面的菜谱_创建测试数据_中给出。一旦这些前提条件满足，我们就准备好继续了。


### 如何操作
我们将尝试写一个查找给定州所有邮编的查询。
1. 执行以下查询以查看此查询的计划：
```
> db.postalCodes.find({state:'Maharashtra'}).
explain('executionStats')
```
注意下面的域：` stage, nReturned, totalDocsExamined,
docsExamined, and executionTimeMillis`出现在结果中以分析计划操作。

2. 让我们再一次执行相同查询，但是这次，我们限制结果仅有100个文档：
```
> db.postalCodes.find({state:'Maharashtra'}).limit(100).explain()
```

3. 注意下结果中的下面这些域：` nReturned, totalDocsExamined,
docsExamined, and executionTimeMillis`。

4. 如下所示，现在我们在`state`和`pincode`域创建索引：
```
> db.postalCodes.createIndex({state:1, pincode:1})
```

5. 执行下面查询：
```
> db.postalCodes.find({state:'Maharashtra'}).explain()
```
注意下结果中的下面这些域：` nReturned, totalDocsExamined,
docsExamined, and executionTimeMillis`。

6. 由于我们只需要pincodes，我们如下所示修改查询并检查它的计划：
```
> db.postalCodes.find({state:'Maharashtra'}, {pincode:1, _id:0}).
explain()
```
注意下结果中的下面这些域：` nReturned, totalDocsExamined,
docsExamined, and executionTimeMillis`。

### 实现原理
这里有大量需要解释的。我们将首先讨论我们做了什么和如何分析这些状态。下一步，我们将讨论创建索引的一些要点和注意事项。

*分析计划*
好的，让我们看看第一步并分析我们执行的输出：
```
db.postalCodes.find({state:'Maharashtra'}).explain()
```

在我们机器上的输出如下所示：（这里我跳过了无关的域）
```
{
 "stage" : "COLLSCAN",
...
 "nReturned" : 6446,
 "totalDocsExamined " : 39732,
 …
 "docsExamined" : 39732,
 …
 "executionTimeMillis" : 12,
…
}
```

在结果中`stage`域的值是`COLLSCAN`，这表明为了在整个集合中查询匹配的文档发生了全集合扫描（所有文档被一个接一个的扫描）。`nReturned`的值是`6446`，这是结果中匹配查询的数目。`totalDocsExamined `和` docsExamined`的值为39732，这个数是在集合中检索结果扫描的文档数。这也是目前集合中文档的总数，和为获取结果扫描的所有文档数。最后，`executionTimeMillis`是遍历结果花费的毫秒数。

### 改进查询时间
到目前为止，查询的性能看起来还不是很好，并且有很大的提升空间。为了演示应用在查询上的限制是如何影响查询计划的，我们可以再次找到那个没有索引但是有限制条件的查询计划，展示如下：
```
> db.postalCodes.find({state:'Maharashtra'}).limit(100).explain()
{
 "stage" : "COLLSCAN",
 …
 "nReturned" : 100,
 "totalDocsExamined" : 19951,

 …
 "docsExamined" : 19951,
 …
 "executionTimeMillis" : 8,
 …
}
```

这个查询计划在这里就很有趣了。尽管我们仍然没有创建索引，但是我们可以看到执行查询时间和遍历结果扫描数目的减少。这是由于一旦文档的数目达到`limit`函数的要求，mongo忽略扫描剩下的文档。我们可以这样总结，推荐使用`limit`功能限制结果的数目为预先知道的被访问文档的最大数目。这也许会获得更好的查询性能。也许这个词是非常重要的，在没有索引的情况下，如果匹配的数目不能满足整个集合依然会被完全扫描。

#### 使用索引提升性能
继续往前，我们在在state和pincode域上创建符合索引。在这种情况下，索引的顺序为升序（值为1），并且除非我们计划执行多键排序，否则索引的顺序不重要。这是一个决定性因素，关于结果是否可以使用索引进行排序或mongo需要在返回结果之前在内存中进行排序。就查询的计划而言，这里我们可以看到一个显著的提升：
```
{
"executionStages" : {
 "stage" : "FETCH",
…
"inputStage" : {
 "stage" : "IXSCAN",
…
 "nReturned" : 6446,
 "totalDocsExamined" : 6446,
 "docsExamined" : 6446,
 …
 "executionTimeMillis" : 4,
…
}
```

`inputStage`域的值现在是`IXSCAN`，这表明现在索引目前明确是在使用的。留下的结果数和预期的一样，都是6446。如结果所示，在索引中扫描的对象数目和在集合中扫描的文档数现在都减少到相同的值了。这是因为现在我们使用索引来提供开始烧苗的文档，只有这样，被扫描的文档数才只是请求的文档数。这类似于使用字典的目录来查找字词，而不是翻阅真笨数来查找这个字词。正如预期，`excutionTimeMillis`的时间大幅减少。

#### 使用覆盖索引提升性能
