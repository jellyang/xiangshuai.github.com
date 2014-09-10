---
layout: post
title: 策略模式
categories:
- 设计模式
tags:
- java
---

##1. 定义
策略模式体现了这样两个原则——封装变化和对接口编程而不是对实现编程。设计模式的作者把策略模式定义如下：

>Define a family of algorithms, encapsulate each one, and make them interchangeable. [The] Strategy [pattern] lets the algorithm vary independently from clients that use it.
>
>(策略模式定义了一系列的算法，并将每一个算法封装起来，而且使它们还可以相互替换。策略模式让算法独立于使用它的客户而变化。)

##2. 实现
###2.1 基本结构
![](http://xiangshuai.github.io/resources/Strategy-Pattern-2014213800.png)
###2.2 流程
1.	为策略对象定义Strategy接口。
2.	编写ConcreteStrategy类实现Strategy接口，ConcreteStrategy包装相关算法和行为。
3.	在Context类中，保持对Strategy对象的私有引用。
4.	在Context类中，实现Strategy对象的set和get方法。
Strategy接口定义了Strategy对象的行为；具体的ConcreteStrategy类实现了Strategy接口；Context类使用Strategy对象。

##3. 应用
1.	许多相关的类仅仅是行为有异。“策略”提供了一种用多个行为中的一个行为来配置一个类的方法。
2.	需要使用一个算法的不同变体。例如，你可能会定义一些反映不同的空间/时间权衡的算法。当这些变体实现为一个算法的类层次时，可以使用策略模式。
3.	算法使用客户不应该知道的数据。可使用策略模式以避免暴露复杂的、与算法相关的数据结构。
4.	一个类定义了多种行为，并且这些行为在这个类的操作中以多个条件语句的形式出现。将相关的条件分支移入它们各自的 Strategy类中以代替这些条件语句。

##4. 实例
###4.1 简单购物车

在实现购物车应用时，关于支付方式可能需要提供多种策略，比如信用卡支付或PayPal等网上支付方式，通过策略模式进行简单实现，类图如下：

![](http://xiangshuai.github.io/resources/Strategy-Pattern-2014213900.png)

具体实现详见：[Strategy Design Pattern in Java – Example Tutorial](http://www.journaldev.com/1754/strategy-design-pattern-in-java-example-tutorial)

###4.2 Java Comparable and Comparator
定义Strategy：

	public interface Comparator {
		public int compare(Object o1, Object o2);
	}

日期类实现Strategy接口：

	public class DateComparator implements Comparator {
  		public int compare(Object o1, Object o2) {
    		return ((Date) o1).compareTo((Date) o2);
  		}
	}

字符串比较类实现Strategy接口：

	public class StringIntegerComparator implements Comparator {
  		public int compare(Object o1, Object o2) {
    		return Integer.parseInt((String) o1) -Integer.parseInt((String) o2);
  		}
	}

倒转行为实现Strategy接口：

	public class ReverseComparator implements Comparator {
 		private final Comparator c;
 		public ReverseComparator(Comparator c) {this.c = c; }
 		public int compare(Object o1, Object o2) {
  			return c.compare(o2, o1);
 		}
	}

在对String数组逆序输出时，便可使用：

	Arrays.sort(stringArray, new ReverseComparator(new StringIntegerComparator()));

后续...

##5 参考：
1. [Java Comparable and Comparator Example to sort Objects](http://www.journaldev.com/780/java-comparable-and-comparator-example-to-sort-objects)
2. [Java Fundamentals Tutorial : Design Patterns](https://thenewcircle.com/static/bookshelf/java_fundamentals_tutorial/design_patterns.html)
3. [Examples of GoF Design Patterns](http://stackoverflow.com/questions/1673841/examples-of-gof-design-patterns)



