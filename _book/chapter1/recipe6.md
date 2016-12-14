# 使用Java客户端连接到单个节点

这个菜单是关于设置MongoDB Java客户端的。当你致力于其他方面时你会反复查找这个菜单，因此请非常小心的阅读他。

### 准备条件
下面是这个菜谱的前提条件：
- 推荐Java SDK 1.6或者以后版本。
- 使用可以使用的最新版本的Maven。在写书的时候最新版本是3.3.3。
- 在写作本书的时候3.0.1版本的MongoDB Java启动是最新版本。
- 连接到互联网，可以防护在线的maven仓库或者是本地仓库。另一种方式是，你可以从计算机中选择一个适当的本地仓库。
- Mongo服务已经启动，并且运行在本地的27017端口。复习一下第一个菜谱，_单节点MongoDB安装_，并启动服务器。

### 如何做...
1. 如果你没有在你的机器上准备好JDK，请从<https://www.java.com/en/download/>安装最新版本。在这个菜谱中我们将不会介绍安装SDK的步骤，但是在我们进入下一步前，JDK应该被准备好。

2. Maven需要从<http://maven.apache.org/download.cgi>下载好。在下载页面我们将会看到类似图片的样子。选择`a.tar.gz`或者`.zip`格式的二进制文件包并下载。这个菜谱是执行在运行Windows系统的机器上的，因此这些步骤是为了在Windows上安装的。

3. 一旦压缩包下载好，我们需要解压它，加解压压缩包中的bin文件夹的绝对路劲放在操作系统的路径变量中。Maven也需要设置为AVA_HOME的环境变量值。

4. 现在我们要做的也就仅是在命令行输入`mvn -version`，如果我们看到如下东西开通的输出，那么说明我们成功的安装了maven：
```
> mvn -version
```

5. 到这一阶段，我们已经有安装好了的maven，现在我们已经准备好了去创建我们简单的项目去编写我们的第一个Java的Mongo客户端。我们通过创建`project`文件夹开始。我们创建一个名为`Mongo Java`的目录。然后我们在`project`文件夹中，创建一个`src/main/java`目录结构。`project`根文件夹现在包含pom.xml文件。一旦这些文件夹创建好，这个文件结构看起来如下所示：
```
 Mongo Java
 +--src
 | +main
 | +java
 |--pom.xml
```

6. 我们只是有了一个项目的骨架。现在我们添加一些内容到pom.xml文件中。不需要太多。下面内容是所有需要在pom.xml文件中的：
```xml
<project>
 <modelVersion>4.0.0</modelVersion>
 <name>Mongo Java</name>
 <groupId>com.packtpub</groupId>
 <artifactId>mongo-cookbook-java</artifactId>
 <version>1.0</version> <packaging>jar</packaging>
 <dependencies>
 <dependency>
 <groupId>org.mongodb</groupId>
 <artifactId>mongo-java-driver</artifactId>
 <version>3.0.1</version>
 </dependency>
 </dependencies>
</project>
```

7. 我们最后来编写我们用来连接Mongo服务并执行一些非常基础操作的Java客户端。下面是在`src/
main/java`的类，位于`com.packtpub.mongo.cookbook`包，另外类的名字是`FirstMongoClient`:
```java
package com.packtpub.mongo.cookbook;
import com.mongodb.BasicDBObject;
import com.mongodb.DB;
import com.mongodb.DBCollection;
import com.mongodb.DBObject;
import com.mongodb.MongoClient;
import java.net.UnknownHostException;
import java.util.List;
/**
 * Simple Mongo Java client
 *
 */
public class FirstMongoClient {
 /**
 * Main method for the First Mongo Client. Here we shall be connecting to a mongo
 * instance running on localhost and port 27017.
 *
 * @param args
 */
 public static final void main(String[] args)
throws UnknownHostException {
 MongoClient client = new MongoClient("localhost", 27017);
 DB testDB = client.getDB("test");
 System.out.println("Dropping person collection in test
database");
 DBCollection collection = testDB.getCollection("person");
 collection.drop();
 System.out.println("Adding a person document in the person
collection of test database");
 DBObject person =
new BasicDBObject("name", "Fred").append("age", 30);
 collection.insert(person);
 System.out.println("Now finding a person using findOne");
 person = collection.findOne();
 if(person != null) {
 System.out.printf("Person found, name is %s and age is
%d\n", person.get("name"), person.get("age"));
 }
 List<String> databases = client.getDatabaseNames();
 System.out.println("Database names are");
 int i = 1;
 for(String database : databases) {
 System.out.println(i++ + ": " + database);
 }
 System.out.println("Closing client");
 client.close();
 }
}
```

