通常来讲，android应用启动的方式分为2种：冷启动和热启动

> 1、冷启动：当启动应用时，后台没有该应用的进程。系统会重新创建一个新的进程分配给该应用。冷启动会先创建和初始化application类，再创建和初始化MainActivity类，最后显示在界面上。
> 2、热启动：当应用启动时，后台有该应用的进程（按back键、home键，应用会退出，当应用的进程依然会保存在内存）。热启动不会走application这一步，而是直接创建初始化MainActivity。

应用启动流程： 当点击app的启动图标时，安卓系统会从Zygote进程中fork创建出一个新 的进程分配给该应用，之后会依次初始化application类，创建mainActivity，加载主题样式Theme中的windowBackground等属性，再inflate布局。当oncreate/onstart/onresume方法都走完，显示界面。

用adb指令测试应用的启动时间

> adb shell am start -W [packageName]/[packageName.MainActivity]

执行成功后将返回三个测量到的时间：

> 1、ThisTime:一般和TotalTime时间一样，除非在应用启动时开了一个透明的Activity预先处理一些事再显示出主Activity，这样将比TotalTime小。
> 2、TotalTime:应用的启动时间，包括创建进程+Application初始化+Activity初始化到界面显示。
> 3、WaitTime:一般比TotalTime大点，包括系统影响的耗时。

减少应用启动的时间

1、在application的构造方法，attachBaseContext(),onCreate()方法中不要进行耗时操作初始化，一些数据如sharedPrefences预存取放在异步线程中。 2、对于mainActivity，由于获取到第一帧前，需要对contetView进行测量，尽量减少布局层次，考虑viewStub的延迟加载策略。在oncreate,onstart,onresume方法中避免耗时操作。 3、加入activity的背景，改善体验。
