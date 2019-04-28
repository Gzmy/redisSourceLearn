## Redis的链表实现

```c
typedef struct listNode {
	struct listNode* prev; // 前置结点
	struct listNode* next; // 后置结点
	void* value; // 结点的值
} listNode;

typedef struct listIter {
	listNode* next;
	int direction;
} listIter;

typedef struct list {
	listNode* head; // 表头结点
	listNode* tail; // 表尾结点
	void* (*dup) (void* ptr); // 结点复制函数
	void (*free)(void* ptr); // 结点释放函数, 可以构成多态, 接收任何类型的指针
	int (*match)(void* ptr, void* key); // 结点对比函数
	unsigned long len; // 链表结点个数
} list;

```

- Redis的链表实现特性总结:
	- 双端: 链表结点都是带有prev和next指针,获取某个节点的前置结点和
	后置结点的时间复杂度都是O(1)
	- 无环: 表头结点的prev指针和表尾结点的next都指向NULL, 对链表的访问
	以NULL为终点
	- 带表头指针和表尾指针: 通过list结构的head指针和tail指针,程序
	获取链表的表头和表尾的时间复杂度是O(1)
	- 带有链表长度计数器: 程序使用list结构的len属性对list持有的链表
	结点进行计数, 程序获取链表长度的时间复杂度为O(1)
	- 多态: 链表使用void * 指针来保存结点值,可以使用list结构的dup,
	free和match三个属性为结点值设置特定类型的函数, 所以链表可以用于保存
	各种不同类型的值


- 链表结点API
函数 | 作用
:-: | :-:
listCreate | 创建一个不包含任何结点的新链表
listRelease| 释放给定链表,以及链表中所有结点
listAddNodeHead | 将一个包含给定值的新节点添加到给定链表的表头
listAddNodeTail | 将一个包含给定值的新节点添加到给定链表的表尾 
listInsertNode | 将一个包含给定值的新节点添加到给定结点的之前或之后
listDelNode | 从链表中删除给定结点
listDup | 复制一个给定链表的副本
listSearchKey | 查找并返回链表中包含给定值的结点
listIndex | 返回链表在给定索引上的结点
listNodeValue | 返回给定结点母亲啊正在保存的值
listPrevNode | 返回给定结点的前置结点
listNextNode | 返回给定结点的后置结点
listLength | 返回结点个数
listFirst | 返回链表头结点
listLast | 返回链表尾结点
listSetDupMethod | 将给定函数设置为链表结点值复制函数
