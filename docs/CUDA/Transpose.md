矩阵转置

代码地址：https://github.com/pesionzhao/Basic-GPU-Kernels/blob/master/transpose/transpose.cu

native版本：行列互换即可

```c++
__global__ void transpose(float *A, float *B, int N, int M) {
    int i = blockIdx.x * blockDim.x + threadIdx.x;
    int j = blockIdx.y * blockDim.y + threadIdx.y;
    if (i < N && j < M) {
        B[j * N + i] = A[i * M + j];
    }
}
```

优化版本：利用shared memory进行优化，合并访存

```c++
template<int BLOCK_SIZE>
__global__ void  _transposeKernel_v1(float* __restrict__ src, float* __restrict__ dst, int M, int N){
    int row = blockIdx.y * blockDim.y + threadIdx.y;
    int col = blockIdx.x * blockDim.x + threadIdx.x;
    __shared__ float cache[BLOCK_SIZE][BLOCK_SIZE];
    if(row < M && col < N){
        //写入的时候就直接转置好，但这里也会引起bank冲突，经测试，读取时避免bank冲突比写入时避免bank冲突性能高
        cache[threadIdx.x][threadIdx.y] = src[row*N+col];
    }
    __syncthreads();
    //块首地址+块内索引
    int dst_row = blockIdx.x * blockDim.x + threadIdx.y;
    int dst_col = blockIdx.y * blockDim.y + threadIdx.x;
    if(dst_row < N && dst_col < M){
        //读取无bank冲突
        dst[dst_row*M+dst_col] = cache[threadIdx.y][threadIdx.x];
    }
}
```

防止bank冲突

```c++
template<int BLOCK_SIZE>
__global__ void  _transposeKernel_v2(float* __restrict__ src, float* __restrict__ dst, int M, int N){
    int row = blockIdx.y * blockDim.y + threadIdx.y;
    int col = blockIdx.x * blockDim.x + threadIdx.x;
    //防止bank冲突，更改cache的大小，使得相邻threadIdx.x相差33，而不是32(同一个bank)
    __shared__ float cache[BLOCK_SIZE][BLOCK_SIZE+1];
    if(row < M && col < N){
        cache[threadIdx.y][threadIdx.x] = src[row*N+col];
    }
    __syncthreads();
    //块首地址+块内索引
    int dst_row = blockIdx.x * blockDim.x + threadIdx.y;
    int dst_col = blockIdx.y * blockDim.y + threadIdx.x;
    if(dst_row < N && dst_col < M){
        dst[dst_row*M+dst_col] = cache[threadIdx.x][threadIdx.y];
    }
}
```

