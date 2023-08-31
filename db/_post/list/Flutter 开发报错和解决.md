Flutter 开发报错和解决 - 简书#### 一. flutter: Another exception was thrown: A GlobalKey was used multiple times inside one widget's child list.

**1. 错误信息：**

```
flutter: Another exception was thrown: A GlobalKey was used multiple times inside one widget's child list'


```

![](http://upload-images.jianshu.io/upload_images/260184-ccde48527c402dfc.png)

添加 NavigatorObserver 报错. png

**2. 原因导致：**

我在添加 NavigatorOberver 时，调用了 previousRoute.settings.name。

```
  MaterialApp (
      ...
      navigatorObservers: [
        GLObserver(),
      ],
)

class GLObserver extends NavigatorObserver {
  @override
  void didPush(Route route, Route previousRoute) {
    // 当调用Navigator.push时回调
    super.didPush(route, previousRoute);
    //可通过route.settings获取路由相关内容
    //route.currentResult获取返回内容
    //....等等
    print('didPush' + route.settings.name +  previousRoute.settings.name);
  }

  @override
  void didPop(Route route, Route previousRoute) {
    super.didPop(route, previousRoute);
    print('didPop' + route.settings.name);
  }
}

```

**3. 深入分析：**

然后我发现，具体得原因是因为第一次打开应用的时候，previousRoute 是 null，然后调用了一个为 null 变量的属性，就会报上述错误。我改为如下代码验证一下：

```
class GLObserver extends NavigatorObserver {

  @override
  void didPush(Route route, Route previousRoute) {
    super.didPush(route, previousRoute);
   if (previousRoute == null) {
      print('null');
    }else {
      print(previousRoute);
      print('notNull');
    }
    //  验证代码
    var ddd = null;
    print(ddd.sdd);
  }


```

然后运行，最后报的错误信息一样

```
flutter: Another exception was thrown: A GlobalKey was used multiple times inside one widgets child list.
flutter: Another exception was thrown: A GlobalKey was used multiple times inside one widgets child list.

```

**但是这次又新的发现，就是拉到报错信息的开始，发现 flutter 是给出了具体报错的详细信息的。😳，原先我一直没往上拉。**

```
2019-04-12 09:47:21.158308+0800 FlutterMix[34482:2965311] flutter: ══╡ EXCEPTION CAUGHT BY WIDGETS LIBRARY ╞═══════════════════════════════════════════════════════════
2019-04-12 09:47:21.163617+0800 FlutterMix[34482:2965311] flutter: The following NoSuchMethodError was thrown building CupertinoTheme:
2019-04-12 09:47:21.163807+0800 FlutterMix[34482:2965311] flutter: The getter 'sdd' was called on null.
2019-04-12 09:47:21.163951+0800 FlutterMix[34482:2965311] flutter: Receiver: null
2019-04-12 09:47:21.164111+0800 FlutterMix[34482:2965311] flutter: Tried calling: sdd
2019-04-12 09:47:21.166094+0800 FlutterMix[34482:2965311] flutter:
2019-04-12 09:47:21.166236+0800 FlutterMix[34482:2965311] flutter: When the exception was thrown, this was the stack:

```

**4. 解决办法：**

相信看到这里，都知道怎么解决了，调用属性之前判断是否为 null 即可。

#### 二、Unhandled Exception: type '_InternalLinkedHashMap<dynamic, dynamic>' is not a subtype of type 'Map<String, dynamic>'

报错：Map 的类型不对

```
// 错误
Unhandled Exception: type '_InternalLinkedHashMap<dynamic, dynamic>' is not a subtype of type 'Map<String, dynamic>'


```

**解决办法：** 转换一下 😁

```
new Map<String, dynamic>.from(snapshot.value);

```

#### 三、 Text 控件设置连续的多个空格

可用在显示时，首行文本头部缩进或者其它用处。

```
 (listBean.defaultFlag == 1 ? '''          ''' : "") + listBean.addressStr,

```

**解决办法：** 使用三个点语法

全文完本文由 [简悦 SimpRead](http://ksria.com/simpread) 优化，用以提升阅读体验

使用了 全新的简悦词法分析引擎^ beta^，[点击查看](http://ksria.com/simpread/docs/#/%E8%AF%8D%E6%B3%95%E5%88%86%E6%9E%90%E5%BC%95%E6%93%8E)详细说明

[]()[]()

[]()[]()[一. flutter: Another exception was thrown: A GlobalKey was used multiple times inside one widget&#39;s child list.](https://www.jianshu.com/p/31256febfa03?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation#sr-toc-0)[二、Unhandled Exception: type &#39;_InternalLinkedHashMapdynamic,&#39; is not a subtype of type &#39;Mapstring,&#39;](https://www.jianshu.com/p/31256febfa03?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation#sr-toc-1)[三、 Text 控件设置连续的多个空格](https://www.jianshu.com/p/31256febfa03?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation#sr-toc-2)
