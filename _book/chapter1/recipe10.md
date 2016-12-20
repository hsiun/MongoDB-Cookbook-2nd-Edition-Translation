# 通过Java客户端连接到副本集以查询和插入数据

在本菜谱中，我们将演示如何从Java客户端连接到副本集，和在primary节点故障时客户端如何自动转移到副本集中的其他节点。


### 前提条件
我们需要复习下_使用Java客户端连接到单个节点_，因为它包含了所有的前提条件和maven的设置步骤，及其他的依赖方面。因为我们正在处理Java客户单的副本集，所有副本集必须启动并运行。查看_作为副本集的一部分启动多个实例_获取更多如何启动副本集的细节。


### 如何做...
1. 粘贴/复制下面的代码片段：（从Packt网站下载的Java类也是可以的。）
```java
package com.packtpub.mongo.cookbook;
import com.mongodb.BasicDBObject;
import com.mongodb.DB;
import com.mongodb.DBCollection;
import com.mongodb.DBObject;
import com.mongodb.MongoClient;
import com.mongodb.ServerAddress;
import java.util.Arrays;
/**
 *
 */
public class ReplicaSetMongoClient {
 /**
 * Main method for the test client connecting to the
 replica set.
 * @param args
 */
 public static final void main(String[] args) throws
 Exception {
 MongoClient client = new MongoClient(
 Arrays.asList(
 new ServerAddress("localhost", 27000),
 new ServerAddress("localhost", 27001),
 new ServerAddress("localhost", 27002)
 )
 );
 DB testDB = client.getDB("test");
 System.out.println("Dropping replTest collection");
 DBCollection collection =
 testDB.getCollection("replTest");
 collection.drop();
 DBObject object = new BasicDBObject("_id",
 1).append("value", "abc");
 System.out.println("Adding a test document to replica
 set");
 collection.insert(object);
 System.out.println("Retrieving document from the
 collection, this one comes from primary node");
 DBObject doc = collection.findOne();
 showDocumentDetails(doc);
 System.out.println("Now Retrieving documents in a loop
 from the collection.");
 System.out.println("Stop the primary instance after few
 iterations ");
 for(int i = 0 ; i < 10; i++) {
 try {
 doc = collection.findOne();
 showDocumentDetails(doc);
 }
 catch (Exception e) {
 //Ignoring or log a message
 }
 Thread.sleep(5000);
 }
 }
 /**
 *
 * @param obj
 */
 private static void showDocumentDetails(DBObject obj) {
 System.out.printf("_id: %d, value is %s\n",
 obj.get("_id"), obj.get("value"));
 }
}
```

2. 从shell中连接到副本集中的任意一个几点，比如说`localhost:27000`，并执行`rs.status()`。注意下副本集中的primary节点，如果`localhost:27000`不是primary，从shell中连接他。在这里，如下所示，转换到管理员数据库：
```
repSetTest:PRIMARY>use admin
```

3. 我们现在从操作系统的shell中执行前面的程序，如下：
```
$ mvn compile exec:java -Dexec.mainClass=com.packtpub.mongo.
cookbook.ReplicaSetMongoClient
```

4. 连接到primary，在mongo shell中执行下面命令关闭primary实例：
```
repSetTest:PRIMARY> db.shutdownServer()
```

5. 当`com.packtpub.mongo.cookbook.ReplicaSetMongoClient`类在使用maven执行时，观察控制台输出。


### 如何做...
一件值得关注的趣事是我们是如何实例化`MongoClient`实例的。它是通过下面代码实现的：
```
MongoClient client = new MongoClient(Arrays.asList(
 new ServerAddress("localhost", 27000),
 new ServerAddress("localhost", 27001),
 new ServerAddress("localhost", 27002)));
```
这个构造器接受一个`com.mongodb.ServerAddress`列表。这个类有大量重载的构造器，但是我们选择使用接受主机名和端口的那一个。我们现在已经做了的是，将副本集中的所有服务器以一个列表的方式提供。我们并没有提到那一个是`PRIMARY`节点，那一个是`SECONDARY`节点。`MongoClinet`足够智能，能够找出并连接到正确的实例。提供的服务器列表称之为种子列表。它不需要包含副本集中整个服务器的集合，虽然我们应该尽可能多的提供。`MongoClinet`将从提供的子集中找出所有服务器的细节。举个例子，如果副本集有五个节点，但是我们只提供了三个服务器，这样是没有问题的。在连接到提供的副本集服务器中时，客户端将查询他们并获取副本集的元信息，并找出副本集中提供对的其余服务器。在前面的例子中，我们使用副本集中对的三个实例实例化客户端。如果副本集有五个成员，然后只使用其中三个初始化客户端也是可以的，并且剩下的两个将会自动被找到。

接下来，我们使用maven从命令行提示符启动客户端。一但客户端运行起来，我们关掉primary实例来查找一个文档。我们应该看到如下所示的输出到控制台：
```
_id: 1, value is abc
Now Retrieving documents in a loop from the collection.
Stop the primary instance manually after few iterations
_id: 1, value is abc

_id: 1, value is abc
Nov 03, 2013 5:21:57 PM com.mongodb.ConnectionStatus$UpdatableNode update
WARNING: Server seen down: Amol-PC/192.168.1.171:27002
java.net.SocketException: Software caused connection abort: recv failed
 at java.net.SocketInputStream.socketRead0(Native Method)
 at java.net.SocketInputStream.read(SocketInputStream.java:150)
 …
WARNING: Primary switching from Amol-PC/192.168.1.171:27002 to AmolPC/192.168.1.171:27001
_id: 1, value is abc
```

正如我们所看到的，当primary节点宕机的时候循环中的查询被中断了。然而，客户端无缝的转移到了一个新的primary节点，因为客户端也行捕获了一个异常，并且在预定间隔过去的时候重试了一遍操作。

