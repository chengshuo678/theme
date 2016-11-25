---
title: 关于JSONP的原理
date: 2016-10-15 10:20:20
categories:
- 技术
tags:
- 实习
---


>由于前端需要用jsonp的格式，之前没用过。特此在网上查了一下，并把学习的内容和大家分享一下。

<!-- more -->


#### Json与Jsonp

- json
```
什么是json：
Json 是一种基于文本的数据描述格式。
    1. 基于纯文本，跨平台传递极其简单；
    2. Javascript原生支持，后台语言几乎全部支持；
    3. 轻量级数据格式，占用字符数量极少，特别适合互联网传递；
    4. 可读性较强(合理的依次缩进之后)；
    5. 容易编写和解析；

json的格式：
    1. JSON只有两种数据类型描述符，大括号{}和方括号[]，其余英文冒号:是映射符，英文逗号,是分隔符，英文双引号""是定义符。
    2. 大括号{}用来描述一组“不同类型的无序键值对集合”（每个键值对可以理解为OOP的属性描述），方括号[]用来描述一组“相同类型的有序数据集合”（可对应OOP的数组）。
    3. 上述两种集合中若有多个子项，则通过英文逗号,进行分隔。
    4. 键值对以英文冒号:进行分隔，并且建议键名都加上英文双引号”，以便于不同语言的解析。
    5. JSON内部常用数据类型无非就是字符串、数字、布尔、日期、null，字符串必须用双引号引起来。
　　JSON实例：
    {
        "Name": "cheng",
        "Age": 24
    }
```

- jsonp


>关于jsonp：我的理解是jsonp是一种借用json作为参数的 数据交换方式。
```
引入：
    1. Ajax直接请求普通文件存在跨域无权限访问的问题。（JavaScript出于安全方面的考虑，不允许跨域调用其他页面的对象。eg：a.com 域名下的js无法操作b.com或是c.a.com域名下的对象。）
    2. 但是，Web页面上调用js文件时则不受是否跨域的影响(具有src属性的标签都具有跨域的能力，iframe，image，script)
    3. 所以,那就是在远程服务器上设法把数据（以json的格式）装进<script>的标签里，供前端处理。
    4. 为了便于客户端使用数据，逐渐形成了一种非正式传输协议，人们把它称作JSONP。
```

#### 代码原理示例

```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title></title>
    <script type="text/javascript">
    var localHandler = function(data){
        alert('我是本地函数，远程js带来的数据是：' + data.result);
    };
    
    
    <script type="text/javascript" src="http://remoteserver.com/remote.js"></script>
    
    </script>
    
</head>
<body>
 
</body>
</html>
```
服务器有个文件remote.js文件代码如下：

```
localHandler({"result":"我是远程js带来的数据"});
```

```
相当于服务器的文件remote.js里的localHandler(xxxx)触发了前端的localHandler(data)去执行。
那么问题来了，服务器的文件remote.js是如何知道去调用前端的哪个函数呢？
so，前端需要把函数名用get的方式?callback=localHandler传给服务器。
服务器知道函数名称返回数据：localHandler({"result":"我是远程js带来的数据"})。
客户端接到数据（其实这数据就是一段代码）成功之后，把它放入<script>中执行即可。

其实关于客户端的这些步骤jquery都封装好了。
```


```
    <script type="text/javascript" src=jquery.min.js"></script>
    <script type="text/javascript">
     jQuery(document).ready(function(){
        $.ajax({
             type: "get",
             async: false,
             url: "http://flightQuery.com/jsonp/flightResult.aspx",
             dataType: "jsonp",
             jsonp: "callback",//传递给请求处理程序或页面的，用以获得jsonp回调函数名的参数名(一般默认为:callback)
             jsonpCallback:"flightHandler",//自定义的jsonp回调函数名称，默认为jQuery自动生成的随机函数名，也可以写"?"，jQuery会自动为你处理数据
             success: function(json){
                 alert('你的名字' + json.name);
             },
             error: function(){
                 alert('fail');
             }
         });
     });
    </script>
```

>这里， jQuery可以自动生成的随机函数名。我会有个大大的疑问，类似与localHandler的回调函数我并没有写啊。其实，善解人意的jquery自动帮我生成回调函数并把数据取出来供success属性方法来调用。

>至此，jsonp的机制原理我就全明白了。

>上述资源、思想来源于互联网。

感谢

[http://kb.cnblogs.com/page/139725/](http://kb.cnblogs.com/page/139725/)

[http://www.jb51.net/article/46463.htm](http://www.jb51.net/article/46463.htm)

[http://www.cnblogs.com/rainman/archive/2011/02/20/1959325.html](http://www.cnblogs.com/rainman/archive/2011/02/20/1959325.html)






