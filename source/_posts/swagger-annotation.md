---
title: swagger-ui注解
date: 2017-03-14 10:30:50
tags: [swagger-ui]
---
今天使用swagger的时候出现了不少一点问题，body传上来的Json字符串在swagger-ui上的形式总是不符合我的意愿，我找了好久才找到了问题。特别来记录一下。

## 注解列表

### @Api

用在类上，说明该类的作用

### @ApiOperation

用在方法上，说明方法的作用(当方法所需参数为对象时，)

### @ApiImplicitParams、@ApiImplicitParam

@ApiImplicitParams({
    @ApiImplicitParam
})
用在@ApiImplicitParams注解中，指定一个请求参数的各个方面
paramType:
参数放在哪个地方
>
header-->请求参数的获取：@RequestHeader
query-->请求参数的获取：@RequestParam
path（用于restful接口）-->请求参数的获取：@PathVariable
body（不常用）
form（不常用）
name：参数名
dataType：参数类型
required：参数是否必须传
value：参数的意思
defaultValue：参数的默认值

### @ApiResponses

用于表示一组响应

### @ApiResponse

用在@ApiResponses中，一般用于表达一个错误的响应信息
code：数字，例如400
message：信息，例如"请求参数没填好"
response：抛出异常的类

### @ApiModel

描述一个Model的信息（这种一般用在post创建的时候，使用@RequestBody这样的场景，请求参数无法使用@ApiImplicitParam注解进行描述的时候）

### @ApiModelProperty

描述一个model的属性

## 注意事项
对于@ApiImplicitParams、@ApiImplicitParam、@ApiModel、@ApiModelProperty的使用情景。
首先，对于类似application/x-www-form-urlencoded等除了application/json以外的传输方式，其传输内容实质上是K-V的方式进行传递的，此时可以用@ApiImplicitParams、@ApiImplicitParam对所传入的参数进行描述，但是如果使用application/json的方式传输JSON字符串，这种表示方式是
不可用的，因为无法准确表示字符串中每个字段所代表的值。此时，就应该使用

```java
    @ApiOperation(value="使用httpClient爬取页面", notes="使用httpClient爬取页面", produces = "application/json")
    @ApiImplicitParam(name = "pageInfoDto", value = "用户详细实体user", required = true, dataType = "PageInfoDto", paramType = "body")
    @RequestMapping("getValue")
    public Result getValue(@RequestBody PageInfoDto pageInfoDto) {}
```

这个时候可以使用一个@ApiImplicitParam来描述所传JSON的信息，千万不可以用@ApiImplicitParams，否则swagger-ui的界面上会出现异常的参数形式。此时，可以配合@ApiModel、 @ApiModelProperty在当前参数的实体类上对信息进行描述。

