> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/Sn_Keys/article/details/126306117)

>         工作中，新接手同事维护老项目，因升级 Android [Gradle](https://so.csdn.net/so/search?q=Gradle&spm=1001.2101.3001.7020) 插件版本与 Gradle 版本不匹配，致使无法构建打包，特此进行了梳理。

**目录**

[1、Android Gradle 插件版本 与 Gradle 版本关系](#t0)

[1.1、修改 Gradle 插件版本](#t1)

[1.2、修改 Gradle 版本](#t2)

[2、JDK 版本 与 Gradle 版本关系](#t3)

[3、Android Gradle 插件版本 和 Android Studio 版本兼容性](#t4)

1、Android Gradle 插件版本 与 Gradle 版本关系
=============================================

为了获得最佳性能，您应使用 Gradle 和插件这两者的最新版本。

<table><tbody><tr><th>插件版本</th><th>所需的 Gradle 版本</th></tr><tr><td>1.0.0 - 1.1.3</td><td>2.2.1 - 2.3</td></tr><tr><td>1.2.0 - 1.3.1</td><td>2.2.1 - 2.9</td></tr><tr><td>1.5.0</td><td>2.2.1 - 2.13</td></tr><tr><td>2.0.0 - 2.1.2</td><td>2.10 - 2.13</td></tr><tr><td>2.1.3 - 2.2.3</td><td>2.14.1 - 3.5</td></tr><tr><td>2.3.0+</td><td>3.3+</td></tr><tr><td>3.0.0+</td><td>4.1+</td></tr><tr><td>3.1.0+</td><td>4.4+</td></tr><tr><td>3.2.0 - 3.2.1</td><td>4.6+</td></tr><tr><td>3.3.0 - 3.3.3</td><td>4.10.1+</td></tr><tr><td>3.4.0 - 3.4.3</td><td>5.1.1+</td></tr><tr><td>3.5.0 - 3.5.4</td><td>5.4.1+</td></tr><tr><td>3.6.0 - 3.6.4</td><td>5.6.4+</td></tr><tr><td>4.0.0+</td><td>6.1.1+</td></tr><tr><td>4.1.0+</td><td>6.5+</td></tr><tr><td>4.2.0+</td><td>6.7.1+</td></tr><tr><td>7.0</td><td>7.0+</td></tr><tr><td>7.1</td><td>7.2+</td></tr><tr><td>7.2</td><td>7.3.3+</td></tr></tbody></table>

1.1、修改 Gradle 插件版本
-------------------------

打开《项目根目录 / build.gradle》文件，写入所需要 android gradle 版本号。

![](https://img-blog.csdnimg.cn/933ad2bfabc84a4dab311fe47497df6d.png)

> ```
> 建议：版本号不要使用动态依赖项，请写入具体的版本号，避免因自动升级致使的难以预料的错误。
>
> ```

1.2、修改 Gradle 版本
---------------------

* 在 Android Studio 的 **File** > **Project Structure** > **Project** 菜单中指定 Gradle 版本
* 在文件《项目根目录 / gradle/wrapper/gradle-wrapper.properties》指定版本号（如：7.4.2）

```
# Gradle配置文件《项目根目录/gradle/wrapper/gradle-wrapper.properties》指定版本号（如：7.4.2）
distributionUrl = "https\://services.gradle.org/distributions/gradle-7.4.2-all.zip"
```

![](https://img-blog.csdnimg.cn/3abe2e5753e14983905c089aa4780b40.png)

[Gradle Distributions![](https://csdnimg.cn/release/blog_editor_html/release2.1.7/ckeditor/plugins/CsdnLink/icons/icon-default.png?t=M666)https://services.gradle.org/distributions/](https://services.gradle.org/distributions/ "Gradle Distributions")![](https://img-blog.csdnimg.cn/463bc2acc6ed46d193e9a014986f3a90.png)

2、JDK 版本 与 Gradle 版本关系
==============================

要正常执行 Gradle 需要 JDK 8~18 版本，目前尚不支持 Java 19 及更高版本。

Java 6 和 7 仍可用于[编译和分叉测试执行](https://docs.gradle.org/current/userguide/building_java_projects.html#sec:java_cross_compilation "编译和分叉测试执行")。

以下为 **Java** 版本与 **Gradle** 版本关系：

<table border="1" cellpadding="1" cellspacing="1"><tbody><tr><th>Java 版本</th><th>第一个支持的 Gradle 版本</th></tr><tr><td>8</td><td>2.0</td></tr><tr><td>9</td><td>4.3</td></tr><tr><td>10</td><td>4.7</td></tr><tr><td>11</td><td>5.0</td></tr><tr><td>12</td><td>5.4</td></tr><tr><td>13</td><td>6.0</td></tr><tr><td>14</td><td>6.3</td></tr><tr><td>15</td><td>6.7</td></tr><tr><td>16</td><td>7.0</td></tr><tr><td>17</td><td>7.3</td></tr><tr><td>18</td><td>7.5</td></tr></tbody></table>

3、Android Gradle 插件版本 和 Android Studio 版本兼容性
=======================================================

Android Studio 构建系统以 Gradle 为基础，并且 Android Gradle 插件添加了几项专用于构建 Android 应用的功能。

以下为 **Android Studio** 版本所需的 **Android Gradle 插件**版本关系：

<table><tbody><tr><th>Android Studio 版本</th><th>所需插件版本</th></tr><tr><td>Arctic Fox   | 2020.3.1</td><td>3.1-7.0</td></tr><tr><td>Bumblebee | 2021.1.1</td><td>3.2-7.1</td></tr><tr><td>Chipmunk   | 2021.2.1</td><td>3.2-7.2</td></tr></tbody></table>

如果您的项目不受某个特定版本的 Android Studio 支持，仍然可以使用[旧版 Android Studio](https://developer.android.google.cn/studio/archive "旧版 Android Studio") 打开和更新您的项目。

**参考：**[Android Gradle 插件版本说明  |  Android 开发者  |  Android DevelopersAndroid Studio 构建系统以 Gradle 为基础，并且 Android Gradle 插件添加了几项专用于构建 Android 应用的功能。![](https://www.gstatic.cn/devrel-devsite/prod/v3f8eafc9e9ec34d001886958ac58f6b3d255ba70e9584b93488d1cf3a23653aa/android/images/favicon.png)https://developer.android.google.cn/studio/releases/gradle-plugin.html#updating-plugin](https://developer.android.google.cn/studio/releases/gradle-plugin.html#updating-plugin "Android Gradle 插件版本说明  |  Android 开发者  |  Android Developers") [Compatibility Matrix![](https://csdnimg.cn/release/blog_editor_html/release2.1.7/ckeditor/plugins/CsdnLink/icons/icon-default.png?t=M666)https://docs.gradle.org/current/userguide/compatibility.html#compatibility](https://docs.gradle.org/current/userguide/compatibility.html#compatibility "Compatibility Matrix")
