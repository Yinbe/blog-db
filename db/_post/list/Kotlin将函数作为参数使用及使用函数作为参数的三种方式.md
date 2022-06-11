Kotlin将函数作为参数使用及使用函数作为参数的三种方式

```kotlin
class Demo {
 
    private fun sayNoWords() {
        println(" no  msg   ")
    }

    fun peopleDo(action: () -> Unit) {
        action()
    }
 
    fun do() {

	//方式1
	peopleDo(::sayNoWords)
	//方式2
	peopleDo(fun() { sayNoWords() })
	//方式3
	peopleDo({ sayNoWords() })
    }
 
}
```

上面的传参方式都会转换为类似如下java代码

```java
	//参数
      var10004.<init>(var10006, var10007, (Function0)(new Function0() {
         // $FF: synthetic method
         // $FF: bridge method
         public Object invoke() {
            this.invoke();
            return Unit.INSTANCE;
         }

         public final void invoke() {
            Demo.this.sayNoWords();
         }
      }));

	//方法
      public void peopleDo(@NotNull Function0 action) {
         this.action = action;
      }
	//调用
      this.action.invoke();
```
