![](https://note.youdao.com/yws/public/resource/d7c23c4f66743638558ec9cbea1d8b5b/xmlnote/WEBRESOURCEadd36676b89c97bb99b6179874f2fd90/7358)![]()

![](![](https://images.cnblogs.com/cnblogs_com/luoyesiqiu/1445698/o_frida_release.png)https://images.cnblogs.com/cnblogs_com/luoyesiqiu/1445698/o_frida_release.png)![]()![](https://images.cnblogs.com/cnblogs_com/luoyesiqiu/1445698/o_frida_release.png)![](https://images.cnblogs.com/cnblogs_com/luoyesiqiu/1445698/o_frida_release.png)

方法区(JVM)

方法区是线程共享的；
保存在着被加载过的每一个类的信息；可以看做是将类（Class）的元数据，保存在方法区里；运行常量池是方法区的一部分(实际存放在堆内存中)，常量池用于存放编译期生成的各种字面量(static变量、文本字符串，声明为final的常量值)和符号引用。

堆（JVM、要GC）

成员（全局）变量随着对象的建立而建立，随着对象的消失而消失，存在于对象所在的堆内存中。

栈（线程）

每个线程包含一个栈区，某一个函数被调用时，这个函数会在栈内存里面申请一片空间，以后在这个函数内部定义的变量，都会分配到这个函数所申请到的栈。当函数运行结束时，分配给函数的栈空间被收回，在这个函数中被定义的变量也随之被释放和消失。
