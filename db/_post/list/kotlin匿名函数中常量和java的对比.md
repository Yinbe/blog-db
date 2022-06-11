##### 问题

kotlin写匿名函数时遇到了和java实现不一致的情况

kotlin 代码

```kotlin
        mViewModel.upPbMaIsImg2(this,uploadPicBody,upimgNumberPBody){
            var rawDimView = dimView
            val pos = this@ProductModifyFragment.picpos
	}
```

上面转换为java代码

```java
((ProductViewModel)this.mViewModel).upPbMaIsImg2((Fragment)this, uploadPicBody, upimgNumberPBody, (Observer)(new Observer() {
         public final void onChanged(NetworkState it) {
            TextView rawDimView = ProductModifyFragment.this.getDimView();
            int pos = ProductModifyFragment.this.picpos;
	}
}
```

~~可以看出lamba转换后还是java匿名内部类的形式,但是常量 `val pos = this@ProductModifyFragment.picpos ` 转换后为 `int pos = ProductModifyFragment.this.picpos; ` 这里变成了变量，每次都会被赋值，导致 pos 第一次赋值后，还会被赋值，失去常量的作用，导致bug。~~

这里每次都会创建匿名内部类

##### 解决方法

```
将匿名内部类 继承为类实现，将参数传入。
```

##### 实例

kotlin代码

```kotlin
class MObserver(val pos:Int,val adapter: PicListAdapter,val action: () -> Unit) :Observer<NetworkState> {}
```

上面转换为java代码

```java
public static final class MObserver implements Observer {
      private final int pos;
      @NotNull
      private final PicListAdapter adapter;
      @NotNull
      private final Function0 action;

      ...

      public MObserver(int pos, @NotNull PicListAdapter adapter, @NotNull Function0 action) {
         Intrinsics.checkParameterIsNotNull(adapter, "adapter");
         Intrinsics.checkParameterIsNotNull(action, "action");
         super();
         this.pos = pos;
         this.adapter = adapter;
         this.action = action;
      }
   }
```

~~`val pos` 转换为 `private final int pos;` 符合常量的要求~~

~~使用全局变量来持有值，保证回调时不会再次赋值~~

使用独立对象持有值

##### 注意这里与Adapter，click注册回调的区别:

`Adapter，click ` 都只注册一次，而上面的方法每次调用都会创建匿名回调，~~这种情况使用lamba做回调，只会创建局部变量，会导致调用时再次赋值，值被覆盖~~。所以需要继承接口使用类实现，~~使用全局变量来持有值，保证回调时不会再次赋值~~。

```java
        view.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

            }
        });
```
