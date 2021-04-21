---
title: design-pattern
category:
  - code
tags:
  - 设计模式
toc: true
date: 2021-03-09 21:42:47
thumbnail:
password:
---

常见的设计模式的简单总结和简单实现，为以后做个参考

<!-- more -->

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
> 上述两个类 已经在 JDK9 中被废弃。愿意是因为其无法控制通知的顺序(因为调用的父类是 notify 方法，而且是从后向前遍历 observer 的)，而且由于 Observable 是一个类形式提供，难以扩展。

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

# 命令模式

命令模式是为了封装不同的操作的不同 api 做的

如`线程池等的实现、工作队列`等也是跟命令模式相关

{% asset_img command.png 报错 %}

```java
public class Client {

    private CommandManager manager;

    public Client(CommandManager manager) {
        this.manager = manager;
    }

    public void execute(int id, Object args) {
        this.manager.execute(id, args);
    }
}

// 对不同 API 的调用封装成同一个 API
public interface Command {
    // 封装命令的细节
    void execute(Object args);
}

public class CommandExecutor {

    void up() {
        System.out.println("executor up !");
    }
}

// 管理并初始化所有的 command 对象
public class CommandManager {

    private Command[] commands;

    public CommandManager() {
        this.commands = new Command[1];
        this.commands[0] = new CommandImpl(new CommandExecutor());
    }

    // 根据传入的 command id 与 args 具体执行
    public void execute(int commandID, Object args) {
        this.commands[commandID].execute(args);
    }
}

```

# 适配器模式

为了适配两种不同的方法的对象，让一种能够适应另一种添加的类

- 接口适配：实现对应的接口，通过组合的方式实现适配。
- 类适配：通过组合继承（java 不可能）

{% asset_img adapter.png 报错 %}

```java

// 客户端调用的接口
public interface Duck {

    void spark();
}

public class Dog {

    public void bark() {
        System.out.println("!!!!!!!!");
    }
}

public class DuckAdapter implements Duck {

    private Dog dog;

    public DuckAdapter(Dog dog) {
        this.dog = dog;
    }

    @Override
    public void spark() {
        this.dog.bark();
    }
}

```

# 外观模式

{% asset_img appearance.png 报错 %}

外观模式是一个向外暴露简单接口，对内进行包装的设计模式。用于封装复杂的逻辑,以减少对象之间的依赖。

`最少知识原则`: 减少对象间的交互，因此只应该调用以下范围的方法。

- 对象本身
- 方法参数传递的对象
- 此方法创建或实例化的对象
- 对象的任何组件

# 模板方法

模板方法用于封装统一的抽象步骤。通过在父类中定义算法的运行流程，提供算法的默认实现并交给不同的子类进行重载完成。

{% asset_img tmplate.png 报错 %}

```java
public interface Action {
    // 定义下面将要使用的接口
    void act();
}

// 定义抽象的流程
public abstract class AbstractProcess implements Action {

    // 保障子类的行为有一定的共同性

    // 提供 action 的默认实现
    public void actionOne() {
        System.out.println("One");
    }

    // 提供 action 的默认实现
    public void actionTwo() {
        System.out.println("Two");
    }

    // 可以再次提供 abstract 的默认实现
    public abstract void actionThree();

    // 可以在子类中实现 hook 方法 hook 住 act 执行的过程
    public abstract void hook();

    @Override
    public void act() {
        // 封装整体的算法调用逻辑
        // 根据运行时 动态的选择方法表里面的方法
        actionOne();
        actionTwo();
        actionThree();
        hook();
    }
}

// 这样可以保证底层组件 只需要关注实现方法细节 不需要关注其他
public class ProcessImpl  extends AbstractProcess {
    @Override
    public void actionThree() {
        System.out.println("process");
    }

    @Override
    public void hook() {
        System.out.println("hook");
    }
}

```

# 迭代器模式

