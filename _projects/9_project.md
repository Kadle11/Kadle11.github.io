---
layout: page
title: "Speeding up Matrix Multiplication" 
description: Matrix Multiplication using Intel's CPU+FPGA platform.
img: assets/img/MatMul.jpg
langs: [cpp, sh]
importance: 4
category: Research
---

## Motivation

Matrix multiplication _(MatMul)_ is an extremely common and important operation. It is used abundantly in fields like ML and Probability Modelling. One of the most commonly used libraries to perform MatMul is [BLAS (Basic Linear Algebra Subroutines)](https://www.netlib.org/blas/) which implements this functionality in C and Fortran. Most platforms like [Intel's MKL (Math Kernel Library)](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-programming-guide/top/api-based-programming/intel-oneapi-math-kernel-library-onemkl.html) use an optimized version of BLAS for MatMul. FPGAs can perform matrix multiplication very efficiently, with one caveat - The design permits multiplication of matrices of fixed sizes. The goal of this project was to split the work between the optimized MKL Library and the FPGA to speedup MatMul.

## The algorithm

I was tasked with developing a low latency program that split matrices of arbitrary sizes into matrcies of fixed sizes that could be fed to the FPGA. Once the FPGA performed the MatMuls, the program would then have to retrieve the results and perform specific matrix additions to obtain the final product. The algorithm I developed invloved 5 major operations -

1. Break Matrix into Smaller Chunks of Size ‘M x N’. (Required by the FPGA)

2. Pad the Chunks if Necessary and send the chunks to the FPGA for multiplication.

3. Retrieve the sub-products and perform the required additions.

4. Coalescing the Chunks to obtain the padded product.

5. De-Padding the padded result to obtain the final product.

The program developed used 4 threads so that operations could be performed in parallel to minimize the time the FPGA was idle. The whole program was able to perform MatMul at a sustained throughput of 1 TFLOP without the help of the MKL library. 

## Fooling the Linker - The _LD_PRELOAD_ Trick

To put the program to the test we decided to use the [Kaldi ASR](https://kaldi-asr.org/doc/) toolkit, which could be configured to use MKL. Our program's imterface was mimicking MKL's SGEMM (Single Precision General Matrix Multiplication) so that any framework could be retrofitted to use our code instead of MKL. The challenge here was to make the binary call our SGEMM instead of MKL's SGEMM. To do this we used the LD_PRELOAD environment variable. This variable can be used to alter the resolution of symbols at runtime. First we inspected MKL's shared object _(.so)_ to find the symbol corresponding to MKL's SGEMM. We then built our own shared object and made sure that our faux SGEMM resloved to the same symbol. Then, LD_PRELOAD was appended with the path to our library. Everytime, the SGEMM symbol was referenced, it resolved to our preloaded function as that was the first one it resolved to and our SGEMM was called. All the other MKL functionalities were from the actual MKL library so Kaldi worked like a charm.


## When a microsecond feels like a lifetime 

I remember profiling code using [Valgrind](https://valgrind.org/) and [gProf](https://ftp.gnu.org/old-gnu/Manuals/gprof-2.9.1/html_mono/gprof.html) trying to find bottlenecks in my code. Fun fact, Valgrind uses [LD_PRELOAD](https://valgrind.org/docs/manual/mc-tech-docs.html#mc-tech-docs.overview) too! While I was profiling my code, I saw many opportunities to overlap operations. For example, A new set of chunks could be created while sending the previous set to the FPGA. I created 4 threads, each taking advantage of a possible overlap and this saved a significant amount of time. 4096x4096 MatMuls showed improvement in the order of 10s of seconds and I was thrilled. Then came the hard part - The next set of inefficiencies I needed to find were more subtle and did not show as much improvement. Everytime I removed an extra initialization, changed a post-increment to a pre-increment, I saw improvement in the order of microseconds. These improvements were neccessary to reach the 1 TFLOP number and I had an amazing time finding these little tweaks.

