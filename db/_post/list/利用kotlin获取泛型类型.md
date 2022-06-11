kotlin提供的一个关键字 `reified`（Reification 实化），它标记泛型使之成为实例化类型参数，使抽象的东西更加具体或真实。配合(必须配合inline，编译器手写) `inline`使用可以直接获取泛型的Class.

修改扩展函数：

```kotlin
inline fun <reified T : Activity> Activity.startActivity() {
    startActivity(Intent(this, T::class.java))
}
```

调用：

```kotlin
startActivity<Main2Activity>()
```

首先做一个简单的测试：

```
// 定义
inline fun <reified T> request(url: String) {
    val clazz = T::class.java
    LogUtil.e(clazz.toString())
}

// 调用
request<List<String>>("www.baidu.com")

```

打印如下：

> -->interface java.util.List

发现 `List`是获取到了，但其中的 `String`还是被擦除了，因此对于嵌套泛型无法完整获取其Type。

回想一下Gson是如何解析类似List `<String>`这种嵌套泛型呢，在Gson的注释中有如下代码：

```

// 通过空的匿名内部类获取List<String>的Type
Type listType = new TypeToken<List<String>>() {}.getType();

List<String> target = new LinkedList<String>();
target.add("blah");
Gson gson = new Gson();
// 对象转json
String json = gson.toJson(target, listType);
// json解析
List<String> target2 = gson.fromJson(json, listType);


```

它是通过空的匿名内部类来获取List `<String>`的Type，查看TypeToken的代码可知

```
class TypeToken<T>{
  
    final Type type;

    protected TypeToken() {
        this.type = getSuperclassTypeParameter(getClass());
    }

    static Type getSuperclassTypeParameter(Class<?> subclass) {
        Type superclass = subclass.getGenericSuperclass();
        ...
        ParameterizedType parameterized = (ParameterizedType) superclass;
        return $Gson$Types.canonicalize(parameterized.getActualTypeArguments()[0]);
    }
    ...
}

```

那我们能否用同样的方式获取泛型T的Type呢？试一下：

```

inline fun <reified T> request(url: String) {
    val type = object : TypeToken<T>() {}.type
    LogUtil.e(type.toString())
}

打印结果：

-->java.util.List<? extends java.lang.String>

发现可以获取到其完整的Type，那就可以将其作为参数传递了。
```

> 版权声明：原文地址
> 本文链接：[[juejin.cn/post/702209…](https://juejin.cn/post/7022091183824306212 "https://juejin.cn/post/7022091183824306212"))
