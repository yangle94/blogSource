---
title: 考虑用静态工厂方法替代构造函数
date: 2017-03-02 22:04:54
tags: [JAVA]
---

## 静态工厂方法优点
1. 静态工厂方法有名称，根据名称可以做到见文知意。对于构造方法而言，由于方法签名的原因，一个类只可能一种相同的方法签名，当然，可以通过参数顺序来克服这个问题，但是，相同参数容易让人混乱，而且有可能错过了编译器的异常。
2. 静态工厂方法可以不必每次调用它的时候都创建一个新的对象。
3. 可以返回原返回类型的任何子类型对象。
```java
public interface Service{
    //do something
}

public interface Provider{
    Service newService();
}

public class Service {
    //不允许外部实例化
    private Service(){};
    
    private static final Map<String,Provider> providers = new ConcurrentHashMap<>();
    public static final String defKey = "<def>";

    //注册
    public static void registerDefaultProvider(Provider provider) {
        registerProvider(defKey, provider);
    }

    public static void registerDefaultProvider(String name, Provider provider) {
        providers.put(name, provider);
    }
    
    //获取
    public static Service newInstance() {
        return newInstance(defkey);
    }

    public static Service newInstance(String name) {
        Provider p =  providers.get(name);

        if(null == p) 
            throw new IlleaglArgumentException("No provider registered with name : " + name);

        return p.newService();
    }
}
```

<!--more-->

## 缺点:
1. 类如果不含有公有或者受保护的构造方法，就不能被子类化。因祸得福的是，我们鼓励复合，而不是继承。
2. 静态工厂方法与普通静态方法没有任何区别，“赶着一等公民（构造方法）的活儿，却享受不到一等公民的待遇（Javadoc等工具并不关注静态工厂方法)

## 何时应该用静态工厂方法
1. 如果需要大量含有共同签名的构造方法的时候。
2. 每次返回的对象是不可变类的时候，可以提前构造好对象供其使用。
3. 需要返回不同的子类的时候。

### 静态工厂方法命名
1. valueOf ---- 返回值与参数具有相同的值
2. of ---- valueOf的替代
3. geteInstance ---- 通过参数描述返回实例，但是值不一定相同
4. newInstance ---- 创建一个新的对象
5. getType ---- Type表示返回的实例的类型
6. newType ---- 返回一个新的实例
