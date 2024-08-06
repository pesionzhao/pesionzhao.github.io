## Python
### 输入输出

```python
# 允许在字符串中插入变量或表达式的值。
print(f'Hello, {name}!')

# format 方法允许您使用占位符 {} 将变量或表达式的值插入到字符串中，然后使用 format 方法中的参数传递这些值。
print("Hello, {}!".format(name))

# 百分号格式化是传统的字符串格式化方法，它使用 % 运算符将变量或表达式的值插入到字符串中
print("Hello, %s!" % name)
```

### json
```python
# 字典的更新
norm_dict = {}
norm_dict.update({f"F_{i}": {"max": imax, "min": imin}})
# json的读写
with open("date_map.json", "w") as json_file:
    json.dump(date_map, json_file)
with open("date_map.json", "r") as json_file:
    date_map = json.load(json_file)
```

```python
# 判断变量属于哪一类型
if isinstance(my_variable, list): 
    print("my_variable是一个列表")
# 判断类中是否有其属性
if hasattr(obj, 'my_attribute'):
    print("obj具有my_attribute属性")
```

```python
# 路径判断
os.path.exists(path) # 文件是否存在
os.makedirs(save_dir) # 创建文件夹
```

## Numpy

### 维度处理
- ndarray = np.expand_dims(x, axis=1) 加dimension length=1的维度 (加在第一位或最后一维也可写作x[np.newaxis, :] or x[np.newaxis])
- ndarray.squeeze()/numpy.squeeze(a, axis=None) 删dimension length=1的维度
- np.transpose(ndarray, axes=(2, 3, 0, 1)) 交换维度
- np.swapaxes(a, axis1, axis2), 交换两个轴的维度
- ndarray.flatten(order='C') order: {‘C’, ‘F’, ‘A’, ‘K’} 展平化, C按行,F按列
- ndarray.ravel(order)/np.ravel(a, order) flatten和ravel用法相同,只不过flatten返回拷贝ravel返回视图,更改ravel()原矩阵也会跟着改变
- ndarray.reshape(shape=(), order='C')/np.reshape(a, newshape=(), order='C')

>这里的order可取c或者f，对应C语言索引和Fortran索引，c表示一行一行的展开，f表示一列一列的展开，后重新组合，三维同理，看子矩阵即可
>>在C语言中的索引顺序是从左到右，从上到下： A[0][0], A[0][1], A[1][0], A[1][1], A[2][0], A[2][1]
>>
>>而在Fortran语言中则是 从上到下，从左到右 ： A[1][1], A[2][1], A[3][1], A[1][2], A[2][2], A[3][2]

### 统计
- np.std(x, axis=None, dtype=None) 计算沿指定轴的标准差，默认
- np.var(x, axis=None, dtype=None) 计算沿指定轴的方差
- np.mean() 均值同上

### 生成矩阵
- np.ones(shape=(), dtype) 生成全一阵
- np.zeros(shape=(), dtype=float) 生成全零阵 
- np.identity(n, dtype) 生成单位阵
- np.tile(a, reps) 通过重复矩阵a来构造一个新矩阵, rep:a在每个轴上重复的次数
- np.repeat(a, repeats, axis=None) 重复矩阵中的每个元素
- np.kron(a,b) 克罗内克积
- np.vstack([]) 垂直vertically顺序堆叠矩阵
- np.hstack([]) 水平horizontally顺序堆叠矩阵
- np.concatenate(seq=[],axis=0) 按照某一轴拼接矩阵
- np.stack([], axis=0, dtype=),在新维度堆叠矩阵
- np.block() 从嵌套的块列表中组装一个矩阵,类似与np.array(list[]),只不过这里的list中的元素为矩阵

### 矩阵乘法
- np.dot(a, b) 如果a和b都是一维数组，则它是向量的内积,二维优先使用matmul :arrow_lower_left:
- np.matmul(x1, x2) 不管原矩阵多大,都是原矩阵的最后两维点乘,再广播到原矩阵前几维的维度, 和@等价
- np.outer(a,b) 计算两向量外积，也就是列向量乘行向量

