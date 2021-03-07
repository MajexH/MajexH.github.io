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