## DeepSeek 技术综述

自回归模式下：  在大语言模型下，token是一个一个预测的，所以会有大量的GEMV，访存密集型，如果想提高并行度，提高带宽，可以选择扩大batch，受限于显存容量，对Attention无影响
 
### MLA 把单词Attention转换成计算密集型的GEMM 

MQA： 减少KV cache ，必然会降低精度

算力密度需求：读一次内存需要做多少次计算

MLA：压缩存储 

### MOE 减小参数激活亮，降低GEMM计算量 FP8: 降低计算量

mlp参数庞大，不是每个 的token都要把所有的W计算一遍的

TP并行 卡内通信

EP并行 机间通信

alltoall通信时间可能和计算时间一样，使用overlap缓解，一次进两个词出两个词，两个batch流水，使吞吐打满

单卡专家数越少，计算越密集，但通信不平衡（每张卡收到的数据量一样），通信时间变大
越小，显存越小，可支持batch变大，计算效率越高
机器越多，资源浪费

## FP8使能

- 细粒度量化：计算在Tensor Core， 量化反量化在Cuda Core 
- 提升累加精度：Tensor Core默认累加精度只有14位，矩阵K维度变大后，影响更明显， 每隔128次计算copy到cuda core(都是FP32？ )中进行累加

- CUDAgraph使能D，由于shape不确定，需要设置一个最大值
- kernel调度问题，提前launch
- 计算均衡：专家不均衡（冗余专家），token不均衡 每一路DP差距大，（切成一样长），attention计算

### 未来

长上下文的稀疏
## DeepGemm

DeepGEMM主要特点包括：

- FP8 支持优化：DeepGEMM采用了 CUDA 核心两级累加。（换句话说这也是NV TensorCore的设计不足之处）FP8 是一种低比特浮点格式，能够在保持一定计算精度的同时大幅提升计算效率。**FP8在大量累加时会累积出现随机误差**。例如FP8 GEMM在英伟达H800 GPU上的累加精度保留14位左右，明显低于FP32累加精度。以K= 4096的两个随机矩阵的GEMM 运算为例，Tensor Core中的有限累加精度可导致最大相对误差接近 2%。DeepSeek将中间结果储存计算升级为FP32（32位浮点），实行高精度累加，然后再转换回 FP8，以降低大量微小误差累加带来的训练偏差。
支持分组GEMM：与 CUTLASS 中传统的分组 GEMM 不同，DeepGEMM 仅对 M 轴进行分组，而 N 和 K 可保持不变。（可专门针对 MoE 模型中的专家量身定制）
即时编译（Just-In-Time, JIT）：通过 JIT 技术，代码可以在运行时动态生成和优化，进一步提升性能和灵活性。这也是跟Cutlass的最大区别。
支持Hopper架构中的TMA加速：包括LHS、LHS 缩放因子和 RHS 矩阵的 TMA 负载，TMA 存储输出矩阵，TMA 多播（LHS 矩阵特有）
使用PTX指令进行性能优化：使用stmatrix PTX 指令。
FFMA SASS 交错：DeepSeek深入分析了SASS编译结果，在FFMA/FADD中调整SASS指令，提高了细粒度 FP8 GEMM 的性能。（这一点很有趣，说明DeepSeek的编译/反编译团队做活儿很细，已然不是普通牛马）
高性能：在 Hopper GPU（例如H100）上，可达到 1350+ TFLOPS 的 FP8 计算性能，这表明DeepSeek针对Hopper进行了深度优化。（把英伟达编译团队该干的活儿干了）特别是对Dense模型的加速比MoE更明显。
仍在发展：DeepGEMM 在某些形状上的表现并不是很好。

EP通信

传统All to all

- 冗余传输，机内冗余和卡内冗余，同一份token要重复发
- 时延问题：网络传输的CPU瓶颈

解决策略

不使用通用all to all
- nvshmem   自定义通信算子 dispatch and combine

prefill实例中的EP通信：两阶段算子设计 （带宽为重）

减小低带宽的机间网络传输，充分利用机内nvlink带宽

decode（时延为重）

DeepGemm

- CDP代替

- MASK GEMM 支持动态shape，mask判断是否计算 

### 专家均衡分析和优化

负载不均： 某些专家收到的token数不一样，token少的专家必须等token多的专家算完才能继续

- token专家选择的随机性，天然热点专家 

静态冗余

-   

dispatch token 均分 


