## Nosql的概念 ##
> NOSQL概念在09年被提出来，NoSQL最常见的解释是"non-relational",也就是非关系，Not Only SQL也被很多人接收。

> NoSQL现在一般理解成非关系型数据库



NoSql的优势
	1.易拓展
	2.大数据量，高性能
	3.灵活的数据模型


## Redis ##
1.Redis 是一个开源的，内存中的数据结构存储系统，他可以用作数据库 缓存和消息中间件，它支持多种类型的数据结构 如字符串（string） 散列（hashes） 列表（lists） 集合（sets） 有序集合（sorted sets) 等数据类型


2.Redis数据都是缓存在内存中，redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加到记录文件中


## Redis 常用的基本命令 ##

	启动Redis服务器端
		redis-cli
	关闭Redis服务器端
		redis-cli shutdown
	
	1.删除键 del 键名 （返回的是删除的行数） 如果不存在返回0
	
	2.查找所有符合给定模式pattern 的key 
			keys * 匹配数据库中所有的key
			keys h?llo 匹配hello 或者hallo 其他等
			keys h*llo 匹配hllo 或和heeeello 等
			keys h[ae]llo 匹配hello 和hallo 不匹配其他

			特殊符号用\隔开
			KEYS 的速度非常快，但在一个大的数据库中使用它仍然可能造成性能问题，如
			果你需要从一个数据集中查找特定的 key ，你最好还是用 Redis 的集合结构(set)来代替。
			
	3.从当前数据库中随机返回一个key
		randomkey
				当数据库不为空时返回一个key
				当数据库为空时返回nil
	4.ttl以秒为单位 返回给定key的剩余生存时间（pttl 是以毫秒）
			当 key 不存在时，返回 -2 。
			当 key 存在但没有设置剩余生存时间时，返回 -1 。
			否则，以秒为单位，返回 key 的剩余生存时间。
	5.exists 检测给定的key是否存在 若存在返回1 不存咋返回0

	6.move 将当前数据库的key移动到给定的数据库db当中
		如果当前数据库和给定数据库有相同的key 那么move没效果成功返回1
		失败返回0
			
	7.type 返回指定key 的类型
	8.expire 指定key 生存时间 当	key过期时会自动被删除 其生存时间不受rename 或者 其他影响 除非用key重新设置生存时间
	9.pexpire 用毫秒指定key生存时间
	10.persist 移除一个键的生命周期 使其变成永久
	11.ping 检查连接状态


## Redis常用命令对应的php方法 ##
	1.$redis = new Redis(); 声明一个Redis对象
		$redis->connect('localhost',6379);连接到一个Redis实例
		$redis->close() 关闭一个redis实例
	2.$redis->ping();
		查看当前连接实例的状态，成功返回PONG 失败跑出一个RedisExperison异常
	3.$redis->del();
			删除指定的一个键 成功返回1 失败返回0
	4.$redis->keys()
			返回所有键的值或者筛选条件的值(数组)
	5.$redis->select('0');
		选择数据库 成功返回bool值true 失败返回false
	6.$redis->move('name2',1);
		移动一个key-value 到另外一个db 返回bool值
	7.取得一个key的生存时间 如果不存在返回-2 没有设置返回-1 
			var_dump($redis->ttl('name1'));
	8.删除一个key的生命周期设置 返回bool
			var_dump($redis->persist('name1'));
	9.设置一个key的生命周期
			var_dump($redis->expire('name1','1000'));
	10.查看一个key的类型 返回对应的类型
			echo  $redis->type('lin');
	11.查看一个key是否存在 返回bool
			var_dump($redis->exists('name2121'));

	
			
