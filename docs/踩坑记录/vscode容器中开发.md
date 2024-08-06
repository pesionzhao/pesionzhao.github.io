## 使用容器开发

### 准备工作

安装WSL->docker-desktop

安装vscode插件

- docker
- dev container

### 进入容器

[VS Code with dockerized build environments for C/C++ projects](https://ddanilov.me/dockerized-cpp-build-with-vscode)

[官方创建devcontainer.json的教程](https://code.visualstudio.com/docs/devcontainers/create-dev-container)

[DockerFlie编写规范](https://docs.docker.com/engine/reference/builder/)

[设置文件挂载宿主机](https://code.visualstudio.com/remote/advancedcontainers/add-local-file-mount)

#### 由镜像生成devcontainer.json

- 进行开发的目录中`ctrl+shift+p`或`F1`打开`open container configuration Flie`
- 选择对应的镜像,之后还可以选择添加的其他配置
- 选完后会在文件夹中看到`devcontainer.json`和`DockerFile`
- 继续`ctrl+shift+p`搜索`Reopen in Container`加载该容器, 即可在当前目录下在容器中进行调试,路径为/Worksapce

#### 由DockFlie构建devcontainer
- 在根目录写好DockerFlie
- `ctrl+shift+p`打开`open container configuration Flie`
- 选择`来自DockerFlie`
- 选择需要附加的软件,可以不选跳过
- 会出现`.devcontainer/devcontainer.json`
- `Reopen in Container`加载该容器可进入容器

之后就可以进入容器进行开发了~

#### 修改/自己编写devcontainer.json

由于在实际的应用中官方

devcontainer.json中各字段的命名规则及其作用 👇

[Dev Container metadata reference](https://containers.dev/implementors/json_reference/)

举个简单的例子,如果我想基于已有镜像构造容器(build和image 字段只能同时存在一个), 使用GPU, 离开容器时自动删除可以这样编写json文件 👇

```json
{
	"name": "CUDA117_env", // UI中的容器名
	"image": "pesioncuda", // 本地镜像的名字
    // 由于不需要build,故不用以下字段
	// "build": {
	// 	// Sets the run context to one level up instead of the .devcontainer folder.
	// 	"context": "..",
	// 	// Update the 'dockerFile' property if you aren't using the standard 'Dockerfile' filename.
	// 	"dockerfile": "../.DockerFile"
	// },
	"runArgs": ["--rm", "--gpus", "all"], // 在进行docker run时需要传的参数, 必须使用--gpus才能正确调用gpu
	"postCreateCommand": "nvidia-smi" // 创建好容器后进行的command, 用于检查是否能正确读取GPU
}
```

`docker run`的参数可参考官方网址[docker run](https://docs.docker.com/engine/reference/commandline/run/)

#### 进入正在运行的容器

还有一种方法可以不通过devcontainer, 先在终端执行docker run 将所需的容器跑起来

之后再vscode->docker->左侧栏可以找到containers，右键已经开启的容器->附加到vscode即可开启一个连接到此容器的vscode窗口

### DockerFlie编写

[使用 Dockerfile 定制镜像](https://yeasy.gitbook.io/docker_practice/image/build)

- `FROM` 指定基础镜像, 可以是本地已经构建好的
- `RUN` 执行命令
- `COPY` 复制文件: 将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置
- `CMD` 容器启动命令, 用于指定默认的容器主进程的启动命令, 比如，ubuntu 镜像默认的 CMD 是 `/bin/bash`


## **Sample**

目标环境pytorch:2.0.0-cuda11.7-cudnn8-devel

`.DockerFlie文件`

```Docker
FROM pytorch/pytorch:2.0.0-cuda11.7-cudnn8-devel

RUN apt-get update\
&& apt-get -y upgrade\
&& apt-get install -y build-essential gdb cmake
```

`.devcontainer.json`

```json
// For format details, see https://aka.ms/devcontainer.json. For config options, see the
// README at: https://github.com/devcontainers/templates/tree/main/src/docker-existing-dockerfile
{
	"name": "CUDA117_env",
	"image": "pesioncuda",
	// "build": {
	// 	// Sets the run context to one level up instead of the .devcontainer folder.
	// 	"context": "..",
	// 	// Update the 'dockerFile' property if you aren't using the standard 'Dockerfile' filename.
	// 	"dockerfile": "../.DockerFile"
	// },
	"runArgs": ["--rm", "--gpus", "all"], //all表示使用全部GPU, 这里可以指定编号，使用指定的GPU
	// Features to add to the dev container. More info: https://containers.dev/features.
	// "features": {},

	// Use 'forwardPorts' to make a list of ports inside the container available locally.
	// "forwardPorts": [],

	// Uncomment the next line to run commands after the container is created.
	"postCreateCommand": "nvidia-smi",
	"customizations": {
		"vscode": {
			"extensions": [ // vscode插件
				"yzhang.markdown-all-in-one"
			]
		}
	}
	// Configure tool-specific properties.
	// "customizations": {},
	// Uncomment to connect as an existing user other than the container default. More info: https://aka.ms/dev-containers-non-root.
	// "remoteUser": "devcontainer"
}
```