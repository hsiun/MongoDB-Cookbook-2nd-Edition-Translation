# 使用Python客户端连接到单个节点

在本菜谱中，我们将使用称之为PyMongo的Python MongoDB驱动来连接到单个MongoDB实例。由于Python的简单语法和它与MongoDB的完整功能集成，许多开发者反向，这种技术栈允许更快速的原型和更少的开发周期。

### 准备条件
下面是这个菜谱的准备条件：

- Python 2.7.x（尽管代码兼容Python 3.X）
- PyMongo 3.0.1：Python MongoDB驱动
- Python包安装器（pip）
- 启动并运行在localhost 27017端口的Mongo服务器。复习下第一个菜谱，_单节点MongoDB安装_，并启动服务器。


### 如何做...
1. 安装pip取决于你所使用的操作系统，在Ubuntu/Debian系统中。你可以使用下面的命令行安装pip：
```
> apt-get install python-pip
```

2. 使用pip安装最新版本的PyMongo：
```
> pip install pymongo
```

3. 最后，创建新的文件，命名为`my_client.py`，并输入下面的代码：
```
from __future__ import print_function
import pymongo
# Connect to server
client = pymongo.MongoClient('localhost', 27017)
# Select the database
testdb = client.test
# Drop collection
print('Dropping collection person')
testdb.person.drop()
# Add a person
print('Adding a person to collection person')
employee = dict(name='Fred', age=30)
testdb.person.insert(employee)
# Fetch the first entry from collection
person = testdb.person.find_one()
if person:
 print('Name: %s, Age: %s' % (person['name'], person['age']))
# Fetch list of all databases
print('DB\'s present on the system:')
for db in client.database_names():
 print(' %s' % db)
# Close connection
print('Closing client connection')
client.close()
```

4. 使用下面命令运行脚本：
```
> python my_client.py
```

### 如何生效...
我们通过借助pip包管理工具的帮助在系统中安装Python MongoDB的驱动PyMongo开始。在给出的Python代码中，我们开始先从` __future__ `模块导入`print_function`来实现对Python 3.x的兼容。下一步，我们导入pymongo，这样我们就能在脚本中使用它了。

我们使用localhost和27017分别作为mongo服务器的主机和端口来初始化`pymongo.MongoClient()`。在pymongo中，通过使用`<client>.<database_name>.<collection_name>`约定格式，我们可以直接引用到数据库和它的集合。

在我们的菜谱中，我们简单的通过引用`client.test`，使用客户端处理程序选中test数据库。这个方法返回一个数据库对象及时数据库不存在。作为这个菜谱的一部分，我们通过调用`testdb.person.drop()`删除集合，这里`testdb`指向`client.test`且`person`是我们希望删掉的集合。对于本菜单，我们故意删掉集合，这样重复运行将总是在集合中产生一条记录。

下一步，我们初始化一个称之为`employee`含有几个名字和年龄值的字典。我们现在讲使用`insert_one()`方法添加这些条目到`person`集合中。

如我们所知，现在有一个条目在person集合中，我们将使用`find_one()`方法来获取这个文档。这个方法返回集合中的第一个文档，具体取决于文档存储在硬盘上的顺序。

按照这个菜谱来，我们也尝试通过客户端调用`get_databases()`方法来获取所有数据库的列表。这个方法返回目前在服务器上数据库名字的列表。当您尝试断言服务器上数据库的存在时，此方法可能派上用场。

最后，我们使用`close()`方法关闭客户端连接。