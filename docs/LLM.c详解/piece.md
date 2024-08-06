llmc/rand.h

## 线性同余生成器

线性同余生成器（Linear Congruential Generator，LCG）是一种经典的伪随机数生成器（PRNG），它使用一个线性递推关系来生成伪随机数序列。LCG 的数学公式如下：

\[ X_{n+1} = (aX_n + c) \mod m \]

其中：
- \( X \) 是生成的伪随机数序列。
- \( n \) 表示当前的步骤。
- \( a \) 是乘数（multiplier）。
- \( c \) 是增量（increment）。
- \( m \) 是模数（modulus）。
- \( X_0 \) 是初始种子（seed）。

### 参数选择
为了确保生成的伪随机数序列有良好的统计性质，需要仔细选择参数 \( a \)、\( c \) 和 \( m \)：
- \( m \) 通常选择为一个大质数或 2 的幂。
- \( a \) 和 \( c \) 需要满足某些条件，以确保生成的序列具有较长的周期和良好的分布特性。例如：
  - 如果 \( m \) 是一个大质数，通常选择 \( c = 0 \) 和 \( a \) 为一个好的乘数。
  - 如果 \( m \) 是 2 的幂，通常选择 \( c \neq 0 \)，以避免生成序列陷入某些周期性行为。

### 例子
一个常见的 LCG 实现是 `rand()` 函数，它在许多编程语言和标准库中都有实现。以 C 标准库为例，其参数通常如下：
- \( a = 1103515245 \)
- \( c = 12345 \)
- \( m = 2^{31} \)

```c
unsigned int lcg(unsigned int seed) {
    return (1103515245 * seed + 12345) % (1 << 31);
}
```

虽然 LCG 在现代应用中逐渐被更复杂和安全的伪随机数生成器（如 Mersenne Twister）取代，但由于其简单高效的特点，在某些应用场景中仍然有用。以下介绍Mersenne Twister随机数

## Mersenne Twister

Mersenne Twister（梅森旋转法）是一种常用的伪随机数生成器，其名称源于其周期长度（一个梅森素数的幂）。Mersenne Twister的设计目的是提供一个快速且高质量的随机数生成算法，满足科学计算和模拟等高要求应用。其基本原理如下：

1. **状态数组**：Mersenne Twister维护一个包含624个32位整数的状态数组。这些整数是生成伪随机数的核心。

2. **初始化**：初始状态数组使用一个种子值来填充。种子值可以是任意整数，通常使用一个或多个初始值来生成数组中的每个元素。

3. **提取随机数**：每次提取一个随机数时，算法会从状态数组中提取一个整数，并对其进行转换以产生最终的伪随机数。转换过程涉及到位移操作、与运算和异或运算，以确保生成的数具有良好的随机性。

4. **状态更新**：当状态数组中的所有元素都已被使用时，算法会通过一系列复杂的运算来更新状态数组。这些运算包括将数组分成两部分，应用特定的线性变换，并重新组合以生成新的状态数组。

5. **周期性**：Mersenne Twister的周期非常长，达到了\(2^{19937}-1\)。这意味着在周期重复之前，它可以生成大量的伪随机数。

### 算法步骤
以下是Mersenne Twister算法的关键步骤：

1. **初始化**：使用种子初始化状态数组。
2. **状态更新**：当需要新随机数且状态数组用尽时，更新状态数组。
3. **提取随机数**：从状态数组中提取一个随机数，并进行转换以得到最终的伪随机数。

### 特点
- **高效率**：Mersenne Twister设计的初衷就是高效，因此在大多数编程语言中都得到了广泛实现。
- **长周期**：其周期长度极大，适用于需要大量随机数的应用。
- **均匀分布**：生成的伪随机数具有良好的均匀分布性质，适合于模拟和蒙特卡罗方法等应用。

生成伪随机数的过程

```c++
#define MERSENNE_STATE_M 397u // 变换的偏移量
#define MERSENNE_STATE_N 624u // 状态数组的大小

//将32位整数分割成高1位和低31位。
#define LMASK 0x7ffffffful //Most significant w-r bits
#define UMASK 0x80000000ul //Least significant r bits
```

```c++
typedef struct {
    unsigned long long seed_; //种子,起始数字
    int left_; //剩余随机数
    unsigned int next_; //下一个随机数的索引
    unsigned int state_[MERSENNE_STATE_N]; //随机数数组
    unsigned int MATRIX_A[2];
} mt19937_state;
```

初始化

```c++
void manual_seed(mt19937_state* state, unsigned int seed) {
    state->MATRIX_A[0] = 0x0u;
    state->MATRIX_A[1] = 0x9908b0df;
    state->state_[0] = seed & 0xffffffff; //seed低32位
    for (unsigned int j = 1; j < MERSENNE_STATE_N; j++) {
        state->state_[j] = 1812433253 * (state->state_[j - 1] ^ (state->state_[j - 1] >> 30)) + j;
        state->state_[j] &= 0xffffffff;
    }
    state->left_ = 1;
    state->next_ = 0;
}
```

更新状态

${\displaystyle x_{k}=x_{k+m}\oplus (({x_{k}}^{u}\mid {x_{k+1}}^{l})>>1) }$

```c++
void next_state(mt19937_state* state) {
    state->left_ = MERSENNE_STATE_N;
    state->next_ = 0;
    unsigned int y, j;
    //更新[0, N-M-1]的状态
    for (j = 0; j < MERSENNE_STATE_N - MERSENNE_STATE_M; j++) {
        y = (state->state_[j] & UMASK) | (state->state_[j + 1] & LMASK);
        state->state_[j] = state->state_[j + MERSENNE_STATE_M] ^ (y >> 1) ^ state->MATRIX_A[y & 0x1];
    }
    //更新[N-M, N-1]的状态(循坏更新)
    for (; j < MERSENNE_STATE_N - 1; j++) {
        y = (state->state_[j] & UMASK) | (state->state_[j + 1] & LMASK);
        state->state_[j] = state->state_[j + (MERSENNE_STATE_M - MERSENNE_STATE_N)] ^ (y >> 1) ^ state->MATRIX_A[y & 0x1];
    }
    //更新最后一个数 之前都是state->state_[j + 1], 这里要改为[0]
    y = (state->state_[MERSENNE_STATE_N - 1] & UMASK) | (state->state_[0] & LMASK);
    state->state_[MERSENNE_STATE_N - 1] = state->state_[MERSENNE_STATE_M - 1] ^ (y >> 1) ^ state->MATRIX_A[y & 0x1];
}
```

[What is Va_list in C? Exploring the Secrets of Ft_printf](https://hackernoon.com/what-is-va_list-in-c-exploring-the-secrets-of-ft_printf)

`va_list`是C函数接受可变数量参数的一种方式。它就像一种特殊的列表，保存着传递给函数的所有额外参数。

在使用`va_list`时，可以搭配以下几个函数使用

`va_start(args, format);` va_start是起始点。它初始化va_list以指向第一个
变量参数(format之后的第一个)

`va_arg(ap,type)` va_arg获取列表中的下一个参数。它将指针向前移动type类型的大小

`va_end` va_end与va_start对应。它清除与va_list相关的内存

