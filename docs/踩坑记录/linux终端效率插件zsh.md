## 提高终端操作效率的插件-->> zsh

安装流程 [玩转WSL(3)之安装配置 oh my zsh](https://blog.mphy.top/posts/2020-03-03-WSL-3.html)

```shell
sudo apt install zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

查看配置文件

```shell
vim ~/.zshrc
```

这里可以修改主题`ZSH_THEME`,为指定主题, 我选择了随机主题 `ZSH_THEME=random`

安装必要插件

#### 高亮插件

```shell
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

#### 自动补全插件

```shell
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

#### 循环查找历史命令

```shell
git clone https://github.com/zsh-users/zsh-history-substring-search ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-history-substring-search
```

下载完插件后,需要在配置文件中指明

```shell
vim ~/.zshrc
```

```.zshrc
plugins=(git 
zsh-syntax-highlighting 
zsh-autosuggestions 
zsh-history-substring-search)
```