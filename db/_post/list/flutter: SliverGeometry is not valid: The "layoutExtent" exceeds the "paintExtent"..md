flutter: SliverGeometry is not valid: The "layoutExtent" exceeds the "paintExtent". 

原因一:
如果出现这个问题, 应该是 SliverPersistentHeaderDelegate 返回的 Widget 中有子 Widget 高度设置不正确.
在 SliverPersistentHeaderDelegate 返回的 Widget 中. 高度不能高于 maxExtent 或者低于 minExtent

```
  @override
  double get maxExtent => 300;

  @override
  double get minExtent => 100;

```

原因二:
如果子控件中使用 Stack+Positioned,Positioned 约束没有写完整, 也会出现这个错误, 将 Positioned 写完整即可解决这个问题.

[参考: SliverGeometry is not valid: The &#34;layoutExtent&#34; exceeds the &#34;paintExtent&#34;. #271](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fxuelongqy%2Fflutter_easyrefresh%2Fissues%2F271)
