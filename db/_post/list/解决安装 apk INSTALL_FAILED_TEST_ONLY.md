deubg 包发现无法安装，提示无法解析，使用 [adb](https://so.csdn.net/so/search?q=adb&spm=1001.2101.3001.7020) install 安装后提示 INSTALL_FAILED_TEST_ONLY，原来是 Android Studio 3.0 会在 debug apk 的 manifest 文件 application 标签里自动添加 android:testOnly="true" 属性

解决方法一
在项目中的 gradle.properties 全局配置中设置：

android.injected.testOnly=false
解决方法二，加 -t ：　　
adb install -t app-debug.apk

dd

ss

ss
