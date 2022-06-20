> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/cb5181418c7a)

Android 4.0 规定的有效可触摸的 UI 元素标准是 48dp，这是一个用户手指能准确并且舒适触摸的区域。

日常开发中，如果我们想扩大一个 View 的点击区域，往往通过给 View 设置 padding 即可实现，但是对于某些特殊的情况，如图

![](http://upload-images.jianshu.io/upload_images/1141787-0ce7d6085855a626.png) 开发者选项 - 显示布局边界

因为布局对齐的关系，这个 SeekBar 不能有 paddingTop，而这时又需要在上方增加可响应区域，就只能用 TouchDelegate 了。

这里提供一篇 Android Developer 上介绍 [TouchDelegate](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.com%2Ftraining%2Fgestures%2Fviewgroup%23delegate) 的文档，TouchDelegate 让父视图能够将子视图的可轻触区域扩展到子视图的边界之外。当子视图必须较小，同时又应该具有较大的轻触区域时，此方法很有用。

TouchDelegate 的使用方法很简单，考虑以下这种情形

![](http://upload-images.jianshu.io/upload_images/1199413-0da4b780dffa2111.png) TouchDelegate.png

我们想扩大 View2 的点击区域至 View1 内部的 Bounds 区域，代码如下：

```
view1.post(new Runnable() {
    @Override
    public void run() {
        Rect bounds = new Rect();
        // 获取View2占据的矩形区域在其父View（也就是View1）中的相对坐标
        view2.getHitRect(bounds);
        // 计算扩展后的j矩形区域Bounds相对于View1的坐标，left、top、right、bottom分别为View2在各个方向上的扩展范围
        bounds.left -= left;
        bounds.top -= top;
        bounds.right += right;
        bounds.bottom += bottom;
        // 创建TouchDelegate，delegateView为View2
        TouchDelegate touchDelegate = new TouchDelegate(bounds, view2);
        // 为View1设置TouchDelegate，原因可以参考View.java中mTouchDelegate的注释
        // The delegate to handle touch events that are physically in this view but should be handled by another view.
        view1.setTouchDelegate(touchDelegate);
    }
});


```

使用 TouchDelegate 的扩展点击区域的原理，可以查看 View.java 源码（基于 API 27），前面使用了下面的代码为 View1 设置了 TouchDelegate

```
/**
 * Sets the TouchDelegate for this View.
 */
public void setTouchDelegate(TouchDelegate delegate) {
    mTouchDelegate = delegate;
}


```

当我们点击 View2 内部的区域，仍然会触发 View2 的 onClick()；而当我们点击 View2 外且在 Bounds 内（亦在 View1 内）的区域，根据 Android Touch 事件的分发原理，最终一定会触发 View1 的 onTouchEvent() 方法

```
public boolean onTouchEvent(MotionEvent event) {
    ......
    if (mTouchDelegate != null) {
        if (mTouchDelegate.onTouchEvent(event)) {
            return true;
        }
    }
    ......
}


```

因为我们为 View1 设置了 TouchDelegate，所以会进入 TouchDelegate 的 onTouchEvent()，如果这个方法返回了 ture，View1 的 onTouchEvent() 也会返回 true 并到此结束，对外宣称 View1 消费了这个事件，但实际上并不会触发 View1 的 onClick()；而如果这个方法返回了 false，则会继续执行后面的逻辑。TouchDelegate 的 onTouchEvent() 源码如下：

```
/**
 * Will forward touch events to the delegate view if the event is within the bounds
 * specified in the constructor.
 *
 * @param event The touch event to forward
 * @return True if the event was forwarded to the delegate, false otherwise.
 */
public boolean onTouchEvent(MotionEvent event) {
    // 这里是触摸点相对于View1的坐标
    int x = (int)event.getX();
    int y = (int)event.getY();
    // 是否将event发送给View2
    boolean sendToDelegate = false;
    // 事件是否发生在Bounds内
    boolean hit = true;
    // 作为返回值，标识View1是否消费了event（实际上可能传递给View2消费了）
    boolean handled = false;

    switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
            Rect bounds = mBounds;

            // Down事件发生在Bounds内
            if (bounds.contains(x, y)) {
                // 存储Down事件的处理策略（传递给View2）供后续的Move和Up参考
                mDelegateTargeted = true;
                sendToDelegate = true;
            }
            // !!! 下面被注释的代码为作者添加的优化代码 ！！！
            // 只有加上下面的代码才能保证在点击Rounds区域触发View2的onClick()后
            // 再点击View1仍会触发View1的onClick()
//          else {
//              mDelegateTargeted = false;
//              sendToDelegate = false;
//          }
            break;
        case MotionEvent.ACTION_UP:
        case MotionEvent.ACTION_MOVE:
            // 这里首先参考前面Down事件的处理策略
            sendToDelegate = mDelegateTargeted;
            if (sendToDelegate) {
                Rect slopBounds = mSlopBounds;
                // 再检查事件是否发生在SlopBounds内，它是由Bounds向外扩展TouchSlop形成的
                if (!slopBounds.contains(x, y)) {
                    // Down事件在发生在Bounds内，但Move和Up事件未发生在SlopBounds内，说明在此期间手指滑出了指定区域
                    hit = false;
                }
            }
            break;
        case MotionEvent.ACTION_CANCEL:
            sendToDelegate = mDelegateTargeted;
            mDelegateTargeted = false;
            break;
    }
    // 这里使用的是局部变量来决定事件的处理策略
    if (sendToDelegate) {
        final View delegateView = mDelegateView;

        if (hit) {
            // Offset event coordinates to be inside the target view
            event.setLocation(delegateView.getWidth() / 2, delegateView.getHeight() / 2);
        } else {
            // Offset event coordinates to be outside the target view (in case it does something like tracking pressed state)
            int slop = mSlop;
            event.setLocation(-(slop * 2), -(slop * 2));
        }
        // 将事件分发给View2处理
        handled = delegateView.dispatchTouchEvent(event);
    }
    return handled;
}


```

然而，直接使用 API 27 的 TouchDelegate 会存在一种 bad case：**当点击过一次扩展区域 Bounds（不包括 View2 内的部分），View1 的点击失效**。这是因为当点击过一次 Bounds 区域，mDelegateTargeted 会被置为 true；当下一次点击 View1 时，由于 sendToDelegate 为 false，**Down 事件会交由 View1 处理**；而由于 mDelegateTargeted 仍为 true，**后续的 Move 和 Up 事件还会交由 View2 处理**，阅读 View.java 的 onTouchEvent() 源码可知，这种情况下 View1 的 performClick() 不会被调用，也就不会触发 View1 的 onClick()

好在 Google 工程师已经发现了这个问题，并在 API 28 进行了修复：

![](http://upload-images.jianshu.io/upload_images/1199413-b5bc527dc629af76.png) API 28 TouchDelegate 修复代码

最后，本文通过对 API 27 的 TouchDelegate 进行扩展，也给出一种使用 TouchDelegate 扩展点击区域的[优化方案](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fstevewang41%2FBestTouchDelegate)，该方案具有以下两个优点：

1. 解决了先点击一次扩展区域 Bounds（不包括 View2 内的部分），View1 点击失效的问题。
2. 内部改用绝对坐标，View2 的扩展区域 Bounds 不再局限于其直接父类 View1 内，即被代理的 View 可以是代理 View 的任意祖先 View，使用者只需提供四个方向需要扩展的大小。
