android 打包后依赖库的so没有打进来。

没有改动过任何配置的代码，查看了gradle缓存目录，aar包是带so的


![](https://raw.githubusercontent.com/Yinbe/blog-db/main/db/images/1672217589256.svg)


今天突然就打包后依赖库的so没有打进来。

最后重新开了个工程，打包没有问题了。

猜测是工程缓存问题：把.idea和.gradle删除后，重新导入工程，打包。正常了。
