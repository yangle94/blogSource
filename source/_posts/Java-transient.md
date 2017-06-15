---
title: Java transient关键字使用记录
date: 2017-02-28 22:01:58
tags: [Java,FastJson]
---

今天对某个对象使用FastJson进行序列化的时候想要把某个属性进行隐藏，找了半天没找到FastJson有什么注解。查询了半天发现谋篇博客有关于transient关键字的介绍，看完突然觉得Java白学了，还有这么一个关键字不知道。

## transient的作用及使用方法
---
我们知道Java中完成对象的序列化需要实现Serilizable接口，实现了该接口则可以进行对象的序列化。然而在实际的开发中，总会有某些属性我们并不想让它进行序列化，可以对此属性加上transient关键字，以后这个属性就不会再被进行序列化和反序列化。当然，这也成为它的局限性，因为没法让序列化、反序列化不同时进行。

样例代码：
```java
class User implements Serializable {

    private transient String passwd;
    
    public String getPasswd() {
        return passwd;
    }
    
    public void setPasswd(String passwd) {
        this.passwd = passwd;
    }

}
```

<!--more-->

## transient使用小结
---
1. &emsp;&emsp;被transient修饰的变量，如果进行序列化：比如写入到文件中，则此变量就不能再被读取到，会被赋值为null
2. &emsp;&emsp;transient关键字只能修饰成员变量，不能修饰方法和类。如果某个类中需要使用此关键字，必须将该类实现Serilizable接口
3. &emsp;&emsp;静态变量不论是否被修饰，均不可被序列化。此时注意，虽然再次反序列化以后static变量可能还会有值，此时这个值对应的为当前虚拟机中整个类所对应的值，并非因为将该属性序列化时对此属性进行了序列化。因为static变量为类共有，此属性为类共有，而并非对象私有。