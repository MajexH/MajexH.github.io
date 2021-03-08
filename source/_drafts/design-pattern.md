---
title: design-pattern
category:
 - code
tags:
 - 设计模式
toc: true
---

# 观察者模式

## 观察者模式简单实现

只需要让 subject 在更新的时候 通知 观察者，基本结构如下

{% asset_img observer.png 报错 %}

```java
// 观察者
public interface Observer {

    // 观察者的更新方法
    void update();
}

// 需要被观察的主题
public interface Subject {

    void registerObserver(Observer ob);

    void unregisterObserver(Observer ob);

    void notifyAllObserver();
}

public class SubjectImpl implements Subject {

    List<Observer> obs;

    public SubjectImpl() {
        this.obs = new ArrayList<>();
    }

    @Override
    public void registerObserver(Observer ob) {
        for (Observer registered : obs) {
            if (ob == registered) return;
        }
        this.obs.add(ob);
    }

    @Override
    public void unregisterObserver(Observer ob) {
        this.obs.remove(ob);
    }

    // 在更改某个数值 或者需要通知的时候 可以将结果通知给注册的 observer
    public void setXXX() {
        this.notifyAllObserver();
    }

    @Override
    public void notifyAllObserver() {
        for (Observer registered : obs) {
            registered.update();
        }
    }
}

public class ObserverImpl implements Observer {

    @Override
    public void update() {
        System.out.println("update");
    }
}

public class Main {

    public static void main(String[] args) {
        Subject s = new SubjectImpl();
        for (int i = 0; i < 10; i++) {
            s.registerObserver(new ObserverImpl());
        }
        s.notifyAllObserver();
    }
}

```

## java 内置的观察者模式

Observable (Subject) 类与 Observor 接口

> This class and the Observer interface have been deprecated. The event model supported by Observer and Observable is quite limited, the order of notifications delivered by Observable is unspecified, and state changes are not in one-for-one correspondence with notifications. For a richer event model, consider using the java.beans package. For reliable and ordered messaging among threads, consider using one of the concurrent data structures in the java.util.concurrent package. For reactive streams style programming, see the java.util.concurrent.Flow API.
> 上述两个类 已经在 JDK9 中被废弃。愿意是因为其无法控制通知的顺序(因为调用的父类是notify方法，而且是从后向前遍历 observer 的)，而且由于 Observable 是一个类形式提供，难以扩展。

```java
public class ObserverImpl implements Observer {

    @Override
    public void update() {
        System.out.println("update");
    }
}

public class SubjectImplUseJDK extends Observable {

    public Object msg;

    public void setMsg(Object msg) {
        this.msg = msg;
        // 1. 要先设置更改
        setChanged();
        // 2. 被动的传递消息
        notifyObservers(msg);
    }
}

public class Main {

    public static void main(String[] args) {
        SubjectImplUseJDK b = new SubjectImplUseJDK();
        for (int i = 0; i < 10; i++) {
            b.addObserver(new ObserverImplUseJDK());
        }
        b.setMsg("test");
    }
}
```

# 装饰器

装饰器可以方便的扩展原来的类不具备的方法。

符合 开闭原则 （对扩展开放，对修改关闭）

## 简单例子

通过子类持有基类的引用，通过构造器对原始的对象进行运行时增强。

```java
public abstract class Base {

    // 基类提供抽象方法
    public abstract void base();
}

public class OneImpl extends Base {
    // 持有 Base 类型的属性
    private Base base;

    public OneImpl(Base base) {
        this.base = base;
    }

    @Override
    public void base() {
        // 对原始的 base 做一个增强
        base.base();
        System.out.println("one");
    }
}

// 还可以用 abstract 进一步加强
public abstract class TwoAbstract extends Base {

    public abstract void anotherAbstract();
}

public class TwoImpl extends TwoAbstract {

    private Base base;

    public TwoImpl(Base base) {
        this.base = base;
    }

    @Override
    public void base() {
        base.base();
        System.out.println("Two");
    }

    @Override
    public void anotherAbstract() {
        System.out.println("another");
    }
}
```

## java 中的例子

java i/o 类

`InputStream` 及其子类，可以使用装饰器增强方法

一个 new BufferedInputStream(new FileInputStream(file)); 
的调用流程分析如下

{% asset_img javaIO-example.png 报错 %}

