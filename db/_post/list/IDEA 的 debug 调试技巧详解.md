> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/future_god_qr/article/details/121250865)

目录

[一、概述](#t0)

[二、debug 操作分析](#t1)

[1、打断点](#t2)

[2、运行 debug 模式](#t3)

[3、重新执行 debug](#t4)

[4、让程序执行到下一次断点后暂停](#t5)

[5、让断点处的代码再加一行代码](#t6)

[6、停止 debug 程序](#t7)

[7、显示所有断点](#t8)

[8、添加断点运行的条件](#t9)

[9、屏蔽所有断点](#t10)

[10、把光标移到当前程序运行位置](#t11)

[11、单步跳过](#t12)

[12、可以跳入方法内部的执行一行代码操作](#t13)

[13、跳出方法](#t14)

[14、直接执行到光标所在位置](#t15)

[15、在控制台改变正在 debug 的数据](#t16)

一、概述
========

* debug 调试也叫断点调试
* 在程序的某一行打上断点，则在 debug 模式下运行到断点位置时会暂停，便于程序员观察代码的执行情况
* 学会 debug，有助于在程序运行未达到理想情况时，对程序的各个流程进行分析
* 本文只详细描述了 debug 的一些基本的常用操作，如果有缺漏欢迎评论区留言~

二、debug 操作分析
==================

1、打断点
---------

* 在程序的某一行位置，数字右边的空白部分使用鼠标左键点击一下，出现红点即为打上了一个断点

![](https://img-blog.csdnimg.cn/9e3026905fcd4c7a9a7b0dce93f324e9.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_11,color_FFFFFF,t_70,g_se,x_16)

2、运行 debug 模式
------------------

* 方式一
  * 选中要进行 debug 的程序，点击右上角的 debug 按钮

![](https://img-blog.csdnimg.cn/f52559e407cf4dd9af9438fc616abca5.png)

* 方式二
  * 在要进行 debug 的程序处右键，选中下图选项

![](https://img-blog.csdnimg.cn/a703e498d8dc4e91be40b3ba06a7a5be.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_10,color_FFFFFF,t_70,g_se,x_16)

3、重新执行 debug
-----------------

* 点击下图按钮，会关闭当前 debug 的程序并重新启动 debug

![](https://img-blog.csdnimg.cn/6a18ce960271427892dbc4a8df8e568d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_17,color_FFFFFF,t_70,g_se,x_16)

4、让程序执行到下一次断点后暂停
-------------------------------

* 点击下图的按钮，debug 会继续运行程序，直到遇到下一次断点后暂停

![](https://img-blog.csdnimg.cn/5347586e6a414e7fbacc2863e4e7f44d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_18,color_FFFFFF,t_70,g_se,x_16)

* 举例
  * 下图是一个循环操作，在打断点的位置点击上面说的按钮，相当于再循环一次，到代码第 9 行时停止

![](https://img-blog.csdnimg.cn/4653841c52c54a3183914354054609dd.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_17,color_FFFFFF,t_70,g_se,x_16)

5、让断点处的代码再加一行代码
-----------------------------

* 点击下图的加号，可以在断点处加一行代码，比如下图中的 count++ 即为新添加的代码
  * 选中 count++，右键点击 Edit 可以编辑该代码
  * 选中该行代码（count++），点击加号下面的减号，可以删除该行代码

![](https://img-blog.csdnimg.cn/b7843e76b1b64b488e93c9cd8b6e72a6.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_18,color_FFFFFF,t_70,g_se,x_16)

* 选中下图的眼镜，变为分屏操作

![](https://img-blog.csdnimg.cn/ac95d9ee4ebb4c2c8e573e623e34f779.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_20,color_FFFFFF,t_70,g_se,x_16)

![](https://img-blog.csdnimg.cn/18ec87566c3340689daa95295557fa12.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_20,color_FFFFFF,t_70,g_se,x_16)

**举例**

* 下图是没添加额外代码之前的截图

![](https://img-blog.csdnimg.cn/155c34d01574452f9924b608cb26a642.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_20,color_FFFFFF,t_70,g_se,x_16)

*  添加一句 count++，并点击左边红色框中的按钮，执行到下一次断点，即循环了一次

![](https://img-blog.csdnimg.cn/33ebf61346304b9c9679c6f836ed99b7.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_20,color_FFFFFF,t_70,g_se,x_16)

*  效果和运行步骤见下图

![](https://img-blog.csdnimg.cn/0994f7950ff1421ca6cf4116e1fafe23.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_20,color_FFFFFF,t_70,g_se,x_16)

6、停止 debug 程序
------------------

* 点击下图按钮停止 debug 程序
* 注意
  * 运行的如果是 javaSE 项目，点一下就停止
  * 运行的如果是 javaWeb 项目，需要点两下
    * 第一下停止代码的当前线程
    * 第二下停止服务器

![](https://img-blog.csdnimg.cn/7ce1d0c26f4a45ec9050155f0f653557.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_18,color_FFFFFF,t_70,g_se,x_16)

7、显示所有断点
---------------

* 点击下图按钮，会显示所有断点

![](https://img-blog.csdnimg.cn/5651d84b51cf45d1bab9072c4c7d1ccc.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_17,color_FFFFFF,t_70,g_se,x_16)

*  点击后出现下图所示界面，可以添加断点运行的条件，见下一条功能解释

![](https://img-blog.csdnimg.cn/93a8e0b13c4d45e3a4adf260d9ca24b6.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_20,color_FFFFFF,t_70,g_se,x_16)

8、添加断点运行的条件
---------------------

* 选中断点，右键后即可编辑断点运行的条件
  * 满足条件时程序才会在该断点处停下

![](https://img-blog.csdnimg.cn/e6b57eeaa0294cdf9364c2b0f6a6cb25.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_20,color_FFFFFF,t_70,g_se,x_16)

*  比如添加 i>=5，重新 debug 后的效果如下图所示

![](https://img-blog.csdnimg.cn/82a7ffe03cd54958a8801e21eceb704a.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_19,color_FFFFFF,t_70,g_se,x_16)

*  此时会发现第 7 条显示所有断点信息处，可以看到下图效果

![](https://img-blog.csdnimg.cn/daab0ee1bfe94a329aa68db1c95f2080.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_20,color_FFFFFF,t_70,g_se,x_16)

9、屏蔽所有断点
---------------

* 点击下图按钮，可以屏蔽所有断点

![](https://img-blog.csdnimg.cn/f7e06ed221044991be85af4fca6748f9.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_12,color_FFFFFF,t_70,g_se,x_16)

* 屏蔽前![](https://img-blog.csdnimg.cn/677a8547e65844eebac684de9f46bc71.png)
* 屏蔽后![](https://img-blog.csdnimg.cn/46bb93d283a640df802eec5d8fd1bbab.png)
* 屏蔽的断点在 debug 的时候不会运行
  * 如果程序调试后觉得没问题了，可以屏蔽掉所有断点继续运行程序查看效果

10、把[光标](https://so.csdn.net/so/search?q=%E5%85%89%E6%A0%87&spm=1001.2101.3001.7020)移到当前程序运行位置
------------------------------

* 点击下图按钮后，会把鼠标光标移动到当前程序运行位置
  * 当程序代码量很大的时候，可以通过该按钮快速定位到程序运行位置

![](https://img-blog.csdnimg.cn/a44a214ccd894d52884cd8cd20e9665c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_13,color_FFFFFF,t_70,g_se,x_16)

* 如下图所示
  * 假设程序运行到第 9 行断点处，鼠标光标在第 11 行，点击该按钮后光标就会移动到第 9 行

![](https://img-blog.csdnimg.cn/15566342c22e47b390f7922e06c080e9.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_13,color_FFFFFF,t_70,g_se,x_16)

11、单步跳过
------------

* 点击下图按钮，会一行一行执行自己编写的代码
  * 如果碰到方法，该按钮不会进入到该方法内部
  * 快捷键 F8

![](https://img-blog.csdnimg.cn/1c54b8240f6e4d719f8aae936a925fb2.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_11,color_FFFFFF,t_70,g_se,x_16)

12、可以跳入方法内部的执行一行代码操作
--------------------------------------

* 下图中的蓝色箭头和红色箭头都可以执行一行代码，如果遇到方法时会进入方法内部，区别在于
  * 蓝色箭头只会跳进自己写的方法，如果是系统已经写好的方法，蓝色箭头无法跳入该方法
  * 红色箭头不管是自己写的方法，还是系统已经定义好的方法，都可以跳入方法内部

![](https://img-blog.csdnimg.cn/23a065624b024b29ba358d96c1e827eb.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_18,color_FFFFFF,t_70,g_se,x_16)

* 如下图所示
  * ArrayList 的 add 方法是系统已经写好的，蓝色箭头无法跳入方法内部，但是红色箭头可以跳入方法内部
  * printMessage（）是自定义方法，红色和蓝色箭头都可以跳入该方法内部

![](https://img-blog.csdnimg.cn/f24471618ac74e69842c047606635c13.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_17,color_FFFFFF,t_70,g_se,x_16)

13、跳出方法
------------

* 下图的两个按钮都可以跳出方法
  * 第二个按钮是关闭窗口的意思，同样可以起到跳出方法的作用
  * 在进入方法内部的时候使用这两个按钮

![](https://img-blog.csdnimg.cn/0deedd480bd94b0f9d93aab8bc5d33d1.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_10,color_FFFFFF,t_70,g_se,x_16)

14、直接执行到光标所在位置
--------------------------

* 点击下图的按钮，程序会执行到光标所在的位置
  * 前提是光标前面没有断点，否则程序还是会在光标前面的断点处暂停

![](https://img-blog.csdnimg.cn/75d500280cad4a7497a31bb49c1a73a4.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_13,color_FFFFFF,t_70,g_se,x_16)

15、在控制台改变正在 debug 的数据
---------------------------------

* 在控制台选中某个变量，右键点击 Set Value 可以改变该变量的值
  * 如果想测试某个地方的数据如果是正确的会是什么效果，可以手动更改该处变量的值

![](https://img-blog.csdnimg.cn/9fcd3a72eebf47a9be694fabec9549cd.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiN6L-b5aSn5Y6C5LiN5pS55ZCNb3Zv,size_20,color_FFFFFF,t_70,g_se,x_16)

补充：debug 调试看代码时，一般用 F9 跳到下一个断点，打断点的目的是你想看程序执行到这个位置时会有什么效果，或者是到达断点的位置后再继续往下看实现的过程；用 F7 去跳进方法内部，看具体的实现细节；用 F8 去看当前位置代码往下的执行情况（不跳入具体方法的内部）
