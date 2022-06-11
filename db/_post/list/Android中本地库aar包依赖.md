> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/u014038624/article/details/113102858)

> 过时了！现在东西变化太快了，在 4.1.2 版本的 Android Studio 上，这种方式已经不支持了。新版上本地引入 aar 或 jar 文件的方式，请参考 [Create an Android library](https://developer.android.google.cn/studio/projects/android-library)

1. 背景

---

前一段时间我在开发中需要用到第三方的控件 Android-PickerView，我在 build.[gradle](https://so.csdn.net/so/search?q=gradle&spm=1001.2101.3001.7020) 中的依赖是这么写的：

```
compile 'com.contrarywind:Android-PickerView:4.1.3'

```

可以看出这是一个网络库依赖。由于本地配置了[网络代理](https://so.csdn.net/so/search?q=%E7%BD%91%E7%BB%9C%E4%BB%A3%E7%90%86&spm=1001.2101.3001.7020)和国内镜像库，除了第一次需要下载慢一些，编译运行没有任何问题。不过在我将代码提交到 svn 上的时候，jenkins 标红了…在有网情况下，使用网络库依赖没问题，但我们的 [Jenkins](https://so.csdn.net/so/search?q=Jenkins&spm=1001.2101.3001.7020) 服务器编译要求离线，所使用的 gradle 版本也相对较低，因此只能就不能使用网络库依赖了。由于上述依赖并不是纯 Java 代码，还包括了资源文件，使用 aar 包依赖比较方便。

2. 使用 aar 包依赖

---

较高版本 gradle 的支持更简单的 aar 引用方式，不过低版本的 gradle 却并不能兼容。网上的文章多数都是较高版本 gradle 使用 aar 依赖的方式，不过我们的 [svn](https://so.csdn.net/so/search?q=svn&spm=1001.2101.3001.7020) 上使用的 gradle 版本较低，这里将描述高低两种版本 gradle 使用 aar 依赖的方式。

### 2.1 较高版本 gradle 中引用 aar 依赖

这里较高版本 gradle 使用是 gradle 4.1，对应 Android Studio gradle 3.0.1 插件。高版本 gradle 上使用 aar 本地依赖很简单，基本上与 jar 的依赖方式一致。
1）将所需要的 aar 包 copy 到应用所在 module 的 libs 目录下
2）在应用所在 module 里的 build.gradle 里添加：

```
compile fileTree(dir: 'libs', include: ['*.aar'])

```

或者如果同时有 jar 依赖的话，可以写成：

```
compile fileTree(dir: 'libs', include: ['*.jar','*.aar'])

```

### 2.2 较低版本 gradle 中引用 aar 依赖

这里较低版本使用的 gradle 版本是 2.2.1，相应的 gradle 插件版本为 1.2.3。使用方式其实也不麻烦。
1）将所需要的 aar 包 copy 到应用所在 module 的 libs 目录下
2）在应用所在 module 里的 build.gradle 添加依赖（以添加的文件 pickerview.aar，wheelview.aar 为例）：

```
 compile(name: 'pickerview', ext: 'aar')
 compile(name: 'wheelview', ext: 'aar')

```

然后在在 build.gradle 最外层添加

```
repositories {
    flatDir {
        dirs 'libs' //this way we can find the .aar file in libs folder
    }
}

```

注意低版本 gradle **不** 支持

```
compile fileTree(dir: 'libs', include: ['*.aar'])

```

这种依赖方式。高版本 gradle 可以兼容低版本的这种使用方式。

3. 从网络库依赖中获取 aar 包

---

aar 包可以很方便的进行离线库依赖。而对于 module 依赖（Eclipse 的项目依赖），也可以将其打包成 aar 以简化项目的结构。但是很多很多开源库的文档基本都是告诉你添加一句 compile 语句进行依赖，例如

```
compile 'com.contrarywind:Android-PickerView:4.1.4'

```

搜索下载 aar 包，有些网络环境下却并不容易找到合适的 aar 包下载。如何比较容易的获取到需要的 aar 包呢？以下以 PickerView 为例描述。

### 3.1 添加网络库依赖

根据开源库文档，在 module 的 build.gradle 中添加网络库依赖

```
compile 'com.contrarywind:Android-PickerView:4.1.3'

```

* 点击编译，编译时候打开网络，必要时添加国内镜像库。编译好后 aar 文件就已经在本地缓存里有了。

  ### 3.2 从缓存中复制 aar 文件

  上述编译好后将 Android Studio 切换到 Project 视图模式。
* 展开 External Libraries，可以在里面找到 com.contrarywind:Android-PickerView-4.1.3 和 com.contrarywind:wheelview-4.0.5
* 右键选择 Library Properties 可以找到依赖包的缓存目录
* 复制路径后并从文件管理进入到该文件所在目录的父级目录。这里要注意的一点是我们要使用的不是里面罗列的 jar 文件。
* 在该目录地下找到某个目录中所包含的 aar 文件，我这里是

```
Android-PickerView-4.1.3.aar

```

将该文件 copy 出来备用
以同样的方式获取并 copy 出

```
wheelview-4.0.5.aar

```

这两个 aar 文件就可以以作为 aar 依赖离线引用了。
**注意**：以上使用的是 Android Studio3.0.1 进行的操作。如果用的 Android Studio 版本较低，右键可能没有 Library Properties 这一选项。这种情况下，可以在 setting 中找到缓存根目录，如

```
C:\Users\user\.gradle\

```

进入到缓存根目录后，进入到

```
C:\Users\user\.gradle\caches\modules-2\files-2.1\

```

通过添加时间等方式，找到刚刚从网上下载的 aar 文件根目录。
比如我们这个案例就在 com.contrarywind 目录下，很容易可以找到所需的两个 aar 文件。

对于获取到的 aar 文件，修改文件名也可以使用。

4. 总结

---

* 搜索工具使用的优先级会影响效率。刚开始使用搜狗搜索到许多排名靠前的文章对我有较大误导，最后还是在 stackoverflow 上找到了答案。
