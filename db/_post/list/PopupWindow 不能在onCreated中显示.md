> 
>
>> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/c94ae135db94)
>>
>
> 1、碰到的问题
> =============
>
> 在 Fragment 的生命周期 onActivityCreated 里弹出 PopupWindow 引导页面的时候，直接运行提示这个错误
>
> ```
> android.view.WindowManager$BadTokenException: Unable to add window -- token null is not valid; is your activity running?
>
>
> ```
>
> 2、分析原因
> ===========
>
>> popupWindow 显示依赖 activity，并且要等 activity onCreated生命周期方法全部执行完成才能显示，这里应该是 activity 的生命周期还没有走完，所以加载出了问题。
>>
>
> 3、解决办法
> ===========
>
> 经过一番折腾，百度后，总结 3 种方法解决。
>
>> * 我们要确保 activity onCreated加载完成后才能加载 PopupWindow
>> * 把要显示的 PopupWindow 放在 Fragment 的父 Activity 中显示
>>
>
> ##### 方法 1:
>
> 在 Fragment 父 Activity 中的 onCreate 方法里面，找到一个页面的组件，然后调用组件的 post 方法，在 Runnable 里面执行初始化 PopupWindow，因为 post() 可以延迟到所有生命周期方法执行完后执行，比如：
>
> ```
> backButton = (Button)findViewById(R.id.btn_back);
> backButton.post(new Runnable(){
>      @Override
>       public void run() {
>           //构建PopupWindow
>           loadFirstGuide();
>       }
> });
>
>
> ```
>
> ##### 方法 2：
>
> 在 Fragment 父 Activity 中可以定义一个 handler，然后发送延时消息，在 onCreate 方法里面
>
> ```
> handler.sendEmptyMessageDelayed(0, 500);  
>
>
> ```
>
> ```
> private Handler handler = new Handler(){  
>      @Override  
>      public void handleMessage(Message msg) {  
>           switch (msg.what) {  
>               case 0:  
>                   //创建PopupWindow
>                   loadFirstGuide();
>                   break;  
>           }  
>       }  
> };
>
>
> ```
>
> ##### 方法 3:
>
> 重写 onWindowFocusChanged 方法，监听 activity 加载状态，获得焦点之后，即 activity 是加载完毕的了，就可以调用显示 PopupWindow 窗口视图了
>
> ```
> @Override
> public void onWindowFocusChanged(boolean hasFocus) {
>     super.onWindowFocusChanged(hasFocus);
>     if(hasFocus){
>         loadFirstGuide();
>     }
> }
>
>
> ```
>
