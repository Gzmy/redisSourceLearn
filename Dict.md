## Redis中的字典
- 字典又称作符号表, 关联数组, 映射, 是一种保存键值对的抽象数据结构
- Redis使用字典作为数据库和哈希键的底层实现, 对数据库的增删改查也是建立在对字典的操作之上
- Redis中的字典使用hash表作为底层实现, 一个hash表中可以有多个hash表结点
每个hash表结点就保存一个键值对


- 底层结构实现
```
// Redis字典使用的hash表由dict.h/dictht定义 
typedef struct dictht {
	dictEntry **table; // hash表数组, 本质上是hash桶

	unsigned long size; // 哈希表大小

	unsigned long sizemask; // hash表大小掩码, 始终为size-1

	unsigned long used; // hash表已经拥有的结点数量
} dictht;

// Redis hash表使用的hash结点由dict.h/dictEntry定义
typedef struct dictEntry {
	void* key; // 键

	union { // 值
		void* val;
		uint64_tu64;
		int64_ts64;
	} v;

	struct dictEntry* next; // 指向下一个链表的结点
} dictEntry;

// Redis 字典的由dict.h/dict结构定义
typedef struct dict {
	dictType* type; // 类型特定函数

	void* privdata; // 私有数据

	dictht ht[2]; // hash表

	int rehashindex; // rehash索引, 当rehash不进行时值为-1
} dict;

// type属性和privdata属性是针对不同类型的键值对, 为创建多态字典设置的
typedef struct ductType {
	unsigned int (*hashFunction)(const void* key); // 计算hash值
	void* (*keydup)(void* privdata, const void* key); // 复制键的函数
	void* (*valdup)(void* privdata, vonst void* obj); // 复制值的函数
	int (*keyCompare)(void* privdata, const void* key1, const void* key2); // 对比键的函数
	void (*keyDestructor)(void* privdata, void* key); // 销毁键
	void (*valDestructor)(void* privdata, void* obj); // 销毁值
} dictType;
```

- hash算法: 要将一个新的键值对添加到新的字典里面, 需要先根据键值对
的键计算出hash值和索引值, 再根据索引值将包含新的键值对的hash结点
放到指定hash数组的指定索引上面
	- hash计算hash值: hash = dict-> type-> hahsFunction(key);
	- 使用hash的sizemask和hash值计算出索引值: index = hash & dict-> ht[x].sizemask;
	- Redis使用**MurmurHash2算法**来实现hash值的计算

- 解决键冲突: 当有两个或者两个以上的键被分配到hash数组的同一个索引上
我们称这些键发生了冲突.   Redis使用链地址法解决hash冲突

- rehash: 随着操作的不读那进行, hash表保存的键值对会增多或者减少,
为了让hash表的负载印在维持在一个合理的范围之内, 当hash表保存的
键值对数量太多或者太少时,程序需要对hash表的大小进行相应的扩展或者收缩
	- 为字典的ht[1]分配空间, 这个空间的大小取决于要执行的操作以及ht[0]
	包含的键的数量
	- 将保存在ht[0]上面的所有键值对都rehash到ht[1]上面, rehash指的是
	重新计算键的hash值和索引值, 然后将键值对放到ht[1]哈希表的指定位置
	- 当ht[0]包含的所有键值对都迁移到ht[1]之后, 释放ht[0], 将ht[1]设置成ht[0]
	并在ht[1]新创建一个空白hash表, 为下一次rehash做准备

- 渐进式rehash: 如果ht[0]中保存的是少量数据, 服务器可以在瞬间完成
rehash操作,但是如果保存的数据是大量数据, 一次性将这些键值对rehash到
ht[1],庞大的计算量可能导致服务器在一段时间内停止服务, 所以有渐进式rehash
	- 为ht[1]分配空间, 让字典同时持有ht[0]和ht[1]两个hash表
	- 在字典中维持rehashindex,将值设置为0, 表示rehash工作正式开始
	- 在rehash期间, 每次对字典进行增删改查时, 程序除了做出指定操作之外,
	在对字典进行插入操作时, 删改查都会在两个hash表上进行, 但是增加会在ht[1]
	还会顺带执行rehash操作, 当rehash操作完成之后, rehashindex会+1
	- 在某个时间点上, ht[0]的所有键值对都被rehash到ht[1]上,这是程序
	rehashindex的值为-1, 表示rehash操作正式完成
- 渐进式hash采取分而治之的思想, 将rehash的计算工作量均摊到对字典进行
增删改查的操作之上, 从而避免了集中式rehash而带来的庞大计算量
