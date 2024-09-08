## 算法
### 回溯法

[回溯算法套路③排列型回溯+N皇后【基础算法精讲 16】](https://www.bilibili.com/video/BV1mY411D7f6/?vd_source=c43347ef375755d298da8f0c05cfe444)

回溯法本质为深度优先遍历，首先选择一条路走到底，递归结束后恢复现场，接着递归

由两个题来对回溯法进行解读, 全排列与N皇后

#### 全排列

**给定一个不含重复数字的数组 `nums` ，返回其 所有可能的全排列 。可以按任意顺序返回答案。**

```
示例
输入：nums = [1,2,3]
输出：[[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

直接拿灵神的代码进行讲解,不理解没关系,注释版在后面,先大概了解一下流程

```c++
class Solution {
public:
    vector<vector<int>> permute(vector<int> &nums) {
        int n = nums.size();
        vector<vector<int>> ans;
        vector<int> path(n), on_path(n); //path为可能的结果, onpath作为记忆化数组,用来在递归时判断这个数字是否被使用
        function<void(int)> dfs = [&](int i) {
            if (i == n) {
                ans.emplace_back(path);
                return;
            }
            for (int j = 0; j < n; ++j) {
                if (!on_path[j]) {
                    path[i] = nums[j];
                    on_path[j] = true;
                    dfs(i + 1);
                    on_path[j] = false; // 恢复现场
                }
            }
        };
        dfs(0);
        return ans;
    }
};
```

其中dfs即为深度优先搜索的实现,基本都使用递归方法,

主体思路为dfs从索引为0开始构造可能的答案,给这个索引赋值答案,然后进行递归给下一个索引赋值,递归结束条件为索引==n, 直接返回答案

怎么给这个索引对应的地方赋值就是算法实现的关键 👇

每次从nums中选择一个数给一个索引赋值都会使剩下可选的数字变少,可以由这个点出发进行构建,这也就是onpath存在的意义,它的True/False代表了这个位置的数字有没有被选,用于保证每个数字只能被选一次,接下来就是for循环,遍历nums中的数字,如果其没有被选,就选择它(将onpath赋值为true),然后进行下一个索引的赋值(也就是下一次递归), **这样子能得到最终的一个正确的结果**, **恢复现场的重要性就是能让其得到所有正确的结果**,在这个索引被赋值之后得到result之后,要将这个索引重新赋其他值,再进行之前的所有操作~这样子就能得到所有全排列的结果啦 🙆

```c++
dfs = (int i) 
{
    if (i == n) { //索引和nums的长度一样,说明已经找到了答案,(此时的onpath必定全为True,因为这时已经选择了所有的数)
        ans.emplace_back(path); // 将答案加到结果数组中
        return;
    }
    for (int j = 0; j < n; ++j) {//遍历所有值(j),在这个循环中只对答案第i的索引进行更改
        if (!on_path[j]) //判断这个值能不能被选,如果能被选的话 ⬇️
        {
            path[i] = nums[j]; //给答案的这个索引(i)赋值
            on_path[j] = true; // 将这个值改为已被选状态,在以后的判断中不能选这个值
            dfs(i + 1); // 对答案的下一个索引(i+1)进行赋值
            on_path[j] = false; // 下一次循环会执行++j, 也就是要重新对答案的(i)索引进行赋值, 在之后的索引依旧可以选择当前j, 所以将这个值改为未被被选状态,之后可以选他
        }
    }
}
```

当然这个dfs实现可以是很多种,以下是我实现的两种,思路和上面的一样,都是要保证每个数字只能被选一次

上面的方法构建了onpath用于记录数字是否被选,我们能不能从原始的nums下手,每选一次就让nums中的这个数字消失,使之后的选择只能在剩下的数字中选,这样符合人的直觉,如果用删除插入的又太耗时,所以这个思路是每选一次就将nums中被选的位置的数改为一个特定的数字(nums不可能出现的数字),下次遍历到nums的值为这个特定数时就跳过(起到了onpath的作用, 说白了就是省去了onpath占用的空间)

```c++
void dfs(vector<vector<int>>& res, int index, vector<int>& nums, vector<int>& cur)
    {
        if(index==nums.size()) // 递归终止
        {
            res.push_back(cur);
            return;
        }
        for(int i = 0; i<nums.size();i++)
        {
            if(nums[i]==-20) // 将nums代替了onpath
                continue;
            else
            {
                cur[index]=nums[i]; // 给答案的这个索引(index)赋值
                nums[i]=-20; //每次被选就让nums中这个数字消失,改为-20(这个数字可以定义为任何nums不可能出现的数字)
                dfs(res, index+1, nums, cur); // 对下一个索引进行赋值
                nums[i]=cur[index]; // 恢复现场
            }
        }
        return;
```

这个onpath决定了数字是否被选,只有两种状态, 即True/False, 而且最多只有[-10-10] 21个数字, 所以我们可以用n个二进制数来代表! 我这里定义了pos为n(nums.size())位二进制数,同样起到onpath的作用, 这种思路很重要,也很实用,各位一定要好好理解!

- 判断元素d是否在集合中：x >> d & 1 
- 把元素d添加到集合中： x | (1 << d)

```c++
vector<vector<int>> permute(vector<int>& nums) {
    int pos = 1<<nums.size();//初始化pos, 1左移n位, 后n位二进制数全为0, 0表示没有被选
    vector<vector<int>> res;
    vector<int> cur(nums.size());
    dfs(res, 0, nums, cur, pos);
    return res;

}
void dfs(vector<vector<int>>& res, int index, vector<int>& nums, vector<int>& cur, int pos)
{
    if(index==nums.size())
    {
        res.push_back(cur);
        return;
    }
    for(int i = 0; i<nums.size();i++)
    {
        if(pos>>i&1)//判断nums的第i位是否被选
            continue;
        else
        {
            cur[index]=nums[i]; // 答案第i位索引赋值
            dfs(res, index+1, nums, cur, pos|1<<i); // 进行递归
        }
    }
    return;

}
```

此时你可能会问, 这儿为什么没有恢复现场呀? 其实是进行了恢复现场的, 仔细看`dfs(res, index+1, nums, cur, pos|1<<i);`这句,它可以写成

```c++
pos = pos|1<<i; //将pos的第i位置1, 表示nums[i]被选了
dfs(res, index+1, nums, cur, pos);//dfs(index+1)进行下一次的递归
pos = pos&~(1<<i); //将pos的第i位置0, 恢复现场
```

这两者是不是一样呀? 是不是恍然大悟了 💡, 所有的dfs都必须恢复现场!只不过有两种写法

#### N皇后
趁热打铁来一到经典的N皇后的题目!

简单来说就是在N*N的棋盘上放置N个皇后,使其相互不能攻击到,也就是这N个皇后不同行不同列不同斜

首先可以确定的是每行每列肯定都有且只有一个皇后, 至于证明方法的话可以看文章开头参考链接中灵神的讲解

可以用for循环对每行皇后进行摆放,这样这些皇后肯定不同行,可以通过dfs得到所有的结果然后判断这些皇后不同列不同斜, 也是最简单最符合直觉的一种写法

```c++
vector<string> queen(n, string(n, '.'));//初始化全为'.',也就是全都不放皇后
void dfs(int row, vector<vector<string>>& res, vector<string>& queen, int n) 
{
    if (row == n) {
        res.push_back(queen);
        return;
    }
    for (int col = 0; col < n; col++) {//遍历列
        if (!isvaild(queen, col, row, n)) //如果当前位置不合法(同行同列或同斜),就跳过
            continue;
        queen[row][col] = 'Q'; //给当前位置摆一个皇后
        dfs(row + 1, res, queen, n); //给下一行摆皇后
        queen[row][col] = '.'; //恢复现场
    }
    return;
}
```

判断合不合法就是看当前列, 当前行, 当前左上斜线, 当前右上斜线有无皇后, 有的话说明不合法,不能摆皇后

```c++
bool isvaild(vector<string>& queen, int col, int row, int n) 
{
    for (int i = 0; i < row; i++) { // 判断当前列没有Q
        if (queen[i][col] == 'Q')
            return false;
    }
    //左上斜线
    for (int i = row - 1, j = col; i >= 0; i--) {
        if ((--j) < 0) //row和col同时-- 表示当前点的左上斜线
            break;
        if (queen[i][j] == 'Q')
            return false;
    }
    //右上斜线
    for (int i = row - 1, j = col; i >= 0; i--) {
        if ((++j) >= n) //row--同时col++, 表示当前点的右上斜线
            break;
        if (queen[i][j] == 'Q')
            return false;
    }
    return true;
}
```

可以看到isvaild占用了很多的时间, 判断不同列的方法完全可以参照全排列的选择不同数!也就是说可以构建onpath或者n二进制数来记忆该列是否被选, 左上右上斜线的话,以左上角点为原点构建坐标系,右上斜线满足x+y相等， 左上斜线满足y-x相等

```c++
class Solution {
public: //回溯法灵神方法一深度优先搜索
    vector<vector<string>> solveNQueens(int n) {
        int queen_col = (1<<n); //使用二进制代表集合，
        // 因为每行每列不重复，每行不重复：可以通过for循环每次循环增加一个皇后表示
        // 每列不重复：可构造一个n个二进制数， 代表了每列是否摆了皇后，全零代表没有被选，第i个数为1代表第i列已经摆了皇后
        // 逻辑为从上到下每行摆一个皇后，
        vector<int> pos(n);
        vector<vector<string>> res;
        dfs(0, res, queen_col, n, pos);
        return res;
    }
    void dfs(int row, vector<vector<string>>& res, int queen_col, int n, vector<int>& pos) {
        if (row == n) {
            vector<string> queen(n);
            for(int i = 0; i<pos.size();i++)
            {
                queen[i] = string(pos[i],'.')+string(1,'Q')+string(n-1-pos[i], '.');
            }
            res.emplace_back(queen);
            return;
        }
        for (int col = 0; col < n; col++) {
            if (!isvaild(col,row,pos)||((queen_col>>col)&1))//第col位为1代表被选了，跳过
                continue;
            pos[row]=col;
            dfs(row + 1, res, (1<<col)|queen_col, n, pos);//将对应位的queen_col置1
            //恢复现场
        }
        return;
    }
    bool isvaild(int col, int row, vector<int>& pos) {//左上满足x+y相等， 右上满足y-x相等, 在外面限制不同列
        for(int i = 0; i<row;i++)//这我之前写的是i<pos.size(), 很明显在没赋值时以下判断条件恒为0+0/0-0所以一直返回false
        {
            if((i+pos[i]==row+col)||(i-pos[i]==row-col))
                return false;
        }
        return true;
    }
};
```

上面的方法是构建了一个数组用来判断列是否被选, 同样可以构建数组来判断斜线是否被选, 结合之前的方法onpath的定义好好理解

```c++
class Solution {
public: // 回溯法 依旧是灵神！
//结合前一种解法可得， 内存换时间
    vector<vector<string>> solveNQueens(int n) {
        vector<vector<string>> res;
        vector<int> col_pos(n), diag1(n * 2 - 1), diag2(n * 2 - 1);
        int on_path = 1<<n; //数组改为集合，每一位0/1代表该列选没选
        function<void(int)> dfs = [&](int r) 
        {
            if(r==n)
            {
                vector<string> queen(n);
                for(int i=0; i<n;i++)
                {
                    queen[i]=string(col_pos[i],'.')+'Q'+string(n-1-col_pos[i],'.');
                }
                res.push_back(queen);
                return;
            }
            for(int i = 0; i<n;i++)
            {
                if(!(on_path>>i&1)&&!diag1[i+r]&&!diag2[i-r+n-1])
                {
                    col_pos[r]=i;
                    on_path = on_path|1<<i; //将该位置1
                    diag1[i+r]=diag2[i-r+n-1]=true;//这里要+n-1以防负数的出现
                    dfs(r+1);
                    on_path = on_path&~(1<<i); // 将该位置0 恢复现场
                    diag1[i+r]=diag2[i-r+n-1]=false;

                } 
            }
            return;
        };
        dfs(0);
        return res;


    }
};
```

#### 模板题 组合总和

一题两解来理解回溯法

[39.组合总和](https://leetcode.cn/problems/combination-sum/description/)

>给你一个 无重复元素 的整数数组 candidates 和一个目标整数 target ，找出 candidates 中可以使数字和为目标数 target 的 所有 不同组合 ，并以列表形式返回。你可以按 任意顺序 返回这些组合。

candidates 中的 同一个 数字可以 无限制重复被选取 。如果至少一个数字的被选数量不同，则两种组合是不同的。

这道题我第一次做的时候就感觉太经典了

##### 解法一：枚举起点的回溯

```c++
vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
    vector<vector<int>> res;
    vector<int> num;
    function<void(int, int)> dfs = [&](int j, int sum) //参数表示起点索引为j,此时总和为sum
    {
        if(sum==target)
        {
            res.push_back(num);
            return;
        }
        else if(sum>target)
        {
            return;
        }
        else
        {
            for(int i = j; i<candidates.size();i++)
            {
                num.push_back(candidates[i]);
                dfs(i,sum+candidates[i]);
                num.pop_back();//num恢复现场
                //由于传的参是sum+candidates[i]，在下一次循环并没有改变sum的值，所以不用sum恢复
                //sum恢复现场的写法 ⬇️
                //sum=sum+candidates[i];
                //dfs(i,sum+candidates[i]);
                //sum-=candidates[i];
            }
        }
    };
    dfs(0,0);
    return res;
}
```

##### 解法二：选或不选的回溯

```c++
vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
    vector<vector<int>> res;
    vector<int> num;
    function<void(int, int)> dfs = [&](int j, int sum) //参数表示起点索引为j,此时总和为sum
    {
        if(sum==target)
        {
            res.push_back(num);
            return;
        }
        else if(i==candidates.size()||sum>target)//递归结束条件多了一个i到达边界
        {
            return;
        }
        else
        {
            dfs(i+1,sum);//不选i
            num.push_back(candidates[i]);//选i
            dfs(i,sum+candidates[i]);
            num.pop_back();//num恢复现场
        }
    };
    dfs(0,0);
    return res;
}
```

剪枝优化

如果剩下可以选的元素里最小值都会使sum>target,那就没有必要继续递归了，所以可以先对数组candidates进行升序排序

然后递归结束条件修改

```c++
else if(i==candidates.size()||sum>target)
{
    return;
}
//==================修改为 ⬇️ ⬇️ ⬇️ =========================
else if(i==candidates.size()||candidates[i]+sum>target)//剪枝优化,i之后的元素全部不进行计算
{
    return;
}
```

### 双指针

### 二分法
#### 升序

我愿称之史上最清晰的二分查找教程！！灵神！ 👇

[二分查找 红蓝染色法](https://www.bilibili.com/video/BV1AP41137w7/?vd_source=c43347ef375755d298da8f0c05cfe444)

灵神二分法初探

一般的二分法

**返回有序数组中第一个大于等于x的位置，如果所有数都小于x，则返回数组长度**

这个问题针对**大于等于x**, 其他条件的转化如下

- 第一个$\gt x$的位置 等价于 $\ge (x+1)$
- 最后一个$\lt x$的位置 等价于 $(\ge x)-1$
- 最后一个$\le x$的位置 等价于 $(\ge x + 1 )-1$


开区间端点更新为mid，而闭区间必须移动mid的值，开区间&闭区间&左闭右开做法很清晰
同时我分析了为什么左开右闭有问题，且实现了左开右闭做法 👇

由于除法是向下取整，所以当`(l,l+1]`时中点会是左端点，如果`nums[l]`满足`l=mid`的条件，开区间侧端点更新为mid也即不变,会成死循环，如果向上取整的话`(l,l+1]`,中点为`l+1`,更新为mid即增1，也就到了循环跳出的条件。**这个只针对半开半闭区间，除法的上取整或下取整不影响全开区间和全闭区间**

```c++
int low_bound(vector<int>& nums, int target)
{
    //闭区间写法，也就是在[l,r]中找
    int l = 0;
    int r = nums.size()-1;
    while(l<=r)//如果l<=r说明一直有元素没有被二分,跳出循环时前一步必定是l==r
    {
        int mid = (l-r)/2+r;//防止加法溢出，上取整，l-r是负数，小数部分被舍弃，+r后也就变成了上取整
        if(nums[mid]>=target)//说明nums[mid]为蓝色,相等时也是蓝色，这样才能求出红蓝分界点
        {
            r = mid-1;//mid已经被分类了，下一个应该是mid-1
        }
        else
        {
            l = mid+1;
        }
    }
    //最后一步必定是l==r
    // 跳出循环时r=l-1或l=r+1,所以此时l或者r+1就是结果
    return l;
}

