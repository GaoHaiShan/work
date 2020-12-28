ConcurrentHashMap		
> 原理		
>> hash表 hash函数计算 key 位置		
>>> hash函数 md5、sha		
>>> 数组存储			
>> hash冲突		
>>> 线性探索 开发寻址			
>>> 链式寻址 hashmap 数组加链表				
>>> hash后在hash 布隆过滤器				
>>> 建立公共溢出区				
>> 线程安全				
>>> hashtable 对整个数组加锁			
>>> chm  每个数组元素设置为 segment 段 1.7			
>>>> 分段锁只对某个段进行加锁			
>>>> 进行hash运算得到分段			
>>>> 每个分段为一个 hashtable			
>>>> 在进行hash运算得到hashtable内下标			
>>>> hash冲突 数组加链表		
>>> chm 1.8			
>>>> 每个元素设置为node (锁node)			
>>>> hash冲突 数组加链表		
>>>> 链表过长优化			
>>>>> 链表长度大于64转红黑树		
> 源码			
>> 初始化  			
>>> 数组为空则进行循环			
>>> 竞争锁			
>>> 初始化node<K,V>数组  		
>>> 扩容 数组长度* 0.75		
>> 赋值			
>>> 内存偏移量操作volatitle 修饰变量 判断当前位置是否为空			
>>> 为空			
>>>> 通过 cas 初始化node,next为空 node记录 key value		
>>> 不为空(hash冲突)			
>>>> 加锁synchrnized所当前node 			
>>>> 有元素			
>>>>> key相等进行覆盖		
>>>>> 不相等将值放在node 的 next 里面			
>>> 添加元素个数			
>>>> 添加元素个数			
>>>> 并发更新			
>>>> 不存在竞争 basecount 			
>>>> 存在竞争countercell[] 储存个数			
>>>> 总个数 = basecount +countercell值		
>>>> 实现			
>>>>> cas成功 修改basecount			
>>>>> cas 失败初始化一个countercell数组长度为2			
>>>>> 随机得到下标 cas 修改下标值		
>>>>> countercell数组扩容		
>>>> put 修改map个数优化原理		
>>>>> 如果 cas 成功则修改base cas 失败则修改countercell值  如果依旧失败 则扩容			
>>> 源码		
>>>> countercell 		
>>>>> 数组为空 则通过cas 进行占位 初始化countercell 并进行赋值		
>>>>> 数组已经初始化 下标位置元素为空 初始化CounterCell 通过 cas 将 counterCell对象 赋给当前元素		
>>>>>  数组元素不为空 通过cas 进行增加值			
>>>>> 数组元素不为空 cas失败 则扩容 每次扩容一倍			
>> 数组扩容(元素个数大于 数组 * 0.75)			
>>> 数据迁移			
>>> 多线程协助扩容（职责划分(定义数据迁移区间)）			
>>> 源码			
>>>> 判断是否需要扩容 			
>>>> 如果当前已经有线程正在进行扩容			
>>>> 判断线程是否需要参与协助扩容			
>>>> 如果线程参与扩容则 进行 sc +1 第一次 sc+2			
>>> 当前没有线程在扩容			
>>>>  sc 高位记录标志  地位记录线程数量			
>>>> 线程扩容分段  根据cpu计算需要参与扩容的线程数			
>>>> 第一个线程初始化 一个为原来数据长度的二倍的新数组		
>>>> 第一个线程 获取迁移区间[16-31]			
>>>> 第二个线程 获取迁移区间[0-15]		
>>>> 进行数据迁移从后往前遍历，判断数据是否需要迁移		
>>>> 迁移过程		
>>>>> 值为为空 fwd 为1 继续往下走		
>>>>> 状态为 moved 表示迁移成功		
>>>>> 否则代表需要进行数据迁移		
>>>>> 高低位迁移：		
>>>>> 地位不需要迁移数据		
>>>>> 高位需要迁移数据 （16扩到32）迁移到当前位置+当前数据长度		
>>>>> 标记高低位		
>>>>> 构建两个 node 链表 		