```java
// inputStream 的基类
public abstract class InputStream implements Closeable {
    // 子类实现其方法
    // 返回下一个字符
    public abstract int read() throws IOException;

    // 基类提供的共有方法 读取 off + len 的 byte 到 byte中
    public int read(byte b[], int off, int len) throws IOException {
        Objects.checkFromIndexSize(off, len, b.length);
        if (len == 0) {
            return 0;
        }

        int c = read();
        if (c == -1) {
            return -1;
        }
        b[off] = (byte)c;

        int i = 1;
        try {
            for (; i < len ; i++) {
                // 关键在这儿进行了动态绑定 实现了增强
                c = read();
                if (c == -1) {
                    break;
                }
                b[off + i] = (byte)c;
            }
        } catch (IOException ee) {
        }
        return i;
    }
}

public class BufferedInputStream extends FilterInputStream {
    // 继承的 FilterInputStream 中包含了一个 inputStream 的实例引用
    public BufferedInputStream(InputStream in) {
        this(in, DEFAULT_BUFFER_SIZE);
    }

    // 实现的 read 方法
    public synchronized int read() throws IOException {
        // 读取的位置上的数字超过了 buffer 中已经保存的
        if (pos >= count) {
            // 调用 read 方法进行读取
            fill();
            if (pos >= count)
                return -1;
        }
        return getBufIfOpen()[pos++] & 0xff;
    }

    private void fill() throws IOException {

        /**
        * …… 读取已经设置游标等方法
        **/
        // 读取 buffer 完毕
        count = pos;
        // 调用持有的 inputStream 的引用进行 read 
        // 关键在这儿进行了增强 会在读取后 赋值 到 buffer 中
        int n = getInIfOpen().read(buffer, pos, buffer.length - pos);
        if (n > 0)
            count = n + pos;
    }
}

public class FileInputStream extends InputStream {
    public int read() throws IOException {
        return read0();
    }

    private native int read0() throws IOException;
}

```

# 工厂模式

抽象对象的新建过程

## 简单工厂

{% asset_img simple-factory.png 报错 %}

简单工厂可以看做是对 new Object() 的简单抽象

## 工厂方法

{% asset_img abstract-method.png 报错 %}

上面是一个简单的例子，子类将会实现 abstract 的抽象工厂方法，将创建细节在子类中实现，提供一个`抽象的工厂方法`。

{% asset_img factory-method.png 报错 %}

进一步，可以将其分为两类

- Object （产品对应的接口）
  - concreateObject(对应的具体产品实现类)
- abstract creator(抽象的工厂方法)
  - creatorImpl(实现对 concreateObject 的初始化)

## 抽象工厂

### 依赖倒置原则

依赖抽象，不能依赖具体类。

因此，抽象工厂针对产品，也是对产品的抽象进行维护和管理，不能对具体产品进行管理，所以需要抽象一下两个部分

- 抽象工厂
- 抽象产品（具体的产品依赖这个 base 的抽象产品）

### 实现

{% asset_img abstract-factory.png 报错 %}

针对产品和工厂进行抽象 甚至可以嵌套抽象工厂 进一步抽象 product 的初始化流程。

这样 client 只需要针对参数等情况 分发到不同的 factory 即可

# 单例模式

全局共享一个变量 

java 的单例模式 

1. 懒汉式

由于 类加载 是线程安全的，所知直接放到 <clinit> 中初始化

```java
public class LazySingleton {
  private static final Object singleton = new Object();
  // 保证不被初始化
  private LazySingleton(){}

  public static Object getSingleton() {
    return singleton;
  }

}
```

2. 饿汉式

需要的时候再创建，为了保障多线程安全，创建的时候加锁

```java
public class HungrySingleton {
  private static final Object singleton;
  // 保证不被初始化
  private HungrySingleton(){}

  public static Object getSingleton() {
    synchronized (HungrySingleton.class) {
        if (singleton == null) {
            singleton = new NeedSingletonClass();
        }
        return singleton;
    }
  }
}
```

3. 双重校验锁

双重校验锁，初始化的时候使用 synchronized 保障线程安全，同时使用 volatile 保障指令不被重排序。

```java

public class DoubleCheckSingleton {

  // 防止指令重排序 因为 Object 对象生成有多个步骤，为了保障
  // 其他线程能够使用该完整对象
  private static volatile Object singleton;

  public static Object getSingleton() {
    // 防止多个线程堵塞 提高多线程性能
    if (singleton == null) {
      synchronized(DoubleCheckSingleton.class) {
        // 在多个线程堵塞在时，防止 singleton 被重复初始化
        if (singleton == null) {
          singleton = new Object();
        }
      }
    }
    return singleton;
  }
}

```

4. 静态内部类

```java
// 使用静态内部类来保障安全
// 原理是因为 静态内部类是懒加载的
public class StaticClassSingleton {

    private StaticClassSingleton() {};

    private static class InnerStaticClass {
        public static NeedSingletonClass singletonClass = new NeedSingletonClass();
    }

    // 这儿是懒加载，在 innerStaticClass 里面的静态内部类被加载的时候
    // 执行内部类的 <clinit> 方法进行初始化
    // final 是为了保证这个方法不会被重写或者重载
    public static final NeedSingletonClass getInstance() {
        return InnerStaticClass.singletonClass;
    }
}
```