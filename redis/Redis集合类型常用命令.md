## 集合基本命令

集合元素特点： 
将一个或多个 member 元素加入到集合 key 当中，已经存在于集合的 member 元素将被忽略。
集合中的元素不允许重复


集合元素特点： 
将一个或多个 member 元素加入到集合 key 当中，已经存在于集合的 member 元素将被忽略。

	//将一个元素添加到集合中
  	sadd 集合名字   元素


	//求集合元素个数
	scard 集合名字


	//求查集 (会保留集合名字1中与集合名字2中不同的元素)
	sdiff 集合名字1  集合名字2

	//求集合交集
	sinter 集合1 集合2

	//求集合并集
	SUNION 集合1  集合2

	//判断 member 元素是否集合 key 的成员。
	SISMEMBER key member


	//返回集合 key 中的所有成员
	SMEMBERS key

	//删除集合中元素
	srem key member member1 ....

## PHP相关命令 ##
	 1.添加一个value到set容器中，如果已经存在该value则返回0,如果该key不是sets类型则返回false
		$redis->sadd('sets','admin','admin2','admin3','admin4');
		var_dump($redis->sadd('sets','admin','admin2','admin3','admin4','admin6'));
	 2.返回指定集合的成员数 如果该key是其他数据类型返回false 如果不存在返回0
	   		var_dump($redis->sCard('sets'));
	 3.返回第一个集合与其他集合的差集
	   		var_dump($redis->sDiff('sets','sets1'));
	 4.同理求集合的交集和并集
	   		var_dump($redis->sInter('sets','sets1'));
	   		var_dump($redis->sUnion('sets','sets1'));
	 5.判断一个元素是否在集合中
	   		var_dump($redis->sIsMember('sets','admin2'));
     6.获取到集合key中所有的成员
			var_dump($redis->sMembers('sets'));
     7.删除集合中的一个元素
			var_dump($redis->sRem('sets','admin6'));