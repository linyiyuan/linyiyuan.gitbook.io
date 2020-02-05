## 字符串类型常用命令 ##

	string是最基本的数据类型 一个键(key) 对应一个值 （value) 它能存储任何形式的字符串 包括二进制数据 你可以用string存储用户的邮箱 图片等 一个字符串类型允许存储的护具最大容量是512mb

	1.set 键名 值
	
	2.get 键名

	3.setnx 键名 值 
		当键名不存在时设置成功， 否则失败
	4.setex 键名 时间 值 （psetex 用毫秒）
		时间单位为s
		
		setex name 5 jack	

		name键只存在5s

	5.一次设置多个键

		mset key value 。。。。。
		
		mset name jack name1 mary ....

	6.一次读取多个键的值
		 
		mget key ...
		
		mget name name1

	7.将key中存储的数字值增一
		
		incr key
	
	8.将key所存储的值加上增量 increment 
		
		incrby key increment

		incrby num 8 (可以是负数)
	9.将key中储存的数字值减一

		decr key

	10.将key 所储存的值减去减量 decrmeng
		decrby key decrement
 		

	11.给key的键值尾部添加值
		
		append key value

	12. 获取key值的长度
		
		strlen key

	13.返回字符串值的一部分
		
			getRange('name1',2,2);

	14.修改字符串的一部分
		
		setRange('name1',5,'adminadmin')
	

## Redis字符串对应的php操作方法 ##
	
		// 1.设置字符串值到key
			var_dump($redis->set('name5','admin5'));
		// 2.获取字符串类型指定键的值 不存在返回false
			var_dump($redis->get('name5'));
			//取不了其他类型的值，返回false
			var_dump($redis->get('lin'));	
		// 3。设置一个key-value 这个函数会优先判断redis是否存在这个set 存在返回false
			var_dump($redis->setnx('name6','admin6')); 
		// 4.设置一个带有生命周期的字符串key
			var_dump($redis->setex('name7',1000,'admin7'));
		// 5.一次性设置多个键
			var_dump($redis->mset(['root' => 'value','root1' => 'value1','root2' => 'value2']));
		// 6.一次性获取多个键的值
			var_dump($redis->mget(['key1','key2']));
		// 7.将某个键的值增一
			var_dump($redis->incr('key1'));
		// 8.将某个键的值增加多少
			var_dump($redis->incrBy('key1',12));
		// 9.将某个键的值数字减一
			var_dump($redis->decr('key1'));
		// 10.将某个键的数字值减多少
			var_dump($redis->decrBy('$key1',12));
		// 11给某个键值尾部追加某个值
			$redis->append('name1','adminadminadmin');
		// 12获取key值的长度
			var_dump($redis->strlen('$key1'));
		// 13返回字符串的一部分
			var_dump($redis->getRange('name1','0','5'))
		// 14修改字符串的一部分
			var_dump($redis->setRands('name1','6','admin'));