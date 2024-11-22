# Reduce

归约在我的理解就是将多个元素合并成一个元素的过程，比如向量求和，向量求极值等。这篇博客以向量求和为例，总结了基本的归约方法

利用并行的思想进行归约可以有两种方法，交错归约和相邻归约


## 交错归约

交错归约，比如一个数组为`[1,2,3,4,5,6,7,8]`,第一次使用四个线程,相加为`[1+5, 2+6, 3+7, 4+8]`，第二次使用两个线程, 相加为`[1+5+3+7, 2+6+4+8]`, 第三次使用一个线程,相加为`[1+5+3+7+2+6+4+8]`

native版代码, 其中input大小为N, output大小为gridDim.x, 存着每个线程块的归约结果, 如果线程块个数大于1的话最终的全局归约结果还要对output进行归约!

```c++
__global__ void reduce_kernel(real *input, real *output, const int N)
{
    int tid = threadIdx.x;
    int bid = blockIdx.x;
    int col = bid*blockDim.x;
    for(int offset = blockDim.x>>1; offset>0; offset>>=1)
    {
        if(tid<offset)//只存到一半数据
            input[tid+col] += input[tid+col+offset];
        __syncthreads(); 
    }
    if(tid==0)
        output[bid] = input[tid+col];
}
```

使用共享内存优化全局内存的读取次数

```c++
template<int BLOCK_SIZE>
__global__ void reduce_kernel(real *input, real *output, const int N)
{
    int tid = threadIdx.x;
    int bid = blockIdx.x;
    int col = bid*blockDim.x;
    __shared__ real sa[BLOCK_SIZE];
    sa[tid] = tid+col<N? input[tid+col]: 0.0;//将input的元素存到共享内存中
    __syncthreads(); //确保共享内存已经完成赋值
    for(int offset = blockDim.x>>1; offset>0; offset>>=1)
    {
        if(tid<offset)//只用offset个线程进行计算
            sa[tid] += sa[tid+offset];
        __syncthreads(); 
    }
    if(tid==0)
        output[bid] = sa[0];
}
```
## 相邻归约

比如一个数组为`[1,2,3,4,5,6,7,8]`,第一次使用四个线程,相加为`[1+2, 3+4, 5+6, 7+8]`，第二次使用两个线程, 相加为`[1+2+3+4, 5+6+7+8]`, 第三次使用一个线程,相加为`[1+2+3+4+5+6+7+8]`, 实现代码如下

```c++
template<int BLOCK_SIZE>
__global__ void reduce_kernel(real *input, real *output, const int N)
{
    int tid = threadIdx.x;
    int bid = blockIdx.x;
    int col = bid*blockDim.x;
    __shared__ real sa[BLOCK_SIZE];
    sa[tid] = tid+col<N? input[tid+col]: 0.0;
    __syncthreads();
    for(int stride = 1; stride<blockDim.x; stride*=2)
    {
        if(tid%(2*stride)==0)
            sa[tid] += sa[tid+stride];
        __syncthreads(); 
    }
    if(tid==0)
        output[bid] = sa[0];
}
```

## 线程束内函数
block内线程数大于32的情况

```c++
template<int BLOCK_SIZE>
__global__ void reduce_kernel(real *input, real *output, const int N)
{
    int tid = threadIdx.x;
    int bid = blockIdx.x;
    int col = bid*blockDim.x;
    __shared__ real sa[BLOCK_SIZE];
    __shared__ real sum[BLOCK_SIZE/32];
    sa[tid] = tid+col<N? input[tid+col]: 0.0;//只需要存一个线程束中的32个数据
    real tmp = sa[tid];
    for(int mask = 32>>1; mask>0; mask>>=1)
    {
        tmp += __shfl_down_sync(0xffffffff, tmp, mask);
    }
    //如果block只有32个线程，此时已经完成了归约，tmp即为结果
    //如果bock有超过32个线程，此时tmp为每个线程束的归约结果，还要进一步的归约

    //此时每个线程束的第一个线程的tmp为束内归约结果
    if(tid%32==0)
        sum[tid/32] = tmp;//转移到共享内存，为了之后放到同一个线程束的寄存器中
    __syncthreads();
    if(tid<BLOCK_SIZE/32)
    {
        tmp = sum[tid];
        for(int mask = BLOCK_SIZE/32>>1; mask>0; mask>>=1)
        {
            tmp += __shfl_down_sync(0xffffffff, tmp, mask);
        }
    }
    //此时完成了整个block的归约
    if(tid==0)
        output[bid] = tmp;
}
```