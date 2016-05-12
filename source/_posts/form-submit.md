---
title: 表单提交问题
date: 2016-05-12 14:47:44
tags: form
categories: HTML
---

## 表单提交方式

* input[type=submit]
* button[type=submit]
* input[type=image]
* form对象调用submit()方法
* submit事件

<!--more-->

## 阻止表单提交

* 设置提交按钮disabled属性
* 调用submit事件

> 1、调用submit方法的时候不会触发submit事件，但是与之对应的有一个重置表单方法reset，调用后会触发reset事件。
> 2、在点击提交按钮的时候，先触发click事件，再触发submit事件
> 3、type=image的表单元素通过表单 elements 属性是获取不到的

## submit事件

### demo1绑定

```js
myform.onsubmit = function(){

   return false;

}
```

这种方式直接返回false即可

### demo2绑定

```js
function addListener(element, type, handler){  
    if (!element) {  
        return;  
    }  
    if (element.addEventListener) {  
        element.addEventListener(type, handler, false);  
    }  
    else {//for ie  
        element.attachEvent("on" + type, handler);  
    }  
}

addListener(obj, 'submit', function(){

     return false

});   
```
以上提交方式在ie下可以阻止表单提交，但是在ff或者chrome却不行，究其原因是addListener中的事件处理函数没有返回值，写了也白写！

> The listener parameter is a EventListener object.Object EventListener：This is an ECMAScript function reference. This method has no return value. The parameter is a Event object.

解决方法：
```js
addListener(obj, 'submit', function(e){

     var e = e || window.event;
     if(e.preventDefault){
        e.preventDefault();
     }else{
        e.returnValue = false;
     }

});
```
脚本处理完自己的工作后由元素处理事件，元素可以通过事件对象判断是否执行默认操作！