8. 现在是时候执行前面的Java代码了。我们将使用maven从shell中执行它。你应该与项目的pom.xml在同一目录中：
```
mvn compile exec:java -Dexec.mainClass=com.packtpub.mongo.
cookbook.FirstMongoClient
```

### 如何生效...
这里有大量的步骤要做。让我们更具体的看下其中一些。所有的事情直到第六步都是简单明了，不需要任何解释的。让我们从第七步开始。

我们这里的`pom.xml`文件是非常简单的。我们将mongo的Java驱动定义为一个依赖。它依靠在线的仓库,`repo.maven.apache.org`，来解析依赖标签。对于本地仓库，所有我们要做的只是在`pom.xml`文件中使用`pluginRepositories`标签定义一个仓库。获取Maven的更多信息，可以在<http://maven.apache.org/guides/index.html>查阅Maven的文档。

对于Java类，`org.mongodb.MongoClient`类是核心骨干。我们先实例化它，通过给它重载的构造器提供服务器主机和端口。在这里，主机名和端口不是真的需要是一个值，提供默认值也是可以的，并且没有参数的构造器也可以工作的很好。下面的代码片段示例话了这个客户端：

```
MongoClient client = new MongoClient("localhost", 27017);
```

下一步是获取数据库，在这个例子中，测试使用`getDB`方法。这个方法返回一个`com.mongodb.DB`类型的对象。注意这一点，这个数据库可能不存在，然而`getDB`不会抛出任何异常。替代的，这个数据库将会被创建，无论何时我们添加新的文档到这个数据库的集合中。类似的，在DB对象上调用`getCollection`将返回一个`com.mongodb.DBCollection`表示数据库汇总的集合类型的对象。这也可能是不存在与数据库中的，那么将会创建这个集合并自动传入一条记录。

下面从我们类中摘录的两行代码展示了如何初始化`DB`和`DBCollection`：
```
DB testDB = client.getDB("test");
DBCollection collection = testDB.getCollection("person");
```

在我们插入文档之前，我们会先删掉这个集合，这样程序执行多次，在person集合中我们依然只有一个文档。文档是使用`DBCollection`对象实例的`drop()`方法删除的。下一步，我们创建了一个`com.mongodb.DBObject`的实例。这个对象表示文档插入集合。这里具体使用的类是`BasicDBObject`，这个类是`java.util.LinkedHashMap`类型的，这个类型的键是字符串，值是对象。值也可以是另外一个`DBObject`，在这里，是一个文档嵌套在另一个文档中。在我们的情况中，我们有两个键，名字和年龄，键是插入到文档中一个区域的名字，而值可以是字符串和整数类型的。`BasicDBObject`的`append`方法向`BasicDBObject`实例添加一个新的键值对，并返回同一个实例，这允许我们链接`append`方法调用来添加多个键值对。接着，这个创建的`DBObject`对象使用`insert`方法 插入集合。如下所示是我们如何为person集合实例化`DBObject`并将其插入大集合中：
```
DBObject person = new BasicDBObject("name", "Fred").append("age", 30);
collection.insert(person);
```

`DBCollection`的`findOne`方法很简单明了，它从集合中返回一个文档。这个版本的`findOne`不接受` DBObject`（否则其充当在选择并返回文档之前执行的查询
）作为参数。这和在shell中执行`db.person.findOne()`是一样的。

最终，我们简单的调用`getDatabaseNames`来从服务器中获取数据库名称的列表。在这一刻，在返回的结果中我们应该至少有`test`和`local`两个数据库。一旦所有的操作完成，我们关闭客户端。`MongoClient`类是线程安全的，并且每个应用程序一般只使用一个实例。为了执行程序，我们使用了Maven的exec插件。在执行第九步是，我们应该在控制台看到下面行一直到结尾。

```
[INFO] [exec:java {execution: default-cli}]
--snip--
Dropping person collection in test database
Adding a person document in the person collection of test database
Now finding a person using findOne
Person found, name is Fred and age is 30
Database names are
1: local
2: test
INFO: Closed connection [connectionId{localValue:2, serverValue:2}] to
localhost:27017 because the pool has been closed.
[INFO] ------------------------------------------------------------------
------
[INFO] BUILD SUCCESSFUL
[INFO] ------------------------------------------------------------------
------
[INFO] Total time: 3 seconds
[INFO] Finished at: Tue May 12 07:33:00 UTC 2015
[INFO] Final Memory: 22M/53M
[INFO] ------------------------------------------------------------------
------ 
```

