> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_34083066/article/details/90901456)

最近遇到一个坑，之前的代码将一个订单对象中的明细，一个 list，进行了分组。然后这个 list 就改变了。再往后，他们认为这个 list 没变，又将这个 list 作为最终数据进行了发送。这就导致这个明细回传错误。

查出问题后，我就想，将这个对象如果赋值一份的吧。应该就没问题了。

这里，一定要实现深复制，不然只进行(clone)浅复制的话，list 内的值还是使用的同一块[内存](https://so.csdn.net/so/search?q=%E5%86%85%E5%AD%98&spm=1001.2101.3001.7020)中的。进行分组后，原参数还是会被改变。

所以这里我想到用 [fastjson](https://so.csdn.net/so/search?q=fastjson&spm=1001.2101.3001.7020) 提供的序列化能力，先将对象序列化成一个 json 字符串，然后再反序列化回来。

这样，相当于按照一个字符串序列化了一个新对象，开辟了一个新的内存用于存放这个对象的堆内存内容。这就不会导致原有对象的改变了。

具体代码很简单：

```
String listStr = JSONObject.toJSONString(param.getDetailItems());
List<Dto> list = JSONObject.parseArray(listStr,Dto.class);
```

目前这方案是没有问题的。这期间我也了解了很多关于对象复制的问题。后续会找机会把最近找到的对象复制的相关内容写一个博客。

获取到json数据后如果这些数据会被改变，必须将对象序列化成一个 json 字符串
