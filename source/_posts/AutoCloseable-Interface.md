---
title: AutoCloseable_Interface
date: 2017-03-08 09:45:02
tags: [Java,AutoClostable]
---

今天在关闭输入输出流的时候，觉得总是手动去关闭太麻烦，有没有什么简单的方法。于是乎发现了AutoCloseable这个接口。

## 重申一下为什么要手动关闭输入输出流

对于平常的Java对象来说，如果Java的gc机制发现某个对象已经不可到达，就会启动对此对象的内存回收机制。当然，他什么时候能够把对象回收完了，是根据操作系统的调度来决定的，并不是说gc机制一起动，就能立刻回收此对象的内存。但是对于InputStream和OutputStream以及他们的子类来说，如果开启了某个流，就会有操作系统之外的资源依附在某个Java对象上，这就导致gc认为其实活着的，而不是死亡的，所以并不能出发gc机制。

## AutoCloseable接口

在Java1.7中提供了新的AutoCloseable接口，以前的Closeable接口扩展了AutoCloseable接口，也就是说，所有的实现了Closeable接口的输入输出流全部实现了AutoCloseable接口，就是说所有的输入输出流都能进行带资源的try catch语句。

```Java
public class InputStreamReaderTest {
    public static void main(String[] args) {
        try(BufferedReader reader = new BufferedReader(new InputStreamReader(        
                new FileInputStream(new File("/home/angle/test.webp")),"UTF8"),1024)){
            System.out.println(reader.readLine());    //这里直接读一行
        }catch(IOException e){
            e.printStackTrace();
        }
    }
}
```

此时，就不需要我们自己手动去关闭输入、输出流了，带资源的try块儿会帮我们把资源管理好。

## 注：关于带资源的try语句的3个关键点：

由带资源的try语句管理的资源必须是实现了AutoCloseable接口的类的对象。

在try代码中声明的资源被隐式声明为fianl。

通过使用分号分隔每个声明可以管理多个资源。