np.matmul和np.dot的区别

```python
a = np.ones([9, 5, 7, 4])
c = np.ones([9, 5, 4, 3])

#np.dot(a,c)为分别计算a@b[0] ... a@b[n]
np.dot(a, c).shape
>> (9, 5, 7, 9, 5, 3)

#np.matmul(a,c)计算最后两维矩阵点乘,再广播到前面的9,5的维度中,也就是对应每个子矩阵点乘,前面的维度必须一样
np.matmul(a, c).shape
>>(9, 5, 7, 3)
```

## Plt

### Axes

- fig, ax = plt.subplots(nrows, ncols, sharex, sharey) 共享x轴或共享y轴

```python
fig, (ax1, ax2) = plt.subplots(1, 2)
fig, ((ax1, ax2), (ax3, ax4)) = plt.subplots(2, 2)

fig, ax = plt.subplots()
ax.plot(x, y, label='line')
ax.set_title('Simple plot')

f, (ax1, ax2) = plt.subplots(1, 2, sharey=True)
ax1.plot(x, y)
ax1.set_title('Sharing Y axis')
ax1.set_xlim(left= , right= ) # 设置x轴范围
ax2.scatter(x, y)

# 修改刻度值，只是仅仅修改刻度显示
# 例如将刻度值加100并格式化
from matplotlib.ticker import FuncFormatter

def format_x_ticks(x, pos):
    return int(x + 100)

# 使用FuncFormatter来应用格式化函数
formatter = FuncFormatter(format_x_ticks)
ax.xaxis.set_major_formatter(formatter)

# 如果想修改成字符
ax.set_xticks(tick=ind, labels=['G1', 'G2', 'G3', 'G4', 'G5'])
```
- Axes.plot(x, y, fmt), 这里fmt代表绘图样式. 具体参照[官方文档](https://matplotlib.org/3.5.3/api/_as_gen/matplotlib.axes.Axes.plot.html#matplotlib.axes.Axes.plot)

> 颜色：可以使用单个字符来指定颜色，比如'r'表示红色，'b'表示蓝色，'g'表示绿色等。也可以使用英文单词来表示颜色，比如'red'表示红色，'blue'表示蓝色。
> 
>标记：标记是在数据点上绘制的符号，例如圆圈、方块、三角形等。可以使用字符来指定标记，例如'o'表示圆圈，'s'表示方块，'^'表示三角形等。
>
>线条样式：可以使用字符来定义线条的样式，比如'-'表示实线，'--'表示虚线，':'表示点线，'-'表示线段等。

- Axes.legend()/ax.legend(['First line', 'Second line']) 图例 
- plt.imshow(X, cmap='viridis', norm=None, aspect=None, vmin=None, vmax=None) 
  
cmp选择合适的色标 [官方文档](https://matplotlib.org/stable/users/explain/colors/colormaps.html); aspect:轴的横纵比 'equal'/'auto'; vmin和vmax定义了颜色映射覆盖的数据范围。默认情况下，颜色映射覆盖所提供数据的完整值范围。当给出norm时，使用vmin/vmax是错误的



## Pandas
强烈建议参考kaggle的教程,网址:https://www.kaggle.com/learn/pandas

- pd.read_csv('data.csv', index_col=0), index_col表示指定data.csv中第0列作为DataFrame的列索引
- Data.head(3) 查看一个 DataFrame 的前 n 行，默认参数值为 5
- Data.tail() 查看一个 DataFrame 的末尾 n 行，默认参数值为 - Data.sample(2) 随机抽取一个 DataFrame 的 n 行
- data = Data[Data.date_id==20230123] 取DataFrame列标签date_id为20230123的数据
- pd.DataFrame(data), 由data生成一个DataFrame
- Data.to_csv('data.csv', index=False)
- data.loc[行名,列名] 可以指定多个列[name1, name2]
- data.iloc[行下标,列下标]

> data.loc和iloc在索引时,如果使用切片,例如[0:9],iloc和python的切片操作一样不包含最后一个元素,但loc包含最后一个元素,因为loc可以进行字符串索引,比如Data.loc['Apple':'Banana']!!


```python
# 还可以进行这样的访问:
Data.loc[Data.country == 'Italy'] 
Data.loc[(Data.country == 'Italy')& (Data.points >= 90)]

# 配合条件选择器
# isin 代表 "is in", 可以理解为&
reviews.loc[reviews.country.isin(['Italy', 'France'])]

# isnull and notnull 表示是否为空(Nan)
reviews.loc[reviews.price.notnull()]
```

- data.set_index("title") 使用现有列(这里为title列)作为索引
- Data['critic'] = 'everyone' 增添列
- Data['index_backwards'] = range(len(reviews), 0, -1)

###  总结函数

- reviews.attribute.describe() 均值,最大值等等...
- reviews.taster_name.unique() 返回taster_name唯一序列
- reviews.taster_name.value_counts() 返回每个taster_name对应的个数

### 映射
- reviews.points.map(lambda p: p - review_points_mean) 返回减去均值的新的Series, 也可以简易写作reviews.points-review_points_mean
- reviews.apply(remean_points, axis='columns') remean_points为自定义方法 如果axis='index'时,需要定义操作行数据的函数

### group操作
group相当于把一样的标签打包成一个group

```python
# 将points一样的行打包成一个group
reviews.groupby('points').points.count() #等价于 reviews.points.value_counts()
# # 将country,province一样的行打包成一个group
reviews.groupby(['country', 'province']).apply(lambda df: df.loc[df.points.idxmax()]) # idxmax()代表最大的索引

reviews.groupby(['country']).price.agg([len, min, max]) # agg可以同时运行不同的函数

# 对于多标签所得到的group会得到多索引,可以rest_index改回但索引,返回的会多一个长度列,代表这个单索引对应原多索引向量的长度
countries_reviewed.reset_index()

# 按照长度进行排序 默认升序
countries_reviewed.sort_values(by='len')
# 降序
countries_reviewed.sort_values(by='len', ascending=False)
# 按照索引排序
countries_reviewed.sort_index()
# 多个排序列
countries_reviewed.sort_values(by=['country', 'len'])
```

### 数据操作

```python
# 更改数据类型
reviews.points.astype('float64')
# 替换缺失值,将每个NaN替换为“unknown”。
reviews.region_2.fillna("Unknown")
# 替换
reviews.taster_twitter_handle.replace("@kerinokeefe", "@kerino")

# 更改列名
reviews.rename(columns={'points': 'score'})
# 更改行名 不常用,常用的为 set_index()用现有列更改索引
reviews.rename(index={0: 'firstEntry', 1: 'secondEntry'})
# 行列都有name属性,可以更改这个name属性
reviews.rename_axis("wines", axis='rows').rename_axis("fields", axis='columns')

# 按指定轴拼接两个data
pd.concat([canadian_youtube, british_youtube]) # 默认axis=0 也就是index
# 按列拼接,index相同
left = canadian_youtube.set_index(['title', 'trending_date'])
right = british_youtube.set_index(['title', 'trending_date'])

left.join(right, lsuffix='_CAN', rsuffix='_UK') # 左侧的frame加_CAN的后缀,右侧的frame加_UK的后缀

```
## Torch
卷积输出的计算**N = (W − F + 2P )/S+1**

### 网络搭建
```python
# 网络搭建相关api
nn.Conv1d(in_channels, out_channels, kernel_size, stride=1, padding=0, dilation=1, groups=1, bias=True, padding_mode='zeros', device=None, dtype=None) # 一维卷积
```

其中dilation为膨胀率,会膨胀卷积核增大感受野, group为组操作,具体可参照[官方文档](https://pytorch.org/docs/1.9.0/generated/torch.nn.Conv1d.html?highlight=conv1d#torch.nn.Conv1d), 每个组进行对应的一部分卷积之后拼接起来.方便理解可以参照[可视化卷积过程](https://github.com/vdumoulin/conv_arithmetic/blob/master/README.md)

### 数据生成

```python
# 数据生成相关API
torch.tensor.uniform_(from=0, to=1)  # 用均匀分布的随机数填充张量x
torch.from_numpy() # 返回的张量和narray共享相同的内存
torch.split(tensor, split_size_or_sections, dim=0) # 可以理解为切片，返回视图


autograd.Variable() # 强烈反对 现在只需指定requires_grad=True即可
autograd_tensor = torch.randn((2, 3, 4), requires_grad=True)
```

### 梯度相关 

```python
# 梯度回传
Tensor.backward(gradient=None, retain_graph=None, create_graph=False, inputs=None) # gradient相当于对梯度进行加权, retain_graph: 计算图释放, create_graph 构造导数图, input 指定求特定输入的梯度,如果没有则计算全部
autograd.backward(tensors, grad_tensors=None, retain_graph=None, create_graph=False, grad_variables=None, inputs=None) # 指定某一tensor回传

autograd.grad(outputs, inputs, grad_outputs=None, retain_graph=None, create_graph=False, only_inputs=True, allow_unused=False)
```

### 模型相关

优化器参数解读 https://yiddishkop.github.io/%E4%B8%AA%E4%BA%BA%E6%B7%B1%E5%BA%A6%E5%AD%A6%E4%B9%A0%E8%AE%BA%E6%96%87(%E8%8D%89%E7%A8%BF)/Adam%E4%B8%89%E9%87%8D%E7%BD%97%E7%94%9F%E9%97%A8.html

```python
# 模型参数
model.load_state_dict(torch.load('', map_location='cpu'))
torch.save(model.state_dict(), savepath)

# 优化器
torch.optim.Adam(params, lr=0.001, betas=(0.9, 0.999), eps=1e-08, weight_decay=0, amsgrad=False) # betas用于计算梯度的平均和平方的系数, weight_decay 权重衰减(如L2惩罚), AMSGrad变体
torch.optim.SGD(params, lr, momentum=0, dampening=0, weight_decay=0, nesterov=False)  # 动量阻尼, 权重衰减, 使用Nesterov动量

# scheduler 
lr_scheduler.ExponentialLR(optimizer, gamma, last_epoch=-1, verb ose=False) # 每个epoch学习率进行gamma的衰减 verbose为打印信息
lr_scheduler.MultiStepLR(optimizer, milestones, gamma=0.1, last_epoch=-1, verbose=False) # 为每到milestones规定的epoch进行gamma衰减
```

### 可视化

```python
# Tensorboard可视化 
# 官方文档 https://pytorch.org/docs/stable/tensorboard.html
from torch.utils.tensorboard import SummaryWriter
writer = SummaryWriter('runs') # 目录为./runs
writer.add_scalar(tag, scalar_value, global_step=None, walltime=None, new_style=False, double_precision=False) # tag: 图的名称 ,scalar_value: 值(纵坐标); global_step: 横坐标
add_histogram(tag, values, global_step=None, bins='tensorflow', walltime=None, max_bins=None) # 柱状图
add_image(tag, img_tensor, global_step=None, walltime=None, dataformats='CHW')
add_images(tag, img_tensor, global_step=None, walltime=None, dataformats='NCHW') # 以dataformats的格式展示tensor

add_figure(tag, figure, global_step=None, close=True, walltime=None) # 展示plt.Figure()
add_graph(model, input_to_model=None, verbose=False, use_strict_trace=True)
```

## Scipy

### 稀疏矩阵构建

```python
import scipy.sparse as sp
sp.csc_array((M, N), [dtype])
```