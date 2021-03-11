---
title: java多线程(一)
category: code
tags:
  - java
  - concurrent
toc: true
date: 2019-12-10 14:18:40
thumbnail:
---


一直以来，对于java的多线程变成都不是十分了解，趁着项目需要一个多线程的爬虫学习了一下，主要是一些基础的相关知识，以上。


<!-- more -->

## Runnable和Thread

踩的第一个坑是关于实现Runnable接口和继承Thread类来build一个新的可运行线程。

```java
// Runnable接口定义
public class MyRunnable implements Runnable {
  private int count = 5;
  @Override
  public void run() {
    System.out.println(this.count--);
  }
}
// Thread类定义
public class MyThread extends Thread {
  private int count = 5;
  @Override
  public void run() {
    System.out.println(this.count--);
  }
}
```

```java
// 启动
public static void main(String[] args) {
  while (int i = 0; i < 5; i++) {
    new MyRunnable().run();
    // new MyThread().start(); 
  }
}
```

在main中分别启动两个类的5个线程，可以看到Runnable接口并没有实际上的并发的启动几个线程，而是在第一个线程执行完毕之后，再去执行第二个线程，阻塞式地执行完new出来的5个线程，究其原因，是因为Runnable接口实际上`并不是实际的Thread的入口`，Runnable接口只是定义了Thread的`任务逻辑`，也就是说Runnable接口中的run()方法只是实现了现在需要的业务逻辑，实际的多线程仍然需要通过Thread类开始。

## 多线程的执行

在实际的线程执行的时候，较为传统的做法是通过`new Thread(new Runnable()).start()`来执行，然而新建线程实际上也是一个较为繁重的操作，以下是反编译的Thread类代码

```java
    public synchronized void start() {
        if (this.threadStatus != 0) {
            throw new IllegalThreadStateException();
        } else {
            this.group.add(this);
            boolean started = false;

            try {
                this.start0();
                started = true;
            } finally {
                try {
                    if (!started) {
                        this.group.threadStartFailed(this);
                    }
                } catch (Throwable var8) {
                }

            }

        }
    }

    private native void start0();
```

以上可以看出，Thread类的start()方法，最终调用了native的start0()，最终通过JNI(java native interface)调用底层提供的pthread_create方法，最终进入linux系统提供的创建线程的接口。

在javase 1.5提供了一个Concurrent的包来提供方便运行的多线程类库，如提供运行的ExecutorService，提供线程安全的各类Blocking Queue、Blocking List、Concurrent List等，以及一些原子操作类库。

其主要通过ExecutorService来启动需要执行的线程

```java
// 创建无限大的线程池，当任务到来的时候
ExecutorService e1 = Executors.newCachedThreadPool();
// 创建指定pool size的线程池
ExecutorService e1 = Executors.newCachedThreadPool(int poolSize);
// 创建一个指定pool size的定时线程池
ExecutorService e3 = Executors.newScheduledThreadPool(int poolSize);
// 创一个执行单个线程的
ExecutorService e4 = Executors.newSingleThreadExecutor();

// 通过submit方法可以将需要执行的Runnable Callable接口实现类启动
e1.submit(() -> {
  // do something
})
```

任务通过ExecutorService提交到指定的线程池，或者执行线程中`异步`的执行，即submit函数不会阻塞主线程的执行，最终子任务的逻辑会异步的在新线程中执行

但是如果主线程依赖子线程的运行结果，在submit方法执行后，返回了一个Future对象，Future对象可以在主线程中控制子线程

```java
ExecutorService ex = Executors.newCachedThreadPool();
Future f = ex.submit(() -> {
  try {
    Thread.sleep(10000)
  } catch (Exception e) {
    e.printStackTrace();
  }
})

// 获取子线程的运行结果
f.get()

// 在1s之后get没有返回执行结果 即没有执行完毕 则出发timeout 可以通过这种方式来对一些任务做超时处理
try {
  f.get(1000, TimeUnit.MilliSeconds)
} catch(TimeoutException | InterruptedException | ExecutionException e) {
  e.printStackTrace();
}

```