---
title: jvm 笔记
category:
  - java
tags:
  - jvm
toc: true
---

## jvm 架构

- jvm 是栈式指令集，所以会在 虚拟机 栈中添加 操作数栈、对数据、局部变量表进行操作。因为为了跨平台，不用使用机器汇编的代码，与平台硬件无关。

## jvm 生命周期

1. 启动。启动是通过 bootstrap-classloader 加载初始类 (init class) 启动的.
2. 执行。启动一个 java 虚拟机进程，在程序结束时停止。
3. 停止。
   - 正常结束
   - 调用 runtime、system 的 exit 方法
   - 异常
   - JNI 的 API 停止

- hotspot 虚拟机.方法区实际上是 hotspot 的概念，只是在 1.7 之前是`永久代`，之后是`元空间`（直接内存）
- Dalvik 虚拟机。基于寄存器，支持 dex 文件

## jvm 结构 (以 hotspot 为例)

{% asset_img jvm_struct.png test %}

{% asset_img jvm_struct_detail.png test %}

类加载子系统

运行时数据区

- 栈
- 堆
- 方法区
- 程序计数器

## 类加载子系统

- 加载 class 文件
- 验证 class 文件
- 加载完毕后将 class content （DNA 元数据模板） 放入到方法区的内存空间中.(方法区中，还包含常量池的信息)

### 类加载过程

#### 类加载

类加载过程的第一步，主要完成下面 3 件事情：

1. 通过全类名获取定义此类的二进制字节流（寻找字节码 也就是 class 文件）
2. 将字节流所代表的静态存储结构转换为方法区的运行时数据结构
3. 在内存中生成一个代表该类的 Class 对象,作为方法区这些数据的访问入口

一个非数组类的加载阶段（加载阶段获取类的二进制字节流的动作）是可控性最强的阶段，这一步我们可以去完成还可以自定义类加载器去控制字节流的获取方式（重写一个类加载器的  loadClass()  方法）。**数组类型不通过类加载器创建**，它由 Java 虚拟机直接创建。（数组类型 如果是局部的 直接就开在栈上 如果是对象的 开在对象所属的堆上）
这个时候拿到的其实是 `class content` 被保存在 类加载器中，实际上的 class 对象是对 `class content 的 java 描述`，保存在方法区中

#### 连接

分为三步：验证，准备，解析。

##### 类验证

{% asset_img verify.png test %}

##### 准备

准备阶段是正式为类变量分配内存并设置`类变量初始值`的阶段，这些内存都将在方法区中分配。对于该阶段有以下几点需要注意：

1. 这时候进行内存分配的仅包括类变量（static），而不包括实例变量，实例变量会在对象实例化时随着对象一块分配在 Java 堆中。
2. 这里所设置的初始值"通常情况"下是数据类型默认的零值（如 0、0L、null、false 等），比如我们定义了 public static int value=111 ，那么 value 变量在准备阶段的初始值就是 0 而不是 111（初始化阶段才会赋值）。特殊情况：比如给 value 变量加上了 fianl 关键字 public static final int value=111 ，那么准备阶段 value 的值就被赋值为 111。

##### 解析

解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用限定符 7 类符号引用进行。
符号引用就是一组符号来描述目标，可以是任何字面量。直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。在程序实际运行时，只有符号引用是不够的，举个例子：在程序执行方法时，系统需要明确知道这个方法所在的位置。Java 虚拟机为每个类都准备了一张方法表来存放类中所有的方法。当需要调用一个类的方法的时候，只要知道这个方法在方发表中的偏移量就可以直接调用该方法了。通过解析操作符号引用就可以直接转变为目标方法在类中方法表的位置，从而使得方法可以被调用。
综上，解析阶段是虚拟机将常量池内的`符号引用替换为直接引用`的过程，也就是得到`类或者字段、方法在内存中的指针或者偏移量`。

#### 初始化

初始化是类加载的最后一步，也是真正执行类中定义的 Java 程序代码(字节码)，初始化阶段是执行类构造器  <clinit> ()方法的过程。`但是<Clinit>如果没有 static 的赋值或者代码块 不会有`。对于<clinit>（）  方法的调用，虚拟机会自己确保其在多线程环境中的安全性。因为  <clinit>（）  方法是带锁线程安全，所以在多线程环境下进行类初始化的话可能会引起死锁，并且这种死锁很难被发现。
对于初始化阶段，虚拟机严格规范了有且只有 5 种情况下，必须对类进行初始化(只有主动去使用类才会初始化类)：

1. 当遇到 new 、 getstatic、putstatic 或 invokestatic 这 4 条直接码指令时，比如 new 一个类，读取一个静态字段(未被 final 修饰)、或调用一个类的静态方法时。
   - 当 jvm 执行 new 指令时会初始化类。即当程序创建一个类的实例对象。
   - 当 jvm 执行 getstatic 指令时会初始化类。即程序访问类的静态变量(不是静态常量，常量会被加载到运行时常量池)。
   - 当 jvm 执行 putstatic 指令时会初始化类。即程序给类的静态变量赋值。
   - 当 jvm 执行 invokestatic 指令时会初始化类。即程序调用类的静态方法。
