---
title: Object.defineProperty用法
date: 2016-06-28 17:45:24
tags: javascrpt
---

### 官网描述如下:
Object.defineProperty() 方法直接在一个对象上定义一个新属性，或者修改一个已经存在的属性， 并返回这个对象。

语法EDIT
Object.defineProperty(obj, prop, descriptor)
参数

obj
需要定义属性的对象。
prop
需被定义或修改的属性名。
descriptor
需被定义或修改的属性的描述符。
描述
该方法允许精确添加或修改对象的属性。一般情况下，我们为对象添加属性是通过赋值来创建并显示在属性枚举中（for...in 或 Object.keys 方法）， 但这种方式添加的属性值可以被改变，也可以被删除。而使用 Object.defineProperty() 则允许改变这些额外细节的默认设置。例如，默认情况下，使用  Object.defineProperty() 增加的属性值是不可改变的。

对象里目前存在的属性描述符有两种主要形式：数据描述符和存取描述符。数据描述符是一个拥有可写或不可写值的属性。存取描述符是由一对 getter-setter 函数功能来描述的属性。描述符必须是两种形式之一；不能同时是两者。

数据描述符和存取描述符均具有以下可选键值：

configurable
当且仅当该属性的 configurable 为 true 时，该属性才能够被改变，也能够被删除。默认为 false。
enumerable
当且仅当该属性的 enumerable 为 true 时，该属性才能够出现在对象的枚举属性中。默认为 false。
数据描述符同时具有以下可选键值：

value
该属性对应的值。可以是任何有效的 JavaScript 值（数值，对象，函数等）。默认为 undefined。
writable
当且仅当仅当该属性的writable为 true 时，该属性才能被赋值运算符改变。默认为 false。
存取描述符同时具有以下可选键值：

get
一个给属性提供 getter 的方法，如果没有 getter 则为 undefined。该方法返回值被用作属性值。默认为undefined。
set
一个给属性提供 setter 的方法，如果没有 setter 则为 undefined。该方法将接受唯一参数，并将该参数的新值分配给该属性。默认为undefined。
记住，这些选项不一定是自身属性，如果是继承来的也要考虑。为了确认保留这些默认值，你可能要在这之前冻结Object.prototype，明确指定所有的选项，或者将__proto__属性指向null。



### 下面作一下解释和对比:

#### 传统给对象设置属性的方式:

    var obj={};
    obj.o=1;

    // 等同于 :
    Object.defineProperty(o, "a", {
      value : 1,
      writable : true,
      configurable : true,
      enumerable : true
    });


1)属性是可以任意修改和删除的
2)可以通过for in ,Object.keys来循环遍历的
3)Object.assign可以复制到该属性


使用Object.defineProperty添加属性:

    var obj={};
    Object.defineProperty(obj,"username",{
      value:'name',
      //为true时，才允许被删除
      configurable:false,
      //为true时，该属性才能够出现在对象的枚举属性中
      //出现在对象的枚举属性中，才可以被Object.assign复制
      //了现在对象的枚举属性中，才可能出现在for in循环中
      enumerable:false,
      //为true时，才允许被修改
      writable:true
    })
    obj.username="update"
    //delete obj.username
    //
    console.log(obj.username)
    //如果 [enumerable]时为false,则无法在循环中
    //即不能在for in 和 Object.keys()中获取到
    for(var item in obj){
      console.log(item)
    }
    //如果 [enumerable]时为false,复制的对象不会有username属性，因为 Object.assign只复制枚举属性中的属性
    
    var copyObj=Object.assign({},obj);
    console.log(copyObj.username)


你可以修改<a class="jsbin-embed" href="http://jsbin.com/yaduxa/embed?html,js,console">在线示例1</a>,查看不同效果

 
 
**总结:可以通过Object.defineProperty来设置私有属性,不允许被修改,被删除,被复制等操作**

### 存取描述符

get
一个给属性提供 getter 的方法，如果没有 getter 则为 undefined。该方法返回值被用作属性值。默认为undefined。
set
一个给属性提供 setter 的方法，如果没有 setter 则为 undefined。该方法将接受唯一参数，并将该参数的新值分配给该属性。默认为undefined。

Object.defineProperty最常用的功能是,通过GET,SET来实现数据的动态绑定,现在流行的数据响应式,双向数据绑定皆是基于此,如:

html:

    <!DOCTYPE html>
    <html >
    <head>

      <meta charset="utf-8">
      <title> es6  DEMO</title>
    </head>
    <body >


      Object.defineProperty 存储描述符 DEMO

      <br><br><br>
      <div class="automatic">3333</div>
      <input type="text" id="myname">

    </body>
    </html>


js:

    var obj={};
    var bVal;

    Object.defineProperty(obj,"username",{
      //当存在get,set时，不能同时存在value,两者不能同时存在
     // value:'name',
      //为true时，才允许被删除
      configurable:true,
      //为true时，该属性才能够出现在对象的枚举属性中
      //出现在对象的枚举属性中，才可以被Object.assign复制
      //了现在对象的枚举属性中，才可能出现在for in循环中
      enumerable:true,
      //为true时，才允许被修改
      //当存在get,set时，不能同时存在value,两者不能同时存在
     // writable:true,
      get:function(){
        console.log('get val');
        return bVal
      },set:function(newVal){
        console.log('set val');
        var container=document.querySelector('.automatic');
        container.innerHTML=newVal;
        bVal=newVal
      }
    });
    //实现简单的数据响应式绑定
    document.addEventListener("DOMContentLoaded", function(event) {
      var myname=document.querySelector('#myname');
        myname.
        addEventListener('input',function(){
          obj.username=this.value
        })
    });



查看<a class="jsbin-embed" href="http://jsbin.com/yimezug/embed?html,js,console,output">在线示例</a>

注意:Object.definePropery,的get,set只支持IE9及以及。
而要在IE8以及下实现动态绑定需要使用 VBSCRIPT


