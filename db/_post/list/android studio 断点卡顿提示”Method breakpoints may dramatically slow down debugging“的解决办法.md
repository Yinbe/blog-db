> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_38084097/article/details/111310067)

项目无法启动了
--------------

简单介绍一下事情的过程：昨天修复代码，代码部分处理完成之后，启动 debug 模式准备调试程序，但是在 IDEA 上点击 debug 按钮后却一直无法正常启动项目，控制台上也能看到日志，但是基本都在几个步骤后无法再继续下去，重复试了几次都无法正常启动项目，日志输出到特定的那几句后就停止了，调试代码也就更无从谈起了。

原因
----

由于是第一次碰到这个问题，不太清除到底是什么原因，只记得当时重复的试了几次项目的 clean install，但是这几次的重新构建也很缓慢，平时重新构建只需要几分钟的等待时间，这几次竟然平均需要一小时构建成功。我此时感觉肯定是哪出了问题，而出问题的地方也就只有以下几处：

1. 代码问题。
2. IDEA 设置问题。
3. 系统问题。

经过反复与 git 分支对比，没有发现代码处理有明显错误，且能通过 run 启动成功且无明显缓慢感，暂时排除 1。
对于 2 和 3，当时感觉是肯定是 2，因为修改代码过程中，并未改动系统本身任何设置问题，不可能说前一个小时还能正常运行的系统，下一个小时就特别缓慢，且能 run 正常启动，也侧面说明系统本身是没有问题的。
既然如此，那就只剩下 2 了，但是找了半天也仍旧没有头绪。问问同事和朋友有没有遇到过类似问题的，但是他们都说没遇到过。心里真是崩溃的。既然如此，祭出大招，重启 idea，结果还是无果，但是无意间看到右下方弹窗提示：Method breakpoints may dramatically slow down debugging 。

Method Breakpoints
------------------

发现提示之后，赶紧 debug 一下，发现也有这个提示，当即就认为，应该就是它了。于是赶紧一顿 google。终于在 idea 官方文档中发现了蛛丝马迹：[IDEA 官方文档](https://intellij-support.jetbrains.com/hc/en-us/articles/206544799-Java-slow-performance-or-hangups-when-starting-debugger-and-stepping)
![](https://img-blog.csdnimg.cn/20201217105720998.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zODA4NDA5Nw==,size_16,color_FFFFFF,t_70#pic_center)

简单翻译如下：
由于 JVM 设计的原因，方法断点将大大降低调试器的速度，评估起来很昂贵。 删除方法断点，并考虑使用常规的行断点。 为了验证您没有任何方法断点，请打开项目根目录中的. idea / workspace.[xml](https://so.csdn.net/so/search?q=xml&spm=1001.2101.3001.7020) 文件（如果使用旧项目格式，则打开 .iws 文件），然后在 method_breakpoints 节点内查找任何断点。

解决
----

到此，果断打开自己 IDEA 的所有断点瞅瞅，果然，发现了一次 Method breakpoint。
![](https://img-blog.csdnimg.cn/20201217110334324.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zODA4NDA5Nw==,size_16,color_FFFFFF,t_70#pic_center)
去除断点，重新 debug，不到一分钟，启动成功，完美解决。

总结
----

到此为止，事件的起因、经过、结果都大致介绍完毕，在以后的搬砖中，还是少用方法断点，也尽量不要在项目里打过多的断点，调试哪里就在哪里打上，调试完把断点去掉就好。
