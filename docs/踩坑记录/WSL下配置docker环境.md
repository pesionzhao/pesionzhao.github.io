[WSL2安装（详细过程）](https://blog.csdn.net/qq_43636384/article/details/128453416)

[史上最全的WSL安装教程](https://blog.csdn.net/wojiuguowei/article/details/122100090)

[docker | 基于 WSL2 在 Windows 下使用 docker](https://zhuanlan.zhihu.com/p/475022079)

[win10启用Wsl2，安装Docker desktop集成Ubuntu，配置docker开发环境](https://config.net.cn/server/microservice/fa3d6ff7-039e-451f-bb66-4a9b92952d7b-p1.html)

[深度学习——docker安装nvidia-container-toolkit的步骤与执行过程中的一些报错](https://blog.csdn.net/weixin_43948284/article/details/131559937)

[win10/11下wsl2安装gpu版的pytorch（避坑指南）](https://zhuanlan.zhihu.com/p/488731878)

### 安装WSL

**可以优先看**[**官方文档!!!**](https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/wsl-containers)

为Linux启用子系统

控制面板->程序->程序与功能->启用或关闭windows功能->适用于Linux的windows子系统 **打开**

打开powershell输入

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

启用虚拟化:以管理员打开powershell输入下列命令

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

下载X64的[WSL2 Linux内核升级包](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)并安装

https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi

设置WSL默认版本

```powershell
wsl --set-default-version 2
```

打开Microsoft Store安装ubuntu 20.04.6 LTS

安装完后打开ubuntu,设置用户名密码即可ok

cmd中输入

```
wsl -l -v
```

即可看到是否安装成功

### 安装docker
[docker | 基于 WSL2 在 Windows 下使用 docker](https://zhuanlan.zhihu.com/p/475022079)

windows安装docker-desktop,按照提示安装

https://www.docker.com/products/docker-desktop

cmd

```
docker --version
```

设置Ubuntu作为默认发行版

打开docker-desktop->设置->Resource->WSL intergration->打开选项ubuntu-20.04

在子系统中输入

```bash
docker run hello-world
```

换源参考链接

### 安装nvidia驱动

**一定要注意cuda版本!!!! [我下载的是11.1.1](https://developer.nvidia.com/cuda-11.1.1-download-archive?target_os=Linux&target_arch=x86_64&target_distro=WSLUbuntu&target_version=20&target_type=runfilelocal)**

官网下载runfile: https://developer.nvidia.com/cuda-downloads

往期版本: https://developer.nvidia.com/cuda-toolkit-archive

选择Linux->x86_64->WSL_Ubuntu->2.0->runfile

将出现的两条指令复制到终端运行,全部默认选项

之后添加环境变量,看好自己的文件夹在哪!!

```shell
vim ~/.bashrc
```

输入i进行编辑,在文件最后加入

```shell
 export PATH=/usr/local/cuda/bin:$PATH
 export LD_LIBRARY_PATH=/usr/local/cuda-11.1/lib64:$LD_LIBRARY_PAT
 ```

 esc输入:wq进行保存退出

 刷新环境变量

 ```shell
 source ~/.bashrc
 ```

 之后在终端输入nvcc -V即可看到cuda版本

 可以用官方sample测试是否可以正常使用

 ```shell
cd /usr/local/cuda/samples/4_Finance/BlackScholes
make BlackScholes
./BlackScholes
```

### 安装nvidia-container-toolkit
首先，1. 确保nvcc -V可以正常输出，2. 确保docker安装成功

在服务器bash中设置stable存储库和密钥：
```shell
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
```

添加NVIDIA Container Toolkit的软件源，并安装nvidia-docker2软件包：

```shell
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
```

重启docker

```bash
sudo systemctl restart docker
```

配置Docker daemon使其可以识别到NVIDIA Container Runtime

```bash
sudo nvidia-ctk runtime configure --runtime=docker
```

输出
```shell
INFO[0000] Loading docker config from /etc/docker/daemon.json
INFO[0000] Successfully loaded config
INFO[0000] Wrote updated config to /etc/docker/daemon.json
INFO[0000] It is recommended that the docker daemon be restarted.
```

重启docker, 安装完成

拉取pytorch官方镜像

```shell
docker pull pytorch/pytorch:2.0.0-cuda11.7-cudnn8-devel
```

进入镜像

```shell
docker run --name cuda117 --gpus all --rm -it pytorch/pytorch:2.0.0-cuda11.7-cudnn8-devel
```

——GPU用于指定容器应该看到哪个GPU, all的意思是“所有的”。如果你只想用一个，你可以传递它的id——gpu 1。你也可以指定一个要使用的gpu列表，——gpu "device=1,2"

[Develop like a Pro with NVIDIA + Docker + VS Code + PyTorch](https://blog.roboflow.com/nvidia-docker-vscode-pytorch/)

由于镜像都过大,下载速度很慢,可以通过更换国内源

[Windows Docker 配置国内镜像源的两种方法](http://www.dfcsoft.com/View.aspx?id=057a2efcdb0d428d80a49e5c00b6f924)

1. 通过Docker-Desktop->设置->Docker Engine 添加
2. 修改daemon.json配置(这个由于我用的是子系统,所以没有使用这个选项)

添加以下地址：

```vim
"registry-mirrors": [
"https://docker.mirrors.ustc.edu.cn",
"https://registry.docker-cn.com",
"http://hub-mirror.c.163.com",
"https://mirror.ccs.tencentyun.com"
]
```

之后重启Docker

输入

```bash
docker info
```

如果在最后显示了Registry Mirrors:,即为配置成功~

docker的具体使用方法,见其他文档~

ps:

WSL设置代理

```bash
host_ip=$(cat /etc/resolv.conf |grep "nameserver" |cut -f 2 -d " ")
export ALL_PROXY="http://$host_ip:7890"
```

[WSL 下使用代理](https://zhuanlan.zhihu.com/p/414627975)

- 1.windows v2ray客户端开启允许来自局域网的连接。
- wsl中关闭自动更新dns nameserver /etc/wsl.conf 文件中设置为false。

```bash
[network]
generateResolvConf = false
```

**这里如果设成false会导致再一次开机时没有此文件，Temporary failure in name resolution导致域名解析失败**

- 然后/etc/resolv.conf中nameserver替换成8.8.8.8或者其他可用的dns服务器。

在~/.bashrc中添加如下内容

```bash
# add for proxy
export hostip=$(ip route | grep default | awk '{print $3}')
export hostport=10810
alias proxy='
    export HTTPS_PROXY="socks5://${hostip}:${hostport}";
    export HTTP_PROXY="socks5://${hostip}:${hostport}";
    export ALL_PROXY="socks5://${hostip}:${hostport}";
    echo -e "Acquire::http::Proxy \"http://${hostip}:${hostport}\";" | sudo tee -a /etc/apt/apt.conf.d/proxy.conf > /dev/null;
    echo -e "Acquire::https::Proxy \"http://${hostip}:${hostport}\";" | sudo tee -a /etc/apt/apt.conf.d/proxy.conf > /dev/null;
'
alias unproxy='
    unset HTTPS_PROXY;
    unset HTTP_PROXY;
    unset ALL_PROXY;
    sudo sed -i -e '/Acquire::http::Proxy/d' /etc/apt/apt.conf.d/proxy.conf;
    sudo sed -i -e '/Acquire::https::Proxy/d' /etc/apt/apt.conf.d/proxy.conf;
'
```

```
source .bashrc
```

关闭防火墙

crtl+shift+c打开终端
