---
layout: post
title: Chan of Responsibility 责任链模式
date: 2019-12-04 22:46 +0800
last_modified_at: 2025-01-01 16:09 +0800
tags: [设计模式, 责任链模式]
toc:  true
---
# Chan of Responsibility 责任链模式

[设计原则学习笔记](https://www.jianshu.com/p/f7f79adad32b)  
[设计模式学习笔记](https://www.jianshu.com/p/08bf9381697c)  
## 作用
将请求以其职责拆成，并以链的结构组合，使请求在链中传递并且可以在链中拦截请求。
## 类图
![责任链模式类图](https://github.com/Charles199310/Charles199310.github.io/blob/main/assets/images/chain_of_responsibility_01.PNG?raw=true)
## Java实现
```Java
//定义handler
public abstract class Handler {
    protected Handler handler;
    public abstract void handleRequest();
}
public class HandlerA extends Handler {
    public HandlerA() {
        handler = new HandlerB();
    }

    @Override
    public void handleRequest() {
        handler.handleRequest();
    }
}
public class HandlerB extends Handler{
    @Override
    public void handleRequest() {

    }
}
//客户端调用责任链
public class Client {
    public static void main(String[] args) {
        Handler handler = new HandlerA();
        handler.handleRequest();
    }
}
```
## Android源码中的应用
* View的点击事件分发
* Okhttp
