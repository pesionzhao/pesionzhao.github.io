## 赛题解读

**任务**

由空间中的特征预测雷达的种类和个数

**数据**

训练集：30个空间的特征，以及每个空间含有的雷达类型和个数
每个空间特征为：[频率, 到达时间, 脉冲宽度, 到达角度, 幅度, 脉内采样数据], 大小为[100’000, 6], 且脉内采样数据可能缺省

**模型**

模型输入: 某一空间的[频率, 到达时间, 脉冲宽度, 到达角度, 幅度, 脉内采样数据]
模型输出: 该空间下的雷达型号及其个数

## 数据预处理

从csv, mat, txt等文件中读取数据

```python
def get_mat_files(folder_path):
    mat_files = []
    csv_files = []
    for root, dirs, files in os.walk(folder_path):
        for file in files:
            if file.endswith('.mat'):
                mat_files.append(os.path.join(root, file))
            elif file.endswith('.csv'):
                csv_files.append(os.path.join(root, file))
    return mat_files, csv_files
def get_label(label_path):
    result = {}
    # 读取文件内容
    with open(label_path, 'r', encoding='utf-8') as file:
        lines = file.readlines()
    # 遍历每一行
    for line in lines:
        line = line.strip()
        if line:
            # 匹配场景名称
            match = re.match(r'场景(\d+)：', line) # ()内为捕获的内容
            if match:
                scene_number = match.group(1)
                # 提取数字及其对应的括号内的值
                items = re.findall(r'(\d+)\((\d+)\)', line)
                result[scene_number] = [(int(key), int(value)) for key, value in items]
    return result
label = get_label('dataset/Train_data/Train_Taget.txt')
mat_files, csv_files = get_mat_files('dataset/Train_data')
```

每个场景中的特征都是都与时间有关,所以需要先画出[频率, 脉冲宽度, 到达角度, 幅度]与到达时间的关系图,以便分析数据

```Python
for name in csv_files:
    csv = pd.read_csv(name, encoding='gbk')
    basename = os.path.basename(name)
    match = re.search(r'(\d+)\.csv$', basename)
    basename = os.path.splitext(basename)[0]
    id = match.group(1)
    value = csv.values
    time = value[:, 1] # 到达时间
    frequcy = value[:, 0] # 频率
    amplitude = value[:, -1] # 幅度
    angle = value[:, -2] # 角度
    width = value[:, -3] # 脉冲宽度
    fig, ax = plt.subplots(2,2)
    ax[0,0 ].scatter(time, frequcy, s=1)
    ax[0,0].set_title('time-frequcy')
    ax[0,1].scatter(time, amplitude, s=1)
    ax[0,1].set_title('time-amplitude')
    ax[1,0].scatter(time, angle, s=1)
    ax[1,0].set_title('time-angle')
    ax[1,1].scatter(time, width, s=1)
    ax[1,1].set_title('time-width')
    fig.suptitle(f'{os.path.basename(name)}, id: {id}, label: {label[id]}')
    plt.tight_layout()
    plt.show()
```

部分结果如下

