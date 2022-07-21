> 
>
>> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/8d6b421e1950)
>>
>
>     在 Java 中，使用 List 时想要删除某个特定个元素怎么办？很好办！因为 List 接口有 remove() 这个方法，我们只需要调用 remove() 方法，就可以删除 list 中的某个元素。但是 list 自带的有一些坑，在**相邻有相同元素**时会掉坑：
>
> 使用 list.remove() 删除：
>
> ![](http://upload-images.jianshu.io/upload_images/16768087-78aa5eb5334b6396.png)
>
> 我们看到有两个 “a” 元素相邻，但是删除时却只删除了一个，这是为何呢？因为删除时，每删除一个元素，后边的元素都会左移一位，也就是下标会减 1，**在 for 循环中，删除第一个 “a” 时，i（下标）为 0，此时 list 重排，后边的元素全部左移 1 位，也就是说第二个 “a” 元素的下标从 1 变为了 0，而此时 for 循环进行已经第二次循环了，执行了 i++，i 的值为 1，对应为 “b” 元素，然后向后循环，再也找不到 “a” 元素了**。所以相邻元素有重复的话，只能删除一个。这明显不符合我们的需求。
>
> 那解决办法是什么呢？
>
> **1. 删除后元素后，i-1**
>
> ![](http://upload-images.jianshu.io/upload_images/16768087-b9570b4576f8bb43.png)
>
> 删除一个元素后，后边的元素左移 1 位，此时 i-1，保证了下次循环能访问到左移了 1 位的元素。
>
> **2. 反向删除**
>
> 我们先从后边的元素开始循环，一个一个的往前面循环，找出特定元素删除
>
> ![](http://upload-images.jianshu.io/upload_images/16768087-b4154911026f4e71.png)
>
> 这样，就算删除了倒数第一个 "a" 元素，list 重排，也只是把后边的元素左移 1 位，此时倒数第一个 “a” 元素（下标为 1）被删除，接着 b 替代了 a 成为了下标为 1 的元素，但前边的元素不变，i--  = 0 后依然能够找到其相邻的 a 元素。
>
> **3. 使用迭代器删除（iterator）（推荐）**
>
> ![](http://upload-images.jianshu.io/upload_images/16768087-2449e797b0f85be7.png) Iterator.remove() 方法
>
> Iterator.remove() 方法会在删除当前迭代对象的同时，会保留原来元素的索引。所以用迭代删除元素是最保险的方法，建议大家使用 List 过程，这其实和上面第一中方法类似，只不过 iterator 内部帮我们做了类似 i-1 的操作。推荐使用这种做法，因为我们不保证每次都记得手动把下标减去 1。
>
> **4. 赋值给新的 list**
>
> ![](http://upload-images.jianshu.io/upload_images/16768087-a9edf38556b95a73.png) 赋值给新 list
>
> 其实我们可以转换思维，可以过滤掉不需要的元素后赋值给新的 list 对象，利用 java8 的 lambda 表达式和强大的 stream 流形式进行内部迭代来过滤掉特定元素，我们只需一行代码就可以实现。虽然这种方式比较简洁，但是定义了新变量，旧的 list 就只能等待漫长的 gc 了。
>
> **注意：**
>
> **1. 在进行普通 for 循环删除时，不要把 list.size() 抽离出去赋值给变量，然后用此变量做为 for 条件, 因为删除时，list.size() 的值是会改变的，要把 list.size 作为 for 条件。**
>
> ![](http://upload-images.jianshu.io/upload_images/16768087-56412460674598a7.png)
>
> **2. 不能在增强 for（foreach）里使用 list.remove() 方法，因为 foreach 循环会把 list 以 iterator 方式进行迭代，调用 list.remove() 后会使 iterator.hasNext() 出问题。**
>
> **3. 在使用 iterator 时，循环体内只能执行一次 iterator.next() 方法，因为执行一次 iterator.next()，会使 list 下标后 1 位**
>
> ![](http://upload-images.jianshu.io/upload_images/16768087-ff1683b12437b47e.png)
>
