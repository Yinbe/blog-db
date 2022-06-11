@JvmStatic

在 “companion object” 中的公共函数必须用使用 @JvmStatic 注解才能暴露为静态方法。

如果没有这个注解，这些函数仅可用作静态 Companion 字段上的实例方法。

@JvmField

在 companion object 中的公共、非 const 的属性 实际上为常量 必须用 @JvmField 注解才能暴露为静态字段。

kotlin中声明常量有两种方式: `const val`以及 `@JvmFiled val`

const 将属性标记为编译期常量 `(编译期常量是在程序编译期会替换成字面量)`

> * 位于顶层或者是 `object` 声明 或 `companion object` 的一个成员
> * 只能 String 或原生类型值初始化
> * 只能修饰不可变变量（val）
> * 没有自定义 `getter`

Kotlin中一个类默认是不可以被继承的。如果想要让一个类可以被继承，需要主动声明open关键字。方法也要主动open。
