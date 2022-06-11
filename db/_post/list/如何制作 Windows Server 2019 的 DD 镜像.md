# 如何制作 Windows Server 2019 的 DD 镜像

[ 技术](https://teddysun.com/category/tech)  [秋水逸冰](https://teddysun.com/author/teddysun)  发布于: 2018-10-07  更新于: 2018-10-07  42024 次围观  [27 次吐槽](https://teddysun.com/544.html#comments)

DD 命令是 Linux 下的磁盘读写常用命令。它可以将已有的硬盘镜像文件直接写到硬盘上。通过 DD 命令，我们可以把系统由 Linux 改造成 Windows，这样不仅能获得一个纯净的系统，而且也能省下不少费用。网上 DD 镜像文件有很多，但是鱼龙混杂，不是版本不合适，就是害怕有后门木马。所以求人不如求己，自己制作的镜像才是最好的。
下面以制作目前最新的 Windows Server 2019 系统的 DD 镜像为例，记录一下整个过程。

### 事前准备

1，MSDN 原版 Windows Server 2019 系统的 iso 文件。此处以英文版的 en_windows_server_2019_x64_dvd_3c2cf1202.iso 为例。
2，[7zip](https://www.7-zip.org/)
3，[Dism++](https://www.chuyu.me/zh-Hans/index.html)
4，[virtio 驱动](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.149-1/virtio-win-0.1.149.iso)
5，一款支持 Hyper-V 的服务器（可选）

### 编辑镜像

1，使用 7zip 打开 en_windows_server_2019_x64_dvd_3c2cf1202.iso 并从中提取文件 install.wim，其路径位于 \sources\install.wim，将其单独解压到本地硬盘。此处假设为 D:\install.wim。
2，新建一个目录，此处假设为 D:\Win2019。将下载回来的 Dism++ 解压到一个单独目录下，根据系统的不同运行 Dism++x64.exe（64 位） 或 Dism++x86.exe （32位）
3，在打开的 Dism++ 界面里选择菜单，文件，挂载映像。在弹出的窗口里，第一个浏览那里，点击选择刚才解压出来的 D:\install.wim，此时 Dism++ 会读出 install.wim 里包含的各个版本，我选择了 ServerDatacenter 版。记住不要选择只读模式，点击确定，等待映像挂载完毕。

![](https://teddysun.com/wp-content/uploads/2018/dd_windows_01.png "dd Windows")

4，挂载准备就绪后，点击打开会话，进入主界面。

![](https://teddysun.com/wp-content/uploads/2018/dd_windows_02.png "dd Windows")

5，将事前准备好的 virtio 驱动 iso 文件，此处下载回来的文件名为 virtio-win-0.1.149.iso，用 7zip 将之解压到 D:\virtio-win-0.1.149。点击驱动管理，添加驱动。选择驱动所在的文件夹后，会自动安装驱动。在弹出窗口，点击确定。

![](https://teddysun.com/wp-content/uploads/2018/dd_windows_04.png "dd Windows")

![](https://teddysun.com/wp-content/uploads/2018/dd_windows_05.png "dd Windows")

6，如果要适配某些独立服务器比如 Kimsufi，需要手动安装 Intel 的网卡驱动。
[点击下载](https://downloadmirror.intel.com/25016/eng/PROWinx64.exe)。下载后，用 7zip 打开该文件，将 PROWinx64.exe\PRO1000\Winx64\NDIS65 目录复制出来备用。
然后需要对文件 e1c65x64.inf 魔改一下才能使用，内容如下，注意对比（此处为天坑，切记）。

```
[Intel.NTamd64.10.0]
; DisplayName                   Section        DeviceID
; -----------                   -------        --------
%E1502NC.DeviceDesc%            = E1502,       PCI\VEN_8086&DEV_1502
%E1502NC.DeviceDesc%            = E1502,       PCI\VEN_8086&DEV_1502&SUBSYS_00008086
%E1502NC.DeviceDesc%            = E1502,       PCI\VEN_8086&DEV_1502&SUBSYS_00011179
%E1502NC.DeviceDesc%            = E1502,       PCI\VEN_8086&DEV_1502&SUBSYS_00021179
%E1502NC.DeviceDesc%            = E1502.10.0.1,       PCI\VEN_8086&DEV_1502
%E1502NC.DeviceDesc%            = E1502.10.0.1,       PCI\VEN_8086&DEV_1502&SUBSYS_00008086
%E1502NC.DeviceDesc%            = E1502.10.0.1,       PCI\VEN_8086&DEV_1502&SUBSYS_00011179
%E1502NC.DeviceDesc%            = E1502.10.0.1,       PCI\VEN_8086&DEV_1502&SUBSYS_00021179
%E1503NC.DeviceDesc%            = E1503.10.0.1,       PCI\VEN_8086&DEV_1503
%E1503NC.DeviceDesc%            = E1503.10.0.1,       PCI\VEN_8086&DEV_1503&SUBSYS_00008086
%E1503NC.DeviceDesc%            = E1503.10.0.1,       PCI\VEN_8086&DEV_1503&SUBSYS_00011179
%E1503NC.DeviceDesc%            = E1503.10.0.1,       PCI\VEN_8086&DEV_1503&SUBSYS_00021179
```

之后重复添加驱动的步骤，将此驱动加入。
7，点击更新管理，扫描，安装，开始安装更新。此处因为是最新版系统，暂时还没有更新包。

![](https://teddysun.com/wp-content/uploads/2018/dd_windows_06.png "dd Windows")

8，点击系统优化，有一系列选项，按需进行优化。我的建议是将防火墙关闭。

![](https://teddysun.com/wp-content/uploads/2018/dd_windows_07.png "dd Windows")

9，编辑完镜像后，另存为新的镜像。此处假设为 D:\win2019.wim。等待新的镜像保存完毕。

![](https://teddysun.com/wp-content/uploads/2018/dd_windows_08.png "dd Windows")

### 创建 VHD 虚拟硬盘

1，依次点击开始菜单，Windows 管理工具，计算机管理。点击磁盘管理，操作，创建 VHD，在弹出的窗口，指定计算机上的虚拟硬盘位置，此处假设为 D:\2019.vhd，选择虚拟硬盘大小为 15GB，点击确定，具体如图所示。注意硬盘不宜设置过大，不然将来 DD 的时候，VPS 或服务器的硬盘小于你指定的磁盘大小的话会出错。

![](https://teddysun.com/wp-content/uploads/2018/dd_windows_11.png "dd Windows")

2，等待片刻，虚拟磁盘创建完毕。然后选中新建的 VHD 硬盘，右键点击初始化磁盘，分区选择 MBR，点击确定。右键点击新建简单卷，并一路下一步确认，盘符任意指定，此处假设为 T 盘。至此虚拟磁盘创建完毕。

![](https://teddysun.com/wp-content/uploads/2018/dd_windows_12.png "dd Windows")

![](https://teddysun.com/wp-content/uploads/2018/dd_windows_13.png "dd Windows")

![](https://teddysun.com/wp-content/uploads/2018/dd_windows_14.png "dd Windows")

### 创建带有 Windows Server 2019 系统的 VHD 虚拟硬盘

1，此时另存为的新镜像 D:\win2019.wim 已经创建完毕。在 Dism++ 界面，依次点击，文件，释放镜像，第一个浏览那里，点击选择 D:\win2019.wim，第二个浏览那里，选择刚建立的 VHD 磁盘 T 盘，选中添加引导和格式化，点击确定，在弹出的窗口里选择更多（此处很重要），选择刚建立的磁盘盘符，点击确认。

![](https://teddysun.com/wp-content/uploads/2018/dd_windows_15.png "dd Windows")

![](https://teddysun.com/wp-content/uploads/2018/dd_windows_16.png "dd Windows")

![](https://teddysun.com/wp-content/uploads/2018/dd_windows_17.png "dd Windows")

2，释放镜像完毕后，关闭 Dism++。此时再将该 VHD 磁盘，此处选中 T 盘，右键点击，弹出。

### 创建无人值守的 DD 包（可选）

之前创建完毕的 VHD 虚拟硬盘实际上就可以使用了。
有些 VPS 提供的控制面板有 VNC，可以用鼠标，还能快捷输入 Ctrl + Alt + Del，那么此 VHD 虚拟硬盘就可以直接拿来使用。
而实际上很多地方是不能 VNC 的，因此就需要做成无人值守，DD 完了立刻就能使用远程登录进入桌面。此时，我们需要借助 Hyper-V 开启远程桌面及定制，优化一下系统。

1，依次打开 Hyper-V 管理器，连接到服务器，本地计算机，操作，新建，虚拟机，指定名称和位置，第一代（1），内存，网络连接，使用现有虚拟硬盘，选择 D:\2019.vhd，完成。以下演示图截自日文系统，实际上界面都是一样的。

![](https://teddysun.com/wp-content/uploads/2018/hyper-v_01.png "hyper-v Windows")

![](https://teddysun.com/wp-content/uploads/2018/hyper-v_02.png "hyper-v Windows")

![](https://teddysun.com/wp-content/uploads/2018/hyper-v_03.png "hyper-v Windows")

![](https://teddysun.com/wp-content/uploads/2018/hyper-v_04.png "hyper-v Windows")

![](https://teddysun.com/wp-content/uploads/2018/hyper-v_05.png "hyper-v Windows")

![](https://teddysun.com/wp-content/uploads/2018/hyper-v_06.png "hyper-v Windows")

![](https://teddysun.com/wp-content/uploads/2018/hyper-v_07.png "hyper-v Windows")

![](https://teddysun.com/wp-content/uploads/2018/hyper-v_08.png "hyper-v Windows")

2，创建完毕虚拟机后，建议取消检查点。选中虚拟机，右键点击，设置，检查点，取消勾选启用检查点。

![](https://teddysun.com/wp-content/uploads/2018/hyper-v_11.png "hyper-v Windows")

3，选中虚拟机，右键点击，连接，启动。然后就像平时安装系统一样，输入一些信息，同意条款，设置 Administrator 密码，进入桌面后，可以进行各种设置了。

![](https://teddysun.com/wp-content/uploads/2018/hyper-v_09.png "hyper-v Windows")

![](https://teddysun.com/wp-content/uploads/2018/hyper-v_10.png "hyper-v Windows")

比如修改注册表关闭 Ctrl + Alt + Del，以及开启远程桌面，不验证用户级别等操作。此处略过。

### 压缩 VHD 虚拟硬盘

1，选中 D:\2019.vhd，右键选择 7zip，添加压缩包，压缩格式选择 gzip，选项默认，点击确认。7zip如果压缩报错，用管理员模式启动即可。
2，等待压缩完成后，将压缩包重命名，上传到你自己的服务器，利用 Apache 或 Nginx 等 WebServer 做一个下载直链即可直接拿来使用了。

### 参考链接：

[https://www.fmqcloud.com/archives/makedd.html](https://www.fmqcloud.com/archives/makedd.html)
[http://www.chinavps.net/zhuanzai/dd-windows-10-zhizuo.html](http://www.chinavps.net/zhuanzai/dd-windows-10-zhizuo.html)
[https://www.hostloc.com/thread-357177-1-1.html](https://www.hostloc.com/thread-357177-1-1.html)
[https://www.hostloc.com/thread-480867-1-1.html](https://www.hostloc.com/thread-480867-1-1.html)
