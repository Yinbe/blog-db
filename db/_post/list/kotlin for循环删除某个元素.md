>
>
> ### 方法一：Iterator迭代器
>
> ```Kotlin
> val list = arrayListOf("1", "2", "3", "4", "5", "6")
>
> val iterator = list.iterator()
> while (iterator.hasNext()){
>     if (iterator.next() == "4"){
>         iterator.remove()
>     }
> }
>       
> println("result: $list")  //result: [1, 2, 3, 5, 6]
> ```
>
> ### 方法二：for循环  (倒序for删除元素)
>
> ```Kotlin
> val list = arrayListOf("1", "2", "3", "4", "5", "6")
>
> for (index in list.size - 1 downTo 0){
>     if (list[index] == "3") {
>         list.removeAt(index)
>     }
> }
>
> println("result: $list") //result: [1, 2, 4, 5, 6]
> ```
>
> ### 方法三：removeIf
>
> ```Kotlin
> val list = arrayListOf("1", "2", "3", "4", "5", "6")
>
> //minSdkVersion >= 24(N)
> list.removeIf {
>     it == "5"
> }
>
> println("result: $list") //result: [1, 2, 3, 4, 6]
> ```
>
> ### 方法四：removeAll + filter
>
> ```Kotlin
> val list = arrayListOf("1", "2", "3", "4", "5", "6")
>
> list.removeAll(list.filter {
>     it == "2"
> })
>
> println("result: $list") //result: [1, 3, 4, 5, 6]
> ```
>
