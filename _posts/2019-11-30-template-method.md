---
layout: post
title: Template Method 模板方法模式
date: 2019-11-30 01:07 +0800
last_modified_at: 2025-01-01 15:46 +0800
tags: [设计模式，模板方法模式]
toc:  true
---

# Template Method 模板方法模式

[设计原则学习笔记](https://www.jianshu.com/p/f7f79adad32b)  
[设计模式学习笔记](https://www.jianshu.com/p/08bf9381697c)  
## 作用
将可变的和不可变的代码分离，可变的代码由子类实现，不可变代码由父类实现，以达到复用的效果和方便扩展的作用。
## 类图
![模板方法模式类图](https://github.com/Charles199310/Charles199310.github.io/blob/main/assets/images/template_method_01.PNG?raw=true)
## Java实现
```Java
// 抽象方法
public abstract class AbstractClass {
    public final void finalMethod() {}
    public abstract void abstractMethod();
}
// 具体方法
public class ConcreteClass extends AbstractClass {
    @Override
    public void abstractMethod() {
    }
}
```
## Android源码中的应用
* Activityy等四大组件
* View
