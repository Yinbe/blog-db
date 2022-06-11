### 配置gradle

[使用 Maven Publish 插件](https://developer.android.com/studio/build/maven-publish-plugin)

```
// Because the components are created only during the afterEvaluate phase, you must
// configure your publications using the afterEvaluate() lifecycle method.
afterEvaluate {
    publishing {
        publications {
            // Creates a Maven publication called "release".
            release(MavenPublication) {
                // Applies the component for the release build variant.
                from components.release

                // You can then customize attributes of the publication as shown below.
                groupId = 'com.example.MyLibrary'
                artifactId = 'final'
                version = '1.0'
            }
            // Creates a Maven publication called “debug”.
            debug(MavenPublication) {
                // Applies the component for the debug build variant.
                from components.debug

                groupId = 'com.example.MyLibrary'
                artifactId = 'final-debug'
                version = '1.0'
            }
        }
    }
```

详细内容大家去官网看吧

`Android Gradle 在升级到 7.0 之后就必须使用 Java 11 以及之后的版本 jitpack.yml`

```
before_install:
  - sdk install java 11.0.10-open
  - sdk use java 11.0.10-open

jdk:
  - openjdk11

```

### 打 Release 包

![github](https://img-blog.csdnimg.cn/20210402100449349.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hhb2ppYWdvdQ==,size_16,color_FFFFFF,t_70)

点击完之后按照下面的格式填写信息：

![release](https://img-blog.csdnimg.cn/20210402100509728.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hhb2ppYWdvdQ==,size_16,color_FFFFFF,t_70)

第一个框是版本号，第二个框是标题，第三个框是描述，根据个人情况填写就行，填写完之后点击绿色按钮进行发布就行了

### JitPack 发布

打开 JitPack 官网：https://jitpack.io/

![jitpack](https://img-blog.csdnimg.cn/20210402100540703.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hhb2ppYWdvdQ==,size_16,color_FFFFFF,t_70)

这块有两个选择，通过右上角登录你的 `Github` 账号或者在搜索框搜索你刚才上传库的地址，都可以，自己选择，选择之后点击 `Look up` 会出现如下：

![get it](https://img-blog.csdnimg.cn/20210402100601240.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hhb2ppYWdvdQ==,size_16,color_FFFFFF,t_70)

点击 `Get it` 静静等待即可。

### 使用依赖

上面完成之后直接就可以使用了，现在工程级目录下的 `build.gradle` 中添加如下代码：

```
	allprojects {
		repositories {
			...
			maven { url 'https://jitpack.io' }
		}
	}

```

然后在你的 `module` 级目录下的 `build.gradle` 中添加如下代码：

```
	dependencies {
	        implementation 'com.github.zhujiang521:Banner:Tag'
	}

```
