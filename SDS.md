## SDS 与 C字符串之间的区别

- SDS:(simple dynameic string)	简单动态字符串

```c
struct sdshdr {
	unsigned int lne; // 记录buf中已经使用的字节数,等于SDS的字符串长度
	unsigned int free; // 记录buf中未使用的字节数的数量
	char buf[]; // 字节数组,支持对二进制的保存也支持对字符串的保存,所以不是字符数组.是一个柔性数组
};
```


- C字符串与Redis中SDS的对比

C字符串|SDS
:-: | :-:
获取字符串长度的时间复杂度是O(N)|获取字符串长度的时间复杂度是O(1)
API是不安全的,可能会造成缓区溢出|API安全,不会造成缓冲区溢出(使用空间预分配以及空间惰性释放)
修改字符串长度N次必然要执行N次内存分配|修改字符串长度N次最多需要执行N次内存重新分配(参照上面内存分配)
只能保存文本数据|可以保存文本或者二进制数据
可以使用所有<string.h>中的函数|可以使用一步分<string.h>中的函数

- SDS API

函数|作用|时间复杂度
:-: | :-: | :-:
sdsnew|创建一个包含给定C字符串的SDS|O(N),N为C字符串的长度
sdsempty|创建一个不包含任何内容的空的SDS|O(1)
sdsfree|释放给定的SDS(底层调用zfree)|O(N),N为释放的长度
sdslen|返回SDS已经使用空间字节数|直接读取len属性,时间复杂度O(1)
sdsvail| 返回SDS已经未使用空间字节数|读取free属性,时间复杂度O(1)
sdsdup | 创建一个SDS副本(copy,调用sdsnewlen) | O(N),N为SDS的长度
sdsclear| 清空SDS保存的字符串内容| 惰性释放,空间复杂度O(1),不是真正的释放,只是修改free和len属性
sdscat| 将给定C字符串拼接到另一个SDS字符串的末尾,会有空间预分配策略 | O(N),N为字符拼接长度
sdscatsds|将给定SDS字符串拼接到另一个SDS字符串的末尾 | 时间复杂度O(N)
sdscpy | 将给定的C字符串复制到SDS中,覆盖SDS原有字符串 | O(N),N为被复制C字符串的长度
sdsgrowzero| 用空字符串将SDS扩展至指定长度 | O(N)
sdsrange | 保留SDS给定区间内的数据,不在区间内部的数据将会被覆盖或者清除|O(N)
sdstrim | 接收一个SDS和一个C字符串作为参数,从SDS移除所有在C字符串中出现过的字符| O(N^2)
sdscmp | 对比两个SDS是否相同 | O(N)
