> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/10d1cd822a4f)

正常情况下，xml 布局文件，使用属性 `android:scrollbars="vertical"`就可开启

但我们如果使用代码 new 出 RecyclerView，访问不到该属性，无法开启滚动。
下面有两种办法，对应 new 或者 LayoutInlator 两种路子
1、需要通过代码显示指定 vertical 的 scrollBar

```
<style >
       <item >vertical</item>
   </style>

   <style >
       <item >horizontal</item>
   </style>
```

```
根据recyclerBar的方向进行判断
RecyclerView view = new RecyclerView(new ContextThemeWrapper(DeliveryStyleActivity.this, R.style.XLRecyclerViewBarVertical));
```

2、或者我们不用 new，自己使用 LayoutInflator.inflate 指定 xml 渲染

```
<android.support.v7.widget.RecyclerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:scrollbars="vertical"/>
```

```
View view = LayoutInflater.from(getContext()).inflate(recyclerLayoutRes, null);
```
