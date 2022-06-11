android中组件必须是view的直接子类或间接子类，其中view有一个名为viewGroup的子类，用于定义容器的组件（LinearLayout都是viewGroup的子类）。两者的区别是如果组件中还有子组件，就一定从viewGroup类继承，否则从view类继承。

要了解view的工作原理，先从activity的组成结构说起

> activity负责容器的生命周期及活动，窗口通过window来管理
> window负责窗口管理（子类为phoneWindow）,窗口的绘制和渲染交给decorView完成 decorView是view的根，layout.xml为decorView的子视图contentParent的子视图
> layout.xml是我们定义的布局文件，最终inflate为decorView的子组件

phonewindow还关联了mWindowManager的windowmanager对象，windowManager会创建一个viewRootlmpl对象来和windowManagerService进行沟通，windowManagerService能获取触摸事件、键盘事件或轨迹球事件，并通过viewRootlmpl将事件分发给各个activity。另外viewRootlmpl还负责activity整个GUI的绘制。而绘制是从viewRootlmpl的performTraversals()开始的，performTraversals()包含三个重要的方法：

> 1、performMeasure()
> 2、performLayout()
> 3、performDraw()

1、performMeasure()
performMeasure()方法负责组件自身尺寸的测量。只有当模式为wrap_content的时候才需要进行尺寸的测量。容器的尺寸依赖于子组件的大小，必须先测量子组件的大小，会调用子view的measure()方法。但组件的大小最终是由setFrame()方法决定的，该方法一般会参考measure出来的尺寸大小。

2、performLayout() performLayout()方法用于确定子组件的位置。该方法只针对viewGroup容器类，为容器中的子view精确定义位置和大小。performLayout()调用layout(),在layout()方法中，在定位之前如果需要重新测量组件大小，则先调用onMeasure()方法，接下来执行setOpticalFrame()或setFrame()方法确定自身位置与大小此时保存了相关值，与具体绘制无关。随后onLayout()的方法被调用。 onLayout()方法在这里是当前组件为容器时，负责定义其子组件的位置。如果子组件也是一个容器，依然要负责其子组件位置的定位。依此类推，从最顶层的decorView开始，从上往下驱动，最后每个组件都放到了它应该出现的位置。 onLayout()方法和onMeasure()一样，是为开发人员预留的，自定义容器时必须重写。

3、performDraw()方法执行组件的绘制功能。每个组件只需负责自身的绘制。performDraw()调用draw()，draw()调用drawSoftware()。绘制组件是通过Canvas类完成的，paint类配置绘制参数。为了提高绘图效率，使用了surface技术，surface提供了双缓存，加快了绘图效率。 drawSoftware()方法中调用了mView的draw()方法，mView是activity中view的根decroview，也是容器，相当于FrameLayout。 下面以FrameLayout为例。FrameLayout的draw()，一是调用父类super.draw()方法绘制自己，二是将前景位图画在canvas上。FrameLayout继承自viewgroup。

先介绍一下view类的draw()方法

> background.draw() 绘制背景
> onDraw() 绘制自己
> dispatchDraw() 绘制子视图
> onDrawScrollBars() 绘制滚动条

background是一个Drawable对象，直接绘制在canvas上，并与组件要绘制的内容不互相干扰。 onDraw()方法和onMeasure()与onLayout()一样，用于绘制组件本身。 dispatchDraw()也是一个空方法，用于容器组件。容器中的子组件必须通过dispatchDraw()方法进行绘制，viewgroup实现了该方法。在dispatchDraw()中，循环遍历每个子组件，并调用drawChild()方法绘制子组件，而子组件又调用view的draw()方法绘制自己。
组件的绘制也是一个递归过程，activity的根一定是容器，根容器绘制结束后开始绘制子组件，子组件如果是容器则继续往下绘制。

总结一下：UI界面绘制从开始到结束要经历以下几个过程 1、测量大小，执行onMeasure()方法 2、组件定位，执行onLayout()方法 3、组件绘制，执行onDraw()方法
