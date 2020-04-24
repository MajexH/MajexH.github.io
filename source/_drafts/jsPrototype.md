---
title: js原型链
category: code
tags:
  - js
toc: true
---

这两天看了下typescript，对于它如何实现class的extends是有点疑问的，因为有在js里面有多种方式来实现继承，一般使用组合式（js高级程序语言设计），即属性继承通过`借用构造函数`实现，方法继承通过`原型链`实现。但是仍然对整体的继承实现有疑问，翻书才发现自己对于js的原型链理解不是很清楚，所以才记录一下。

## instance、prototype、constructor的关系

{% blockquote 《js高级程序语言设计》%}
每个构造函数都有一个原型对象，原型对象都包含一个指向构造函数的指针，而实例都包含一个指向原型对象的内部指针。
{% endblockquote %}

也就是说上面三者的关系如下图所示
{% asset_img prototypeRelationship.png test %}

实例可以通过`this.__proto__`来访问prototype属性，constructor可以通过`constructor.prototype`来访问prototype属性，prototype可以通过`prototype.constructor`来访问原来的构造方法。

## js原型链继承

首先看一下下面的组合继承的例子

```js
function Father(name) {
  this.name = name;
}

Father.prototype.sayName = function () {
  console.log(this.name);
}

function Child(name, child) {
  Father.call(name); // 第一次调用Father的构造函数
  this.child = child;
}

Child.prototype = new Father(); // 继承 第二次调用Father的构造函数
```

其次要知道js的原型链的查找顺序

{% blockquote %}
js在试图使用对象(instance)的属性是，会首先在对象内部寻找这个属性，如果找不到，则会在该对象的原型(prototype)上寻找
{% endblockquote %}

在上面的例子中通过`Child.prototype = new Father()`来实现了继承的操作，也就是说现在的`Child的实例(instance)`会通过`__proto__`属性找到一个`Father对象`，Father对象通过自己的`__proto__`对象指向`Father的prototype`对象，也就是说现在Child继承Father后，constructor实际上是指向了`Father`(因为从Child原型链上是最后指向了Father的prototype的Constructor，也就是Father自己)，也就是说这个原型链的查找过程就是

{% blockquote %}
Child---->Father instance(Child.prototype)--Child.prototype.`__proto__`-->Father.prototype
{% endblockquote %}
