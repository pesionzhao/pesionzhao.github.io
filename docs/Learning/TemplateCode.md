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
- 