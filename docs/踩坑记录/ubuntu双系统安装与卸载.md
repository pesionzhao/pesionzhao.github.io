# windows安装ubuntu双系统
最后删除系统了,简而言之,由于安装了双系统后开机很慢,而且英伟达驱动安装了两次全都黑屏,而且要等巨长时间才能crtl+alt+f2进入命令行,最后放弃了这个系统,转用WSL

[Windows安装Ubuntu双系统（Win11+最新Ubuntu22.04.1LTS）](https://blog.csdn.net/syluxhch/article/details/128071404?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522169798119316800215033882%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=169798119316800215033882&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-128071404-null-null.142^v96^pc_search_result_base1&utm_term=ubuntu%E5%8F%8C%E7%B3%BB%E7%BB%9Fwin11&spm=1018.2226.3001.4187)

[windows11安装ubuntu22.04双系统教程（亲测）](https://zhuanlan.zhihu.com/p/644425528)

[Windows+Ubuntu20.04双系统安装教程](https://zhuanlan.zhihu.com/p/363640824)

### 制作启动盘
下载ubuntu ISO文件

https://ubuntu.com/download/desktop

往期版本下载,找20.04: https://cn.ubuntu.com/download/alternative-downloads

或者国内镜像 [阿里](https://mirrors.aliyun.com/ubuntu-releases/?spm=a2c6h.25603864.0.0.409445f8KikYzb)

下载写入软件 https://rufus.ie/zh/

打开Rufus软件，导入镜像文件；分区类型选择GPT，目标分区类型选UEFI（非CSM）；文件系统选择NTFS，簇大小选默认：4096字节（默认），设置完成后点击开始。接下来提示选择写入模式，选择第一个：以ISO镜像模式写入（推荐），点击OK。

### 磁盘分区

开始->右键->磁盘管理->选中对应分区->压缩卷

如果只有两块硬盘的话,需要在c盘所在的硬盘压缩出400M当作ubuntu引导,如果只有一块硬盘的话则不用管这个,反正也只能在c盘所在硬盘压缩

之后再另一块硬盘压缩出想给ubuntu的大小,我给了300G

### 安装过程

重启机器按delete进入bios或者在设置->系统->恢复->高级启动进入

![Alt text](https://img-blog.csdnimg.cn/aa1a7765a538407793f478f4887edab0.jpeg)

或者

![](https://pic1.zhimg.com/80/v2-9cf9f2a77e9d27444360e3eb09840230_720w.webp)

选择“Ubuntu （safe graphics）”->选择语言“中文（简体）”->“安装Ubuntu”

选择**最小安装**, 之后

![](https://pic4.zhimg.com/80/v2-80415169d39eafc47d7bd734c4280daf_720w.webp)

对自己之前所分好的区进行安装ubuntu,找到空闲的分区而且空间对的上的!!一定不要看错了!!!,其他小容量空闲的空间不用管

![](https://pic4.zhimg.com/v2-5abd6f09c7ba35464d5cee983ba56587_r.jpg)

之前所专门分好的400m作为efi引导,单硬盘就选整个空闲的空间里自行选出来400m就行

|  空间大小 |  分区类型 | 位置 | 用作| 挂载点|
|---|---|---|---|---|
|  C盘所在硬盘的400M | **逻辑分区**/主分区  | 起始位置| EFI System Partition| **无** 或 /boot |
|16G|逻辑分区|起始位置|swap area|无|
|150G|主分区|起始位置|ext4|/|
|140G|逻辑分区|起始位置|ext4|/home|

下面的这一步很重要：在分区界面的下方，选择安装启动项的位置，我们刚刚不是创建了400M的efi分区吗，现在你看看这个区前面的编号是多少，比如是/dev/sda1,不同的机子会有不同的编号，。下拉列表选择这个efi分区编号（这里一定要注意，windows的启动项也是efi文件，大小大概是104M，而我们创建的ubuntu的efi大小是400M，一定要选对），之后点击"Install Now"

之后一路安装就OK

### 安装之后

#### 选择最佳服务器

软件和更新->设置 下载自... 其他站点->选择最佳服务器 选择速度最快的 镜像源

换源(不是太推荐)

备份

```bash
sudo cp /etc/apt/sources.list  /etc/apt/sources-bak.list 
```

更改

```bash
sudo gedit /etc/apt/sources.list
```

换成

```shell
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse

# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
# # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse

deb http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse
# deb-src http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
# # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
```
更新,安装一些必要的东西

```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install build-essential 
sudo apt-get install vim
```

#### 安装英伟达驱动(慎重,纯踩坑)

安装失败--[Ubuntu18.04安装 NVIDIA驱动](https://zhuanlan.zhihu.com/p/109909317)

失败--[Ubuntu18.04安装 NVIDIA驱动](https://zhuanlan.zhihu.com/p/109909317)

可能会成功--[双系统下 Ubuntu安装NVIDIA显卡驱动及错误解决办法](https://zhuanlan.zhihu.com/p/261314033)

由于现在在终端输入nvidia-smi没用,所以需要安装驱动

这里有三种方法,我试了两种都失败了,听说第三种可以但我没有试过

安装驱动前先禁用nouveau驱动

```shell
sudo gedit /etc/modprobe.d/blacklist.conf
```

在最后添加

```
blacklist nouveau
options nouveau modeset=0
```

重启后验证

```bash
lsmod | grep nouveau
```

如果没有反应则禁用成功

方法一

```bash
ubuntu-drivers devices
```

找到标有recommended的驱动的名字

软件与更新->附加驱动->选择该驱动->应用更改

等一段安装好后会提示要重启,重启完就卡死在clean block那,然后跳出一堆OK,证明安装失败了,所以要卸载掉

ctrl+alt+f2进入命令行,登陆后,输入

```shell
sudo apt-get remove --purge nvidia*
```

方法二

添加PPA源,据说可以解决黑屏

```
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt-get update
sudo apt-get install nvidia-driver-435 #此处数字要对应上面查询到的recommend版本号
sudo apt-get install mesa-common-dev
```

然后重启,又是之前那样,所以我就不想继续搞了

方法三(未尝试)可能会成功

[双系统下 Ubuntu安装NVIDIA显卡驱动及错误解决办法](https://zhuanlan.zhihu.com/p/261314033)

具体方法为到英伟达官网下载runfile到本地进行安装

### 卸载ubuntu

#### 空间清理

[教你彻底卸载Ubuntu双系统，去污不残留](https://blog.csdn.net/qq_42257666/article/details/120721561)

首先一定要进bios确定windows在首选项,也就是开机优先进入windows系统

进入windows后,磁盘管理,将之前给ubuntu划的分区全都删除,然后将上一个卷进行扩展,那个400m的efi可能没法在这里进行删除,所以要借助一个工具DiskGenius,下载网址

https://www.diskgenius.cn/download.php

将400m的引导进行删除分区操作即可

#### 删除引导项
上一步就已经删干净了,但是bios里还会有ubuntu引导项,如果想删的话可以进行如下操作

cmd输入

```shell
diskpart
list disk
```

选择c盘所在的磁盘

```shell
select disk 0
list partition
```

确定Windows的EFI分区，一般是200多M，然后为它分配盘符。

盘符不可与已有盘符重复，比如你电脑已有CDE盘，那么就分配26个字母中排在E后面的字母F、G、H等等，最好隔几个字母，防止你插上U盘和驱动器

```shell
select partition 1
assign letter=J
```

此时，Win+E 打开此电脑，就会有刚刚分配的盘符J. 由于权限不够，不能直接打开该磁盘，可通过记事本间接打开。在Windows附件中，用管理员权限运行记事本。

记事本->以管理员方式打开->左上角文件->打开->找到EFI文件夹->进入->删除ubuntu文件夹

返回刚刚命令行

```shell
remove letter=J
```

大功告成