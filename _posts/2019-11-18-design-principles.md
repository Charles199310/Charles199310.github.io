---
layout: post
title: 设计原则
date: 2019-11-18 00:10 +0800
last_modified_at: 2025-01-01 16:12 +0800
tags: [设计模式, 设计原则]
toc:  true
---
# 设计原则

我们在面向对象开发过程中，总是希望我们的代码是稳定并且易于维护的，那什么样的的代码就是稳定的并且易于维护的呢？前人总结出了六条原则，这六条原则需要在我们开发的时候时时注意，遵循了这六条原则，我们就可以说我们的代码是稳定的，易于维护的。
* 开闭原则
* 单一职责原则
* 接口隔离原则
* 迪米特原则
* 依赖倒置原则
* 理氏替换原则  

接下来我们将仔细讲讲这六大原则：  
## 开闭原则 Open Closed Principle
> Software entities like classes, modules and functions should be open for extension but closed for modification

> 一个软件实体(可以是一个模块或一个独立的类)应当对扩展开放，对修改关闭。即软件实体应尽量在不修改原有代码的情况下进行扩展。  

开闭原则是六大原则中，说的最简单明了的，也是与设计原则的目的最直接相关的。对修改闭合——稳定，修改就有风险，原先写好的代码需要重新考虑是否有风险，甚至从新测试。对扩展开发——灵活，新修改的代码一定是拓展出来的，这样插拔式的修改，方便我们接入或删除新功能（需求）。  

开闭原则是设计原则的总纲，其他原则是对开闭原则的解释或具体实现

## 依赖倒置原则 Dependence Inversion Principle
> High level modules should not depend upon low level modules. Both should depend upon abstractions.
> Abstractions should not depend upon details. Details should depend upon abstractions.

> a. 高层模块不应该依赖于底层模块，两者应该依赖于其抽象。  
> b. 抽象不应该依赖具体实现，具体实现应该依赖抽象。  

换而言之依赖倒置原则的含义就是面向接口编程，相较于实现，抽象要更稳定一些，变化较少。例如对于一个需要List接口的地方，如过声明我们声明为ArrayList，假如我们将来随着业务扩展，要提供对LinkedList的支持，就需要我们重新定义接口。相反如果我们声明为List就不需要再进行修改了。

依赖倒置本身支持类对修改闭合的效果，同时和里氏替换原则配合支持了对扩展开放。

## 里氏替换原则 Liskov Substitution Principle
> Functions that use  pointers or references to base classes must be able to use objects of derived classes without knowing it.

> 所有引用基类的地方必须能透明地使用其子类的对象

继承是一种入侵性很强的代码组织方式，里氏替换原则就是承认这种入侵是合理的，是我们想要的。这样保证了我们在扩展时，最终的结果是与原先的设计相符的，避免出现不符合原先设计而出现bug。

里氏替换原则要求我们尽量做到：
1. 子类尽量避免重写父类的非抽象方法。
2. 重写的方法，子类的输入范围要包含父类的输入范围，父类的输出范围要包含子类的输出范围。
3. 当做不到以上两点时尽量考虑使用聚合等方式代替继承。  

里氏替换原则认为重写也修改的一种，避免重写就是对修改闭合，同时和依赖倒置配合共同确保了对扩展开放。

## 单一职责原则 Single Responsibility Principle
> There should never be more than one reason for a class to change.

> 一个类应该只有一个发生变化的原因

单一职责要求一个类只负责一个职责，这样将类的复杂被降到了最低，也可能将这个类与其他类的耦合。方便我们维护。同时当与这个类不相关的功能有变动的时候，避免了对这个类的修改。

## 接口隔离原则 Interface Segregation Principle
> Clients should not be forced to depend upon interfaces that they don\`t use.  
The dependency of one class to another one should depend on the smallest possible.  

> 客户端不应该依赖它不需要的接口。  
类间的依赖关系应该建立在最小的接口上。

接口隔离第一条要求我们以合适的颗粒度拆分，以避免与其他使用到不需要的接口的耦合。第二条是避免与不必要的类耦合。这样避免了当某一个接口发生修改时，某个类被动的发生了修改。

## 迪米特法则 Law of Demeter
> Talk only to your immediate friends and not to strangers

> 只与直接朋友交谈，不与陌生人交谈。

迪米特法则说的就是避免耦合，比如说，类A同时依赖于类B和类C,类B依赖于类C,这时因该考虑，A是不是没有必要和C通信，而是通过和B的通信间接达到和C通信的目的。

迪米特法则避免了陌生类的修改影响。
