## 杂记代码

### 字符串


#### 子序列判断

>字符串删除一定元素得到的串为子序列,字符串匹配就是给定两个字符串s,t,判断s是否为t的子序列

```c++
bool isSub(string& s, string& t) {//s是不是t的子序列，s.size()必定小于t.size()
    int i = 0;
    for(char c:t)
    {
        if(s[i]==c)//此时一一匹配了,两个指针都++
        {
            i++;
            if(i==s.size())
            {
                return true;
            }
        }
        // if(s[i]==c&&++i==s.size()) return true; //是上面代码的简洁版
        //此时没有一一匹配,只有t指针++
    }
    return false;
}
```

#### 字符串分割

```c++
stringstream ss(s);
string w;
while (ss >> w)//继承了istream,空格为分隔符
{
    std::cout<<w<<endl;
}
```

分组循环

```c++
for(int i = 0; i<n; i++>)
    //初始化....
    while (i < n && ...): //分组循环,寻炸符合条件的连续元素
        i++;
    //一个分组结束,进行其他操作
```

#### 字符串转浮点数并保留位数

`std::stringstream` 是C++标准库中的一个类，可以用来进行字符串流操作。它适用于更复杂的转换需求，尤其是需要格式化输出时

```c++
long long price = std::stoll(sentence);//字符串转longlong
int price = std::stoi(sentence);//字符串转longlong
double price = std::stod(sentence);//字符串转double
float price = std::stof(sentence);//字符串转float

std::stringstream ss;
ss << std::fixed << std::setprecision(2) << price;//保留两位数字
string res = ss.str();
```


### 细节

```c++
int res = 0, i = 0;
vector<string>& strs;
//在比较长度时必须要转成int!!!!!!!!!
if((int)strs[i].length()<=res);
max(res, static_cast<int>(strs[i].size()));
```

### 报错原因

- `==`运算优先级大于`&`, 所以要`(s&1)==0` 


### 算法思路与技巧

- [小M的光明之魂速通挑战](https://www.marscode.cn/practice/lnx2n6ojwjpod3?problem_id=7414004855077093420)： 构建哈希表用于回溯，构建vis数组判断是否用过该武器
- [79翻转增益的最大子数组和](https://www.marscode.cn/practice/lnx2n6ojwjpod3?problem_id=7414004855075979308): 将翻转问题转化为删除问题
- [LC102. 二叉树的层序遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal/description/): 带有深度的层序遍历，先序遍历实现层序遍历的结果
- [55. 跳跃游戏](https://leetcode.cn/problems/jump-game/description/), 贪心维护最远距离或者反向模拟,  [45. 跳跃游戏 II](https://leetcode.cn/problems/jump-game-ii/description/)  贪心维护当前最大值和之前经过的点能到的最大值

有点绕

- [688. 骑士在棋盘上的概率](https://leetcode.cn/problems/knight-probability-in-chessboard/description/): dp数组要存概率， 要注意什么问题可以记忆化， 马可以通过不同的策略跳相同的步数到相同的位置，所以可以记忆化

常识

- 质数筛
- 离线查找 [1847. 最近的房间](https://leetcode.cn/problems/closest-room/description/), 小面积是大面积的子集，所以要先算小面积，离线查找：维护一个数组存查找顺序
- 矩阵快速幂
- lcp，z函数 [3292. 形成目标字符串需要的最少字符串数 II](https://leetcode.cn/problems/minimum-number-of-valid-strings-to-form-target-ii/description/)
- 

#### 最小堆

#### 单调栈

#### 