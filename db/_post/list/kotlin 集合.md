集合在我们实际开发中用的还是比较频繁的，Kotlin中的集合不同于Java中的集合， Kotlin中的集合根据“是否可变”，分为：可变集合与不可变集合。

> 可变集合可以在初始化后add新数据，不可变集合只能get数据，不能add数据
>
>> * **列表** ：**List** /**MutableList** ；
>> * **集** ：**Set** /**MutableSet** ；
>> * **映射** ：**Map** /**MutableMap** ；
>> * **集合** ：**Collection** /**MutableCollection** ；
>> * **迭代器** ：**Iterable** /**MutableIterable** ；
>>

### List

```
    var emptyList= emptyList<String>()  //创建空的list 不可变
    var list= listOf(1,2,3)   //创建普通list 不可变
    var mlist= mutableListOf(1,2,3)  //创建可修改的List
    var arrayList= arrayListOf(1,2,3)   //创建一个arrayList
```

### Set

```
    var emptySet= emptySet<String>()  //创建空的set
    var set = setOf(1,2,3)  //创建一个普通的set
    var mset= mutableSetOf(1,2,3)    //创建一个可变的set
    var hashSet= hashSetOf(1,2,3)    //创建一个hashSet
    var linkedSet= linkedSetOf(1,2,3)  //创建一个linkedSet
    var sortedSet= sortedSetOf(1,2,3)   //创建一个sortedSet
```

### Map

```
var emptyMap= emptyMap<Int,String>()  //创建一个空的map
var map= mapOf(1 to "one",2 to "two")  //创建一个普通的map
var mMap= mutableMapOf(1 to "one",2 to "two") //创建一个可变的map
var hashMap= hashMapOf(1 to "one",2 to "two")  //创建一个hashMap
var linkedMap= linkedMapOf(1 to "one",2 to "two") //创建一个linkedMap
var sortedMap= sortedSetOf(1 to "one",2 to "two") //创建一个sortedMap
```

## 集合操作符速览

Kotlin中关于集合的操作符有六类：

* **元素操作符**
* **总数操作符**
* **过滤操作符**
* **映射操作符**
* **顺序操作符**
* **生产操作符**

### 元素操作符

> contains —— 判断集合中是否有指定元素，有返回true
> elementAt —— 查找下标对应的元素，如果下标越界会抛IndexOutOfBoundsException
> elementAtOrElse —— 查找下标对应元素，如果越界会根据方法返回默认值(最大下标经方法后的值)
> elementAtOrNull —— 查找下标对应元素，越界会返回Null
> first —— 返回符合条件的第一个元素，没有 抛NoSuchElementException
> firstOrNull —— 返回符合条件的第一个元素，没有 返回null
> indexOf —— 返回指定下标的元素，没有 返回-1
> indexOfFirst —— 返回第一个符合条件的元素下标，没有 返回-1
> indexOfLast —— 返回最后一个符合条件的元素下标，没有 返回-1
> last —— 返回符合条件的最后一个元素，没有 抛NoSuchElementException
> lastIndexOf —— 返回符合条件的最后一个元素，没有 返回-1
> lastOrNull —— 返回符合条件的最后一个元素，没有 返回null
> single —— 返回符合条件的单个元素，如有没有符合或超过一个，抛异常
> singleOrNull —— 返回符合条件的单个元素，如有没有符合或超过一个，返回null

### 总数操作符

> any —— 判断集合中 是否有满足条件 的元素；
> all —— 判断集合中的元素 是否都满足条件；
> none —— 判断集合中是否 都不满足条件，是则返回true；
> count —— 查询集合中 满足条件 的 元素个数；
> reduce —— 从 第一项到最后一项进行累计 ；
> reduceRight —— 从 最后一下到第一项进行累计；
> fold —— 与reduce类似，不过有初始值，而不是从0开始累计；
> foldRight —— 和reduceRight类似，有初始值，不是从0开始累计；
> forEach —— 循环遍历元素，元素是it，可对每个元素进行相关操作；
> forEachIndexed —— 循环遍历元素，同时得到元素index(下标)；
> max —— 查询最大的元素，如果没有则返回null；
> maxBy —— 获取方法处理后返回结果最大值对应的那个元素的初始值，如果没有则返回null；
> min —— 查询最小的元素，如果没有则返回null；
> minBy —— 获取方法处理后返回结果最小值对应那个元素的初始值，如果没有则返回null；
> sumBy —— 获取 方法处理后返回结果值 的 总和；
> dropWhile —— 返回从第一项起，去掉满足条件的元素，直到不满足条件的一项为止

### 过滤操作符

通过 某个条件 来对集合中的元素进行过滤，过滤后会返回一个处理后的列表结果，不会改变原列表

> filter —— 过滤 掉所有 满足条件 的元素
> filterNot —— 过滤所有不满足条件的元素
> filterNotNull —— 过滤NULL
> take —— 返回从第一个开始的n个元素
> takeLast —— 返回从最后一个开始的n个元素
> takeWhile —— 返回不满足条件的下标前面的所有元素的集合
> drop —— 返回 去掉前N个元素后 的列表
> dropLastWhile —— 返回从最后一项起，去掉满足条件的元素，直到不满足条件的一项为止
> slice —— 过滤掉 非指定下标 的元素，即保留下标对应的元素过滤list中
> 指定下标的元素(比如这里只保留下标为1,3,4的元素)

### 映射操作符

> map —— 将集合中的元素通过某个 方法转换 后的结果存到一个集合中;
> mapIndexed —— 除了得到 转换后的结果 ，还可以拿到Index(下标);
> mapNotNull —— 执行方法 转换前过滤掉 为 NULL 的元素
> flatMap —— 合并两个集合，可以在合并的时候做些小动作；
> groupBy —— 将集合中的元素按照某个条件分组，返回Map；

### 顺序操作符

> reversed —— 相反顺序
> sorted —— 自然排序(升序)
> sortedBy —— 根据方法处理结果进行自然(升序)排序
> sortedDescending —— 降序排序
> sortedByDescending —— 根据方法处理结果进行降序排序

### 生产操作符

> zip —— 两个集合按照下标组合成一个个的Pair塞到集合中返回
> partition —— 根据判断条件是否成立，拆分成两个 Pair
> plus —— 合并两个List，可以用”+”替代
> unzip —— 将包含多个Pair的List 转换成 含List的Pair
