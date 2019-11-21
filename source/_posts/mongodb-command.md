layout: '[post]'
title: MongoDB Shell命令
tags:
  - 备忘翻阅
categories:
  - Mongodb
  - ''
date: 2017-11-30 16:39:00
---

#### 开启MongoDB数据库服务

&emsp;&emsp;mongo根目录`/bin/mongod -f` 配置文件目录/配置文件名
例：`./bin/mongod -f conf/mongod.conf`

<!-- more -->

#### 强行关闭MongoDB

&emsp;&emsp;先用命令 `ps -ef | grep mongod `查出mongod 的进程pid
然后`kill pid `即可

#### 数据库

##### 连接MongoDB数据库

&emsp;&emsp;mongo根目录`/bin/mongo` 数据库地址:端口号/库名称
例： `./bin/mongo 127.0.0.1:12345/test`

##### 关闭MongoDB数据库

&emsp;&emsp;`db.shutdownServer()`

##### 新建和切换数据库

&emsp;&emsp;MongoDB不用特别地去声明新建一个数据库，直接用`use 数据库名` 就可以了。

##### 删除数据库

&emsp;&emsp;先用`db.use`切换到要删除的数据库，然后使用`db.dropDatabase()`来删除数据库

#### 数据操作

##### 插入一条数据

`db.collection.insert({key: value})`

其中集名称可以自己起

##### 插入多条数据

&emsp;&emsp;可以使用for循环插入： `for(i＝3;i<100;i++)db.collection.insert({key: value})`

##### 查询所有数据

`db.collection.find()`

##### 查询单条数据

`db.collection.find({key: value})`

##### 查询后有条件地进行处理

`db.collection.find({key: value}).skip(3).limit(5).sort({key: value})`

上面查询语句后的限制分别是skip（跳过多少条数据）、limit（限制查多少条数据）、sort(将查询出来的结果集排序)

##### 显示库中的所有集名称

`show collections`

##### 删除数据库中的集合

`db.collection.drop()`

#### 索引

##### 查看索引

`db.imooc_2.getIndexes()`

##### _id索引

- _id索引是绝大多数集合默认建立的索引。
- 对于每个插入的数据，MongoDB都会自动生成一条唯一的_id字段。

##### 创建一个单键索引

- 单键索引是最普通的索引
- 单键索引不会自动创建

&emsp;&emsp;例如： 一条记录为`{x: 1, y: 2, z: 3}`，如果我们在x上建立了索引，就可以使用x为条件进行查寻。

`db.collection.ensureIndex({index: order})`

- index: 索引
- order: 1表示升序， -1表示降序

##### 创建一个多键索引

多键索引与单键索引创建形式相同，区别在于字段的值。 

- 单键索引：值为一个单一的值，如字符串，数字或日期。 
- 多键索引：值具有多个记录，如数组。

`db.collection.insert({x:[1,2,3,4,5]}) //插入一条数组数据`

`db.collection.insert({x:new Date()}) //插入一条数组数据`

##### 创建一个复合索引

&emsp;&emsp;当我们的查询条件不止一个的时候，就需要建立复合索引

&emsp;&emsp;例如{x:1,y:2,z:3}这样一条数据，要按照x与y的值进行查询，就需要创建复合索引`db.collection.ensureIndex({x:1, y:1})`，然后就可以使用`{x: 1, y:1}`作为条件进行查询

`db.collection.ensureIndex({x:1, y:1})`

##### 创建一个过期索引

- 在一段时间后会过期的索引 
- 在索引过期后，相应的数据会被删除 
- 适合存储在一段时间之后会失效的数据，比如用户的登录信息、存储的日志等。

`db.imooc_2.ensureIndex({time:1},{expireAfterSeconds: seconds})` 创建过期索引，time-字段，expireAfterSeconds在多少秒后过期，单位：秒

&emsp;&emsp;过30秒后再find，刚才的数据就已经不存在了。

&emsp;&emsp;过期索引的限制： 

1. 存储在过期索引字段的值必须是指定的时间类型，必须是ISODate或者ISODate数组，不能使用时间戳，否则不能自动删除。 
例如 >db.imooc_2.insert({time:1})，这种是不能被自动删除的 
2. 如果指定了ISODate数组，则按照最小的时间进行删除。 
3. 过期索引不能是复合索引。因为不能指定两个过期时间。 
4. 删除时间是不精确的。删除过程是由MongoDB的后台进程每60s跑一次的，而且删除也需要一定时间，所以存在误差

