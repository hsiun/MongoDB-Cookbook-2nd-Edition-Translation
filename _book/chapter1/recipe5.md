# 通过JavaScript在Mongo shell中连接到单个节点

这个菜单是关于启动mongo shell并连接到MongoDB服务器的。我们也在这里演示了如何在shell中加载JavaScript代码。尽管这并不是经常用到，当我们有含有变量和函数含有业务逻辑在其中很大的JavaScript代码块，需要在shell中频繁执行且我们需要这些功能总是在shell中可用是，它还是很方便的。


### 准备条件
尽管可以使用`mongo --nodb`命令不连接到MongoDB服务器运行mongo shell，我们只需要知道这样做可以就行了。在本地主机启动服务器没有太多麻烦，再看一眼第一个菜单，_单节点MongoDB安装_，并启动服务器。


### 如何做...
1. 首先，我们创建一个简单的JavaScript文件，并将其命名为`hello.js`。在`hello.js`文件中输入下面内容：
```
function sayHello(name) {
 print('Hello ' + name + ', how are you?')
}
```

2. 在本地`/mongo/scripts/hello.js`保存这个文件。（这个文件也可以保存在本地的任何其他目录。）

3. 在命令提示符中，执行下面命令：
```
> mongo --shell /mongo/scripts/hello.js
```

4. 执行这条命令后，我们可以在控制台看到如下输出：
```
MongoDB shell version: 3.0.2
connecting to: test
>
```

5. 通过输入下面命令来测试shell已连接到的数据库：
```
> db
```
这个命令将会在控制台输出`test`。

6. 现在，在shell中输入下面命令：
```
> sayHello('Fred')
```

7. 你应该可以获取到下面输出：
```
Hello Fred, how are you?
```

> 注意：这本书是写给3.0.2版本MongoDB的。这对你来说是个很好的机会，你可能会使用更新的版本并因此在mongo shell中看到不同的版本号。


### 如何生效...
我们这里执行的JavaScript函数是没有实际用处的，只是用来展示在启动时函数怎样预加载到shell中。在`.js`文件中可以包含多个有用的JavaScript函数代码--可能是一些负责的业务逻辑。

以不带参数的方式执行`mongo`命令，我们可以连接到运行在本机并且监听在默认27017端口等待新连接的MongoDB服务器。大体上来说，命令的格式是下面展示这样的：
```
mongo <options> <db address> <.js files>
```

在不传递参数给mongo执行的情况下，等价于传递了`db address`为`localhost:27017/test`。

一起看一些命令行选项`db address`值的例子，和它的说明：

- mydb：这将连接到运行在本地且监听27017端口的服务。连接到的数据库将是mydb。
- mongo.server.host/mydb：这将会连接到运行在`mongo.
server.host`且监听默认端口`27017`的服务。连接到的数据库将是mydb。
- mongo.server.host:27000/mydb：这将会连接到运行在`mongo.
server.host`且监听端口`27000`的服务。连接到的数据库将是mydb。
- mongo.server.host:27000：这将会连接到运行在`mongo.
server.host`且监听端口`27000`的服务。连接到数据库将是默认的test数据库。

现在，在mongo客户端也有少量几个选项可以使用。我们将在下面的表格中了解下其中几个：
| 选项              | 描述                                      | 
|:----------------- |:------------------------------------------| 
| --help or -h      | 这个选项用来打印各个可用的启动是选项信息  | 
| --shell           | 当`.js`文件作为一个参数给出是，这些脚本   | 
|                   | 将会执行然后mongo客户端将会推出。提供这个 |
|                   | 选项保证JavaScript文件执行完之后shell保持 |
|                   | 运行。在启动时所有这些.js文件中定义的函数 |
|                   | 和变量在shell中可用。如前所示，定义在     |
|                   | JavaScript文件中`sayHello`函数可以在shell |
|                   | 调用。                                    |
| --port            | 说明了客户端连接需要的mongo服务的端口。   |
| --host            | 说明了客户端连接需要的mongo服务的主机名。 |
|                   | 如果db address提供了主机名，端口和数据库，|
|                   | 那么--host和--port都不用做特殊说明了。    |
| --username        | 当mongo的安全性开启时这个选项是有用的。   |
| or -u             | 需要提供用户的用户名来登录。              |
| --password        | 当mongo的安全性开启时这个选项是有用的。   |
| or -p             | 需要提供用户的密码来登录。                |
