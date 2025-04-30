- [git使用](#git使用)
  - [基础](#基础)
  - [本地仓库推送](#本地仓库推送)
  - [远程重命名仓库](#远程重命名仓库)
  - [回退版本](#回退版本)
  - [代理](#代理)
- [C++](#c)
  - [Algorithms](#algorithms)
    - [ranges](#ranges)
  - [iterator](#iterator)
  - [lambda表达式](#lambda表达式)
  - [gcc](#gcc)
  - [initializer\_list](#initializer_list)
  - [string](#string)
  - [vector](#vector)
  - [Set](#set)
    - [multiset](#multiset)
    - [bitset](#bitset)
  - [map](#map)
  - [queue](#queue)
    - [priority\_queue](#priority_queue)
- [正则表达式](#正则表达式)
- [Linux](#linux)
  - [常用命令](#常用命令)
  - [脚本使用](#脚本使用)
  - [VIM使用](#vim使用)
  - [文本处理](#文本处理)
    - [grep](#grep)
    - [sed](#sed)
    - [awk](#awk)
  - [进阶指令](#进阶指令)
- [Docker](#docker)
  - [基础查看命令](#基础查看命令)
  - [常用指令](#常用指令)
  - [删除](#删除)
  - [多容器](#多容器)
  - [打包](#打包)
  - [运行容器 docker run](#运行容器-docker-run)
- [CMake](#cmake)

## git使用

### 基础

- `git config --list` 查看配置


### 本地仓库推送
获取token方法->develop settings

```bash
git init
git remote add origin http://[Token]@github/pesionzhao/ # 绑定将origin设置为远程仓库地址
git remote set-url origin URL_ADDRESS # 更改远程地址
git branch -M main # 重命名默认分支
git checkout -b master1 # 创建新分支并转到
git push -u origin main # -u 选项也可以写作 --set-upstream，关联本地与远程，之后不用指定分支
```

### 远程重命名仓库
```bash
# 远程重命名仓库
git branch -m main master
git fetch origin
git branch -u origin/master master
git remote set-head origin -a
```

### 回退版本
```bash
git reset # 撤销add操作
git log # 查看往期版本号和commit
git reset --hard 64d3f4bccb4d00 # 回退到某一版本号
git push -f origin master # 把回退完的上传
```

### 代理

[一文让你了解如何为 Git 设置代理](https://ericclose.github.io/git-proxy-config.html)

```bash
git config --global http.proxy http://127.0.0.1:10809
# unset
git config --global --unset http.proxy
```

## C++

### Algorithms

```c++
//返回最小值的第一个元素的迭代器
Iterator min_element(ForwardIterator first, ForwardIterator last);
// [first,last) 默认升序排列
void sort (RandomAccessIterator first, RandomAccessIterator last， comp); 
// 返回范围 [first,last) 中比较等于 val 的元素数量。
int count (first, last, val)
//计算两个迭代器之间的元素之和，第三个参数为init
std::accumulate(v.begin(), v.end(), 0, std::plus<>{});
//迭代器的下一个迭代器
next_it = next(it);
ForwardIterator std::unique(ForwardIterator first, ForwardIterator last);//Remove consecutive duplicates in range  区间中连续重复项去重，返回指向最后一个未移除元素后面的元素的迭代器。
//二分查找
ForwardIterator lower_bound (ForwardIterator first, ForwardIterator last, const T& val);//区间内第一个不小于val的位置，可能相等
ForwardIterator upper_bound (ForwardIterator first, ForwardIterator last, const T& val);//区间内第一个大于val的位置

std::vector<int>::iterator low,up;  // v =  10 10 10 20 20 20 30 30
low=std::lower_bound (v.begin(), v.end(), 20);//      ^
up= std::upper_bound (v.begin(), v.end(), 20);//               ^
```

- `void *memset(void *str, int c, size_t n)` str指向的空间中n个字节被设置为c
- `void iota (ForwardIterator first, ForwardIterator last, T val);` 将迭代器区间的元素assgin为val开始并不断自增

**trick**

```c++
//降序排列
ranges::sort(v, greater());
//自定义按照数组首元素降序
sort(items.begin(), items.end(), [&](const vector<int> &item1, const vector<int> &item2) -> bool { return item1[0] > item2[0];});
```

std::accumulate()和std::redce()的区别

[std::accumulate vs. std::reduce](https://blog.tartanllama.xyz/accumulate-vs-reduce/)

- std::reduce可以进行并行计算
- reduce要计算的需满足结合律和交换律
- reduce默认初始值是由默认构造生成的，要注意一下

#### ranges

```c++
sort(R&& r, Comp comp = {}, Proj proj = {});//按照comp排序({}默认升序),在proj方向进行排序
```

### iterator

```c++
distance (InputIterator first, InputIterator last);//两迭代器之间的元素个数
```

### lambda表达式

[C++ Lambda表达式的完整介绍](https://zhuanlan.zhihu.com/p/384314474)

使用std::function在函数内部构造dfs时经常使用！！

```c++
function<int(int, int, int)> dfs = [&](int t, int j, int k) -> int
{};
function<void(int)> dfs = [&](int i){};
```

lambda表达式语法

`[capture list] (params list) mutable exception-> return type { function body }`

`[capture list]`中为捕获的参数, 分为值捕获和引用捕获,例如

`[]`: 不捕获任何变量
`[&]`: 以引用方式捕获所有外部变量
`[=]`: 以值的方式捕获所有外部变量
`[=]`: 以值的方式捕获所有外部变量
`[=,&x]`: 以引用方式捕获x,其余都用值
`[&,x]`: 以值方式捕获x,其余都用引用

`(params list)`函数参数列表

`mutable`指示符：用来说用是否可以修改捕获的变量

`exception`：异常设定

### gcc

- `__builtin_popcount()` : 统计二进制数中1的个数
- `__builtin_ctz(x)`：x末尾0的个数。x=0时结果未定义。(count the trailing zeros)
- `__builtin_clz(x)`：x前导0的个数。x=0时结果未定义。(count the leading zeros)
- `__builtin_ffs(x)`：返回x中最后一个为1的位是从后向前的第几位

上述仅适用于unsigned int

- `__builtin_parity(x)`: x中1个数的奇偶性,返回1为奇,返回0为偶

### initializer_list

其中元素不可更改

### string

```c++
//获取子串
str.substr(start, length);
//查找子串
str.find(substr);
//查找字符
str.find('Q') != string::npos;
//访问string元素 可以通过charAt()或者[]
std::string str = "Hello";
char character = str.charAt(2);
char character = charArray[2];

//追加字符/字符串
str.push_back(char c);
str.append(char* s, size_t n)//各种重载
```
### vector

```c++
vector<vector<int>> vec(m, vector<int>(n));//m行n列的空数组
//重新构造
vec.assgin(5,10);//整个数组元素都赋为10
vec.resize(5,10);//新元素用10
vec.insert(iterator position, size_type n, const value_type& val);//在pos位置插入n个val
vec.insert(iterator position, InputIterator first, InputIterator last);//在pos位置插入[first,last)
```

### Set

```c++
set.insert(value);
//从集合容器中删除单个元素或[first,last)范围内的元素。
set.erase(pos);
void erase (iterator first, iterator last);

iterator upper_bound (ForwardIterator first, ForwardIterator last, const T& val, Compare comp);
;//val之后第一个元素
iterator lower_bound (val);//不在val前面的元素，要么相同，要么之后
```

#### multiset
可重复集合(按照升序排列的数组)

```c++
std::multiset<int> first；//空构造
int myints[]= {10,20,30,20,20};
std::multiset<int> second (myints,myints+5);//通过指针构造
std::multiset<int> third (second);//拷贝
std::multiset<int> fourth (second.begin(), second.end());//通过迭代器构造
std::multiset<int,classcomp> fifth;//通过指定排序方式构造
```

```c++ 
//默认构造是升序，也可指定降序构造
struct DescendingCompare {
    bool operator()(const int& lhs, const int& rhs) const {
        return lhs > rhs; // 定义降序排列
    }
};

multiset<int, DescendingCompare> temp;
//也可通过lambda表达式
multiset<int, std::function<bool(int, int)>> temp{
        [](int lhs, int rhs) {
            return lhs > rhs; // 定义降序排列
        }
    };
```

#### bitset

```c++
template< std::size_t N >
class bitset;
```

类模板bitset表示固定长度的N位序列。bitset可以通过标准逻辑运算符进行操作，并可以在字符串和整数之间进行转换。出于字符串表示和移位操作的命名方向的目的，序列被认为在其最低索引的元素在右边，就像整数的二进制表示一样。

```c++
#include <bitset>
#include <iostream>
std::bitset<4> b1;
std::bitset<4> b3{"0011"};
b3.set(size_t pos)//设定pos位置为ture
b3.reset(size_t pos)//pos位置为false
b3[2] = true; //由于低索引在最右边，所以b3 = "0111"
```

### map

```c++
//字典,哈斯表
unordered_map<string, int> myMap;
//插入键值对
myMap.insert({"Alice", 18});
myMap["Bob"] = 20; 
myMap["zhia"];//会默认构造myMap["zhia"]=0
//访问元素
int age1 = myMap["Alice"]; // 通过键来访问对应的值
int age2 = myMap.at("Bob")
//查找
myMap.count("Alice");
//遍历
for (auto it = myMap.begin(); it != myMap.end(); it++) {
    string key = it->first; // 访问键
    int value = it->second; // 访问值
}
for(auto &[k, v]:distance)
//删除
myMap.erase("Alice"); // 删除键为"Alice"的键值对
myMap.clear(); // 清空哈希表中所有的键值对
```

**trick**

```c++
//默认构造map会占用大量时间，可根据所需在插入键值对之前加入判断是否存在
if (!m_map.contains(temp)) {[temp]++;}
//或者
if(m_map.count(temp)==0)

//
```

### queue

```c++
q.emplace()
q.push()
q.front()
q.pop();
q.back();
```

#### priority_queue

```c++
priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;//小根堆
```

## 正则表达式

[55分钟学会正则表达式（转载自Github）](https://zhuanlan.zhihu.com/p/91689180)

[原文链接](https://github.com/EZLippi/practical-programming-books/blob/master/src/30-minutes-to-learn-regex.md)

- `.` 任意**一个**字符
- `[]` 匹配[]里面的任何一个 
- `[^]` 可以理解为非门
- `\d` 等价于[0-9] 匹配数字
- `\w` 等价于[0-9A-Za-z] 匹配数字或字母
- `\s` 匹配一个空字符
- `\b` 单词分隔符 `\w`和`\W`之间一定存在单词分隔符
- `\D` \d的非
- `\W` \w的非
- `\S` \s的非
- `{}` 表示重复 {4}: 重复四次 {2,4}: 重复2或4次 {1, }: 匹配一个或一个以上 **优先匹配最长的**
- `?` 等价于{0,1} 有或没有
- `*` 等价于{0, } 匹配任意
- `+` 等价于{1,} 匹配一个或一个以上, 即`\w+` 表示匹配一个单词
- `|` 表示或
- `^` 匹配行的开始位置 注意由于`[]`中的^代表非,所以应该这样写`[\^]`, 而`[$]`可以直接写
- `$` 匹配行的结束位置
- `^&` 空行

捕捉和替换

- `()` 用于捕获分组
- `\1 \2...` 反向引用,用于匹配之前第几次捕获的字符 例如`([abc])\1`表示匹配aa或bb或cc
- 
未完待续....

练习

```bash
echo "apple banana" | sed -E 's/(\w+) (\w+)/\2 \1/' # 交换两个单词的位置 s表示交换
```
## Linux

深入理解linux可以看[linux学习](./Linux.md)

[你必须知道的四十个linux指令](https://kinsta.com/blog/linux-commands/)

[菜鸟教程 Linux 命令大全](https://www.runoob.com/linux/linux-command-manual.html)

### 常用命令

```bash
alias gc="git clone" # 给指令起别名
alias # 显示设置的别名
unalias gc # 删除刚刚设置的别名
pwd # 显示当前路径
cd - # 返回刚刚的目录
cp file_to_copy.txt new_file.txt
cp -r dir_to_copy/ new_copy_dir/ # /r表示recursive 递归
rm -r empty_folder/
rm -rf folder/ # f表示force
mv file.txt folder/
mkdir -p movies/2004/ # -p 表示 parent 构建子目录 
man command # manuals of command
touch file
chmod # 更改文件模式(r: read, w:write, x:execute)
./ # 执行文件 但要chomd增加权限
sudo # superuser
htop # 管理后台资源
echo "Hey $USER" # 在终端打印东西
cat # concatenate
ps # process snapshot
kill Pid or name
history # 过去使用过的指令
passwd #　修改密码
which command #　command的绝对路径
shred # 完全损坏文件 -u 删除
less # 审查文件(cat 的升级版)
tail # 审查文件的后10行 可以通过-n指定几行
head # 前10行同理
grep #　匹配正则表达式　-c 计数 -E 正则匹配扩展
whatis command # 命令的简介
wc file # word count print (lines->words->byte-size->filename) -w is words 
uname -a # short for unix name
neofetch # 显示有关您的系统的信息
find [flags] [path] -name [expression]  # 正则匹配, -type f:文件 d:文件夹
```

- `du -sh` 查看所有子目录占的空间
- `df -h` 查看剩余空间
- `du -h --max-depth=1`等价于`du -h -d=1` 当前文件夹下的一级目录占用空间大小
- `chmod`: change mode, [修改文件权限](https://www.runoob.com/linux/linux-comm-chmod.html)

### 脚本使用

- `export ALL_PROXY=""` 设置环境变量
- ` unset ALL_PROXY` 取消设置的环境变量

### VIM使用

[how-do-i-exit-vim](https://stackoverflow.com/questions/11828270/how-do-i-exit-vim)

![](https://i.stack.imgur.com/BN6vl.png)

esc 进入普通模式

- `:q` 退出
- `:q!` 不保存退出
- `:wq` 写入保存退出
- `:wq!`  如果无权限, 强制写入保存
- `:x` 写入退出
- `:qa` 退出全部
- `dd` 删除一行

### 文本处理

#### grep
查看

#### sed
查看修改

#### awk
对本文执行复杂的操作

- `$1` 分割后的第`1`个字段
  
### 进阶指令

- `cammond | xargs` 将前一个指令输出作为下一个指令的输入

## Docker

[常用删除指令](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes#purging-all-unused-or-dangling-images-containers-volumes-and-networks)

[docker compose指令参考](https://docs.docker.com/compose/reference/)

Source是主机上的位置，Destination是容器内的位置

### 基础查看命令
- `docker container ls`: 查看运行容器的情况
- `docker inspect container`: 查看该容器的信息 Source是主机上的位置，Destination是容器内的位置
- `docker images -a` 查看所有镜像id
- `docker images -a | grep "mul" | awk '{print $3}'` 结合grep&awk进行查找 输出id
- `docker ps -a` 查看容器, `--size` 查看空间占用
- `docker ps -a -f status=exited -q` status有`created`, `restarting`, `running`, `paused`, or `exited` -f 表示 filter -q表示只显示ID
- `docker volume ls -f dangling=true` 显示dangling状态的volume

### 常用指令
- `docker pull busybox` pull一个docker
- `docker stop "containername"`
- `docker exec [OPTIONS] CONTAINER COMMAND` 命令用于在运行中的容器内执行一个新的命令。这对于调试、运行附加的进程或在容器内部进行管理操作非常有用。 例如`docker exec -it my_container /bin/bash`
- `docker system df` 查看占用空间 `-v`详细信息

### 删除
- `docker system prune` 删除所有悬挂的镜像(未标记或与容器无关联),容器和卷轴
- `docker system prune -a` 删除所有没有使用的镜像,容器和卷轴
- `docker rmi` 后指定image的id或标签可完成删除
- `docker images -f dangling=true` 显示dangling的image
- `docker image prune` 删除dangling镜像
- `docker rm` 指定id或名字删除容器
- `docker run --rm image_name` 离开时删除容器
- `docker rm $(docker ps -a -q -f status=exited)` 等价于 `docker container prune` $()表示指令替换,这一段先执行
- `docker volume rm` 指定id或名字删除卷
- `docker volume prune` 删除dangling状态的volume
- 
### 多容器

- `docker compose up -d`: 由compose.yaml构建多个镜像之间的链接
- `docker compose watch`: 用于更改代码,实时监视

### 打包
- `docker init`

### 运行容器 docker run
- `-d`: detach 后台运行容器
- `-it`: interactive 交互
- `--name`: 为容器指定名字
- `-p`: publish 容器端口映射到主机端口 `host_port:container_port`
- `-v`: volume 挂载卷 `host_path:container_path`
- `--rm`: 离开容器自动删除
- `-e`: 设置容器内环境变量
- `--network`: 指定容器连接的网络 host 
- `-u`: 指定用户
- `--entrypoint`: 容器入口点
- `-w`: 设置容器工作目录


## CMake

常用的命令有：

- 如何设置项目源码目录
- 如何设置项目 include 目录
- 如何设置依赖的库目录
- 如何指定可执行程序的生成方式（包括可直接运行的和依赖库（.so 和 .a 文件））
- 指定查找依赖的方式
- 条件编译
- 自定义变量以及如何执行 cmake 命令时给这些变量传递值。

常用的功能基本上就这么多，其他的命令可以在用到的时候去查一下即可。

```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)

# 项目信息
project (Demo3)

# 查找当前目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_SRCS)

# 指定生成目标
add_executable(Demo ${DIR_SRCS})

# 添加 math 子目录
add_subdirectory(math)

# 指定生成目标 
add_executable(Demo main.cc)

# 添加链接库
target_link_libraries(Demo MathFunctions)
```

静态库生成

```cmake
# 生成静态库
add_library(static_lib test1.cpp)
```

调用静态库

```cmake
link_directories(${CMAKE_SOURCE_DIR}/lib)
target_link_libraries(Demo static_lib)
```

动态库生成

```cmake
add_library(static_lib SHARED test1.cpp)
```

dll放在可执行文件目录下


modern cmake

头文件的包含

为减轻依赖，废弃`incude_directories`, 改用`target_include_directories`

```cmake
# 仅对当前目标可见，不会传播到依赖该目标的其他目标。
target_include_directories(target PRIVATE include_dir)
# 对当前目标以及所有依赖它的目标可见。
target_include_directories(target PUBLIC include_dir)
# 对当前目标不可见，依赖该目标的其他目标可见，（如纯头文件库
target_include_directories(target INTERFACE include_di)
```

`find_package`: 寻找`.cmake`文件, 可以在这个文件中找到各种依赖库的路径