迭代器模式用来提供统一的访问元素的接口，可以不用知道元素存储的具体细节访问。

{% asset_img iterator.png 报错 %}

通过 iterator 暴露统一的 遍历 接口。客户端通过保存所有数据的 Items 实现类创建对应的 iterator 完成数据的遍历操作。

> 单一责任：一个类应该只有一个引起变化的原因，所以需要把 Items 与 Iterator 分开实现。

- 简单的 iterator 实现

使用一个数值来表明现在访问到的位置

```java
package iterator;

// 作为 iterator 的基类
public interface BaseIterator {

    boolean hasNext();
    Object next();
}

package iterator;

public class ArrayIterator implements BaseIterator {

    // 保存遍历的下标
    private int index;
    // 遍历的数组
    private int[] nums;

    public ArrayIterator(int[] nums) {
        this.nums = nums;
        this.index = 0;
    }

    @Override
    public boolean hasNext() {
        return this.index < nums.length;
    }

    @Override
    public Object next() {
        return this.nums[this.index++];
    }
}

```

## 在树状结构中进行迭代的方式

相当于组合迭代器，问如果在 `[1,[2,3,[2,34],3]]` 这种中间实现 iterator 该如何处理。

例题- [扁平化嵌套列表迭代器](https://leetcode-cn.com/problems/flatten-nested-list-iterator/)

### 空数组即 [[]] 返回 null 的解法

使用递归的思想，在 iterator 中保存每次遍历的 iterator

这样在遇到深层次的嵌套的时候，会在每一个 iterator 中保存 下一层次的引用，最后得到结果

```java
import java.util.*;

public class NestedIteratorNew implements Iterator<Integer> {
    // 给定一个 nexted 的数组
    // 问如何去遍历它
    // 这儿使用组合的方式去生成 nextedList 然后再用 stack 存储遍历的 iterator
    private static interface NestedInteger {
        // @return true if this NestedInteger holds a single integer, rather than a nested list.
        public boolean isInteger();

        // @return the single integer that this NestedInteger holds, if it holds a single integer
        // Return null if this NestedInteger holds a nested list
        public Integer getInteger();

        // @return the nested list that this NestedInteger holds, if it holds a nested list
        // Return null if this NestedInteger holds a single integer
        public List<NestedInteger> getList();
    }

    // 提供的默认实现类
    private static class NestedIntegerImpl implements NestedInteger {

        private int num;
        private List<NestedInteger> list;
        private boolean isNum;

        public NestedIntegerImpl(int num) {
            this.num = num;
            this.isNum = true;
        }

        public NestedIntegerImpl(List<NestedInteger> list) {
            this.list = list;
            this.isNum = false;
        }

        @Override
        public boolean isInteger() {
            return this.isNum;
        }

        @Override
        public Integer getInteger() {
            return this.num;
        }

        @Override
        public List<NestedInteger> getList() {
            return this.list;
        }
    }

    // stack 来保存每次转换的出来的 iterator 相当于 next 调用的时候去找到的是 stack top 里的 iterator
    Deque<Iterator<NestedInteger>> stack;

    public NestedIteratorNew(List<NestedInteger> nestedList) {
        this.stack = new LinkedList<>();
        // 初始化加入的节点
        this.stack.addLast(nestedList.iterator());
    }

    @Override
    public Integer next() {
        if (hasNext()) {
            Iterator<NestedInteger> iterator = this.stack.peekLast();
            NestedInteger i = iterator.next();
            if (!i.isInteger()) {
                stack.addLast(i.getList().iterator());
                return next();
            }
            return i.getInteger();
        }
        return null;
    }

    @Override
    public boolean hasNext() {
        if (this.stack.size() > 0) {
            Iterator<NestedInteger> top = this.stack.peekLast();
            if (top.hasNext()) return true;
            // 这个时候 top 的 iterator 访问完毕了 去访问下面的 iterator
            this.stack.removeLast();
            // 只需要递归的调用 访问 栈即可
            return hasNext();
        }
        return false;
    }

    public static void main(String[] args) {
        List<NestedInteger> test = new ArrayList<>();
//        List<NestedInteger> one = new ArrayList<>();
//        one.add(new NestedIntegerImpl(1));
//        one.add(new NestedIntegerImpl(1));
//        test.add(new NestedIntegerImpl(one));
//        test.add(new NestedIntegerImpl(2));
//        List<NestedInteger> three = new ArrayList<>();

        test.add(new NestedIntegerImpl(new ArrayList<>()));
//
//        three.add(new NestedIntegerImpl(1));
//        three.add(new NestedIntegerImpl(1));
//        test.add(new NestedIntegerImpl(three));
//        test.add(new NestedIteratorNew.NestedIntegerImpl(new ArrayList<>()));
//        test.add(new NestedIteratorNew.NestedIntegerImpl(2));
//        test.add(new NestedIteratorNew.NestedIntegerImpl(new ArrayList<>() {{
//            List<NestedInteger> two = new ArrayList<>();
//            two.add(new NestedIteratorNew.NestedIntegerImpl(new ArrayList<>()));
//            add(new NestedIteratorNew.NestedIntegerImpl(two));
//            add(new NestedIteratorNew.NestedIntegerImpl(3));
//        }}));

        NestedIteratorNew it = new NestedIteratorNew(test);
        while (it.hasNext()) {
            System.out.println(it.next());
        }
    }
}
```

### 空数组不返回的结果

由于上一个方法不能保证返回的时候，嵌套空数组返回 null。

所以，实际上只能在 hasNext() 调用的过程中去处理这种事情，保证每次从 stack 栈中获取的都是数字，不会存在 null 这种情况。

所以需要在 hasNext() 调用的过程中进行铺平操作，递归的调用，直到栈顶是一个带有数字的 iterator

```java
import java.util.*;

public class NestedIteratorNotReturnNull implements Iterator<Integer> {

    // 为了不返回 null 所以要在 hasNext 中就将所有数据准备好
    private Deque<Iterator<NestedInteger>> stack;

    public NestedIteratorNotReturnNull(List<NestedInteger> nestedList) {
        this.stack = new LinkedList<>();
        this.stack.addLast(nestedList.iterator());
    }

    @Override
    public boolean hasNext() {
        // 拉平所有的 iterator
        while (!this.stack.isEmpty()) {
            // 已经拉平了一个状态了
            Iterator<NestedInteger> top = this.stack.peekLast();
            if (!top.hasNext()) {
                this.stack.removeLast();
                continue;
            }
            // 每次处理一个
            NestedInteger next = top.next();
            if (next.isInteger()) {
                // 已经找到了一个数字 直接返回
                this.stack.addLast(Collections.singletonList(next).iterator());
                return true;
            }
            this.stack.addLast(next.getList().iterator());
        }
        return false;
    }

    @Override
    public Integer next() {
        // hasNext 中已经将数据和结果准备好
        return stack.peekLast().next().getInteger();
    }

    private static interface NestedInteger {
        // @return true if this NestedInteger holds a single integer, rather than a nested list.
        public boolean isInteger();

        // @return the single integer that this NestedInteger holds, if it holds a single integer
        // Return null if this NestedInteger holds a nested list
        public Integer getInteger();

        // @return the nested list that this NestedInteger holds, if it holds a nested list
        // Return null if this NestedInteger holds a single integer
        public List<NestedInteger> getList();
    }

    // 提供的默认实现类
    private static class NestedIntegerImpl implements NestedInteger {

        private int num;
        private List<NestedInteger> list;
        private boolean isNum;

        public NestedIntegerImpl(int num) {
            this.num = num;
            this.isNum = true;
        }

        public NestedIntegerImpl(List<NestedInteger> list) {
            this.list = list;
            this.isNum = false;
        }

        @Override
        public boolean isInteger() {
            return this.isNum;
        }

        @Override
        public Integer getInteger() {
            return this.num;
        }

        @Override
        public List<NestedInteger> getList() {
            return this.list;
        }
    }

}

```

# 状态模式（状态机）

状态模式抽象状态操作，进行统一的调配，这样可以方便修改添加功能。因为这样才能满足对扩展开放，对修改关闭的效果。

{% asset_img status.png 报错 %}

以书上的例子为例，这个有以下几个状态及其转换关系

{% asset_img status-change.png 报错 %}

1. 抽象 state 状态

```java
package status;

public interface State {

    void insertQuarter() throws Exception;

    void turnCrank() throws Exception;

    void ejectQuarter() throws Exception;

    void dispense() throws Exception;
}
```

2. 实现各种状态

```java
package status;

public class HasQuarterState implements State {

    // 暴露 context 是为了能够在 state 中操作 context 进行状态转移
    private Context context;

    public HasQuarterState(Context context) {
        this.context = context;
    }

    @Override
    public void insertQuarter() throws Exception {
        throw new RuntimeException("无法重复投币");
    }

    @Override
    public void turnCrank() throws Exception {
        this.context.setCurrentState(context.getSoldState());
    }

    @Override
    public void ejectQuarter() throws Exception {
        // 操作 context 进行状态转移
        this.context.setCurrentState(context.getNoQuarterState());
    }

    @Override
    public void dispense() throws Exception {
        throw new RuntimeException("还未 turn crank 无法获取");
    }
}

```

3. context 实现对于 state 的管理

```java
package status;

public class Context implements State {

    private final State noQuarterState;
    private final State hasQuarterState;
    private final State soldOutState;
    private final State soldState;

    private State currentState;
    private int count;

    public Context(int count) {
        this.count = count;
        this.hasQuarterState = new HasQuarterState(this);
        this.noQuarterState = new NoQuarterState(this);
        this.soldOutState = new SoldOutState(this);
        this.soldState = new SoldState(this);

        this.currentState = this.noQuarterState;
    }

    // 表示剩余资源的数量 进行减一操作
    public void releaseCount() {
        this.count--;
    }

    public int getCount() {
        return count;
    }

    public void setCurrentState(State currentState) {
        this.currentState = currentState;
    }

    public State getNoQuarterState() {
        return noQuarterState;
    }

    public State getHasQuarterState() {
        return hasQuarterState;
    }

    public State getSoldOutState() {
        return soldOutState;
    }

    public State getSoldState() {
        return soldState;
    }

    // 暴露 state 的操作
    @Override
    public void insertQuarter() throws Exception {
        this.currentState.insertQuarter();
    }

    @Override
    public void turnCrank() throws Exception {
        this.currentState.turnCrank();
    }

    @Override
    public void ejectQuarter() throws Exception {
        this.currentState.ejectQuarter();
    }

    @Override
    public void dispense() throws Exception {
        this.currentState.dispense();
    }

    @Override
    public String toString() {
        return "Context{" +
                "currentState=" + currentState +
                ", count=" + count +
                '}';
    }

    public static void main(String[] args) {
        State c = new Context(5);
        try {
            c.insertQuarter();
            c.turnCrank();
            c.dispense();
            System.out.println(c);
            c.insertQuarter();
            c.ejectQuarter();
//            c.turnCrank();
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
    }
}
```

# 代理模式

代理模式也是对一个对象进行增强的模式，我理解的话。

通过持有真实的对象，向外暴露相同的方法，对数据进行动态增强。

{% asset_img proxy.png 报错 %}

## java 动态代理

java 会在运行时创建代理类(返回生成类的 byte[] 并使用类加载器加载)，通过继承 Proxy 类实现 Subject 接口的方式实现。具体可以参考下面的[文章](https://segmentfault.com/a/1190000022789831)

{% asset_img dynamic-proxy.png 报错 %}

# 责任链模式

责任链设计模式是一种对链式调用更高级的抽象，能够把链式函数执行的控制权下放到链中间，在执行链的过程中，可以在调用下一个链式过程前和后处理进行额外的处理。java 中实现的主体逻辑也采用下面的方法实现。

{% asset_img chain.png %}

主要分为两个部分

- 处理函数，即承担链式函数逻辑的实现
- 管理部分，即管理链调用过程的部分

```java
package chain;

@FunctionalInterface
public interface Process {

    void process(Chain c);
}

import java.util.ArrayList;
import java.util.List;

public class Chain {

    List<Process> handlers;
    int index;

    public Chain() {
        this.index = -1;
        this.handlers = new ArrayList<>();
    }

    public void register(Process ...processes) {
        this.handlers.addAll(Arrays.asList(processes));
    }

    public void next() {
        this.index++;
        // 为了保证链式函数中没有被显式调用 .next() 的地方也能够被执行
        while (this.index < this.handlers.size()) {
            this.handlers.get(this.index).process(this);
            this.index++;
        }
    }

    public static void main(String[] args) {
        Process p1 = (c) -> {
            System.out.println("first start");
            c.next();
            System.out.println("first end");
        };

        Process p2 = (c) -> {
            System.out.println("second start");
            c.next();
            System.out.println("second end");
        };
        Chain c = new Chain();
        c.register(p1);
        c.register(p2);

        c.next();
    }
}

```

## golang gin 中的链实现

[gin](https://github.com/gin-gonic/gin)

gin 也是使用责任链的方式来处理对某一个请求的链式调用的。可以通过如下的方法实现一个链式调用的过程。

```golang
router.GET("/example", func(c *Context) {

}, func(context *Context) {
    
})
```

其中 Context 是一个被复用的上下文，保存 http 请求的 header、writer、body 等信息。在运行时被动态的添加对应的 handlerChain 到 context 中进行链式调用执行。

TODO：字符匹配的前缀树。goland 使用一个 radix 树来存储请求路径，每个 request method 会包含一个对应的 tree。

```golang
// serveHttp 的入口
func (engine *Engine) handleHTTPRequest(c *Context) {
	httpMethod := c.Request.Method
	rPath := c.Request.URL.Path
	unescape := false
	if engine.UseRawPath && len(c.Request.URL.RawPath) > 0 {
		rPath = c.Request.URL.RawPath
		unescape = engine.UnescapePathValues
	}

	if engine.RemoveExtraSlash {
		rPath = cleanPath(rPath)
	}

	// Find root of the tree for the given HTTP method
    // 1. 找到对应的 tree 上的节点
	t := engine.trees
	for i, tl := 0, len(t); i < tl; i++ {
		if t[i].method != httpMethod {
			continue
		}
		root := t[i].root
		// Find route in tree
		value := root.getValue(rPath, c.params, unescape)
		if value.params != nil {
			c.Params = *value.params
		}
		if value.handlers != nil {
            // 2. 将当前的 tree 上注册的 handler 填充到 context 中
			c.handlers = value.handlers
			c.fullPath = value.fullPath
            // 3. 执行 handler chain
			c.Next()
			c.writermem.WriteHeaderNow()
			return
		}
		if httpMethod != "CONNECT" && rPath != "/" {
			if value.tsr && engine.RedirectTrailingSlash {
				redirectTrailingSlash(c)
				return
			}
			if engine.RedirectFixedPath && redirectFixedPath(c, root, engine.RedirectFixedPath) {
				return
			}
		}
		break
	}

	if engine.HandleMethodNotAllowed {
		for _, tree := range engine.trees {
			if tree.method == httpMethod {
				continue
			}
			if value := tree.root.getValue(rPath, nil, unescape); value.handlers != nil {
				c.handlers = engine.allNoMethod
				serveError(c, http.StatusMethodNotAllowed, default405Body)
				return
			}
		}
	}
	c.handlers = engine.allNoRoute
	serveError(c, http.StatusNotFound, default404Body)
}

```

所以其实，gin 中将表示和实际执行分为了两个部分

- 执行

链的执行包含在 context 中，通过 context.next() 调用链的下一个方法

```golang
func (c *Context) Next() {
    // 使用 index 标识现在访问到的节点下标
	c.index++
    // 使用 for 循环，这样可以保证在 handler 中不显示调用 next 方法 
    // 也能执行完调用链
	for c.index < int8(len(c.handlers)) {
		c.handlers[c.index](c)
		c.index++
	}
}
```

- 表示

链自身的标识，由于链需要在调用的时候调用下一个方法，所以需要一个 context 的引用进行调用，golang 使用 type 给了对应的 chain alias，实际其就是一个 handlerFunc 的切片

```java
// HandlersChain defines a HandlerFunc array.
type HandlersChain []HandlerFunc

// HandlerFunc defines the handler used by gin middleware as return value.
type HandlerFunc func(*Context)
```

链的注册，是通过 routerGroup 管理，每个 routerGroup 对应一个大链接下的小链接

```golang
// POST is a shortcut for router.Handle("POST", path, handle).
// 1. 首先将当前针对一个 request method 的方法的链式调用注册
func (group *RouterGroup) POST(relativePath string, handlers ...HandlerFunc) IRoutes {
	return group.handle(http.MethodPost, relativePath, handlers)
}

// 2. 与之前注册到 group 上的 handler 进行合并
func (group *RouterGroup) handle(httpMethod, relativePath string, handlers HandlersChain) IRoutes {
	absolutePath := group.calculateAbsolutePath(relativePath)
	handlers = group.combineHandlers(handlers)
	group.engine.addRoute(httpMethod, absolutePath, handlers)
	return group.returnObj()
}

// 3. 添加 handler 的绝对路径 与 对应的 handlerChain 到 radix tree 结构中
func (engine *Engine) addRoute(method, path string, handlers HandlersChain) {
	assert1(path[0] == '/', "path must begin with '/'")
	assert1(method != "", "HTTP method can not be empty")
	assert1(len(handlers) > 0, "there must be at least one handler")

	debugPrintRoute(method, path, handlers)

	root := engine.trees.get(method)
	if root == nil {
		root = new(node)
		root.fullPath = "/"
		engine.trees = append(engine.trees, methodTree{method: method, root: root})
	}
	root.addRoute(path, handlers)

	// Update maxParams
	if paramsCount := countParams(path); paramsCount > engine.maxParams {
		engine.maxParams = paramsCount
	}
}
```

## js koa 中的洋葱模型实现

js 的实现就简单很多了，koa 的实现可以参考 [koa-compose](https://www.npmjs.com/package/koa-compose)

其本质思想与上面一个差不多，但是由于 js 支持闭包，也支持将 func bind 一个运行时，因此其在调用的时候，传递的是注册到 chain 中的下一个 function，所以直接执行即可。但是也因此，其必须要调用显示的 `next()` 方法。

```js
function compose (middleware) {
  if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!')
  for (const fn of middleware) {
    if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!')
  }

  /**
   * @param {Object} context
   * @return {Promise}
   * @api public
   */
  // compose 执行完毕后，相当于把这个闭包结构声明完毕
  return function (context, next) {
    // last called middleware #
    // 用闭包来保存当前调用的到 middleware 中的哪一个
    let index = -1
    // 返回 middleware 的入口
    return dispatch(0)
    // 只需要 compose 结束后 执行该方法即可
    function dispatch (i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      let fn = middleware[i]
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        // fn 会执行当前的 middleware 同时将 下一个 dispatch 传递给新的 fn 方法作为第二个参数调用
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}
```
