---
title: SpringMVC_Annotation_@RequestBody
date: 2017-03-01 19:33:40
tags: [SpringMVC]
---
## @RequestBody注解的源码
```java
@Target({ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RequestBody {
    boolean required() default true;
}
```
由源码可见，@RequestBody只有一个required属性，默认值为true，该注解会保留至.class文件中，利用反射可以进行查找。

<!--more-->

## @RequestBody的作用
该注解常用来处理Content-Type: 不是application/x-www-form-urlencoded编码的内容，例如application/json, application/xml等；

它是通过使用HandlerAdapter 配置的HttpMessageConverters来解析post data body，然后绑定到相应的bean上的。

因为配置有FormHttpMessageConverter，所以也可以用来处理 application/x-www-form-urlencoded的内容，处理完的结果放在一个MultiValueMap<String, String>里，这种情况在某些特殊需求下使用，详情查看FormHttpMessageConverter api;

例如
```JAVA
    @RequestMapping(value = "sendMessage", method = RequestMethod.POST)
    public String sendMesscage(@RequestBody UserIdCodeDTO userIdCodeMessageDTO) {
        return userIdCodeMessageDTO.toString();
    }
```

运用次注解后，SprimgMVC将会把post或者get请求来的信息进行逆序列化，组成对象，前提是你传过来的值跟该对象匹配。

### 注意！！
使用该注解以后，若想传送非json格式代码，需要指定请求头Content-Type为application/json，否则对象所有属性将为null。