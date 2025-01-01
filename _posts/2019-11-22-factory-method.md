---
layout: post
title: Factory Method 工厂方法模式
date: 2019-11-22 01:35 +0800
last_modified_at: 2025-01-01 15:08 +0800
tags: [设计模式，工厂方法模式]
toc:  true
---
# Factory Method 工厂方法模式

[设计原则学习笔记](https://www.jianshu.com/p/f7f79adad32b)  
[设计模式学习笔记](https://www.jianshu.com/p/08bf9381697c)  
## 作用
将本来某类与一个系列下的每一个类耦合，通过工厂间接联系，避免直接耦合，从而避免与不必要的类耦合。

比如：有n个A的子类A1,A2……和m个B的子类B1,B2……每个A类都会依赖每一个B的子类，这是。这时，所有A类和所有B类的依赖关系一个有m*n条线，扩展一个A类的代价是要添加m个B类，扩展一个B类要修改n个A类。如果所有A类都通过同一个1个工厂类那么，依赖关系为m+n条线，扩展一个A类的代价是添加1个工厂，扩展一个B类是修改一个工厂（简单工厂）。
## 类图
![工厂方法模式类图](https://github.com/Charles199310/Charles199310.github.io/blob/main/assets/images/factory_method_01.PNG?raw=true)  
如图客户端通过持有工厂类的实现，避免了直接持有具体的产品。根据工厂类的策略，间接的生成不同的产品类。
## Java实现
```JAVA
public abstract class Product {
    public abstract String getProductName();
}

public class ProductA extends Product {
    @Override
    public String getProductName() {
        return "产品A";
    }
}

public abstract class Factory {
    public abstract Product createProduct();
}

public class ConcreteFactory extends Factory {
    @Override
    public Product createProduct() {
        return new ProductA();
    }
}

public class Client {
    public static void main(String[] args) {
        Factory factory = new ConcreteFactory();
        Product product = factory.createProduct();
        System.out.println(product.getProductName());
    }
}
```
在上面的代码中客户端避免了与具体的产品直接依赖，具体使用哪个产品的逻辑由具体的Factory来处理。未来扩展出新产品也由Factory来扩展而避免了客户端的修改。  
当没有抽象的工厂类，只有一个具体的工厂类，甚至是通过静态的方法生产产品的时候这时就是简单工厂模式（静态工厂模式）。

## Android源码中的应用
* Spannable.Factory
