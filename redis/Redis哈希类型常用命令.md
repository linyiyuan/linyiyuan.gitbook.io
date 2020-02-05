## Hash（哈希表）类型常用命令

1. Redis的hash类型是一个string类型的field(字段)和value(值)的映射表。

2. Hash特别适合用于存储对象(属性、属性值)。相对于将对象的每个属性存成单个string类型。

3. 将一个对象存储在Hash类型中会占用更少的内存，并且可以更方便地存取整个对象信息。



   1. 将哈希表 key 中的field 的值设为 value(如果已经存在键则修改当前键的值并返回0)

       HSET  key field value

       例子：  hset mm  height  175cm

   2. 返回哈希表 key 中给定 field 的值

		hget key field

       例子：  hget mm height  //可以得到175cm

   3. 将哈希表 key 中的 field 的值设置为 value ，当且仅当字段 field 不存在

		HSETNX key field value

   4. 同时将多个 field-value (键值)对设置到哈希表 key 中

		HMSET key field value [field value ...]

   5. 返回哈希表 key 中，一个或多个给定字段的值

		HMGET key field [field ...]

   6. 为哈希表 key 中的字段 field 的值加上增量 increment

       	HINCRBY key field increment

   7. 查看哈希表 key 中，给定字段field 是否存在

		HEXISTS key field

		返回值：
			如果哈希表含有给定字段，返回 1 。
			如果哈希表不含有给定字段，或 key 不存在，返回 0

   8. 返回哈希表 key 中字段的数量
	   
		HLEN key

   9. 删除哈希表 key 中的一个或多个指定字段，不存在的字段将被忽略

		HDEL key field [field ...]

   10. 获取key中的所有字段名和字段值
		  
		HGETALL key

	11 .获取某个键所有属性
	
			hkeys key

	12. 获取某个键的所有值

		hvals  key


## PHP操作哈希命令 ##

		// 1.设置哈希的值 ('key','field','value') 如果存在则返回0并且覆盖
			var_dump($redis->hset('gg','height','17322cm'));
		// 2.获取某个哈希键字段的值 不存在返回false
			var_dump($redis->hget('gg','height'));
		// 3.添加一个哈希字段如果存在的话返回false
			var_dump($redis->hsetnx('gg','weight','55kg'));
		// 4.获取哈希的长度
			var_dump($redis->hlen('gg'));
		// 5.删除哈希的某个值
			var_dump($redis->hdel('gg','weight'));
		// 6.获取哈希的全部信息
			var_dump($redis->hgetAll('gg'));
		// 7.获取哈希的所有键 返回数组
			var_dump($redis->hkeys('gg'));
		// 8.获取哈希的所有值 返回数组
			var_dump($redis->hvals('gg'));
		// 9.检测哈希是否存在某个值 返回bool类型
			var_dump($redis->hexists('gg','height'));
		// 10批量添加hash表 不是字符串类型的VALUE，自动转换成字符串类型
			var_dump($redis->hmset('mm',['height'=>'178','weight'=>'88kg']));
			var_dump($redis->hgetAll('mm'));                                                                    
		 