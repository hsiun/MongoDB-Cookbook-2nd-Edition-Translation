# 使用collMod命令修改集合行为


这是mongo中的一个命令，执行这个命令可以改变集合的行为。它可以被认为是集合修改（collMod）操作（官方并没有提到这一点）。

作为本菜谱的一部分，需要掌握TTL索引的知识。


### 准备 ###
在本菜谱中，我们将会在集合上执行`collMod`操作。我们需要启动一个单独的服务，监听任何端口等待客户端连接都可；在我们这，我们将依旧使用默认的27017端口。如果你不知道如何启动单独的服务器，参考第一章_安装和启动服务器_中的_安装单节点MongoDB_菜谱。另外我们需要启动一个shell用来执行这些管理操作。如果你不熟悉的话，强烈建议看一下第二章的菜谱_使用TTL索引在固定时间间隔后过期文档_和_使用TTL索引在给定时间间隔后过期文档_。


### 实现 ###
这个操作可以被用来做两件事：
1. 假设我们有一个集合带有TTL索引，真如我们在_第二章，命令行操作_中看到的，执行下面命令查看索引列表：
```
> db.ttlTest.getIndexes()
```

2. 执行下面命令，将到期时间有300ms改为800m。
```
> db.runCommand({collMod: 'ttlTest', index: {keyPattern:
{createDate:1}, expireAfterSeconds:800}})
```


### 工作原理 ###
`collMod`命令总是使用下列各式：` {collMod : <name of the
collection>, <collmod operation>}.`

我们使用`collMod`的索引操作来修改TTL索引。如果TTL索引已经创建并且创建之后需要修改，我们可以使用`collMod`命令。操作指定字段的命令如下：
```
{index: {keyPattern: <the field on which the index was originally
created>, expireAfterSeconds:<new time to be used for TTL of the index>}}
```

`keyPattern`是集合的一个字段。`keyPattern`是集合的字段，在其上创建TTL索引，`expireAfterSeconds`将包含要更改的新时间。成功执行后，我们将会在shell中看到下面内容：
```
{ "expireAfterSeconds_old" : 300, "expireAfterSeconds_new" : 800, "ok" :
1 }
```
