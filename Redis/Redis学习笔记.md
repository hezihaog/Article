### Redis学习笔记

Redis是用C语言开发的一个开源的高性能键值对（key-value）数据库，官方提供测试数据，50个并发执行100000个请求,读的速度是110000次/s,写的速度是81000次/s ，且Redis通过提供多种键值数据类型来适应不同场景下的存储需求。

#### 什么是NOSQL

NoSQL(NoSQL = Not Only SQL)，意即“不仅仅是SQL”，是一项全新的数据库理念，泛指非关系型的数据库。

随着互联网web2.0网站的兴起，传统的关系数据库在应付web2.0网站，特别是超大规模和高并发的SNS类型的web2.0纯动态网站已经显得力不从心，暴露了很多难以克服的问题，而非关系型的数据库则由于其本身的特点得到了非常迅速的发展。

NoSQL数据库的产生就是为了解决大规模数据集合多重数据种类带来的挑战，尤其是大数据应用难题。

#### NOSQL和关系型数据库比较

- 优点

    1. 成本：nosql数据库简单易部署，基本都是开源软件，不需要像使用oracle那样花费大量成本购买使用，相比关系型数据库价格便宜。
    2. 查询速度：nosql数据库将数据存储于缓存之中，关系型数据库将数据存储在硬盘中，自然查询速度远不及nosql数据库。
    3. 存储数据的格式：nosql的存储格式是key,value形式、文档形式、图片形式等等，所以可以存储基础类型以及对象或者是集合等各种格式，而数据库则只支持基础类型。
    4. 扩展性：关系型数据库有类似join这样的多表查询机制的限制导致扩展很艰难。

- 缺点

    1. 维护的工具和资料有限，因为nosql是属于新的技术，不能和关系型数据库10几年的技术同日而语。
    2. 不提供对sql的支持，如果不支持sql这样的工业标准，将产生一定用户的学习和使用成本。
    3. 不提供关系型数据库对事务的处理。
    
#### 关系型数据库的优势

- 复杂查询可以用SQL语句方便的在一个表以及多个表之间做非常复杂的数据查询。
- 事务支持使得对于安全性能很高的数据访问要求得以实现。对于这两类数据库，对方的优势就是自己的弱势，反之亦然。

#### 总结 

- 关系型数据库与NoSQL数据库并非对立而是互补的关系，即通常情况下使用关系型数据库，在适合使用NoSQL的时候使用NoSQL数据库，让NoSQL数据库对关系型数据库的不足进行弥补。
- 一般会将数据存储在关系型数据库中，在nosql数据库中备份存储关系型数据库的数据。

#### 主流的NOSQL产品

- 键值(Key-Value)存储数据库
    - 相关产品： Tokyo Cabinet/Tyrant、Redis、Voldemort、Berkeley DB
    - 典型应用： 内容缓存，主要用于处理大量数据的高访问负载。 
    - 数据模型： 一系列键值对
    - 优势： 快速查询
    - 劣势： 存储的数据缺少结构化
        
- 列存储数据库
    - 相关产品：Cassandra, HBase, Riak
    - 典型应用：分布式的文件系统
    - 数据模型：以列簇式存储，将同一列数据存在一起
    - 优势：查找速度快，可扩展性强，更容易进行分布式扩展
    - 劣势：功能相对局限
        
- 文档型数据库
    - 相关产品：CouchDB、MongoDB
    - 典型应用：Web应用（与Key-Value类似，Value是结构化的）
    - 数据模型： 一系列键值对
    - 优势：数据结构要求不严格
    - 劣势： 查询性能不高，而且缺乏统一的查询语法
        
- 图形(Graph)数据库
    - 相关数据库：Neo4J、InfoGrid、Infinite Graph
    - 典型应用：社交网络
    - 数据模型：图结构
    - 优势：利用图结构相关算法。
    - 劣势：需要对整个图做计算才能得出结果，不容易做分布式的集群方案。

#### Redis的应用场景

- 缓存（数据查询、短连接、新闻内容、商品内容等等）
- 聊天室的在线好友列表
- 任务队列。（秒杀、抢购、12306等等）
- 应用排行榜
- 网站访问统计
- 数据过期处理（可以精确到毫秒
- 分布式集群架构中的session分离

#### Redis数据结构

目前为止Redis支持的键值数据类型如下：

1. 字符串类型 string
2. 哈希类型 hash：相当于HashMap
3. 列表类型 list：相当于LinkedList格式，支持重复元素
4. 集合类型 set：相当于HashSet，不允许重复元素，并且无序排列
5. 有序集合类型 sortedset：相当于TreeSet，不允许重复元素，且元素有顺序

- 字符串类型 string
	- 存储：set key value
		- 例：set username zhangsan
	- 获取：get key
		- 例：get username
	- 删除：del key
		- 例：del age