2. 使用  java.lang.reflect  包的方法对类进行反射调用时如 Class.forname("..."),newInstance()等等。 ，如果类没初始化，需要触发其初始化。
3. 初始化一个类，如果其父类还未初始化，则先触发该父类的初始化。
4. 当虚拟机启动时，用户需要定义一个要执行的主类 (包含 main 方法的那个类)，虚拟机会先初始化这个类。
   MethodHandle 和 VarHandle 可以看作是轻量级的反射调用机制，而要想使用这 2 个调用， 就必须先使用 findStaticVarHandle 来初始化要调用的类。

### 类加载器分类

- bootstrap classloader （jvm 提供）
- user-defined classloader （自定义类加载器，包括 ext classloader、application classloader）

为什么要有自定义类加载器

- 隔离类（因为一个类是有其标识符以及其 classloader 决定的）
- 修改加载方式
- 防止源码泄露
- 以及保护被其他恶意攻击（比如 基础类库都由 bootstrap 加载，就没办法重新 load 修改，保障安全）

双亲委派机制

- java 采用了按需加载
- 加载类 会被先被委派到父类进行加载，如果父类加载器加载失败，则会尝试自己加载
- 避免了 类 的重复加载
- 沙箱安全机制
  - 保障了核心的 java 基础类不会被修改

#### bootstrap classloader

- jvm 提供
- 加载核心类库 (java_home/jre/lib 的一系列的 jar 包、boot.class.path 下的内容)
- 没有 parent 类加载器
- 只加载 java、javax、sun 等开头的类

#### ext classloader

- Laucher 的内部类
- 加载 jre/lib/ext 子目录下的类库

#### app classloader

- Laucher 内部类
- 加载 classpath 路径下的类库
- 默认加载器，一般 java 类都是尤其加载

## 运行时数据区

包括以下几个部分

{% asset_img data_area.png test %}

- 程序计数器
- 虚拟机栈
- 本地方法栈
- 堆
- 方法区（hotspot 元空间 永久代）

### 进程&线程

一个 java 程序，运行在一个 jvm 进程中，对应一个 runtime。

这个进程里面的线程对应一个`内核线程`，其生命周期与其基本相同。

### 程序计数器(PC)

`c 语言中 PC 是一个指向了当前执行下一条命令地址的地方。`
因为实际上，程序在运行的时候会被编译成对应的底层汇编，再到机器码。对其反编译能够拿到对应的一行命令的地址。pc 就是对这个命令地址的保存。

为了能够在方法调用后返回到之前执行的地址，因此在栈帧中，会把返回地址（pc 里面的值）放入到调用者栈中进行保存。

与 c 语言类似

java 也会有类似的地址与指令的对应

javap 反编译出来的代码，code 属性中左侧的就是 指令地址，右侧的是指令

<pre>
0: bipush 10
2: istore_1
</pre>

使用 pc 寄存器的原因:

我认为，首先，所以的代码被编译后，实际上对应的都是一条一条的指令在流水线上运行

1. 不管是函数调用过程、内核调用过程或者是 cpu 被中断抢占等一系列情况，都需要知道在上个过程结束后下条指令的位置。
2. 在流水线上操作的时候，cpu 不一定在针对同一个线程执行，它关心的只是我要运行的要一条指令是什么，在哪个地方，pc 可以提供这样的抽象。

### 虚拟机栈

虚拟机栈式保存局部变量的地方，由一对`栈帧框定一个区域`

c 语言执行过程中，会保存以下的数据

- 在调用者栈。在调用者栈会计算 1. 传入函数的参数 2. 返回地址（pc 寄存器指向的下个指令地址 在 call 命令被执行后压入调用者栈）
- 在被调用者，会保存 帧指针 %ebp
- 调用 ret 指令，会返回，通过保存的 %ebp 恢复原来的栈帧
- 同时可以使用 %eax 等寄存器保存返回值

java 栈帧里面会保存

- 局部变量表。
  - 主要存储局部变量，包括引用、基本数据类型以及 returnAddress。其大小在`编译时呗确认下来`(在写 pl0 编译器的时候，实际上也用了局部变量表，在编译的时候把对应的参数存储)，保存在 `code 属性`中。
  - 局部变量表存放的基本单位是 `slot` (一个 32 位的大小，64 位的需要两个 slot 存储)
  - 局部表量表，从属于类实例的方法，会在局部变量表第一个 index 的地方存储 this 指针，this 指针，在 invokeSpecial 调用构造函数的时候就已经赋值了。
  - 局部变量必须要进行显示的复制（没办法 invoke 一个特殊的初始化函数）
- 操作数栈
- 动态链接(运行时常量池的方法引用)
- 方法返回地址（对应上述的返回地址）
- 一些附加信息

!!! 对方法的理解 !!!

