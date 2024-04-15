---
title: 除 0 真的会 panic 吗？
date: 2021-10-16 07:47:20
tags:
- Go
- Rust
- C
- C++
- Python
---

某天，小伙伴反馈说程序 panic 在一个比较诡异的地方，差不多是下面的代码。

```go
type Point struct {
		X float32
		Y float32
}

func Get(p Point) {
		// import github.com/shopspring/decimal
		x, _ := decimal.NewFromFloat32(p.X).Round(7).Float64()
}
```
<!--more-->

查看 decimal 的源码，下面是其注释，发现**若 value 为 NaN 或者 +/- inf 时，会 panic**。

```go
// NewFromFloat32 converts a float32 to Decimal.
//
// The converted number will contain the number of significant digits that can be
// represented in a float with reliable roundtrip.
// This is typically 6-8 digits depending on the input.
// See https://www.exploringbinary.com/decimal-precision-of-binary-floating-point-numbers/ for more information.
//
// For slightly faster conversion, use NewFromFloatWithExponent where you can specify the precision in absolute terms.
//
// NOTE: this will panic on NaN, +/-inf
func NewFromFloat32(value float32) Decimal {
	if value == 0 {
		return New(0, 0)
	}
	// XOR is workaround for https://github.com/golang/go/issues/26285
	a := math.Float32bits(value) ^ 0x80808080
	return newFromFloat(float64(value), uint64(a)^0x80808080, &float32info)
}
```

显然我们不可能传递一个非法的 `p`，在我们的认知里面，X 不应该有 NaN 这种数值，即使 p 没有初始化，那也应该不是 **NaN**。

最终分析的结果是，X 有一个赋值操作，出现 `p.X = number / float32(a)`  ，而 a 有可能是 0，因为在我的认知里面，除法除 0 就会 panic，而不可能到调用 **`NewFromFloat32`** 时才出问题，最后发现一个惊人的事实：**只有整数除 0 才会 panic，除以 0.0 并不会引发 panic，而会得到 NaN/+Inf/-Inf**。 

这让我突然有想研究各个语言在在除 0 以及 0.0 的想法了。

关于浮点数除 0 为什么不会报错，此处是相关的[链接](https://stackoverflow.com/questions/12954193/why-does-division-by-zero-with-floating-point-or-double-precision-numbers-not)。

## Rust

浮点数相除的结果是**注释掉编译错误**之后得到的结果。

`rustc 1.55.0 (c8dfcfe04 2021-09-06)`

```rust
fn main() {
    let a = 0;
    let b = 0;
    println!("{}", a/b); // attempt to divide `0_i32` by zero
    let a = 3;
    let b = 3;
    println!("{}", 9/(a-b)); // attempt to divide `9_i32` by zero
    println!("{}", 0.0f64/0f64); // NaN
    println!("{}", 0.1f64/0f64); // inf
    println!("{}", -0.1f64/0f64); // -inf
    println!("{}", 0.0f32/0f32); // NaN
    println!("{}", 0.1f32/0f32); // inf
    println!("{}", -0.1f32/0f32); // -inf
}
```

## Go

`go version go1.17 windows/amd64`

```go
package main

func main() {
	// var a int32 = 3
	// var b int32 = 3
	// var c int32 = 9
	// println(c / (a - b)) // 运行出错 panic: runtime error: integer divide by zero
	// println(c / 0)       // 编译出错 division by zero
	// println(float64(0) / float64(0)) // 编译出错 division by zero
	var a float64 = 0.0
	println(float64(0) / a)    // NaN
	println(float64(0.1) / a)  // +Inf
	println(float64(-0.1) / a) // -Inf
	var b float32 = 0.0
	println(float32(0) / b)    // NaN
	println(float32(0.1) / b)  // +Inf
	println(float32(-0.1) / b) // -Inf
}
```

## C

`gcc version 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC)`

```c
#include <stdio.h>

int main() {
	// printf("%d\n", 0/0); // 编译警告 warning: division by zero [-Wdiv-by-zero]
	int a = 0;
	int b = 0;
	// printf("%d\n", a/b); // 运行出错 Floating point exception
	printf("%f\n", (float)0/(float)0); // -nan
	printf("%f\n", (float)0.1/(float)0); // +inf
	printf("%f\n", (float)-0.1/(float)0); // -inf
	printf("%f\n", (double)0/(double)0); // -nan
	printf("%f\n", (double)0.1/(double)0); // +inf
	printf("%f\n", (double)-0.1/(double)0); // -inf
}
```

## C++

`gcc version 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC)`

```cpp
#include <iostream>
using namespace std;

int main() {
	// cout << 0 / 0 << endl; // 编译警告 warning: division by zero [-Wdiv-by-zero] 运行报错：Floating point exception
	int ia = 0;
	int ib = 0;
	// cout << ia / ib << endl; // 运行出错：Floating point exception
	cout << float(0) / float(0) << endl; // -nan
	cout << float(0.1) / float(0) << endl; // +inf
	cout << float(-0.1) / float(0) << endl; // -inf
	cout << double(0) / double(0) << endl; // -nan
	cout << double(0.1) / double(0) << endl; // +inf
	cout << double(-0.1) / double(0) << endl; // -inf
}
```

## Python

### python 3.9

```bash
Python 3.9.2 (tags/v3.9.2:1a79785, Feb 19 2021, 13:44:55) [MSC v.1928 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> 0/0
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ZeroDivisionError: division by zero
>>> 0.1/0.0
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ZeroDivisionError: float division by zero
>>> 0.0/0.0
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ZeroDivisionError: float division by zero
>>> -0.1/0.0
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ZeroDivisionError: float division by zero
```

### python 2.7

```bash
[GCC 4.8.5 20150623 (Red Hat 4.8.5-39)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> 0/0
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ZeroDivisionError: integer division or modulo by zero
>>> 0.1/0.0
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ZeroDivisionError: float division by zero
>>> 0.0/0.0
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ZeroDivisionError: float division by zero
>>> -0.1/0.0
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ZeroDivisionError: float division by zero
```

## 结论

**Python** 的表现最好，无论是整数除 0 还是浮点除 0 都返回的错误：**ZeroDivisionError**。但跟编译型语言无法比较，因为无论如何，运行到的时候都挂了。

编译报错的情况：**Rust 整数除 0**，不论是**简直还是直接**，**代码均编译不过**，Go 直接整数除 0 会编译不通过。但是直接除浮点数 0 时，不会报编译问题，但是 Go 能正常识别，编译不过。此处 Rust 和 Go 几乎五五开。

C、C++ 直接除 0 编译器都会给出警告，但是可以运行，运行的时候报错。
