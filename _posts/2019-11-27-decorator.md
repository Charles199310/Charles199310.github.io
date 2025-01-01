---
layout: post
title: Decorator 装饰模式
date: 2019-11-27 01:02 +0800
last_modified_at: 2025-01-01 15:32 +0800
tags: [设计模式，装饰模式]
toc:  true
---

# Decorator 装饰模式

[设计原则学习笔记](https://www.jianshu.com/p/f7f79adad32b)  
[设计模式学习笔记](https://www.jianshu.com/p/08bf9381697c)  
## 作用
动态的给一系列类添加新的属性以及操作。  
装饰模式和桥接模式有点像，桥接模式是将一系列类的属性和操作提取出来。两者都是提供了对扩展的开放性。
## 类图
![装饰模式](https://github.com/Charles199310/Charles199310.github.io/blob/main/assets/images/decorator_01.PNG?raw=true)
## Java实现
```Java
// 定义一个接口
public interface Component {
    void operation();
}
// 被修饰的类
public class ConcreteComponent implements Component {
    @Override
    public void operation() {
        //todo do something.
    }
}
//抽象的以及具体的装饰类
public abstract class Decorator implements Component{
    Component component;

    public Decorator(Component component) {
        this.component = component;
    }

    @Override
    public void operation() {
        component.operation();
    }
}
public class ConcreteDecorator extends Decorator {
    public ConcreteDecorator(Component component) {
        super(component);
    }

    public void addOperation() {
        //todo do something.
    }
}
```
装饰类本质也是一个Component实现，单不能单独使用，需要配合被修饰的类实用，其接口提供的方法主要还是由被修饰的类来实现。
## Android源码中的应用
* 各种IO接口
