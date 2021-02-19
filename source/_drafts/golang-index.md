---
title: golang-index
category: 
  - index
tags:
  - program
  - code
toc: true
---

{% pullquote mindmap mindmap-md %}
- [the way to go](https://github.com/unknwon/the-way-to-go_ZH_CN)
  - go 基础
    - 环境变量(基础的变量含义)
    - go runtime
    - go 编译器
  - go 基本结构和数据类型
    - 包(package)
      - 编译产物 .a
      - 编译顺序（引用顺序、循环引用）
      - 包的可导出性
    - 函数 func
      - 函数定义
      - 函数返回值
    - 注释
      - go doc
      - package
      - func
      - 其他注释
    - 数据
      - 常量
      - 变量
        - 变量类型推断（编译时添加类型类型，可能会存在小坑 注意 int int32 int64 等的区别）(包括 :=)
        - 指针与指针指向的数值
    - 类型
      - 基本类型：int、float、bool、string
        - 与架构相关的: uint int （所以使用的时候 为了精度 最好使用 int32 等强制声明的）
        - a := uint64(0)
        - 复数
        - 字符类型(byte: uint8, rune: int32)
        - 没有三元运算符
      - 结构类型：struct、array、slice、map、channel
      - 类型行为：interface
      - 日期
        - time.time
      - 没有继承（可以组合）
      - 类型转换 (int() or XXX.(xxx))
  - 控制结构
    - if-else、switch
    - 多返回值
    - for (break、continue)
    - 标签与 goto （goto 最好不要乱用）
  - 函数
    - 闭包
  - 数组与切片
{% endpullquote %}

# go 基础

## 环境变量

> - $GOROOT 表示 Go 在你的电脑上的安装位置，它的值一般都是 $HOME/go，当然，你也可以安装在别的地方。
> - $GOARCH 表示目标机器的处理器架构，它的值可以是 386、amd64 或 arm。
> - $GOOS 表示目标机器的操作系统，它的值可以是 darwin、freebsd、linux 或 windows。
> - $GOBIN 表示编译器和链接器的安装位置，默认是 $GOROOT/bin，如果你使用的是 Go 1.0.3 及以后的版本，一般情况下你可以将它的值设置为空，Go 将会使用前面提到的默认值。

## go runtime

> go 的运行时类似于 java 的 jvm，其管理包括`内存分配`、`垃圾回收（第 10.8 节）`、`栈处理`、`goroutine`、`channel`、`切片（slice）`、`map`和`反射（reflection）`等等

- go 的 gc 采用的是`标记-清除`回收器

- 由于 go 没有 jvm 那样的虚拟机，所有每个编辑产物会带着一个`完整的 go runtime` 

# go 基本数据结构

> Go 语言虽然看起来不使用分号作为语句的结束，但实际上这一过程是由编译器自动完成




`bao的顺序（先根据引入的顺序 去引入包（每个包只会被引入一次） 然后再从最里面被引入的包开始执行 init 等函数）`



## 包的概念

- go 采用包来管理一组相关的代码。包在编译完成后会生成对应的 `.a` 文件

- go 编译顺序。A.go(import B.go) B.go(import C.go)。那么在编译的时候，会首先编译 C.go，然后编译 B.go，最后编译 A.go。`因此，golang 不能出现循环依赖`!

- 可见性。在 golang package 中向外暴露的方法和函数是通过 `大写的首个字母`, 如 func Abc 即为向外导出的 

## 函数

func 是 build-in 的关键字，仍然遵循`可导出性`的原则

> 只有当某个函数需要被外部包调用的时候才使用大写字母开头，并遵循 Pascal 命名法（第一个单词首字母和之后的单词首字母均大写）；否则就遵循骆驼命名法，即第一个单词的首字母小写，其余单词的首字母大写。

- func 定义，分为三部分，多返回值需要用 () 

```golang
// a 函数名 b、c 均为入参 返回值为 int
func a(b int, c string) int {}
```

- func 支持多返回值，且支持命名返回值（即可以返回在 func 内的局部变量名的值）

```golang
// a 中定义为命名返回值
func a() (b, c, d int) {}
```

## 注释

- package 注释。golang 采用 package 的方式管理 go 文件，只需要在包内的一个 go 文件声明该注释即可，如标准库的注释

```golang
// Copyright 2011 The Go Authors. All rights reserved.
// Use of this source code is governed by a BSD-style
// license that can be found in the LICENSE file.

package cgo
```

- func 注释。func 的注释需要加上方法名称

```golang
// siftDown implements the heap property on data[lo, hi).
// first is an offset into the array where the root of the heap lies.
func siftDown(data Interface, lo, hi, first int) {…………}
```

- 其他注释。对于导出的字段等均建议注释解释

## 数据

### 常量

- 常量使用 `const a [type] = 1` 的方式定义
- 对常量的赋值会compile error
{% asset_img const-val.png 报错 %}
- 常量一定是在`编译时`就能够确定的
- 可以使用`itoa`枚举

```golang
// 赋值一个常量时，之后没赋值的常量都会应用上一行的赋值表达式
const (
	a = iota  // a = 0
	b         // b = 1
	c         // c = 2
	d = 5     // d = 5   
	e         // e = 5
)
// 或者会重复第一个表达式的内容
const (
	_           = iota             // 使用 _ 忽略不需要的 iota
	KB = 1 << (10 * iota)          // 1 << (10*1)
	MB                             // 1 << (10*2)
	GB                             // 1 << (10*3)
	TB                             // 1 << (10*4)
	PB                             // 1 << (10*5)
	EB                             // 1 << (10*6)
	ZB                             // 1 << (10*7)
	YB                             // 1 << (10*8)
)
```

### 变量

- golang 变量统一使用 var 定义，形如 `var a [type]`
- golang 变量声明后都会被初始化 `所有的内存在 Go 中都是经过初始化的`
- 遵循可见性原则，即`首字母大写`
- 变量会在编译的时候`自动推断`**(注意这个地方有点小坑)**
- 变量存在指针
  - 不能获取字面量或常量的地址（字面量应该是作为一个数字直接背推到栈里面了 获取地址也没有意义）
- :=

leet code 137 题
> 在计算机中，数字都是用补码的形式表现的，即最高位作为符号位。一般来说 c、java 等 int 是4字节（32位）大小。而 golang 的 int 不是 int32 alias，所以会存在64位的问题。

**下面的代码 因为是 64 位 int 也就是说 一直是正数 所以出现问题**

```golang
func singleNumberTest(nums []int) int {
	res := 0
	// int ... 在 64 位机器上 int 为 64 位大小
	// 与答案的 int 32位不符 所以无法出现负数的情况
	fmt.Println(reflect.TypeOf(res))
	for i := 0; i < 32; i++ {
		count := 0
		for _, num := range nums {
			count += (num >> i) & 1
		}
		res += (count % 3) << 32
		res >>= 1
	}
	return res
}
```
指针类型 需要抓到他的 elem 再操作（很多库函数都有这个）
```golang
	res := 0
	var a *int
	a = &res
	t := reflect.TypeOf(a)
	b := reflect.ValueOf(a)
	fmt.Println(t.Elem(), b.Elem())
```

## 类型

### 基本类型

#### bool(无异，注意类型转换)
#### 数字类型

##### 数据类型

- int、uint 都是于架构相关的类型 int、uint 在32位机器上为4字节，在64位机器上为8字节
- 无符号整数
  - uint8
  - uint16
  - uint32
  - uint64
- 整数
  - int8
  - int16
  - int32
  - int64
- 浮点数
  - float32(小数点后7位)
  - float64(小数点后15位)
  - 浮点数的比较可能会存在精度问题
- 8进制、16进制
  - 添加前缀 0 来表示 8 进制数（如：077）
  - 添加前缀 0x 来表示 16 进制数（如：0xFF）
- 可以使用 a := uint64(0) 来同时完成类型转换和赋值操作，这样 a 的类型就是 uint64。
- 复数
  - complex64 (32 位实数和虚数)
  - complex128 (64 位实数和虚数)
  - real(c) 和 imag(c) 可以分别获得相应的实数和虚数部分
- 字符类型
  - byte (uint8，代表之前的 ASCII 码)
  - rune (int32，采用 unicode 编码的字符)
- string 类型 (可以看成是 byte 的 slice)
  - 获取字符串中某个字节的地址的行为是非法的，例如：&str[i] 
  - strings 包 封装了 replace、index 等常见的 string 操作
  - strconv 包封装了 Atoi、 Itoa 等常见的转换方法

##### 运算

- 二进制
  - 运算 & | ^ !
  - << 左移 >> 右移（只有无符号右移 左侧填充0）
- 逻辑运算符 == != < > <= >=
- 算数运算符 ++ -- / % （注意 不允许 a := b++ + c 出现）(因为`带有 ++ 和 -- 的只能作为语句，而非表达式`)
- 注意 golang 没有三元运算符

##### 时间与日期

参考以下的使用方法
```golang
package main
import (
	"fmt"
	"time"
)

var week time.Duration
func main() {
	t := time.Now()
	fmt.Println(t) // e.g. Wed Dec 21 09:52:14 +0100 RST 2011
	fmt.Printf("%02d.%02d.%4d\n", t.Day(), t.Month(), t.Year())
	// 21.12.2011
	t = time.Now().UTC()
	fmt.Println(t) // Wed Dec 21 08:52:14 +0000 UTC 2011
	fmt.Println(time.Now()) // Wed Dec 21 09:52:14 +0100 RST 2011
	// calculating times:
	week = 60 * 60 * 24 * 7 * 1e9 // must be in nanosec
	week_from_now := t.Add(time.Duration(week))
	fmt.Println(week_from_now) // Wed Dec 28 08:52:14 +0000 UTC 2011
	// formatting times:
	fmt.Println(t.Format(time.RFC822)) // 21 Dec 11 0852 UTC
	fmt.Println(t.Format(time.ANSIC)) // Wed Dec 21 08:56:34 2011
	// The time must be 2006-01-02 15:04:05
	fmt.Println(t.Format("02 Jan 2006 15:04")) // 21 Dec 2011 08:52
	s := t.Format("20060102")
  fmt.Println(t, "=>", s)
}
```

### 结构类型

### 自定义类型

#### 类型转换