int low_bound2(vector<int>& nums, int target)
{
    //开区间写法，也就是在(l,r)中找
    int l = -1;
    int r = nums.size();
    while(l+1<r)//如果l+1<r说明一直有元素没有被二分 由于(l,l+1)中无元素,跳出循环时l+1=r
    {
        int mid = l+(r-l)/2;//防止加法溢出，下取整
        if(nums[mid]>=target)//说明nums[mid]为蓝色,相等时也是蓝色，这样才能求出红蓝分界点
        {
            r = mid;//mid已经被分类了，下一个应该是mid-1，但由于是开区间，所以r=mid
        }
        else
        {
            l = mid;//mid已经被分类了,但由于是开区间，所以l=mid
        }
    }
    //最后一步必定是l+1=r
    // 跳出循环时区间为(l,l+1),所以此时l+1或者r就是结果
    return r;
}
int low_bound3(vector<int>& nums, int target)
{
    //左开右闭写法，也就是在(l,r]中找
    int l = -1;
    int r = nums.size()-1;
    while(l<r)//如果l<r说明一直有元素没有被二分,跳出循环时l==r
    {
        int mid = (l-r)/2+r;//上取整！！
        if(nums[mid]>=target)//说明nums[mid]为蓝色,相等时也是蓝色，这样才能求出红蓝分界点
        {
            r = mid-1;//mid已经被分类了，下一个应该是mid-1
        }
        else
        {
            l = mid;//mid已经被分类了,但由于是开区间，所以l=mid
        }
    }
    // 跳出循环时区间为(l,l],且一定是r = mid-1导致的跳出循环，所以此时l+1或者r+1就是结果
    return l+1;
}
int low_bound4(vector<int>& nums, int target)
{
    //左闭右开写法，也就是在[l,r)中找
    int l = 0;
    int r = nums.size();
    while(l<r)//如果l<r说明一直有元素没有被二分,跳出循环时l==r
    {
        int mid = l+(r-l)/2;//下取整！！
        if(nums[mid]>=target)//说明nums[mid]为蓝色,相等时也是蓝色，这样才能求出红蓝分界点
        {
            r = mid;//mid已经被分类了，由于是开区间，下一个应该是mid
        }
        else
        {
            l = mid+1;//mid已经被分类了,所以l=mid+1
        }
    }
    // 跳出循环时区间为(l,l],且一定是l = mid+1导致的跳出循环，所以此时l或者r就是结果
    return l;
}
```

#### 降序
在上面的讨论中我们已经了解了二分的模板，接下来深入理解二分

如果一个数组为降序，我们要找到大于val的最小值，可以使用二分查找

同样使用红蓝染色，通过判断mid是红还是蓝最终将数组全部染色

- 闭区间二分时, 更新策略时l=mid+1; r=mid-1; 也就是l-1始终为红色，r+1始终为蓝色
- 开区间二分时, 更新策略时l=mid; r=mid; 也就是l始终为红色，r始终为蓝色
- 左开右闭区间二分时, 更新策略时l=mid; r=mid-1; 也就是l始终为红色，r+1始终为蓝色

掌握上面的规律后，降序数组二分的写法也就一目了然了

#### 第K大的值





### 深度优先搜索/广度优先搜索

### 贡献法

### 分治

### 滑动窗口

[滑动窗口【基础算法精讲 03】](https://www.bilibili.com/video/BV1hd4y1r7Gq/?vd_source=c43347ef375755d298da8f0c05cfe444)

题单

[3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/description/)

[209. 长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/description/)

[713. 乘积小于 K 的子数组](https://leetcode.cn/problems/subarray-product-less-than-k/description/)

初始化区间左边界为0，右边界为0，遍历取区间**右边界**，如果不符合题意就左边界向里缩，知道刚好满足条件，再继续向右取右边界，直到遍历结束

### 最短路

#### Dijkstra算法

模板题 [743. 网络延迟时间](https://leetcode.cn/problems/network-delay-time/description/)

问题描述

图中,求k点到某一点的最短路径

算法流程

- 构建邻接矩阵`g[i][j]`,表示从某一点到某一点的边权,如果没有这样的边,就置为无穷, `dis[n]`数组表示k到n的最短路径长度,初始化为无穷,`dis[k]=0`
- 找到dis中最小的一点`dis[y]`,即为k到该点y的最小距离,固定,之后不再进行计算
- 更新计算y相邻点z的dis, 如果`dis[z]>dis[y]+g[y][z]`,就把`dis[z]`更新,作为临时最短路

代码实现

```c++
//稠密图,朴素Dijkstra
//构建邻接矩阵
int n = 100;//节点个数
vector<vector<int>> g(n, vector<int>(n, INT_MAX/2));
for(auto &v: edges)
{
    int from = v[0];
    int to = v[1];
    int cost = v[2];
    g[from][to] = v[2];
    //g[to][from] = v[2]; //无向图情况
}
vector<int> dis(n, INT_MAX);//最短路数组
vector<int> done(n);//代表最短路是否被固定
dis[k] = 0;
while(true)
{
    int x = -1;
    for(int i = 0; i<n; i++)
    {
        if(!done[n]&&(x==-1||dis[i]<dis[x]))
        {
            x = i;//找未被固定的最短路
        }
    }
    if(x==-1)//所有节点均被固定
    {
        return ranges::max(dis);//走完所有节点的最短时间
    }
    if(dis[x]==INT_MAX/2)
    {
        return; //有节点无法到达
    }
    done[x] = 1;//固定最短路
    for(int i =0; i<n; i++)//更新x相邻节点
    {
        dis[i] = min(dis[i], dis[x]+g[x][i]);
    }
}
```

小根堆优化,适用于稀疏图,利用邻接表存边权信息

```c++
//适合稀疏图
//构建邻接表
int n = 100;//节点个数
unordered_map<int, vector<pair<int,int>>> g;
for(auto &v: edges)
{
    int from = v[0];
    int to = v[1];
    int cost = v[2];
    g[from].emplace_back(to, cost);
    // g[to].emplace_back(from, cost); //无向图
}
vector<int> dis(n, INT_MAX/2);
dis[k] = 0;
priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;//小根堆
pq.emplace(dis[k], k);//源点加入
while(!pq.empty())
{
    auto [x, cost] = pq.front();
    pq.pop();//每次弹出最小值
    if(cost>dis[x])
    {
        continue;//之前计算过这个点
    }
    for(auto &[to, c]:g[x])//更新相邻点
    {
        int sum = dis[x]+c;
        if(dis[to]>sum)
        {
            dis[to] = sum;
            pq.emplace(sum, to);//入队
        }
    }
}
return ranges::max(dis);
```

#### Floyed算法

思路比较简单,可以得到任意两点的最短路, 不断遍历得到任意两点的最短路

外层循环遍历中转点(只有这样才能保证最短路),内存遍历起点和终点, 修改邻接矩阵

```c++
int n = 100;//节点个数
vector<vector<int>> g(n, vector<int>(n, INT_MAX/2));//构建边权矩阵
for(auto& v: times)
{
    g[v[0]][v[1]] = v[2];
}
for(int i = 0; i<n; i++)//自身和自身的距离为0,注意!!!!!!!!
{
    g[i][i]=0;
}
for(int s = 0; s<n; s++)
{
    for(int from = 0; from<n; from++)
    {
        for(int to = 0; to<0; to++)
        {
            int sum = g[from][s] + g[s][to];
            g[from][to] = min(sum, g[from][to]);
        }
    }
}
//此时的g[i][j]就表示i到k的最短路
return ranges::max(g[k]);
```

原始floyd算法为三维DP

`g[k+1][i][j] = min(g[k][i][j], g[k][i][k]+g[k][k][j]);`

[带你发明 Floyd 算法：从记忆化搜索到递推（Python/Java/C++/Go/JS/Rust）](https://leetcode.cn/problems/find-the-city-with-the-smallest-number-of-neighbors-at-a-threshold-distance/solutions/2525946/dai-ni-fa-ming-floyd-suan-fa-cong-ji-yi-m8s51/)

#### dfs暴力算法

暴力搜索每一条可行的路,计算最小值

```c++
//构建邻接表
unordered_map<int, vector<pair<int,int>>> g;
for(auto v: edges)
{
    g[v[0]].emplace_back(v[1], v[2]);
    // g[v[1]].emplace_back(v[0], v[2]);//无向图
}
vector<int> dis(n);
dis[k] = 0;
auto dfs(auto&& dfs, int i)->void
{
    for(auto &[to, c]: g[i])
    {
        int cost = dis[i]+c;
        if(dis[to]>cost)//筛选最小值
        {
            dis[to] = cost;
            dfs(dfs, to);
        }
    }
};
dfs(dfs, k);
return ranges::max(dis);
```

#### BFS暴力算法

```c++
//构建邻接表
unordered_map<int, vector<pair<int,int>>> g;
for(auto v: edges)
{
    g[v[0]].emplace_back(v[1], v[2]);
    // g[v[1]].emplace_back(v[0], v[2]);//无向图
}
vector<int> dis(n, INT_MAX/2);
dis[k] = 0;
queue<int> q;
q.push(k);
while(!q.empty())
{
    int t = q.top();
    q.pop();
    for(auto &[to, c]: g[t])
    {
        int sum = dis[t]+c;
        if(dis[to]>sum)//筛选最小值
        {
            dis[to] = sum;
            q.push(to);
        }
    }
}
return ranges::max(dis);
```

### 求解最大公约数
[求两个数的最大公约数3种算法](https://zhuanlan.zhihu.com/p/338809271)

简单来说就是合并求取连通域的算法

如果想合并两个域，需要找到这两个域的根节点，将两个根节点相互连接(谁是父谁是子呢？慢慢往下看)

```c++
int find(int x) //找到x的根节点
{
    if(fa[x] == x) //如果x的父节点是自己的话,则它就是根节点
        return x;
    else
        return find(fa[x]);//x父节点进行递归
}
```

```c++
inline void merge(int i, int j)
{
    fa[find(i)] = find(j); // 使j根节点成为i根节点的父节点(说白了就是让j的根节点成为合并后的根节点,而i的根节点是其子节点)
}
```

路径压缩


#### 辗转相除法

#### 更相减损法

#### Stein算法（结合辗转相除法和更相减损法的优势以及移位运算）

### 记忆化搜索

举一个很简单的例子：求解第n项斐波那契数列的值，递归代码(朴素dfs)如下

```c++
int feibo(int n) //第n项 n>=0
{
    if(n==0||n==1)
        return 1;
    else
        return feibo(n-1)+feibo(n-2);
}
```

但是画一张图就会发现, 计算f(5)时需要计算f(4)和f(3),而计算f(4)也要计算f(3), 这里就会造成重复计算

复杂度分析

由于计算一个f需要计算其他两个值, 时间复杂度$O(2^n)$

计算完会释放空间, 所以只会开辟长度为n的隐式栈, 空间复杂度$O(n)$, (如果不释放的话可能会是$O(n+n^2)$?利用等差数列求和计算)

能否阻止重复计算,从而降低算法的时间复杂度呢?, 可以,就是把计算过的值保存下来

```c++
vector<int> temp(n,-1);  // 创建一个空字典，用来记录第i个斐波那契数列的值
int feibo(int n)
{
    if(n==0||n==1)
        return 1;
    if (temp[n]==-1)
        temp[n] = feibo(n-1) + feibo(n-2);
    return temp[n];
}
```

构建额外空间temp,空间换时间

时间复杂度$O(n)$, 空间复杂度$O(2n)$,分别为temp所占空间于递归构建的隐式栈

题外话

也可以通过减少递归次数达到降低时间复杂度的效果, 以往的递归都是从最小的子问题开始从后往前递归, 这里的代码为从前往后递归, 本质类似于迭代, 就是给迭代换了个马甲!

```c++
int feibo(int first, int second, int n) {
    if (n <= 0) {
        return 0;
    }
    if (n < 3) {
        return 1;
    }
    else if (n == 3) {
        return first + second;
    }
    else {
        return feibo(second, first + second, n - 1);
    }
}
```

```c++
int feibo(int n) { // 迭代法代码
    int first = 0;
    int second = 1;
    if(n==0)
        return 1;
    for(;n>0;n--)//遍历更新first和second的值
    {
        int temp = second;
        second = first+second;
        first = temp;
        
    }
    return second;
}
```

上两种代码几乎一摸一样, 时间复杂度$O(n)$, 空间复杂度O(1)

### 复杂度分析

#### 递归算法

#### 二分算法的时间复杂度为什么是O(logN)?

#### 分治算法--牺牲空间复杂度换时间复杂度

### 考点整理

- 回溯
- 二分查找
- 滑动窗口
- 双指针
- 最短路数组
- 动态规划
  - 背包DP
  - 状态机DP
  - 状态压缩DP
  - 数位DP

数据结构

- 并查集
- 树状数组
- 单调栈
- 前缀树
