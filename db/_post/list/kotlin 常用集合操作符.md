* ### any

说明：如果至少有一个元素符合判断条件，则返回true，否则false,例：

```
val list = listOf(1, 2, 3, 4, 5)
list.any { it > 10 }
```

结果为false

* ### all

说明：如果集合中所有的元素都符合判断条件，则返回true否则false,例：

```
val list = listOf(1, 2, 3, 4, 5)
list.any { it < 10 }
```

结果为true

* ### count

说明：返回集合中符合判断条件的元素总数。例：

```
val list = listOf(1, 2, 3, 4, 5)
list.count { it <3 }
```

结果为2

* ### forEach

说明：遍历所有元素，并执行给定的操作（类似于Java 中的for循环）。例：

```
val list = listOf(1, 2, 3, 4, 5)
list.forEach{ Log.i(TAG,it.toString()) }
```

结果为：1 2 3 4 5

* ### forEachIndexed

说明：与forEach作用一样，但是同时可以得到元素的index。例：

```
val list = listOf(1, 2, 3, 4, 5)
list.forEachIndexed { index, i -> Log.i(TAG, "index:" + index + " value:" + i.toString()) }
 
结果为
index:0 value:1
index:1 value:2
index:2 value:3
index:3 value:4
index:4 value:5
```

* ### drop

说明：返回去掉前n个元素的列表。例：

```
val list = listOf(1, 2, 3, 4, 5)
var s = list.drop(2)
s.forEach {
Log.i(TAG, it.toString())
}
```

结果为 ：3 4 5（去掉了前两个元素）

* ### filter

说明：过滤所有符合给定函数条件的元素。例：

```
list.filter { it > 2 }
```

结果为：3 4 5

* ### flatMap

说明：遍历所有的元素，为每一个创建一个集合，最后把所有的集合放在一个集合中。例：

```
list.flatMap { listOf(it, it + 1) }
```

结果为： 1 2 2 3 3 4 4 5 5 6（每个元素都执行加1后作为一个新元素）

* ### map

说明：返回一个每一个元素根据给定的函数转换所组成的集合。例：

```
list.map { it * 2 }
```

结果为：2 4 6 8 10

* ### zip

说明：返回由pair组成的List，每个pair由两个集合中相同index的元素组成。这个返回的List的大小由最小的那个集合决定。例：

```
list.zip(listOf(7, 8))
```

结果为：(1, 7) (2, 8)

* ### first

说明：返回符合给定函数条件的第一个元素。例：

```
list.first { it > 2 }
```

结果为：3

* ### last

说明：返回符合给定函数条件的最后一个元素。例：

```
list.last { it % 2 == 0 }
```

结果为：4
