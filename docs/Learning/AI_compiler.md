首先介绍编译流程， 以GCC为例

**预处理**

```shell
gcc -E test.c -o test.i
```

包含宏定义替换，文件包含，条件编译，删除注释和空白字符串， 生成`.i`文件

**编译**

```shell
gcc -S test.i -o test.s
```

得到汇编文件

**汇编**

```shell
gcc -c test.s -o test.o
```

**链接**

```shell
gcc test.o -o test
```

编译器基本构成

前端->优化->后端

