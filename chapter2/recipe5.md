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

`inputStage`域的值现在是`IXSCAN`，这表明现在索引目前明确是在使用的。留下的结果数和预期的一样，都是6446。如结果所示，在索引中扫描的对象数目和在集合中扫描的文档数现在都减少到相同的值了。这是因为现在我们使用索引来提供开始烧苗的文档，只有这样，被扫描的文档数才只是请求的文档数。这类似于使用字典的目录来查找字词，而不是翻阅整本书来查找这个字词。正如预期，`excutionTimeMillis`的时间大幅减少。


#### 使用覆盖索引提升性能
这需要我们注意一个域，`executionStages`的值是`FETCH`，并且我们将了解这是什么意思。为了明白这个值是什么意思，我们需要简单的看看索引是如何执行的。

索引存储了集合中正常文档域的子集。索引中的字段与创建索引的字段相同。然而，这个区域保持索引创建时候的排序。除了这个域以外，在索引中还存储了另外的值，这个值的作用是指向集合中原文档。因此，无论用户何时执行查询，如果查询包含存在索引的字段，则索引关联获取到的匹配集合。指针和匹配查询的索引实体一起存储，之后通过其他IO操作从集合中获取整个文档时，这个指针被直接返回给用户。

这里`executionStages`的值是`FETCH`，这表明用户在查询中请求的数据不是整个出现在索引中的，但需要一个额外的IO操作通过索引的指针去集合中获取到这个文档。如果这个值是索引本身，就不再需要额外的操作去集合中获取文档，索引数据会直接返回。这就是我们所说的覆盖索引，这种情况下，`executionStages`的值将会是`IXSCAN`。

在我们这里，我们只需要PIN码。因此，为什么不使用投影只获取我们需要的呢？这将是的索引只包含州名和PIN码作为索引，这样获取请求的数据就完全不需要从集合中检索原文档。这里的查询计划也是很有趣的。

执行下列命令：
```
db.postalCodes.find({state:'Maharashtra'}, {pincode:1, _id:0}).explain()
```

这给了我们下面返回：
```
{
"executionStages" : {
 "stage" : "PROJECTION",
…
"inputStage" : {
 "stage" : "IXSCAN",
…
 "nReturned" : 6446,
 "totalDocsExamined" : 0,
 "totalKeysExamined": 6446
 "executionTimeMillis" : 4,
…
}
```

`totalDocsExamined`和`executionStage: PROJECTION `这两个域的值是值得关注的。如我们所料，我么从投影中请求的数据可以只从索引中获取。在这里，我们在索引中扫描了6446个试听，因此`totalKeysExamined`的值是6446。

因为整个结果是从索引中获取的，我么恩的查询不会从集合中获取任何文档。因此，`totalDocsExamined`的值是0。

因为这个集合很小，我么无法在查询时间上看到明显的差异。查询时间在打的集合上会更加明显。使用索引是很好的，且给我们提供了很好的性能。使用覆盖索引给我们提供了更好的性能。

> MongoDB的结果解释在第三版中有了很大的改变。我建议稍微花几分钟浏览下文档< http://docs.mongodb.org/manual/
reference/explain-results/>

> 另外一件需要记得的事就是，如果文档有很多的域，尝试使用投影来获取这个域我们需要的数目。默认情况下`_id`域每次都会被获取。如果他不是索引的一部分，除非我们的执行计划设置` _id:0`来不获取他。执行覆盖查询是查询集合一个更有效的方式。

#### 创建索引的一些忠告
We will now see some pitfalls in index creation and some facts when an array field is used in
the index.
Some of the operators that do not use the index efficiently are the $where, $nin, and
$exists operators. Whenever these operators are used in the query, one should bear in
mind that a possible performance bottleneck might occur when the data size increases.
Similarly, the $in operator must be preferred over the $or operator as both can be used to
achieve more or less the same result. As an exercise, try to find the pincodes in the state of
Maharashtra and Gujarat in the postalCodes collection. Write two queries: one using $or
and one using the $in operator. Explain the plan for both these queries.
What happens when an array field is used in the index?
Mongo creates an index entry for each element present in the array field of a document. So, if
there are 10 elements in an array in a document, there will be 10 index entries, one for each
element in the array. However, there is a constraint while creating indexes containing array
fields. When creating indexes using multiple fields, no more than one field can be of a type
array, and this is done to prevent a possible explosion in the number of indexes on adding
even a single element to the array used in the index. If we think of it carefully, an index entry
is created for each element in the array. If multiple fields of the type array were allowed to be
a part of an index, then we would have a large number of entries in the index, which would be
a product of the length of these array fields. For example, a document added with two array
fields, each of length 10, would add 100 entries to the index if it is allowed to create one index
using these two array fields.
This should be good enough, for now, to scratch the surfaces of a plain, vanilla index. We will
see more options and types in the following few recipes.