- 哈希类型 hash
	- 存储：hset key field value（每次操作，往map中加一对键值对）
		- 例：hset myhash username lisi
		- 例：hset myhash password 123
	- 获取：hget key field（获取指定的键的值）
		- 例：hget myhash username
	- 获取所有：hgetall key
		- 例：hgetall myhash
	- 删除：hdel key field
		- 例：hdel myhash username
- 列表类型 list（可以添加一个元素到列表的头部（左边）或者尾部（右边））
	- 存储：
		- lpush key value: 将元素加入列表左边（头部）
		- rpush key value：将元素加入列表右边（尾部）
	- 获取：lrange key start end ：范围获取，end为-1时，则为所有
		- 获取所有：lrange myList 0 -1
	- 删除：
		- lpop key： 删除列表最左边的元素，并将元素返回
		- rpop key： 删除列表最右边的元素，并将元素返回
- 集合类型 set：不允许重复元素，并且无序排列
	- 存储：sadd key value
		- 例：sadd myset a
	- 获取：smembers key:获取set集合中所有元素
		- 例：smembers myset
	- 删除：srem key value:删除set集合中的某个元素
		- 例：srem myset a
- 有序集合类型 sortedset：不允许重复元素，且元素有顺序.每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。
	- 存储：zadd key score value
		- 例：
			- zadd mysort 60 zhangsan
			- zadd mysort 50 lisi
			- zadd mysort 80 wangwu
	- 获取：zrange key start end [withscores]
		- 例：zrange mysort 0 -1（获取所有，end为-1则代表所有）
		- 例：zrange mysort 0 -1 withscores（withscores代表将分数也拿出来）
	- 删除：zrem key value
		- 例：zrem mysort lisi（删除某一个元素）

- 通用命令
	- keys * : 查询所有的键
	- type key ： 获取键对应的value的类型（如果不知道key对应的value值是什么类型，使用这个命令查询，确定后，再使用对应类型的增、删命令）
	- del key：删除指定的key value

#### 持久化

Redis是一个内存数据库，当redis服务器重启，获取电脑重启，数据会丢失，我们可以将redis内存中的数据持久化保存到硬盘的文件中。

- Redis持久化机制
	- RDB：默认方式，不需要进行配置，默认就使用这种机制（定时备份）
		- 策略：在一定的间隔时间中，检测key的变化情况，然后持久化数据。
	- AOF：日志记录的方式，可以记录每一条命令的操作。（性能会差一点）
		- 策略：每次每一次命令操作后，就持久化一次数据。

##### 策略修改

- RDB的策略修改

	- 步骤一：编辑Redis解压目录下的redis.windwos.conf文件，找到以下配置，可以针对服务器配置进行修改！

	```
	# after 900 sec (15 min) if at least 1 key changed（15分钟之后，如果至少有1个key发生改变，就持久化一次）
	save 900 1
	# after 300 sec (5 min) if at least 10 keys changed（5分钟之后，如果至少有10个key发生改变，就持久化一次）
	save 300 10
	# after 60 sec if at least 10000 keys changed（60秒之后，如果至少有1万个key发生改变，就持久化一次）
	save 60 10000
	```

	- 步骤二：重新启动redis服务器，并指定配置文件名称

	```
	redis-server.exe redis.windows.conf
	```

- AOF的策略修改
	- 步骤一：编辑redis.windwos.conf文件，找到appendonly no，改为yes，就可以开启AOF，默认为no（关闭）

	- 步骤二：找到以下配置，默认只有中间那一项是打开的，另外两项都是关的，可以根据你的需要进行更改
	
	```
	# appendfsync always ： 每一次操作都进行持久化
	appendfsync everysec ： 每隔一秒进行一次持久化
	# appendfsync no	 ： 不进行持久化
	```
	
#### Jedis

Jedis是Redis的Java客户端，操作类似JDBC。

操作一般分3步

- 获取连接
- 操作
- 关闭连接

使用Jedis，需要导入2个jar包，**commons-pool2-2.3.jar**和**jedis-2.7.0.jar**。下面是Jedis操作Redis的5种数据结构，Jedis中的方法和上面行的一致。

默认Redis的端口是6379，具体看Redis启动时显示的端口号来决定。

- 字符串

```
//获取连接
Jedis jedis = new Jedis("localhost", 6379);
String key = "username";
//存储
jedis.set(key, "zhangsan");
//获取
String value = jedis.get(key);
System.out.println("测试String获取：key = " + key + " value = " + value);
//20秒后，自动过期，将该键值对移除（key = activecode , value = hehe）
jedis.setex("activecode", 20, "hehe");
//关闭连接
jedis.close();
```

- 哈希，相当于Map

