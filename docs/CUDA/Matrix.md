# GEMM

General Matrix Multiplication

部分代码: https://github.com/pesionzhao/Basic-GPU-Kernels/blob/master/gemm/gemm.cuh

本文将从最朴素的版本一步步优化，且有详细注释，方便大家学习与理解，可以更好的入门！！！

参考连接: 

[cuda实现matmul的重新解读](https://www.bilibili.com/video/BV1Mi421a7cZ/?vd_source=c43347ef375755d298da8f0c05cfe444)

**A@B=C**, A的维度为`M*K`, B的维度为`K*N`, C的维度为`M*N`

## v1 native
最native的版本, 每个线程计算C一个点的值，访存比 1/2K

```c++
__global__ void matrix_mul(float *da, float *db, float *dc, int M, int N, int K)
{
    //结果矩阵的一个点的计算
    //x对应列，y对应行
    int row = threadIdx.y + blockIdx.y * blockDim.y;
    int col =  threadIdx.x + blockIdx.x * blockDim.x;
    float sum=0;
    //边界判断
    if(row<M && col<N)
    {
        //内积，a矩阵的行和b矩阵的列每个元素相乘相加
        for(int i = 0; i<K; i++)
        {
            sum+=da[row*K+i]*db[col+i*N];
        }
        dc[row*N+col]=sum;
    }
}
```
## v2 使用共享内存并分块

v1的算法中一个线程中重复访问2k个全局内存(A的一行和B的一列),共享内存的读写速度要快于全局内存，我们可以借助共享内存而减少对全局内存的访问

共享内存是对整个线程块可见的区域,对于每个线程都有一个副本(同一线程块数据共享，不同线程块之间数据不共享)。生命周期在核函数内,所以在核函数结束之前必须把其保存到全局内存中

因为共享内存是对线程块共享的，所以应该存整个线程块(block_size)中都会使用到的数据, 由于v1中一个线程就要访问2k个元素， 所以对于一个线程块来说，就需要2k*block_size个元素，也就是说我们的目标是要计算线程块内的全部元素，需要a的block_size行和b的block_size列，由于k可能会很大，共享内存存不下，我们就要采用分块的办法

a按行b按列把k分成block_size大小的块，依次计算先计算两个块的结果，相加就得到了线程块的输出，至于为什么是block_size大小的块，是因为正好可以以一个线程存一个数，方便写代码，当然之后我们也会改它的大小，访存比 1/(2k/BLOCK_SIZE)

```c++
template <int BLOCK_SIZE>
__global__ void matrix_mul(float *da, float *db, float *dc, int M, int N, int K)
{
    //共享内存维度为[blockDim.x, blockDim.y], 这里都为BLOCK_SIZE， 所以大小为BLOCK_SIZE*BLOCK_SIZE
    __shared__ float sa[BLOCK_SIZE*BLOCK_SIZE];
    __shared__ float sb[BLOCK_SIZE*BLOCK_SIZE];
    int num_packs = (K+ BLOCK_SIZE - 1)/BLOCK_SIZE;//上取整计算分块的个数
    int row = threadIdx.y + blockIdx.y*blockDim.y;
    int col = threadIdx.x + blockIdx.x*blockDim.x;
    float sum = 0;
    //计算每个块
    //要注意对于A矩阵是在列方向分块，对于B是在行方向分块，所以计算索引是不同的
    for(int i = 0; i<num_packs; i++)
    {
        int s_id = threadIdx.y*BLOCK_SIZE + threadIdx.x;//共享内存块内索引
        //给sa赋值, 对该线程块在中的共享内存， 要确定当前sa是来自da的哪个位置, 也就是全局索引
        //da_col = i*BLOCK_SIZE+threadIdx.x
        //da_row = blockIdx.y*blockDim.y+threadIdx.y
        sa[s_id] = row<M&&i*BLOCK_SIZE+threadIdx.x<K ? da[row*K+i*BLOCK_SIZE+threadIdx.x]:0.0;
        //给sb赋值
        //db_col = threadIdx.x + blockIdx.x*blockDim.x;
        //db_row = i*BLOCK_SIZE+threadIdx.y
        sb[s_id] = i*BLOCK_SIZE+threadIdx.y<K&&col<N ? db[(i*BLOCK_SIZE+threadIdx.y)*N+col]:0.0;
        __syncthreads();//保证数据全都载入
        for(int j = 0; j< BLOCK_SIZE; j++)//相当于v1中的k
        {
            //sa的行和sb的列计算内积
            sum += sa[threadIdx.y*BLOCK_SIZE+j]*sb[j*BLOCK_SIZE+threadIdx.x];
        }
        __syncthreads();//这个块计算完在进行下个块的计算，要不然没计算完共享内存就被更改了
    }
    if(row<M && col<N)
        dc[row*N+col] = sum;//转移到全局内存
}
```

## v3 一个线程处理多个元素

访存比 (2*num_per_thread/num_per_thread^2) * (1/(k/TILE)), num_per_thread=1时和v2一样， num_per_thread为一个线程处理的元素个数

```c++
//TILE为分块的大小， num_per_thread为每个线程处理的元素个数， 默认为4
template<int TILE, int num_per_thread=4>
__global__ void matrix_mul(float *da, float *db, float *dc, int M, int N, int K)
{
    //一个线程，对da处理num_per_thread行， 对db处理num_per_thread列, 所以要扩充共享内存的大小
    __shared__ float sa[num_per_thread*TILE*TILE];//sa shape: [num_per_thread*TILE, TILE]
    __shared__ float sb[num_per_thread*TILE*TILE];//sb shape: [TILE, num_per_thread*TILE]
    //当前行列
    int row = num_per_thread*blockIdx.y*blockDim.y+threadIdx.y*num_per_thread;
    int col = num_per_thread*blockIdx.x*blockDim.x+threadIdx.x*num_per_thread;
    //结果数组，shape: [num_per_thread, num_per_thread]
    float tmp[num_per_thread * num_per_thread] = {0.0f};
    int num_packs = (K+TILE-1)/TILE;
    //依次计算每个块
    for(int i = 0; i<num_packs; ++i)
    {
        //一个线程要循环计算num_per_thread个元素
        for(int j = 0; j<num_per_thread; ++j)
        {
            //以下只是一种共享内存的存数据的顺序，不唯一！！当时写完之后发现有点复杂，但是也是一种思路，就没有删，之后的还会有其他排列方式，如果不理解可以跳过，不影响之后的进程
            //因为一个线程同时计算A的num_per_thread行和B的num_per_thread列
            //所以对于sa来说每个threadIdx.y要存num_per_thread个元素， 对于sb来说每个threadIdx.x要存num_per_thread个元素
            //所以要分别计算sa和sb的块内索引

            //对于sa, y方向连续存num_per_thread个值
            int sa_id = threadIdx.x+(threadIdx.y*num_per_thread+j)*TILE;
            //对于sb, x方向连续存num_per_thread个值
            int sb_id = (threadIdx.x*num_per_thread+j)+threadIdx.y*TILE*num_per_thread;

            //共享内存载入数据， 
            //对于da来说: row = row+j, col = i*TILE + threadIdx.x
            sa[sa_id] = row+j<M&&i*TILE+threadIdx.x<K? da[(row+j)*K+i*TILE+threadIdx.x]:0.0;
            //对于db来说: row = i*TILE+threadIdx.y, col = col+j
            sb[sb_id] = i*TILE+threadIdx.y<K&&col+j<N? db[(i*TILE+threadIdx.y)*N+col+j]:0.0;
        }
        __syncthreads();
        //当前的sa 和 sb 会计算出一个大小为[num_per_thread * num_per_thread]的临时结果
        //这里也可以选择使用二重循环
        for(int j = 0; j<num_per_thread*num_per_thread; ++j)
        {
            for(int k = 0; k<TILE; ++k)
            {
                //行优先, 所以要优先改变sb的列才能得到下一个tmp的值
                tmp[j] += sa[k+(threadIdx.y*num_per_thread+j/num_per_thread)*TILE]*sb[k*TILE*num_per_thread+threadIdx.x*num_per_thread+j%num_per_thread];
            }
        }
        __syncthreads();
    }
    if(row<M&&col<N)
    {
        for(int j = 0; j<num_per_thread*num_per_thread; ++j)
        {
            if((row+j/num_per_thread<M)&&(col+j%num_per_thread<N))
                dc[(row+j/num_per_thread)*N+col+j%num_per_thread] = tmp[j];
        }
    }
}
```

## v4 索引重排

和v3类似，只不过将共享内存设置为和线程块一样的大小

比如线程块为`32*32`， 需要计算的大小为`32*num_per_thread*32*num_per_thread`, v3的sa和sb共享内存的大小为`32*32*num_per_thread`, v4的共享内存大小设置为和线程块一样的大小，这样一个线程只用存一个数据，设置大小为`32*32`=`32/num_per_thread*32*num_per_thread`, 缩短分块(TILE)的长度用来扩大一个线程处理的数据个数

简单来说v3中一个线程要存4个共享内存， v4一个线程只需要存1个共享内存，只不过增加了分块的数量

```c++
template<int BLOCK_SIZE, int num_per_thread=4>
__global__ void matrix_mul(float *da, float *db, float *dc, int M, int N, int K)
{
    //num_per_thread必须时BLOCK_SIZE的因子
    __shared__ float sa[BLOCK_SIZE*BLOCK_SIZE];//sa shape: [BLOCK_SIZE*num_per_thread, BLOCK_SIZE/num_per_thread]
    __shared__ float sb[BLOCK_SIZE*BLOCK_SIZE];//sb shape: [BLOCK_SIZE/num_per_thread, BLOCK_SIZE*num_per_thread]
    //这里由于sa的列数和sb的行数为B/num_per_thread, 所以共享内存的索引要重新计算
    int row = num_per_thread*blockIdx.x*blockDim.x;
    int col = num_per_thread*blockIdx.y*blockDim.y;
    int TILE = BLOCK_SIZE/num_per_thread;// 分块大小 例如：32/4 = 8
    int LEN = BLOCK_SIZE*num_per_thread; // 共享内存的(在计算num_per_thread个数据的方向)另一个维度， 例如：32*4 = 128
    //线程块内索引
    int tid = threadIdx.y*blockDim.x+threadIdx.x;
    //对应的sa,sb索引
    int sa_row = tid/TILE;
    int sa_col = tid%TILE;
    int sb_row = tid/LEN;
    int sb_col = tid%LEN;;
    //结果数组，shape: [num_per_thread, num_per_thread]
    float tmp[num_per_thread * num_per_thread] = {0.0f};
    //依次计算每个块
    for(int i = 0; i<(K+TILE-1)/TILE; ++i)
    {
        //共享内存载入数据， 
        //这里的共享内存访问顺序不像v3一个线程要存num_per_thread个数据，而且块内索引一样和v2一样，代码简洁
        sa[tid] = row+sa_row<M&&i*TILE+sa_col<K? da[(row+sa_row)*K+i*TILE+sa_col]:0.0;
        sb[tid] = i*TILE+sb_row<K&&col+sb_col<N? db[(i*TILE+sb_row)*N+col+sb_col]:0.0;
        __syncthreads();
        //当前的sa 和 sb 会计算出一个大小为[num_per_thread * num_per_thread]的临时结果
        //这里也可以选择使用二重循环
        for(int j = 0; j<num_per_thread*num_per_thread; ++j)
        {
            for(int k = 0; k<TILE; ++k)
            {
                //行优先, 所以要优先改变sb的列才能得到下一个tmp的值
                tmp[j] += sa[k+(threadIdx.y*num_per_thread+j/num_per_thread)*TILE]*sb[k*LEN+threadIdx.x*num_per_thread+j%num_per_thread];
            }
        }
        __syncthreads();
    }
    if(row<M&&col<N)
    {
        for(int j = 0; j<num_per_thread*num_per_thread; ++j)
        {
            if((row+threadIdx.y*num_per_thread+j/num_per_thread<M)&&(col+threadIdx.x*num_per_thread+j%num_per_thread<N))
                dc[(row+threadIdx.y*num_per_thread+j/num_per_thread)*N+col+threadIdx.x*num_per_thread+j%num_per_thread] = tmp[j];
        }
    }
}
```

## v5 float4类型
使用内置cuda类型float4, 一次读取4个元素, 优化数据读取， 也就是一个线程读取四个数据，
与v4不同的是， v4每个线程会存到一次共享内存，共享内存一位是一个数据， 而v5每个线程会存到4个数据，共享内存一位4个数据, 这里就和v3梦幻联动了，我们的v3实现的不就是朴素的float4访存吗！但是float4数据是按行优先存的，也就是没法像v3一样对sb的列方向同时存num_per_thread个数字，只能对行进行存，所以需要v4线程重排的技巧

由于一个线程直接存四个元素，省略for循环，共享内存大小就成了之前的四倍，由于对行进行float4访存，所以列数要乘4，

所以sa的shape为`[TILE*num_per_thread, TILE*4/num_per_thread]`，sb的shape为`[TILE*4/num_per_thread, TILE*num_per_thread]`

```c++
template<int TILE, int num_per_thread=4>
__global__ void matrix_mul(float *da, float *db, float *dc, int M, int N, int K)
{
    //行方向
    __shared__ float sa[4*TILE*TILE];//sa shape: [TILE*num_per_thread, TILE*4/num_per_thread]
    __shared__ float sb[4*TILE*TILE];//sb shape: [TILE*4/num_per_thread, TILE*num_per_thread]
    //索引重排
    int row = num_per_thread*blockIdx.y*blockDim.y;
    int col = num_per_thread*blockIdx.x*blockDim.x;
    int num_packs = (K+TILE*4/num_per_thread-1)/(TILE*4/num_per_thread);
    //线程块索引
    int tid = threadIdx.y*blockDim.x+threadIdx.x;
    int sa_row = tid/(TILE*4/num_per_thread/4);//由于每个线程处理四个元素，所以要除以4，下面的都同理
    int sa_col = tid%(TILE*4/num_per_thread/4);
    int sb_row = tid/(TILE*num_per_thread/4);//由于每个线程处理四个元素，所以要除以4
    int sb_col = tid%(TILE*num_per_thread/4);
    float tmp[num_per_thread*num_per_thread] = {0.0f};
    for(int patch = 0; patch<num_packs; ++patch)
    {
        //使用float4直接存4个元素，所以tid要*4
        (float4 &)sa[tid*4] = (float4 &)da[(row+sa_row)*K+patch*TILE*4/num_per_thread+sa_col*4];
        (float4 &)sb[tid*4] = (float4 &)db[(patch*TILE*4/num_per_thread+sb_row)*N+sb_col*4+col];
        //边界判断
        for(int i = 4; i>=1; i--)
        {
            if(row+sa_row>=M||patch*TILE+4*sa_col+i>=K)
                sa[4*tid+i] = 0.0;
            if(patch*TILE+sb_row>=K||col+4*sb_col+i>=N)
                sb[4*tid+i] = 0.0;
        }
        __syncthreads();
        //计算元素，和之前一样
        for(int j = 0; j<num_per_thread*num_per_thread; ++j)
        {
            for(int k = 0; k<(TILE*4/num_per_thread); ++k)
            {
                tmp[j] += sa[k+(threadIdx.y*num_per_thread+j/num_per_thread)*TILE*4/num_per_thread]*sb[k*num_per_thread*TILE+threadIdx.x*num_per_thread+j%num_per_thread];
            }
        }
        __syncthreads();
    }
    //赋值
    for(int j = 0; j<num_per_thread*num_per_thread; ++j)
    {
        if((row+threadIdx.y*num_per_thread+j/num_per_thread<M)&&(col+threadIdx.x*num_per_thread+j%num_per_thread<N))
            dc[(row+threadIdx.y*num_per_thread+j/num_per_thread)*N+col+threadIdx.x*num_per_thread+j%num_per_thread]= tmp[j];
    }

}
```

## v6 减少bank冲突
共享内存在物理上被分为32个bank, 除开普勒架构外每个bank的一层宽度为4字节，一个线程束内(相邻32个线程)访问同一个bank的不同层级会造成bank冲突，也就是说线程束中的每个线程只访问共享内存的一个bank, 对于v5来说，可以观察到对sb访问时有`threadIdx.x*num_per_thread+j%num_per_thread`，也就是说同一个线程访问了相邻的num_per_thread个bank, 这必然导致bank冲突，所以优化策略为修改sb的存储顺序`(float4 &)sb[tid*4]`为`sb[tid]`,`sb[tid+TILE*TILE]`,`sb[tid+2*TILE*TILE]`,`sb[tid+3*TILE*TILE]`

## v7 减小共享内存的读取
共享内存的访存速度不如寄存器，在计算内积时需要不断访问共享内存，可以更改循环次序，将循环利用的数存到寄存器中
```c++
template<int TILE, int num_per_thread=4>
__global__ void matrix_mul(float *da, float *db, float *dc, int M, int N, int K)
{
    __shared__ float sa[4*TILE*TILE];//sa shape: [TILE*num_per_thread, TILE*4/num_per_thread]
    __shared__ float sb[4*TILE*TILE];//sb shape: [TILE*4/num_per_thread, TILE*num_per_thread]
    //索引重排
    int row = num_per_thread*blockIdx.x*blockDim.x;
    int col = num_per_thread*blockIdx.y*blockDim.y;
    int num_packs = (K+TILE*4/num_per_thread-1)/(TILE*4/num_per_thread);
    //线程块索引
    int tid = (threadIdx.y*blockDim.x+threadIdx.x);
    int sa_row = tid/(TILE*4/num_per_thread/4);
    int sa_col = tid%(TILE*4/num_per_thread/4);
    int sb_row = tid/(TILE*num_per_thread/4);
    int sb_col = tid%(TILE*num_per_thread/4);
    float tmp[num_per_thread*num_per_thread] = {0.0f};
    //寄存器，用于减小共享内存的读取
    float com_a[num_per_thread];
    float com_b[num_per_thread];
    for(int patch = 0; patch<num_packs; ++patch)
    {
        (float4 &)sa[tid*4] = (float4 &)da[(row+sa_row)*K+patch*TILE*4/num_per_thread+sa_col*4];
        (float4 &)sb[tid*4] = (float4 &)db[(patch*TILE*4/num_per_thread+sb_row)*N+sb_col*4+col];
        //边界判断
        for(int i = 3; i>=1; i--)
        {
            if(row+sa_row>=M||patch*TILE*4/num_per_thread+4*sa_col+i>=K)
                sa[4*tid+i] = 0.0;
            if(patch*TILE*4/num_per_thread+sb_row>=K||col+4*sb_col+i>=N)
                sb[4*tid+i] = 0.0;
        }
        __syncthreads();
        for(int k = 0; k<TILE*4/num_per_thread; ++k)
        {
            //将共享内存的数据移到寄存器中
            for(int i = 0; id<num_per_thread; i+=4)
            {
                com_a[i] = sa[k + (threadIdx.y*num_per_thread+i)*TILE*4/num_per_thread];
                com_a[i+1] = sa[k + (threadIdx.y*num_per_thread+i+1)*TILE*4/num_per_thread];
                com_a[i+2] = sa[k + (threadIdx.y*num_per_thread+i+2)*TILE*4/num_per_thread];
                com_a[i+3] = sa[k + (threadIdx.y*num_per_thread+i+3)*TILE*4/num_per_thread];
                (float4 &)com_b[i/4] = (float4 &)sb[k*num_per_thread*TILE + threadIdx.x*num_per_thread];
            }
            for(int j = 0; j<num_per_thread*num_per_thread; ++j)
            {
                // tmp[j] += sa[k+(threadIdx.y*num_per_thread+j/num_per_thread)*TILE]*sb[k*num_per_thread*TILE+threadIdx.x*num_per_thread+j%num_per_thread];
                tmp[j] += com_a[j/num_per_thread]*com_b[j%num_per_thread];
            }
        }
        __syncthreads();
    }
    for(int j = 0; j<num_per_thread*num_per_thread; ++j)
    {
        if((row+threadIdx.y*num_per_thread+j/num_per_thread<M)&&(col+threadIdx.x*num_per_thread+j%num_per_thread<N))
            dc[(row+threadIdx.y*num_per_thread+j/num_per_thread)*N+col+threadIdx.x*num_per_thread+j%num_per_thread]= tmp[j];
    }

}
```

## v8 流水并行

在循环`for(int patch = 0; patch<num_packs; ++patch)`中，要做两次同步，一次是确保共享内存赋值完成可以进行计算，一次是确保计算完成确保共享内存可以进行更改，我们可以使用流水并行，减小一次同步，也就是不同线程之间可以计算或者赋值同时进行，需要扩大一倍共享内存，赋值新的数据，计算旧的数据，之后做一次同步就可以，要多处理一次起始的赋值和结束的计算，代码如下~

```c++
template<int TILE,  int num_per_thread=8>
__global__ void matrix_mul(float* da, float* db, float* dc, int M, int N, int K)
{
    //原始TILE SIZE = [TILE/num_per_thread, TILE*num_per_thread]
    //扩大了一倍是流水并行需要存上一时刻和当前时刻的，4是float4能一次写4个元素
    __shared__ float sa[2*4*TILE*TILE]; //sa shape: [TILE*num_per_thread, TILE*4/num_per_thread]
    __shared__ float sb[2*4*TILE*TILE]; //sb shape: [TILE*4/num_per_thread, TILE*num_per_thread]
    int tid = threadIdx.x + threadIdx.y * blockDim.x;
    int row = num_per_thread*blockIdx.y*blockDim.y;
    int col = num_per_thread*blockIdx.x*blockDim.x;
    //假设num_per_thread=8
    int sa_row = tid/(TILE/2/4);
    int sa_col = tid%(TILE/2/4);
    int sb_row = tid/(TILE*2*4/4);
    int sb_col = tid%(TILE*2*4/4);
    float tmp[num_per_thread*num_per_thread] = {0.0f};
    float a[4];
    /*================第一次只存数据================*/
    //转置sa， 方便使用float4读取纵向排布的四个数据
    (float4 &)a[0] = (float4 &)da[(row+sa_row)*K+sa_col*4];
    for(int id = 0; id<4; id++)
    {
        sa[(sa_col*4+id)*TILE*4*2+sa_row]=a[id];//sa转置的情况
    }
    //寄存器，用于减小共享内存的读取
    float com_a[num_per_thread];
    float com_b[num_per_thread];
    // (float4 &)sa[tid*4] = (float4 &)da[(row+sa_row)*K+sa_col*4]; //sa没有转置的情况
    (float4 &)sb[tid*4] = (float4 &)db[(sb_row)*N+col+sb_col*4];
    //分块的数量， 块宽度为TILE*4/num_per_thread， num_per_thread=8时，即为TILE/2
    int num_packs = (K+TILE/2-1)/(TILE/2);
    __syncthreads();
    /*================流水并行================*/
    for(int ph = 1; ph<(K+TILE/2-1)/(TILE/2); ++ph)
    {
        (float4 &)a[0] = (float4 &)da[(row+sa_row)*K+ph*(TILE/2)+sa_col*4];
        for(int id = 0; id<4; id++)
        {
            sa[TILE*TILE*4*(ph&1)+(sa_col*4+id)*TILE*4*2+sa_row]=a[id];
        }
        // (float4 &)sa[TILE*TILE*4*(ph&1)+tid*4] = (float4 &)da[(row+sa_row)*K+ph*(TILE/2)+sa_col*4];
        (float4 &)sb[TILE*TILE*4*(ph&1)+tid*4] = (float4 &)db[(ph*(TILE/2)+sb_row)*N+col+sb_col*4];
        for(int k = 0; k<TILE/2; ++k)
        {
            //转置sa之后可以类似sb一样使用float4读取，减小代码量，但是速度不一定会块
            for(int idx = 0; idx<num_per_thread/4; ++idx)
            {
                (float4 &)com_a[idx*4] = (float4 &)sa[4*TILE*TILE*((ph-1)&1)+k*TILE*2*4+threadIdx.y*num_per_thread+idx*4];
                (float4 &)com_b[idx*4] = (float4 &)sb[4*TILE*TILE*((ph-1)&1)+threadIdx.x*num_per_thread+idx*4+k*TILE*4*2];   
            }
            for(int j = 0; j<num_per_thread*num_per_thread; ++j)
            {
                // tmp[j] += sa[4*TILE*TILE*((ph-1)&1)+k+(threadIdx.y*num_per_thread+j/num_per_thread)*TILE/2]*sb[4*TILE*TILE*((ph-1)&1)+threadIdx.x*num_per_thread+j%num_per_thread+k*TILE*4*2];
                tmp[j] += com_a[j/num_per_thread]*com_b[j%num_per_thread];
            }
        }
        __syncthreads();
    }
    /*================最后一次只计算================*/
    int ph = (K+TILE/2-1)/(TILE/2);
    for(int k = 0; k<TILE/2; ++k)
    {
        for(int idx = 0; idx<num_per_thread/4; ++idx)
        {
            (float4 &)com_a[idx*4] = (float4 &)sa[4*TILE*TILE*((ph-1)&1)+k*TILE*2*4+threadIdx.y*num_per_thread+idx*4];
            (float4 &)com_b[idx*4] = (float4 &)sb[4*TILE*TILE*((ph-1)&1)+threadIdx.x*num_per_thread+idx*4+k*TILE*4*2];   
        }
        for(int j = 0; j<num_per_thread*num_per_thread; ++j)
        {
            // tmp[j] += sa[TILE*TILE*4*((ph-1)&1)+k+(threadIdx.y*num_per_thread+j/num_per_thread)*TILE/2]*sb[4*TILE*TILE*((ph-1)&1)+threadIdx.x*num_per_thread+j%num_per_thread+k*TILE*4*2];
            tmp[j] += com_a[j/num_per_thread]*com_b[j%num_per_thread];
        }
    }
    for(int j = 0; j<num_per_thread*num_per_thread; ++j)
    {
        if((row+threadIdx.y*num_per_thread+j/num_per_thread<M)&&(col+threadIdx.x*num_per_thread+j%num_per_thread<N))
            dc[(row+threadIdx.y*num_per_thread+j/num_per_thread)*N+col+threadIdx.x*num_per_thread+j%num_per_thread]= tmp[j];
    }  
}
```

## v9 最终版

大方向

- 减小对全局内存读取的次数
- 能优先使用寄存器就使用寄存器
- 共享内存要避免bank冲突
- 全局内存合并访问和合并写入

小技巧：

- 能用float4都用float4
- 循环展开，减小指令使用