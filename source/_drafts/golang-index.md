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
    - 基本结构
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
      - 类型
        - 基本类型：int、float、bool、string
        - 结构类型：struct、array、slice、map、channel
        - 类型行为：interface
        - 没有继承（可以组合）
        - 类型转换 (int() or XXX.(xxx))
      - 数据
        - 常量
        - 变量
    - 基本类型
  - 控制结构
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