```
Jedis jedis = new Jedis("localhost", 6379);
//存储
String key = "user";
jedis.hset(key, "name", "lishi");
jedis.hset(key, "age", "18");
jedis.hset(key, "gender", "female");
//获取
String name = jedis.hget(key, "name");
System.out.println("测试Hash，key = " + key + "，field = name" + name);
//获取所有
Map<String, String> value = jedis.hgetAll(key);
for (Map.Entry<String, String> entry : value.entrySet()) {
    System.out.println("测试Hash，获取所有" + "field = " + entry.getKey() + "，value" + entry.getValue());
}
jedis.close();
```

- 列表，链表，有序，可重复

```
Jedis jedis = new Jedis("localhost", 6379);
String key = "myList";
//存储
jedis.lpush(key, "a", "b", "c");//从左边存
jedis.rpush(key, "a", "b", "c");//从右边存
//获取所有
List<String> myList = jedis.lrange(key, 0, -1);
System.out.println("测试List，获取所有" + myList);
//获取左边第一个
String element1 = jedis.lpop(key);
System.out.println("element1 = " + element1);
//获取右边第一个
String element2 = jedis.rpop(key);
System.out.println("element2 = " + element2);
jedis.close();
```

- Set集合，不可重复

```
Jedis jedis = new Jedis("localhost", 6379);
String key = "mySet";
//存储
jedis.sadd(key, "java", "php", "c++");
//获取
Set<String> mySet = jedis.smembers(key);
System.out.println(mySet);
//移除
jedis.srem(key, "java");
jedis.close();
```

- 有序集合，不可重复，并自动排序

```
Jedis jedis = new Jedis("localhost", 6379);
String key = "mySortedSet";
//存储
jedis.zadd(key, 50, "Rose");
jedis.zadd(key, 3, "Wally");
jedis.zadd(key, 30, "Barry");
//获取
Set<String> mySortedSet = jedis.zrange(key, 0, -1);
System.out.println("测试SortedSet，获取所有：" + mySortedSet);
//移除
jedis.zrem(key, "Wally");
jedis.close();
```

#### JedisPool

每次操作都重新创建、销毁连接是很耗性能的，一般我们会使用JedisPool这个库，每次操作完后，归还连接到池子中，类似线程池。

- 创建JedisPool，可传入一个JedisPoolConfig来配置连接池，不传，则使用默认的配置

```
//连接池配置
JedisPoolConfig config = new JedisPoolConfig();
//最大连接数
config.setMaxTotal(50);
//最大空闲连接
config.setMaxIdle(10);
JedisPool pool = new JedisPool(config, "localhost", 6379);
```

- 从连接池中，获取Redis连接

```
JedisPool pool = new JedisPool(config, "localhost", 6379);
//获取连接
Jedis jedis = pool.getResource();
```

- 操作，这里就是Jedis的操作，上面一斤介绍了

```
Jedis jedis = pool.getResource();
String key = "myName";
jedis.set(key, "zihe");
String value = jedis.get(key);
```

- 归还连接到连接池中，还是调用Jedis对象的close

```
jedis.close();
```

- 完整代码

```
//连接池配置
JedisPoolConfig config = new JedisPoolConfig();
//最大连接数
config.setMaxTotal(50);
//最大空闲连接
config.setMaxIdle(10);
JedisPool pool = new JedisPool(config, "localhost", 6379);
//获取连接
Jedis jedis = pool.getResource();
String key = "myName";
jedis.set(key, "zihe");
String value = jedis.get(key);
System.out.println("测试连接池，获取数据" + "key = " + key + "，value = " + value);
//归还连接
jedis.close();
```

#### Jedis连接池常用配置

Jedis常用配置，在JedisPool中都有提供对应的set方法。

```
#最大活动对象数
redis.pool.maxTotal=1000
#最大能够保持idel状态的对象数
redis.pool.maxIdle=100  
#最小能够保持idel状态的对象数
redis.pool.minIdle=50
#当池内没有返回对象时，最大等待时间    
redis.pool.maxWaitMillis=10000
#当调用borrow Object方法时，是否进行有效性检查    
redis.pool.testOnBorrow=true
#当调用return Object方法时，是否进行有效性检查    
redis.pool.testOnReturn=true
#“空闲链接”检测线程，检测的周期，毫秒数。如果为负值，表示不运行“检测线程”。默认为-1.  
redis.pool.timeBetweenEvictionRunsMillis=30000
#向调用者输出“链接”对象时，是否检测它的空闲超时；  
redis.pool.testWhileIdle=true
# 对于“空闲链接”检测线程而言，每次检测的链接资源的个数。默认为3.  
redis.pool.numTestsPerEvictionRun=50
#redis服务器的IP
redis.ip=xxxxxx  
#redis服务器的Port
redis1.port=6379
```