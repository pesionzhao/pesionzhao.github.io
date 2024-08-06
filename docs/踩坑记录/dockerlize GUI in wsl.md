背景：在docker中使用图形化界面，例如opencv中的imshow或pyqt5开发

### 跑dockerhub的pyqt5
[发现WSL（Windows Subsystem for Linux）支持GUI（图形界面）的Linux应用了](https://blog.csdn.net/ddrfan/article/details/125651928)

windows端安装vcXsrv https://sourceforge.net/projects/vcxsrv/

然后打开Xlaunch对启动vsXsrv进行设置, additional parameter要添加`-ac`

之后打开wsl, pull一个docker镜像,例如

[fadawar/docker-pyqt5](https://hub.docker.com/r/fadawar/docker-pyqt5)

指令如下

```shell
docker pull fadawar/docker-pyqt5
export DISPLAY=10.180.73.62:0.0 #这里要换成自己windows端的ipv4地址,0.0表示显示编号和屏幕编号
docker run --rm -it -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=$DISPLAY -u qtuser  jozo/pyqt5 python3 /tmp/hello.py
```

参数解读

- `-v` 本地目录:容器目录  或 -v 容器目录. 将本机的当前目录，挂载到镜像中的目录　
- `-e` 向容器内部传递环境变量　
- `-u` 用户user

###  自己构建镜像

#### 方法一：由容器构建镜像

##### 在容器内中配置环境

以下是在容器中下载qt5并且构建为images的方法

首先进入一个基本容器

安装相关的包

```shell
apt-get update #更新包索引
apt install pyqt5* qttools5-dev-tools qt5-default python3-pyqt5 #下载qttool
pip3 install pyqt5
```

更改环境便变量

```bash
export DISPLAY=10.180.73.62:0.0
```

创建下面的python脚本为检查环境是否配置好

```python
from PyQt5.QtWidgets import QApplication, QMainWindow, QPushButton, QVBoxLayout, QWidget
import sys

class MainWindow(QMainWindow):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        self.setWindowTitle('win')
        self.setGeometry(100, 100, 1000, 500)

        self.button = QPushButton('click me!!', self)
        self.button.clicked.connect(self.on_button_clicked)

        self.layout = QVBoxLayout()
        self.layout.addWidget(self.button)

        self.widget = QWidget()
        self.widget.setLayout(self.layout)
        self.setCentralWidget(self.widget)

    def on_button_clicked(self):
        print("按钮被点击了！")
        self.button.setText("already click")

if __name__ == '__main__':
    app = QApplication(sys.argv)
    ex = MainWindow()
    ex.show()
    sys.exit(app.exec_())
```

运行该脚本会弹出vcXsrv的窗口界面,即证明环境安装完成

在终端输入

```bash
designer
```

同样会弹出qt designer的界面

##### 提交为镜像
wsl终端中输入

```bash
docker ps
```

可查看到当前运行的容器, 找到我们刚刚配置好环境的容器的ID,例如(CONTAINER ID为c3f279d17e0a)

```bash
docker commit c3f279d17e0a
```

此时查看镜像会发现多了一个名为`<none>`的镜像, 我们找到它的镜像id, 例如(`IMAGE ID`为0c17f0798823), 并给它标号为my_image

```bash
docker tag 0c17f0798823 my_image
```

之后我们继续查看images就会发现之前的`<none>`已经成了我们预设的my_image, **TAG默认为latest, REPOSITORY后面加:可以指定tag**

以上的两条命令可以通过一条命令实现

```bash
docker commit c3f279d17e0a  my_image:1.0
```

执行之后的tag为1.0而不是latest

大功告成,之前的容器可以停止并且删除了

```bash
docker stop c3f279d17e0a
docker rm c3f279d17e0a
```

由于没有规定cmd或者entrypoint, 所以在run时要添加`--entrypoint /bin/bash`

```bash
docker run --name my_app -it --rm --entrypoint /bin/bash -e DISPLAY=10.180.73.62:0.0 my_image
```

#### 方法二

将上述的过程编写为Dockerfile

```Dockerfile
FROM pytorch/pytorch:2.0.0-cuda11.7-cudnn8-devel

ENV DEBIAN_FRONTEND=noninteractive

# 还没有遇到这个问题
# # This fix: libGL error: No matching fbConfigs or visuals found
# ENV LIBGL_ALWAYS_INDIRECT=1

# Install Python 3, PyQt5
RUN apt-get update && apt install -y pyqt5* qttools5-dev-tools qt5-default python3-pyqt5 git vim&& pip3 install pyqt5

CMD /bin/bash
```

运行镜像
```bash
docker run --name my_app -it --rm --entrypoint /bin/bash -e DISPLAY=10.180.73.62:0.0 my_image
```

之后在vscode中调试可以通过写一个`.devcontainer.json`方便配置

### vscode中的.devcontainer编写

```json
// For format details, see https://aka.ms/devcontainer.json. For config options, see the
// README at: https://github.com/devcontainers/templates/tree/main/src/docker-existing-dockerfile
{
	"name": "app_container",
	"image": "my_image",
	"runArgs": ["--rm", "--gpus", "all", "-e", "DISPLAY=10.180.73.62:0.0", "--entrypoint","/bin/bash"]
}
```

### ssh连接远程docker

```bash
ssh-keygen -t rsa -C "PesionKey"
```

生成本地公钥和私钥，默认存在.ssh/中， -C指定注释，否则默认为本地主机的名字

之后可以指定是否设置passphares, 可以理解之后登录远程机子就仅需输入passphares, 可以为空

```bash
ssh-copy-id user@userip
```

将本地的公钥复制到远程user@userip的主机上， 之后会要求你输入远程机子密码

```bash
ssh user@userip
```

之后就可以通过刚刚设置的passphares登录主机了