`class 文件中会有对应 method 区存储对应的 method，其中 name（方法名）、descriptor（方法描述符）、方法参数会存储到其中，然后还会有一个 attributes 对应各种属性，其中最重要的是 code 属性，对应 code 表的一个项，存储编译后的方法体，即字节码指令。`

- stackoverflow。采用固定大小的虚拟机栈，可能分配的局部变量表、操作数栈过多，stackoverflow 了
- oom。采用动态扩展的，在扩展时没有足够的内存就会 OOM。

#### 局部变量表

局部变量表是以 slot 为单位存储数据的，每个 slot 的大小为 32 位长度，因此对于 long、double 类型的数据需要两个 slot 来存储数据。

其中会存储 reference 类型的数据，作为对堆数据的引用，需要满足:

- 能够根据引用找到堆中存储数据的地址
- 能够根据引用找到方法区存储的类元数据信息

注意：在局部变量中必须同时对变量进行初始化！因为 `int a = 0` 这样的句子，实际上会编译成多条 jvm 指令，其中有声明、赋值等。如果不赋初值，会造成错误。

#### 操作数栈

- 字节码指令的操作过程中局部数据的操作空间
- 同样采用类似 slot 的大小来存储
- 栈顶缓存技术：由于操作数栈放在主存里面，为了效率，会把栈顶的热点数据放到寄存器中，加快

#### 动态链接

将常量池中的符号引用转换为直接引用。

- 静态链接。在编译时就已经确定了，对应`早期绑定`（在类加载时，就已经知道其运行的地址了，对应的是静态分派，比如说`类的 static 方法，通过 invokeStatic 指令调用`或者`私有的方法`）
- 动态链接。在运行时确定的，对应晚期绑定，实际运行中找到的。（动态分派，根据实际类型寻找虚方法 invokeVirtual）

- 为了支持 invokeVirtual 的多态
- 方法在编译后都会将其 `方法描述符` 存入到 静态常量池 中

##### 方法的解析和分派

###### 解析

方法的调用解析放在是在编译器阶段，他会将对方法的调用翻译成以下四种类型的 jvm 指令调用。

> invokeDynamic <br/>
> invokeVirtual <br/>
> invokeSpecial <br/>
> invokeStatic  <br/>
> invokeInterface

###### 分派

方法的分配是发生在运行时。

- 静态分派

```java
public static class A {}

public static class B extends A {}

public static void test(A a) {
    System.out.println("A");
}

public static void test(B b) {
    System.out.println("B");
}

public static void main(String[] args) {
    A a = new A();
    A b = new B();

    test(a);
    test(b);
}
```

方法均会调用 `test(A a)`，因为这儿是静态的分派，根据方法的定义类型确定，在编译时就已经确定了调用的方法，其反编译的 class 文件如下。

```
// 初始化两个 对象
164 invokespecial #74 <SubarraysWithKDistinctNew_992$A.<init>>
167 astore_1
168 new #75 <SubarraysWithKDistinctNew_992$B>
171 dup
172 invokespecial #77 <SubarraysWithKDistinctNew_992$B.<init>>
175 astore_2
176 aload_1
// 可以看到两个对象调用的都是同一个 常量池 的引用
// 即 test(LSubarraysWithKDistinctNew_992$A;)V
// 调用的 test(A a) 方法
177 invokestatic #78 <SubarraysWithKDistinctNew_992.test>
180 aload_2
181 invokestatic #78 <SubarraysWithKDistinctNew_992.test>
184 return
```

- 动态分派

根据运行时的实际对象类型，调用其上的实际方法。

```java
public static class A {
    public void say() {
        System.out.println("A");
    }
}

public static class B extends A {
    @Override
    public void say() {
        System.out.println("B");
    }
}

public static void main(String[] args) {

    A a = new A();
    A b = new B();

    a.say();
    b.say();
}
```

```
164 invokespecial #74 <SubarraysWithKDistinctNew_992$A.<init>>
167 astore_1
168 new #75 <SubarraysWithKDistinctNew_992$B>
171 dup
172 invokespecial #77 <SubarraysWithKDistinctNew_992$B.<init>>
175 astore_2
176 aload_1
// invokeVirtual 的查找是不一样的
177 invokevirtual #78 <SubarraysWithKDistinctNew_992$A.say>
180 aload_2
181 invokevirtual #78 <SubarraysWithKDistinctNew_992$A.say>
184 return
```

1. 找到`接受者`的实际类型
2. 在实际类型中找到方法描述符相同的方法，检查器是否可以访问
3. 如果没有找到，则从下向上开始寻找方法，并进行验证
4. 如果均没有找到，就返回错误

同时为了实现动态的分配查找过程，使用了`虚方法表`（在方法区中）来优化查找性能，对于 `虚的方法` 在方发表中会替换成实际类型调用的方法。

#### 返回地址

一般来说，pc 指向了下一个指令运行的地址，那么在函数调用完成之后，返回调用者函数执行的时候，就需要还原包括 pc 寄存器在内的各种信息。所以一般会将这个作为返回地址，在退出堆栈的时候，以此还原 pc 指针，执行下面的指令操作。
