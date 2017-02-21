---
title: java内部类实现原理以及注意事项
date: 2016-12-28 15:08:32
categories: ß
tags: [Java]
---
### 1.什么是内部类：
定义在某个类中的类叫做内部类。

### 2.内部类有什么好处：
1）内部类方法可以访问类定义所在的做哦用语中的数据，包括私有数据。
2）内部类可以对同一个包中的其他类隐藏起来。
3）当想要定义一个回调函数且又不想编写大量代码的时候，使用匿名内部类比较便捷。

### 3.内部类实现原理
#### 代码样例:
```java
class TalkingClock {
    private int interval;
    private boolean beep;
    public TalkingClock(int interval, boolean beep) {
        this.interval = interval;
        this.beep = beep;
    }
    public void start() {
        ActionListener listener = new TimePrinter();
        Time t = new Time(interval,listener);
        t.start();
    }
    public class TimePrinter implements ActionListener {
        public void actionPerformed(ActionEvent event) {
            Date now = new Date();
            if(beep) Toolkit.getDefaultToolkit().beep();
        } 
    }
}
```

#### 解释:
首先，内部类是一种编译器现象，与java虚拟机无关。也就是说，在虚拟机中，类并不存在内部类一说。在上面两个类中，TimePrinterl类是TalkingClock的内部类，当java编译器对此两个类进行编译时，会将内部类TimePrinter编译为TalkingClock$TimePrinter，并在内部类中增加一条外部类的引用，以此来访问外部类对象中的域，并会修改其无参的构造方法为有参的构造方法（增加一个TalkingClock参数），对于外部类TalkingClock，会增加一个static boolean access$0(TalkingClock)方法，具体可以使用java -private innerClass.TalkingClock$TimePrinter(由于在linux等系统的shell中$为关键字，所以此时需要转义)查看。

```java
public class TalkingClock$TimePrinter {
    public TalkingClock$TimePrinter(TalkingClock);
    public void actionPerformed(java.awt.event.ActionEvent);
    final TalkingClock this$0;
}
class TalkingClock {
    private int interval;
    private boolean beep;
    public TalkingClock(int interval, boolean beep);
    public void start();
    static boolean access$0(TalkingClock);
}
```
由此，当内部类中需要访问外部类的私有成员时，内部类可以调用外部类的静态方法进行对私有成员的访问。当然，access$0并不是一个合法的java方法名，因此，在java层面调用此方法名并不现实。但可以用底层的其他方法进行对此方法的调用。