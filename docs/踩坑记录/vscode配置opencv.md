# vscode配置opencv4.1.1（c++)环境

**所需环境**

- vscode
- cmake
- mingw-w64

## mingw下载

mingw下载地址: [https://sourceforge.net/projects/mingw-w64/files/](https://sourceforge.net/projects/mingw-w64/files/)

建议直接下载官方给的编译好的版本
如果使用online installer会出现 *the file has been downloaded incorrectly* 的错


![](https://s2.loli.net/2022/09/08/epBQYO81xuL46kg.png)


将bin文件夹加入环境变量

> H:\mingw\mingw64\bin

在终端输入g++ -verion, gdb --version检查是否安装成功

![](https://s2.loli.net/2022/09/08/DV67ClEKhHvi1Za.png)

**重要的点!!,一定要将mingw的bin文件放到环境变量第一的位置，如果之前添加过git的环境变量,二者之间有重名的libstdc++-6.dll，导致编译出现”no such file or directory“**

## opencv配置

### 方法一 使用预编译好的opencv库，省去编译的麻烦

opencv预编译库下载地址： [https://github.com/huihut/OpenCV-MinGW-Build](https://github.com/huihut/OpenCV-MinGW-Build)
注意 **一定要按照对应的configuration核对mingw版本，否则会编译会出错**

![](https://s2.loli.net/2022/09/08/5xuQYFvDXJh3mLZ.png)

配置环境变量
例如：
> H:\Packages\OpenCV-MinGW-Build-OpenCV-4.1.1-x64\x64\mingw\bin


### 方法二 自己编译opencv库

创建build文件夹存放编译文件，打开cmake选择source和build文件夹进行编译，不出意外会出现一堆红色信息，最后显示Configure done，是正常的。如果执行时中断，则存在其他问题。在执行完后，勾选**ENABLE_CXX11**，(可选BUILD_opencv_world，BUILD_EXAMPLES)，不勾选WITH_MSMF。
一般会出现ffmpeg下载失败的问题，是由于无法访问github导致的，可以发现在source文件夹下的.cache/有三个大小为0kb的文件，可以通过自行找CMakeDownloadLog.txt中的下载网址进行进行下载，下载后在命令行中输入

```shell
certutil -hashfile xxx MD5
```
xxx代表那三个文件

查看文件md码与对应0kb文件名是否一致，**之后将文件名字改为原0kb文件**

![](https://s2.loli.net/2022/09/08/nYKGpDrOaX1dMUQ.png)

之后再点击config 没有报错后点击generate

之后在build文件夹下在命令行中输入minGW32-make -j 4
完成后输入minGW32-make install

将bin目录加入环境变量

## 配置vscode环境
安装c/c++ extension,不再赘述了。

之后主要配置3个json 
点击生成活动文件选择g++会生成tasks.json
主要更改"args"的内容，添加
```json
"args": [
    "-fdiagnostics-color=always",
    "-g",
    "${file}",
    "-o",
    "${fileDirname}\\${fileBasenameNoExtension}.exe",
    "-I","${workspaceFolder}/include",
    "${workspaceFolder}/lib/basic.cpp",
    "-I","H:/Packages/OpenCV-MinGW-Build-OpenCV-4.1.1-x64//include",
    "-I","H:/Packages/OpenCV-MinGW-Build-OpenCV-4.1.1-x64/include/opencv2",
    "-L","H:/Packages/OpenCV-MinGW-Build-OpenCV-4.1.1-x64/x64/mingw/lib",                
    "-llibopencv_calib3d411",
    "-llibopencv_core411",
    "-llibopencv_dnn411",
    "-llibopencv_features2d411",
    "-llibopencv_flann411",
    "-llibopencv_highgui411",
    "-llibopencv_imgcodecs411",
    "-llibopencv_imgproc411",
    "-llibopencv_ml411",
    "-llibopencv_objdetect411",
    "-llibopencv_photo411",
    "-llibopencv_stitching411",
    "-llibopencv_video411",
    "-llibopencv_videoio411"
],
```
在调试按钮的右边点击设置配置调试文件会生成launch.json 不用更改
ctrl+shift+P搜索c/c++ UI配置会生成c_cpp_properties.json
在includePath下添加得到
```json
"includePath": [
            "${workspaceFolder}/**",
            "H:/Packages/OpenCV-MinGW-Build-OpenCV-4.1.1-x64/include",
            "H:/Packages/OpenCV-MinGW-Build-OpenCV-4.1.1-x64/include/opencv2",
            "${workspaceFolder}/include/**"
        ],
```

环境配置好，可以测试一下代码，如果

ps::使用msys2配置mingw的心酸历程
根据vscode和msys2的官方文档一步一步配置
[https://code.visualstudio.com/docs/languages/cpp](https://code.visualstudio.com/docs/languages/cpp)

下载msys2 [https://www.msys2.org/](https://www.msys2.org/) 是一个包管理器，看起来很方便
下载完成后在msys2命令行中输入
```shell
pacman -S --needed base-devel
pacman -S mingw-w64-x86_64-gcc mingw-w64-x86_64-gdb mingw-w64-x86_64-make
```

之后检查安装 g++ --version;  gdb --version 一切正常

msys居然也可以下载opencv包，但是我只找到了最新版opencv4.6(可能会在开发过程遇到不稳定问题)
```shell
pacman -S mingw-w64-x86_64-opencv
```
安装完可以看到dll都在mingw64\bin 里了，系统变量路径就不用再配了。要配的头文件在mingw64\include，库文件在mingw64\lib,简直不要太爽。

之后按照第二步找到对应的路径配置vscode的json即可
