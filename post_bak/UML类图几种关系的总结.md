---
title: UML类图几种关系的总结
date: 2016-05-17 19:58:50
categories: 设计模式
tags: [Java,设计模式]
---
在UML类图中，常见的有以下几种关系:泛化（Generalization）,  实现（Realization）,关联（Association）,聚合（Aggregation）,组合(Composition)，依赖(Dependency)



1.泛化(Generalization)

【泛化关系】：是一种继承关系,它指定了子类如何特化父类的所有特征和行为例如：老虎是动物的一种.

【箭头指向】：带三角箭头的实线，箭头指向父类
![1.gif](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2016/UML%E7%B1%BB%E5%9B%BE%E5%87%A0%E7%A7%8D%E5%85%B3%E7%B3%BB%E7%9A%84%E6%80%BB%E7%BB%93_1.gif-watermark)

2.实现（Realization)

【实现关系】：是一种类与接口的关系，表示类是接口所有特征和行为的实现

【箭头指向】：带三角箭头的虚线，箭头指向接口
![2.gif](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2016/UML%E7%B1%BB%E5%9B%BE%E5%87%A0%E7%A7%8D%E5%85%B3%E7%B3%BB%E7%9A%84%E6%80%BB%E7%BB%93_2.gif-watermark)

3.关联（Association）

【关联关系】：是一种拥有的关系,它使一个类知道另一个类的属性和方法；如：老师与学生，丈夫与妻子

关联可以是双向的，也可以是单向的。双向的关联可以有两个箭头或者没有箭头，单向的关联有一个箭头。

【代码体现】：成员变量

【箭头及指向】：带普通箭头的实心线，指向被拥有者
![3.gif](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2016/UML%E7%B1%BB%E5%9B%BE%E5%87%A0%E7%A7%8D%E5%85%B3%E7%B3%BB%E7%9A%84%E6%80%BB%E7%BB%93_3.gif-watermark)

上图中，老师与学生是双向关联，老师有多名学生，学生也可能有多名老师。但学生与某课程间的关系为单向关联，一名学生可能要上多门课程，课程是个抽象的东西他不拥有学生。
![4.gif](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2016/UML%E7%B1%BB%E5%9B%BE%E5%87%A0%E7%A7%8D%E5%85%B3%E7%B3%BB%E7%9A%84%E6%80%BB%E7%BB%93_4.gif-watermark)

上图为自身关联：
4. 聚合（Aggregation）

【聚合关系】：是整体与部分的关系.如车和轮胎是整体和部分的关系.

聚合关系是关联关系的一种，是强的关联关系；关联和聚合在语法上无法区分，必须考察具体的逻辑关系。

【代码体现】：成员变量

【箭头及指向】：带空心菱形的实心线，菱形指向整体
![5.gif](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2016/UML%E7%B1%BB%E5%9B%BE%E5%87%A0%E7%A7%8D%E5%85%B3%E7%B3%BB%E7%9A%84%E6%80%BB%E7%BB%93_5.gif-watermark)

5. 组合(Composition)

【组合关系】：是整体与部分的关系.,没有公司就不存在部门      组合关系是关联关系的一种，是比聚合关系还要强的关系，它要求普通的聚合关系中代表整体的对象负责代表部分的对象的生命周期

【代码体现】：成员变量

【箭头及指向】：带实心菱形的实线，菱形指向整体
![6.gif](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2016/UML%E7%B1%BB%E5%9B%BE%E5%87%A0%E7%A7%8D%E5%85%B3%E7%B3%BB%E7%9A%84%E6%80%BB%E7%BB%93_6.gif-watermark)

6. 依赖(Dependency)

【依赖关系】：是一种使用的关系,所以要尽量不使用双向的互相依赖。

【代码表现】：局部变量、方法的参数或者对静态方法的调用

【箭头及指向】：带箭头的虚线，指向被使用者
![7.gif](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2016/UML%E7%B1%BB%E5%9B%BE%E5%87%A0%E7%A7%8D%E5%85%B3%E7%B3%BB%E7%9A%84%E6%80%BB%E7%BB%93_7.gif-watermark)

各种关系的强弱顺序：

泛化= 实现> 组合> 聚合> 关联> 依赖
下面这张UML图，比较形象地展示了各种类图关系：
![8.gif](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2016/UML%E7%B1%BB%E5%9B%BE%E5%87%A0%E7%A7%8D%E5%85%B3%E7%B3%BB%E7%9A%84%E6%80%BB%E7%BB%93_8.gif-watermark)
