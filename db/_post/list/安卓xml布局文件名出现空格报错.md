**报错信息** ：The layout in layout has no declaration in the base layout folder; this can lead to crashes when the resource is queried in a [configuration](https://so.csdn.net/so/search?q=configuration&spm=1001.2101.3001.7020) that does not match this qualifier

**位置** ：xml 的根布局

**我的项目是** tv 项目，res 下有 layout（默认布局）、layout_land（横屏）、layout_port（竖屏）

**解决方法** ：把横屏布局 layout_land 里所有的 xml 复制一份到 layout 文件夹下。

如果还是解决不了参考 Stack Overflow 的这篇：[https://stackoverflow.com/questions/52547657/the-layout-layout-in-layout-has-no-declaration-in-the-base-layout-folder-erro](https://stackoverflow.com/questions/52547657/the-layout-layout-in-layout-has-no-declaration-in-the-base-layout-folder-erro)

上面是查搜索引擎，检查过没有类似的。后来发现xml文件名称有空格导致的。android studio 居然没有校验
