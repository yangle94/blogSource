---
title: jq的ajax请求后台未出错却进入error问题
date: 2016-12-27 15:08:32
categories: 
tags: [ajax,jq]
---
### 现象:
```javascript
jq进行ajax请求，后端并为报错，状态码为200但ajax请求进入error方法。
	$.ajax({
	url : "http:/www.baidu.com",
	data : {
		param : "param"
	}
	dataType : "json",
	success : function(data) {
		alert("返回成功，进入成功方法");
	}
	error : function() {
		alert("返回失败，进入失败方法")
	}		
})
```
### 检查思路:
1.查看后端日志，发现后端并未报错。
2.打开谷歌浏览器调试模式，发现状态吗为200。

由此可推断出后台并为发生任何异常，判断并不是因为后端错误所导致。由此可以判断能够接触到返回数据并出现错误的只能是js部分。

### 最终结果
查询资料后发现，对于ajax请求如果规定了返回值的格式（dataType），则返回值一定要符合dataType的格式。若返回的格式跟dataType的格式不相符，js会在解析的时候报错，由此进入了error的方法。后端以前返回值为”success”，将其改为{“isSuccess” : “success”}后进行返回，程序运行成功，进入ajax的success方法。
由此可见，若规定了dataType以后，后端返回的数据必须符合此格式，否则，前端js在解析的时候会发生错误。
