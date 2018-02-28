---
title: String
date: 2018-02-28 13:48:26
tags: ['String']
---

今天遇到了一个很有意思的关于String的问题，

```java

String a = "hello2";
final String b = "hello";
String d = "hello";
String c = b + 2;
String e = d + 2;

System.out.println("c:": c);
System.out.println("e:": e);
System.out.println(a == c);
System.out.println(a == e);

```

<!--more-->

首先，因为编译器在编译的时候发现，String b是一个常量（final修饰，当然，虚拟机内部是没有final这个修饰符的，会在编译期将代码中的final擦除），相对应的再生成c的时候会直接计算结果，然后将结果（“hello2”）赋值给c，所以，如果反编译（我这里直接使用idea打开生成的class文件）会发现，生成的代码如下：

```java

String a = "hello2";
String b = "hello";
String d = "hello";
String c = "hello2";
String e = d + 2;

```

```java

StringBuilder

    @Override
    public String toString() {
        // Create a copy, don't share the array
        return new String(value, 0, count);
    }

```
当然，使用javap反编译后能够看到，String e = d + 2;这里在class文件中，会使用StringBuilder来负责将d与2进行相加（append()），最后，会调用toString方法，然后将结果赋给e。所以，a与c全部指向常量池的“hello2”，而由于StringBuilder的toString方法会在堆中new一个新的String对象，所以a 与 e是不相等的（内存地址不同）。