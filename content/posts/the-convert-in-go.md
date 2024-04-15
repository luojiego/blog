---
title: Go 中"坑爹"的类型转换
date: 2020-09-20 00:34:56
tags:
- go
---

## go 代码

```go
package main

import (
	"fmt"
	"math"
)

func main() {
	var count uint32
	fmt.Printf("count: %d\n", count)
	fmt.Printf("count - 1: %d\n", count - 1)
	// num := math.Pow(2, float64(count-1))
	fmt.Printf("math.Pow(2, float64(count-1)): %f\n", math.Pow(2, float64(count-1)))
	fmt.Printf("int(math.Pow(2, float64(count-1))): %d\n", int(math.Pow(2, float64(count-1))))
	fmt.Printf("uint32(math.Pow(2, float64(count-1))): %d\n", uint32(math.Pow(2, float64(count-1))))
	fmt.Printf("int32(math.Pow(2, float64(count-1))): %d\n", int32(math.Pow(2, float64(count-1))))
	fmt.Printf("uint64(math.Pow(2, float64(count-1))): %d\n", uint64(math.Pow(2, float64(count-1))))

	fmt.Printf("int64(pow(2,1023): %d\n", int64(math.Pow(2, 1023)))
	fmt.Printf("uint64(pow(2,1023): %d\n", uint64(math.Pow(2, 1023)))
}

```
<!--more-->

## 输出

测试环境：go version go1.14.4 darwin/amd64

```
count: 0
count - 1: 4294967295
math.Pow(2, float64(count-1)): +Inf
int(math.Pow(2, float64(count-1))): -9223372036854775808
uint32(math.Pow(2, float64(count-1))): 0
int32(math.Pow(2, float64(count-1))): -2147483648
uint64(math.Pow(2, float64(count-1))): 9223372036854775808
int64(pow(2,1023): -9223372036854775808
uint64(pow(2,1023): 9223372036854775808
```



## 坑点: +Inf 转换为 int 为 0

刚工作的时候，就开始使用了 protobuf 协议了，由于没有 int 类型，看到好多博客或者说明文档都用的是 uint32 类型，加上本身 C 和 C++ 都是支持无符号类型的，最后我也觉得有些数据压根就没有负数，所以我们的数据存储中大量使用了 uint32 类型，但是在实际的工作中，这个设定却一次次的让我头破血流。

首先我们写 go 代码的时候，在需要转换的时候，根本就不思考，直接就对 uint32 转换为了 int，float32, float64，大部分情况下，这个转换都是正确的。但是在一些地方会有灾难。

正如上图的代码，几乎所有稍微有经验的同鞋，都会看出来，这个溢出了。但是有些对这些基础掌握不清楚的同学，函数传参是 uint32, 但是并未对参数判断，就执行了 math.Pow(2, float64(count-1))，**由于函数返回还是 uint32，又对结果再次转换为了 uint32，这个时候 +Inf 可以正常的转换，转换的结果是 0.** 其实我对这个设定也不清楚，我甚至也不知道 pow(2, n)，这里 n 超过多少时，结果会越界。经过测试，结果是 1023，**超过 1023 之后**，结果将会越界。

刚开始的时候，我以为这是 go 的设定，但是在我写这个博客的时候，我突然想到，可能是我的错觉，只是我在 C 和 C++ 没有遇到过而已，果然不出我所料，C 的测试结果和 go 结果基本一致。

## C 代码

```c
#include <stdio.h>
#include <stdint.h>
#include <math.h>

int main() {
	uint32_t count = 0;
	printf("count: %d\n", count);
	printf("count - 1: %u\n", count-1);
	printf("pow(2, %d): %lf\n", count-1, pow(2, count-1));
	printf("(int32_t)pow(2, %d): %d\n", count-1, (int32_t)pow(2, count-1));
	printf("(uint32_t)pow(2, %d): %u\n", count-1, (uint32_t)pow(2, count-1));
	printf("(uint64_t)pow(2, %d): %llu\n", count-1, (uint64_t)pow(2, count-1));
	printf("(int64_t)pow(2,1023): %lld\n", (int64_t)pow(2,1023));
	printf("(uint64_t)pow(2,1023): %lld\n", (uint64_t)pow(2,1023));
}
```

## 输出

环境： 

Apple clang version 12.0.0 (clang-1200.0.31.1)

Target: x86_64-apple-darwin19.6.0

```
count: 0
count - 1: 4294967295
pow(2, -1): inf
(int32_t)pow(2, -1): -2147483648
(uint32_t)pow(2, -1): 0
(uint64_t)pow(2, -1): 0
(int64_t)pow(2,1023): -9223372036854775808
(uint64_t)pow(2,1023): 0
```

# 总结

其实我们以为是某种语言"坑"，大部分是我们自己用的不正确，或者理解的有误差，但是正是一次次的"打脸"，让我们对语言有了更深的了解。另外之前我也被**有符号到无符号的转换**坑过，我以为有符号和无符号操作之后是有符号数，但是 C++ 的 auto 把我"坑"的不要不要的。
