# SIMD_SM4

个人完成。

# 实现原理

单指令流多数据流 SIMD 是一种采用一个控制器来控制多个处理器，它能够以同步的方式复制多个操作数，将他们打包在大型寄存器中并在同一时间执行同一条指令。在本次实验中运用了 Intel 的 AVX 指令集实现，利用 Microsoft Visual C++ 提供
的支持 SIMD 指令集的 intrinsics 内联函数，使用 immintrin.h 库实现。实验中需要注意的是 SIMD 指令集的共用体类型通常为 128 位或 256 位大小，而对于 SM4 加密，对于一个 128bit 的明文串，每次只进行 32bit 加密，而且查 S 盒时将 32bit
分成 4 组 8bit 进行查表，这样就只存在 32bit 的异或，但是我们为了使用 SIMD，需要将异或等操作扩充成 128（256）bit。因此利用 AVX 指令集，对 SM4 算法进行等价代换。


在多线程优化中，应用了 pthread 库与 OpenMP 库进行多线程的实现与时间对比，OpenMP 是一种用于共享内存并行系统的多线程程序设计方案，采用 fork-hoin 执行
模式，开始的时候只存在一个主线程，当需要进行并行计算的时候，派生出若干个分支线程来执行并行任务。当并行代码执行完成之后，分支线程会合，并把控制流程交给单
独的主线程。

pthread 全称为 POSIX THREAD，按照 POSIX 对线程的标准而设计的，书写上相对麻烦一些。openMP 不同于 pthread 的地方是，它不会将软件锁定在事先设定的线程数量
中，但是相对的查错更难也更麻烦。它更偏向于将原来串行化的程序，通过加入一些适当的编译器指令变成并行执行，从而提高代码运行的速率。

在使用 pthread 库实现多线程的时候，对于线程任务的分配我们总共采取两种策略，第一种策略是将每个线程分配相邻的一组数，第二种策略是将索引模 num 同余的元素
分配到一个线程上计算，考虑到多线程会共用缓存，如果多个线程依次访问在内存中相隔很远的数据时，空间局部性较差，后者的分配方式使得多个线程访问的数据地址十分
接近进而提高 cache 的命中率。

![image](https://user-images.githubusercontent.com/105580300/181919658-6616c46b-8990-4c65-83a3-a9334b6dd682.png)

为了通过 SIMD 指令集实现 SM4 算法，需要使用 SIMD 指令集中大小为 128
位或者 256 位的共用体，但是对于 SM4 加密来说，对于一个 128bit 的明文串，每次只
进行 32bit 加密，而且查 S 盒时将 32bit 分成 4 组 8bit 进行查表，这样就只存在 32bit 的
异或，但是为了使用 SIMD，需要将异或等操作扩充成 128（256）bit。因此想
到了利用 AVX 指令集，对 SM4 算法进行等价代换。

# 执行结果

![image](https://user-images.githubusercontent.com/105580300/181920149-11597d24-c205-4033-bb9c-e288b4ad4b44.png)
