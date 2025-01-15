# SoftMax cuda算子解读

参考链接

[如何实现一个高效的Softmax CUDA kernel？](https://www.oneflow.org/a/share/jishuboke/54.html)

[How to write a fast Softmax CUDA kernel?](https://github.com/facebookincubator/AITemplate/wiki/How-to-write-a-fast-Softmax-CUDA-kernel%3F)

softmax公式为

$$y_i = \frac{e^{x_i-C}}{\sum_i^N e^{x_i-C}}$$

其中$C = \max x$, 根据公式，其中有sum与max的操作，所以需要两次归约，第一次归约向量$X$得到最大值$C$，第二次归约向量$e^{x_i-C}$得到和

如果有多个线程块的话，还需要对不同线程块的归约结果进行归约，为了方便将kernel融合为一个，我们将只使用一个block计算softmax

## 线程块使用32个线程
由于线程块只有32个线程，所以可以只用线程束函数进行归约，但在这之前每个线程要先计算cols_per_thread的归约结果, cols_per_thread=N/32

```c++
template <typename T>
__inline__ __device__ T Inf();

template <>
__inline__ __device__ float Inf<float>() {
return CUDART_INF_F;
}
//束内归约 
//method=0: 求最大值，method=1: 求和
template<int NUM>
inline __device__ void warpReduce(real *val, int method = 0)
{
    for(int i = 0; i<NUM; i++)
    {
        for(int offset = 32>>1; offset>0; offset>>=1)
        {
            if(method==1)
                val[i] = val[i]+__shfl_xor_sync(0xffffffff, val[i], offset);
            else
                val[i] = max(val[i], __shfl_xor_sync(0xffffffff, val[i], offset));
        }
    }
}
//一个 Warp 处理一行的计算，适用于 num_cols <= 1024 情况, block = <32, 4>, grid = <M/4> 
template<int cols_per_thread>
__global__ void softmax_kernel(real* input, real* output, int M, int N)
{
    constexpr int num_packs = (cols_per_thread+3) / 4;
    int m_idx  = blockIdx.x*blockDim.y + threadIdx.y;
    const int tid = threadIdx.x;
    float4 buf[num_packs];//寄存器存num_packs*4个变量
    for(int row = m_idx ; row<M; row+=gridDim.x*blockDim.y)
    {
        //当前操作的索引
        const int row_idx = row*(N);
        real* row_ptr = input+row_idx;
        real* out_row_ptr = output+row_idx;
        real local_max[1] = {-Inf<float>()};
        //分块
        for(int pack_id = 0; pack_id<num_packs; ++pack_id)
        {
            const int col = (pack_id*32+tid)*4;//重排了线程，使得相邻线程访问相邻四个元素，可以换一种思考方式，一个线程处理的元素是cols_per_thread,这是不连续的，每隔32个处理一次
            if(col<N)
            {
                buf[pack_id] = {row_ptr[col], row_ptr[col+1], row_ptr[col+2], row_ptr[col+3]};
                local_max[0] = max(local_max[0], max(max(buf[pack_id].x, buf[pack_id].y), max(buf[pack_id].z, buf[pack_id].w)));
            }
            else {
                buf[pack_id].x = -Inf<float>();
                buf[pack_id].y = -Inf<float>();
                buf[pack_id].z = -Inf<float>();
                buf[pack_id].w = -Inf<float>();
            }

        }
        //此时local_max[0]是当前线程中cols_per_thread个值的最大值
        warpReduce<1>(local_max);//32个线程束内归约，得到束内最大值，对于此函数也就是全局最大值，且每个线程这个值都一样
        float local_sum[1] = {0.0f};//归约和
        for(int pack_id = 0; pack_id<num_packs; ++pack_id)
        {
            buf[pack_id].x = exp(buf[pack_id].x - local_max[0]);
            buf[pack_id].y = exp(buf[pack_id].y - local_max[0]);
            buf[pack_id].z = exp(buf[pack_id].z - local_max[0]);
            buf[pack_id].w = exp(buf[pack_id].w - local_max[0]);
            local_sum[0] += buf[pack_id].x + buf[pack_id].y + buf[pack_id].z + buf[pack_id].w;
        }
        warpReduce<1>(local_sum, 1);//同上
        for(int pack_id = 0; pack_id<num_packs; ++pack_id)
        {
            int col = (pack_id*32+tid)*4; 
            if(col<N)
            {
                out_row_ptr[col] =  buf[pack_id].x/local_sum[0];
                out_row_ptr[col+1] =  buf[pack_id].y/local_sum[0];
                out_row_ptr[col+2] =  buf[pack_id].z/local_sum[0];
                out_row_ptr[col+3] =  buf[pack_id].w/local_sum[0];
            }
        }
    } 
}
```

## 线程块使用超过32个线程
如果线程块中超过x方向超过32个线程，使用束内归约只能得到每个warp的归约结果，还需要对每个warp的归约结果进行进一步归约, 代码如下

```c++
//线程块归约， blockDim.x > 32
template<int NUM>
inline __device__ void blockReduce(real *val, int method = 0)
{
    __shared__ real s_val[NUM][32];//共享内存存每个warp的归约结果，线程数最大为1024， 所以这里最大为32
    int lane = threadIdx.x&0x1f;//threadIdx.x % 32
    int wid = threadIdx.x>>5;//threadIdx.x/32

    warpReduce<NUM>(val, method);

    //此时block中每个warp完成的归约，且在warp首个线程的寄存器中
    if(lane==0)//找每个warp的第一个线程
    {
        for(int i = 0; i<NUM; i++)
            s_val[i][wid] = val[i];//把该寄存器的值赋给共享内存
    }
    __syncthreads();
    //将共享内存里的结果赋给寄存器
    for(int i = 0; i<NUM; i++)
    {
        val[i] = (threadIdx.x<(blockDim.x/32.f))? s_val[i][lane]: 0.0;
    }
    if(wid==0)//此时第一个warp中的数据归约结果
    {
        warpReduce<NUM>(val,method);
    }
}
template<int cols_per_thread, int block_size>
__global__ void softmax_block_kernel(const real* input, real* output, int M, int N)
{
    constexpr int num_packs = (cols_per_thread+3) / 4;
    int m_idx  = blockIdx.x;
    const int tid = threadIdx.x;
    float4 buf[num_packs];//寄存器存num_packs*4个变量
    for(int row = m_idx ; row<M; row+=gridDim.x)
    {
        //当前操作的索引
        const int row_idx = row*(N);
        const real* row_ptr = input+row_idx;
        real* out_row_ptr = output+row_idx;
        real local_max[1] = {-Inf<float>()};
        //分块
        for(int pack_id = 0; pack_id<num_packs; ++pack_id)
        {
            const int col = (pack_id*block_size+tid)*4;//重排了线程，使得相邻线程访问相邻元素，可以换一种思考方式，一个线程处理的元素是cols_per_thread,这是不连续的，每隔32个处理一次
            if(col<N)
            {
                buf[pack_id] = {row_ptr[col], row_ptr[col+1], row_ptr[col+2], row_ptr[col+3]};
                local_max[0] = max(local_max[0], max(max(buf[pack_id].x, buf[pack_id].y), max(buf[pack_id].z, buf[pack_id].w)));
            }
            else {
                buf[pack_id].x = -Inf<float>();
                buf[pack_id].y = -Inf<float>();
                buf[pack_id].z = -Inf<float>();
                buf[pack_id].w = -Inf<float>();
            }

        }
        blockReduce<1>(local_max);//此时归约结果在第一个warp中
        //将全局最大值赋给共享内存以便全部线程访问
        __shared__  float global_max;
        if(tid==0)
            global_max = local_max[0];
        __syncthreads();//让所有线程得到最大值, 也就是对于tid!=0的线程保证共享内存global_max已经被赋值
        float local_sum[1] = {0.0f};//归约和
        for(int pack_id = 0; pack_id<num_packs; ++pack_id)
        {
            buf[pack_id].x = exp(buf[pack_id].x - global_max);
            buf[pack_id].y = exp(buf[pack_id].y - global_max);
            buf[pack_id].z = exp(buf[pack_id].z - global_max);
            buf[pack_id].w = exp(buf[pack_id].w - global_max);
            local_sum[0] += buf[pack_id].x + buf[pack_id].y + buf[pack_id].z + buf[pack_id].w;
        }
        blockReduce<1>(local_sum, 1);//此时归约结果在第一个warp中
        __shared__ float global_sum;//同上
        if(tid==0)
            global_sum = local_sum[0];
        __syncthreads();
        for(int pack_id = 0; pack_id<num_packs; ++pack_id)
        {
            int col = (pack_id*block_size+tid)*4; 
            if(col<N)
            {
                out_row_ptr[col] =  buf[pack_id].x/global_sum;
                out_row_ptr[col+1] =  buf[pack_id].y/global_sum;
                out_row_ptr[col+2] =  buf[pack_id].z/global_sum;
                out_row_ptr[col+3] =  buf[pack_id].w/global_sum;
            }
        }
    }
        
}
```

## 使用共享内存存数据
如果寄存器不能保存全部的数据，就需要使用共享内存了, 然后根据线程索引对共享内存进行访问

```c++
template<int block_size>
__global__ void softmax_shared_kernel(const real* input, real* output, int M, int N)
{
    //blockDim.y = 1
    int m_idx = blockIdx.x;
    int tid = threadIdx.x;
    extern __shared__ float buf[]; //大小为N*sizeof(real)
    int num_packs = N>>2;//所有的线程的pack数量
    // int num_packs = (N/block_size+3)/4;
    for(int row = m_idx; row<M; row+=gridDim.x)
    {
        const int row_offset = row*N;
        const real* row_ptr = input+row_offset;
        real* out_row_ptr = output+row_offset;
        real local_max[1] = {-Inf<float>()};

        for(int pack_id = tid; pack_id<num_packs; pack_id+=block_size)
        {
            const int col = tid;
            // if(col<N/4)
            {
                float4 pack = {row_ptr[col], row_ptr[col+1], row_ptr[col+2], row_ptr[col+3]};
                //这样子存是为了防止bank冲突， 这样线程束中不同线程访问的不同的bank
                buf[col] = pack.x;
                buf[col+num_packs] = pack.y;
                buf[col+2*num_packs] = pack.z;
                buf[col+3*num_packs] = pack.w;
                local_max[0] = max(local_max[0], max(max(pack.x, pack.y), max(pack.z, pack.w)));
            }
        }
        blockReduce<1>(local_max);//此时规约结果在第一个warp中
        __shared__  float global_max;
        if(tid==0)
            global_max = local_max[0];
        __syncthreads();//让所有线程得到最大值, 也就是对于tid!=0的线程保证共享内存global_max已经被赋值
        //求和
        real local_sum[1] = {0.0f};
        for(int pack_id = tid; pack_id<N; pack_id+=block_size) 
        {
            float local_val = exp(buf[pack_id]-global_max);
            buf[pack_id] = local_val;
            local_sum[0] += local_val;
        }
        blockReduce<1>(local_sum, 1);//此时归约结果在第一个warp中
        __shared__ float global_sum;
        if(tid==0)
            global_sum = local_sum[0];
        __syncthreads();//同上
        for(int pack_id=tid; pack_id<num_packs; pack_id+=block_size)
        {
            int col = pack_id;
            out_row_ptr[col] = buf[pack_id]/global_sum;
            out_row_ptr[col+1] = buf[pack_id+num_packs]/global_sum;
            out_row_ptr[col+2] = buf[pack_id+2*num_packs]/global_sum;
            out_row_ptr[col+3] = buf[pack_id+3*num_packs]/global_sum;
        }
        
    }
}
```


总结：
- 如果一个block线程只有32个，所有线程访问同一个值可以通过束内函数去访问寄存器的值，而不用共享内存
- 如果一个block线程超过32个，所有线程访问同一个值就需要使用共享内存，同时要注意同步线程
- 对分块有了更深刻的理解
- 共享内存的赋值可以任意分配，不必按照分块时方法分配，要保证优先避免bank冲突!! 同一个线程束中的不同线程尽量不访问同一个bank

## 适用于flashattention的softmax

根据flashattention的思路，要分块计算softmax(q@k.T)@V, 但是softmax需要全局的结果才能计算，根据前面的代码也可得知，计算softmax只需要一个block, 所以这里并不是让softmax分块，**而是让softmax在blockIdx.y相等的block中重复计算归约结果， 也就是blockIdx.y=0的block就已经可以得到TILE行的全局的归约结果**

参考矩阵乘法的策略，因为对于矩阵乘法来说，输出矩阵一个元素是由输入矩阵的一行和一列计算而来的，所以也可以看成归约，所以我们可以借鉴其思路，使用多个block覆盖输出矩阵，并使用TILE对输入矩阵进行分块，通过循环得到归约结果，这样做适用于FlashAttention

在遍历tile时，计算每个tile的局部最大值和局部和，也就是`tile_max`和`tile_sum`, 并维护`per_max`和`per_max`保存计算过的所有tile的全局归约结果，每次在计算完当前的`tile_max`和`tile_sum`，都要对`per_max`和`per_max`进行更新才能进入下一次循环，

- 对于`per_max`的更新很简单，就是取`max(per_max, tile_max)`
- 对于`per_sum`的更新， 根据softmax公式，这个值是与`per_max`有关的，如果`per_max`没有更新，说明最大值是`per_max`而不是`tile_max`, 则需要把当前`tile_sum`结果更新为以`per_max`计算得到的，然后加`per_sum`, 如果`per_max`发生了更新，说明最大值是`tile_max`而不是`per_max`, 则要把`per_sum`先更新为以`tile_max`计算得到的，然后加`tile_sum`， 归根结底就是以下的公式

$$
\sum e^{x_i-C} = (\sum e^{x_i-C_{old}})*e^{C_{old}-C}
$$

```c++
template<int TILE>
__global__ void softmax_kernel(const real* input, real* output, int M, int N)
{
    __shared__ real tile_max[TILE*TILE];
    __shared__ real tile_sum[TILE*TILE];
    __shared__ real per_max[TILE];//之前所有TILE的最大值
    __shared__ real per_sum[TILE];//之前所有TILE的和
    int num_pack = (N+TILE-1)/TILE;
    int tid = threadIdx.x+threadIdx.y*blockDim.x;//块内索引
    int row = blockDim.y*blockIdx.y+threadIdx.y;//全局索引
    int col = blockDim.x*blockIdx.x+threadIdx.x;
    per_max[threadIdx.y] = -__FLT_MAX__;//初始化
    per_sum[threadIdx.y] = 1;//初始化
    //类似矩阵乘法一样的分块
    for(int pack = 0; pack<num_pack; pack++)
    {
        int global_index = pack*TILE+threadIdx.x+row*N;
        tile_max[tid] = input[global_index];
        tile_sum[tid] = 1;
        __syncthreads();//为共享内存赋值
        //最大值和求和同时进行
        for(int offset=TILE>>1; offset>0; offset>>=1)
        {
            if(threadIdx.x<offset)
            {
                if(tile_max[tid]<tile_max[tid+offset])
                {
                    tile_sum[tid] = tile_sum[tid+offset]+tile_sum[tid]*__expf(tile_max[tid]-tile_max[tid+offset]);
                    tile_max[tid] = tile_max[tid+offset];
                }
                else
                {
                    tile_sum[tid] += tile_sum[tid+offset]*__expf(tile_max[tid+offset]-tile_max[tid]);
                }
            }
            __syncthreads();
        }
        //此时只是得到了一个TILE的归约结果，要通过循环num_pack才能得到全局最大值
        if(per_max[threadIdx.y]>tile_max[threadIdx.y*blockDim.x])
        {
            per_sum[threadIdx.y] += tile_sum[threadIdx.y*blockDim.x]*__expf(tile_max[threadIdx.y*blockDim.x]-per_max[threadIdx.y]);
        }
        else
        {
            per_sum[threadIdx.y] = tile_sum[threadIdx.y*blockDim.x]+per_sum[threadIdx.y]*__expf(per_max[threadIdx.y]-tile_max[threadIdx.y*blockDim.x]);
            per_max[threadIdx.y] = tile_max[threadIdx.y*blockDim.x];
        }
    }
    //此时per_max和per_max为全局的归约结果，且同一blockIdx.y的线程块规约结果一样
    int out_index = row*N+col;
    output[out_index] = __expf(input[out_index]-per_max[threadIdx.y])/per_sum[threadIdx.y];
}
```