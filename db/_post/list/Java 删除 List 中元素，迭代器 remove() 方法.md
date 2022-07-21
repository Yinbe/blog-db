> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.ygbks.com](http://www.ygbks.com/1133.html)

> List list =new ArrayList(); list.add(“asdfffff“); list.add(“sx“); list.add(“kkj“); list.add(“r“); Sy

```
List list =new ArrayList();
list.add("asdfffff");
list.add("sx");
list.add("kkj");
list.add("r");
	
System.out.println("原list：" + list);
	
Iterator<String> iter = list.iterator();
while (iter.hasNext()) {
 String item = iter.next();
 if (item.equals("sx")) {
    iter.remove();
 }
}
	
System.out.println("删除元素后list：" + list);

```

运行结果：

```
原list：[asdfffff, sx, kkj, r]
删除元素后list：[asdfffff, kkj, r]

```

迭代器 remove() 方法虽然方便，但仍有需要注意的地方，要用此法删除元素的前提是该 List 的实现类的 iterator() 方法返回的 Iterator 实现类支持 remove() 方法，否则会报 _java.lang.UnsupportedOperationException_ 异常，常用的 ArrayList 的 Iterator 支持 remove() 方法，但有些情况下就会有问题，来看看以下代码：

```
Integer[] arr = {1, 2, 3, 4, 5};
		List<Integer> list = Arrays.asList(arr);
		Iterator<Integer> iter = list.iterator();
		while (iter.hasNext()) {
		    Integer item = iter.next();
		    if (item == 2) {
		        iter.remove();
		    }
		}
	
		System.out.println("删除元素后list：" + list);

```

这种情况就会运行失败，报 _java.lang.UnsupportedOperationException_ 异常。

使用迭代器 remove() 方法还有需要注意的问题，接着看下面的代码：

```
List<Integer> list = new ArrayList<>();
list.add(1);
list.add(2);
list.add(3);
Iterator<Integer> iter = list.iterator();
while (iter.hasNext()) {
	iter.remove();
}

```

这个例子只是为了展示，比较极端，如果想用这种方法删除 List 所有元素，则会报 _java.lang.IllegalStateException_ 异常，原因就是没有在删除前调用 Iterator 的 next() 方法。

还有一种变体，如下所示：

```
List<String> list = new ArrayList<>();
list.add("a");
list.add("b");
list.add("c");
Iterator<String> iter = list.iterator();
while (iter.hasNext()) {
	String item = iter.next();
    if (item.equals("a") || item.equals("c")) {
        list.remove(item);
    }
}

```

注意，上面的代码调用了 List 的 remove() 而不是 Iterator 的 remove()，如果只删除一个元素，那么在删除后调用 break 语句即可，但这里目的是删除多于 1 个的元素，会报 _java.util.ConcurrentModificationException_ 异常。
