minSDKVersion

设置sdk最低版本的。作用就是操作系统会拒绝低于该标准的APP的安装。

compileSdkVersion

compileSdkVersion 告诉 Gradle 用哪个 Android SDK 版本编译你的应用。

buildToolVersion

buildeToolVersion是你构建工具的版本，其中包括了打包工具aapt、dx等等

targetSdkVersion

targetSDKVersion就是设置SDK目标版本，目标版本的设置就是为了告诉Android系统：本APP是设计计划给哪个API级别运行的。

汇总：

minSdkVersion <= targetSdkVersion <= compileSdkVersion==buildToolVersion

如果 compileSdkVersion 是你的最大值，minSdkVersion 是最小值，那么最大值必需至少和最小值一样大且 target 必需在二者之间。

理想上，在稳定状态下三者的关系应该更像这样：

minSdkVersion (lowest possible) <= targetSdkVersion == compileSdkVersion(latest SDK) ==buildToolVersion(latest SDK)

用较低的 minSdkVersion 来覆盖最大的人群，用最新的 SDK 设置 targetSdkVersion、compileSdkVersion和buildToolVersion。
