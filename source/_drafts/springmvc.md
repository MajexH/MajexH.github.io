---
title: spring mvc
category: essay
tags:
  - java
  - spring
toc: true
---

最近温习了一下spring，首先看了一下spring mvc。传统的javaEE是使用servlet进行编程，每个servlet通过自己的`service()`方法来相应客户端的请求，其实现类HttpServlet通过`doget() dopost()`等方法完成对各类不同请求方法的请求的响应。

那么就有一个问题spring是不是也是使用servlet来进行请求响应的呢，从查的资料来看，不然。

spring提供了一个controller层的实现spring mvc，该spring mvc通过DispatcherServlet来实现对请求的分发。使用spring mvc时，经常使用`@controller`注解来标识一个真正处理请求的类，实际上这种方法是通过`@controller`标识了一个需要spring auto detect的`java bean`，在开启了的自动扫描后，spring会自动将这个java bean纳入spring的容器中。

那么既然这个spring mvc是通过一个DipatcherServlet通过原生的servlet来实现了请求的转发，也就是说其背后是一套的自己的容器，那么他是怎么拿到servletContext（上下文）的呢，在applicationContext（spring提供的容器中），有一个servletContext的引用，通过该引用保障了spring的容器中能够获取到上下文。

## @RequestMapping controller 方法注入

翻看源码 可以知道，在DispatchServlet被初始化的时候，同时初始化了一个HandlerMapping和HandlerAdapter

