# 通过Python客户端连接到副本集以查询和插入数据

在这个菜谱中，我们见演示如何使用Python客户端连接到副本集，和在primary节点宕机的时候客户端将如何自动转移到其他副本集的其他节点。


### 前提条件
参考_使用Python客户端连接到单个节点_菜谱，因为它描述了如何设置和安装PyMongo，PyMongo是是Python的驱动。灵位副本集必须启动并运行。参考_作为副本集的一部分启动多个实例_菜谱获取如何启动副本集的细节。


### 如何做...
1. 手打/复制下面的代码片段到replicaset.py：（这个脚本也可以从Packt网站下载。）
```Python
from __future__ import print_function
import pymongo
import time

client = pymongo.MongoClient(['localhost:27002',
'localhost:27001', 'localhost:27000'], replicaSet='repSetTest')
# Select the collection and drop it before using
collection = client.test.repTest
collection.drop()
#insert a record in
collection.insert_one(dict(name='Foo', age='30'))
for x in range(5):
 try:
 print('Fetching record: %s' % collection.find_one())
 except Exception as e:
 print('Could not connect to primary')
 time.sleep(3)
```

2. 连接到副本集中的任何节点，比如说`localhost:27000`。并且在shell中执行`rs.status()`。注意下副本集中的primary实例，如果`localhost:27000`不是primary，从shell连接到primary。这里，如下所示转换到管理员数据库：
```
> repSetTest:PRIMARY>use admin
```

3. 如下，现在我们从操作系统的shell执行前面的脚本：
```
$ python replicaset_client.py
```

4. 在连接到primary的mongo shell中执行下面命令关闭primary实例：
```
> repSetTest:PRIMARY> db.shutdownServer()
```

5. 在Python脚本执行的控制台观察输出。


### 如何做...
你将注意到，在这个脚本中，我们通过提供一个列主机替代单个主机来实例化mongo客户端。在3.0版本中，在初始化期间pymongo驱动的`MongoClient()`类技能接受一列主机，也可以是单个主机，但是不推荐`MongoReplicaSetClient()`。客户端将临时连接到列表中的第一个主机，如果成功，会探测副本集中的其他节点。我们也传递了` replicaSet='repSetTest'`参数来保证客户端检查连接到的节点是否是副本集的一部分。

一旦连接上，我们执行普通的数据库操作，如选中测试数据库，删除`repTest`集合，并且插入一条文档到集合中。


依次下来，我们将进入一个条件循环，遍历五次。每一次，我们获取记录，展示他，并休眠三秒。当脚本执行到循环是，我们如第四步中提到的关掉副本集中primary节点。我们将看类似下面的输出：
```
Fetching record: {u'age': u'30', u'_id': ObjectId('5558bfaa0640fd1923fce
1a1'), u'name': u'Foo'}
Fetching record: {u'age': u'30', u'_id': ObjectId('5558bfaa0640fd1923fce
1a1'), u'name': u'Foo'}
Fetching record: {u'age': u'30', u'_id': ObjectId('5558bfaa0640fd1923fce
1a1'), u'name': u'Foo'}
Could not connect to primary
Fetching record: {u'age': u'30', u'_id': ObjectId('5558bfaa0640fd1923fce
1a1'), u'name': u'Foo'}
```

在前面的输出中，客户端中途从primary节点断开连接。然而，不就，一个新的primary节点从剩余的节点中选出，mongo客户端将恢复连接。

