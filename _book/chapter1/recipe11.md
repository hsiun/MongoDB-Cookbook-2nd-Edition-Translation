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

2. 连接到副本集中的任何节点，比如说`localhost:27000`。并且在shell中执行`rs.status()`。