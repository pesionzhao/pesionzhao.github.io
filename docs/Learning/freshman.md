NCU使用指南

`ncu --target-processes all  --set full --import-source yes -f -o stable_unzip -k tokens_unzip_stable_kernel python test_stable_unzip.py`

更新ncu

下载最新安装包（替换为实际版本号）

```
wget https://developer.nvidia.com/downloads/assets/tools/secure/nsight-compute/2025_2_0/nsight-compute-linux-2025.2.0.11-35613519.run
```

2. 赋予执行权限

```
chmod +x nsight-compute-linux-2025.2.0.11-35613519.run
```

3. 运行安装程序（可能需要卸载旧版本）

```
sudo ./nsight-compute-linux-2025.2.0.11-35613519.run
```

ctrl + z 挂起程序

jobs: 显示后台的程序

vscode快捷键

- `ctrl`/`command`+a: 行首
- `ctrl`/`command`+e: 行尾
- `ctrl`/`command`+g: 跳转行数
- `ctrl`+`-` / `option`+左右箭头: 光标跳转
- `ctrl`/`command`+`enter`: 创建新的一行
- `ctrl` + k: 删除光标以后的内容 
- `command`+``shift`+t: 恢复上一次删除的浏览器标签页
- `command`+w: 关闭当前标签页