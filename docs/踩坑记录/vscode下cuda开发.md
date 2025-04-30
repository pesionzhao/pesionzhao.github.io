## 配置开发环境

docker的话一定要下载devel版本的镜像 否则没有nvcc

本地安装：

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
 export LD_LIBRARY_PATH=/usr/local/cuda-11.1/lib64:$LD_LIBRARY_PATH
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

此时cuda环境已经安装好

## vscode配置

下载插件Nsight Visual Studio Code Edition用于调试

[vscode使用nsight调试cuda代码](https://blog.csdn.net/dzxxbj/article/details/135684839)

task.json： 用于编译, 

```json
{
	"version": "2.0.0",
	"tasks": [
		{
			"label": "build CUDA program",
			"type": "shell",
			"command": "nvcc",
			"args": [
				"-g",
				"-G",
				"-o",
				"${workspaceFolder}/${fileBasenameNoExtension}",
				"${file}",
				"-I${workspaceFolder}"
			],
			"group": {
				"kind": "build",
				"isDefault": true
			},
			"problemMatcher": []
		},
		{
			"type": "cppbuild",
			"label": "C/C++: g++ 生成活动文件",
			"command": "/usr/bin/g++",
			"args": [
				"-fdiagnostics-color=always",
				"-g",
				"${file}",
				"-o",
				"${fileDirname}/${fileBasenameNoExtension}"
			],
			"options": {
				"cwd": "${fileDirname}"
			},
			"problemMatcher": [
				"$gcc"
			],
			"group": "build",
			"detail": "编译器: /usr/bin/g++"
		}
	]
}
```

launch.json： 用于调试

c_cpp_properties.json： 用于自动补全、代码提示、跳转等

## CMake构建

[使用Vscode调试cuda代码](https://fancyerii.github.io/2024/01/17/vscode-cuda-debug/#%E4%BF%AE%E6%94%B9cmakeliststxt)

[Libtorch系列教程1：一个丝滑的C++ Tensor库](https://cloud.tencent.com/developer/article/2349772)

```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 3.15)

# 项目信息
project (Demo3 LANGUAGES CUDA CXX)
# set(CMAKE_BUILD_TYPE Debug)  # 启用调试模式

# 添加 NVCC 编译选项，启用调试模式
set(CMAKE_CUDA_FLAGS_DEBUG "-G -g")
```