![](https://s2.loli.net/2024/10/18/WkG1ZyVfnMgKIFL.png)

![](https://s2.loli.net/2024/10/18/xb7m9IWVsPj5nBD.png)

相同雷达发出的频率应该是固定的某些值, 同时观察到场景中的频率主要集中在固定某些值, 也就是可以从场景中的所含频率的值来作为特征进行预测

所以可以先将场景中的信号按照频率聚类,可以分出不同的频段的信号, 然后再对每个波段的信号进行角度,脉冲宽度,幅度进行分析

```python
from sklearn.cluster import AgglomerativeClustering
def culster(y, show=False):
    # 聚类
    clustering = AgglomerativeClustering(n_clusters=None, distance_threshold=1e6, affinity='euclidean', linkage='single')
    labels = clustering.fit_predict(y.reshape(-1, 1))  # 只使用 y 值进行分类
    # label即为每个点的类别
    if show:
        unique_labels = np.unique(labels)
        colors = plt.cm.get_cmap("tab10", len(unique_labels))  # 使用 colormap "tab10" 生成不同颜色
        
        # 创建散点图
        plt.figure(figsize=(10, 6))
        for i, label in enumerate(unique_labels):
            if label == -1:
                # -1 表示噪声点，使用灰色
                color = "grey"
            else:
                color = colors(i)  # 为每个类分配一种颜色
        
            # 筛选出该类的数据
            class_data = data[labels == label]
        
            # 绘制散点图
            plt.scatter(class_data[:, 0], class_data[:, 1], color=color, label=f"{label}", s=1, alpha=0.6)
        
        # 添加图例
        plt.legend()
        plt.title("time-frequency")
        plt.xlabel("time")
        plt.ylabel("frequency")
        plt.show()

    return labels
```

展示的结果如下

![](https://s2.loli.net/2024/10/18/OeK68aZy73kDMjV.png)

分类正确, 即可将30类场景的频率进行分类

```Python
for name in csv_files:
    csv = pd.read_csv(name, encoding='gbk')
    basename = os.path.basename(name)
    match = re.search(r'(\d+)\.csv$', basename)
    basename = os.path.splitext(basename)[0]
    id = match.group(1)
    value = csv.values
    time = value[:, 1]
    frequcy = value[:, 0]
    amplitude = value[:, -1]
    angle = value[:, -2]
    width = value[:, -3]
    # 根据frequcy进行聚类
    pos = culster(frequcy)
    # 将分好的类进行保存
    scio.savemat(f'dataset/freq_pos/{basename}.mat', {'pos': pos})
    # 打印出每个场景的频率数
    print(f'{basename} is done, class = {np.unique(pos).shape[0]}')
```

按照频率分好的类,将相同频率信号的角度,脉冲宽度,幅度进行分类,并画出

```Python
for name in csv_files:
    csv = pd.read_csv(name, encoding='gbk')
    basename = os.path.basename(name)
    match = re.search(r'(\d+)\.csv$', basename)
    basename = os.path.splitext(basename)[0]
    id = match.group(1)
    value = csv.values
    time = value[:, 1]
    frequcy = value[:, 0]
    amplitude = value[:, -1]
    angle = value[:, -2]
    width = value[:, -3]
    # 展示聚类后的频率数据
    pos = scio.loadmat(f'dataset/freq_pos1e7/{basename}.mat')['pos']
    pos = np.squeeze(pos,0)
    unique_labels = np.unique(pos)
    # pos = update_cluster(frequcy, pos)
    colors = plt.cm.get_cmap("tab10", len(unique_labels))  # 使用 colormap "tab10" 生成不同颜色
    fig, ax = plt.subplots(2,2)
    for i, p in enumerate(unique_labels):
        if i == -1:
            # -1 表示噪声点，使用灰色
            color = "grey"
        else:
            color = colors(i)  # 为每个类分配一种颜色

        # 筛选出该类的数据
        class_freq = frequcy[pos == p]
        class_time = time[pos == p]
        class_amplitude = amplitude[pos == p]
        class_angle = angle[pos == p]
        class_width = width[pos == p]
        ax[0,0].scatter(class_time, class_freq, color=color, label=f"{p}", s=1, alpha=0.6)
        ax[0,0].set_title('time-frequcy')
        ax[0,1].scatter(class_time, class_amplitude, color=color, label=f"{p}", s=1, alpha=0.6)
        ax[0,1].set_title('time-amplitude')
        ax[1,0].scatter(class_time, class_angle, color=color, label=f"{p}", s=1, alpha=0.6)
        ax[1,0].set_title('time-angle')
        ax[1,1].scatter(class_time, class_width, color=color, label=f"{p}", s=1, alpha=0.6)
        ax[1,1].set_title('time-width')

    plt.legend()
    plt.show()
```

结果如下

![](https://s2.loli.net/2024/10/18/VgLmG45XNxv7CRd.png)

可以看出相同频率的信号角度,脉冲宽度,幅度上的分布是比较一致的, 所以可以将角度,脉冲宽度,幅度作为特征进行预测,可以构建向量空间, 向量长度为频率个数, 向量特征为[角度,脉冲宽度,幅度], 

具体实现: 通过分析得到30个空间的频率范围为2e8~1.47e10, 所以可以将频率分为145份,每份宽度为1e8 hz,  也就是向量长度为145, 向量宽度代表[初始角度,最后角度,脉冲宽度,幅度], 为4, 结果为50个雷达的个数

模型输入 data.shape = [145, 4]

模型输出 label.shape = [50, 1]

## 生成数据

观察每个数据的分布以进行预处理

```
class is 7
amp_max:0.76411207455, amp_min:0.26620381523371683, width_max:2.052930435332028e-06, width_min:1.000977088544272e-06, angle_max:88.02397296, angle_min:88.39031187
class is 18
amp_max:0.7680373747999999, amp_min:0.569752868, width_max:3.1424472646201333e-06, width_min:9.9865450180072e-07, angle_max:88.04380572, angle_min:88.44179694
class is 12
amp_max:0.7653497896, amp_min:0.2676597103333333, width_max:3.001172468987595e-06, width_min:1.9992597038815526e-06, angle_max:88.02397296, angle_min:88.44179694
class is 22
amp_max:1.1243949759999998, amp_min:0.5678371488333334, width_max:4.001540616246498e-06, width_min:2.0012104841936774e-06, angle_max:175.90854530000001, angle_min:88.39031187
class is 31
amp_max:0.766928774, amp_min:0.11893168799999998, width_max:6.000588235294117e-06, width_min:1.998789515806323e-06, angle_max:88.02397296, angle_min:88.39031187
class is 66
amp_max:0.7657542412, amp_min:0.11125264699999998, width_max:4.001442977190877e-06, width_min:9.95328931572629e-07, angle_max:88.02397296, angle_min:88.39031187
class is 38
amp_max:0.872980201, amp_min:0.24023715419999994, width_max:2.4996338535414163e-06, width_min:9.996422569027613e-07, angle_max:88.02397296, angle_min:88.39031187
class is 87
amp_max:0.7768302029999999, amp_min:0.4487626909999999, width_max:6.004165666266507e-06, width_min:2.9952941176470587e-06, angle_max:88.04380572, angle_min:88.44179694
class is 80
amp_max:0.776859914, amp_min:0.14046983600000001, width_max:1.0046170468187275e-06, width_min:9.96795918367347e-07, angle_max:88.04380572, angle_min:88.44179694
class is 74
amp_max:0.776806206, amp_min:0.141151507, width_max:3.999387755102041e-06, width_min:9.961908763505402e-07, angle_max:88.04380572, angle_min:88.44179694
class is 64
amp_max:1.5532265820000004, amp_min:0.2724507057857143, width_max:6.001296518607442e-06, width_min:1.999888080581981e-06, angle_max:175.78056189999998, angle_min:88.44179694
class is 5
amp_max:0.7652766232222221, amp_min:0.7329277794793507, width_max:5.000049462159047e-06, width_min:2.999730839704303e-06, angle_max:88.04380572, angle_min:88.44179694
class is 9
amp_max:0.7670647923333332, amp_min:0.26711427543081545, width_max:3.0554788582099502e-06, width_min:9.997065532766383e-07, angle_max:88.02397296, angle_min:88.45415785
class is 21
amp_max:1.008827406, amp_min:0.2445645775401241, width_max:3.2859921111301668e-06, width_min:9.994533013205283e-07, angle_max:175.7798208, angle_min:88.44179694
class is 10
amp_max:0.7662765956, amp_min:0.2779798915, width_max:3.0006376308944173e-06, width_min:1.4006559423769507e-06, angle_max:88.04380572, angle_min:88.46026507
class is 8
amp_max:0.7683727005333334, amp_min:0.5535565518333334, width_max:2.3557562491663335e-06, width_min:9.995966386554623e-07, angle_max:88.04380572, angle_min:88.51878484
class is 21
amp_max:1.1375659509999996, amp_min:0.5627994821111111, width_max:3.333030926656377e-06, width_min:9.99367106842737e-07, angle_max:175.7573519, angle_min:88.44179694
class is 27
amp_max:0.77535528, amp_min:0.6077169904999999, width_max:3.99938775510204e-06, width_min:1.997046818727491e-06, angle_max:88.02397296, angle_min:88.44179694
class is 31
amp_max:0.7687933324999999, amp_min:0.11194344000000005, width_max:3.5999435774309725e-06, width_min:1.9992256902761104e-06, angle_max:88.03393933, angle_min:88.45415785
class is 36
amp_max:0.9049455240000003, amp_min:0.11848450800000003, width_max:4.004129651860744e-06, width_min:9.999027611044418e-07, angle_max:176.05791230000003, angle_min:88.46026507
class is 44
amp_max:0.7753617140000002, amp_min:0.26083463425000003, width_max:5.000726290516207e-06, width_min:9.995951380552222e-07, angle_max:88.03393933, angle_min:88.44179694
class is 47
amp_max:0.780404844142857, amp_min:0.11194344000000005, width_max:6.002941176470589e-06, width_min:9.967839135654262e-07, angle_max:88.0434051, angle_min:88.44179694
class is 12
amp_max:0.7986596809375, amp_min:0.5717107596666667, width_max:2.499752400960384e-06, width_min:9.993908763505402e-07, angle_max:88.02397296, angle_min:88.39031187
class is 65
amp_max:0.7765707770000001, amp_min:0.102249845, width_max:3.998247298919567e-06, width_min:9.971212484993998e-07, angle_max:88.02356419, angle_min:88.44179694
class is 6
amp_max:0.7838408363157895, amp_min:0.2779798915, width_max:3.2107651481645294e-06, width_min:2.9996451914098972e-06, angle_max:88.04380572, angle_min:88.39031187
class is 15
amp_max:0.7671324993333333, amp_min:0.55027191561677, width_max:2.000213170982679e-06, width_min:9.987951980792317e-07, angle_max:88.02397296, angle_min:88.39031187
class is 15
amp_max:0.8324584575, amp_min:0.5678371488333334, width_max:4.000492196878751e-06, width_min:1.998937575030012e-06, angle_max:88.02397296, angle_min:88.39031187
class is 11
amp_max:0.7646298163999998, amp_min:0.5775280338590726, width_max:3.250427671068427e-06, width_min:1.2006942376950784e-06, angle_max:88.02397296, angle_min:88.39031187
class is 21
amp_max:0.7687933324999999, amp_min:0.2330405214508705, width_max:3.0004606842737094e-06, width_min:9.981020408163267e-07, angle_max:88.02397296, angle_min:88.39031187
class is 21
amp_max:0.7660576141999998, amp_min:0.2585017576666666, width_max:3.166650660264106e-06, width_min:9.987188475390157e-07, angle_max:88.02397296, angle_min:88.39031187
```

角度集中在87~177, 脉冲宽度集中在1e-6~6e-6, 幅度集中在0~1.5e-6,
## 模型训练

输入数据是每个频率对应的[初始角度,最后角度,脉冲宽度,幅度], 

[145, 4] -> embedding -> [145, 128] -> Linear -> [145, 1] -> Linear -> [50, ]