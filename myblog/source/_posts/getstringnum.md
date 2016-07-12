---
title: 记一次面试中,遇到的一个问题:统计一个字符串中,以空格分格的词的数量,并以JSON的格式返回
date: 2016-07-12 17:12:26
tags: javascript
---


面试题如下:

统计一个字符串中,**字符串内容为随机**,**以空格分格**的词的数量,并以JSON的格式返回

字符串示例:

    //字符串是随机内容
    var a="this is a string and is a xx ";

返回格式为:

    {
        this:1
        is:2
        a:2
    }

面试时第一次想到的答案:

        var a="this is string test which is then";
        var b={};
        var c =a.split(" ");
        for(var t=0;t<c.length;t++){
            if(!b[c[t]]){
                b[c[t]]=1;
            }else{
                b[c[t]]++;
            }
        }
        console.log(b)

初看没有什么问题,却忽略了,字符串中可能会存在,创建对象时,对象已有的属性名,如


    var a="this is string test which is then prototype constructor";
            var b={};
            var c =a.split(" ");
            for(var t=0;t<c.length;t++){
                if(!b[c[t]]){
                    b[c[t]]=1;
                }else{
                    b[c[t]]++;
                }
            }
            console.log(b)

这种情况下,返回的json就会有问题。

当时还没想明白,后来做了第二次修改,添加了一个类型判断:

    var b={};
    var c =a.split(" ");
    for(var t=0;t<c.length;t++){
        if(!b[c[t]] || typeof(b[c[t]])!=="number"){
            b[c[t]]=1;
        }else{
            b[c[t]]++;
        }
    }
    console.log(b)

这样看好象是对了,但是实际上,还有一个严重问题,如果字符串中存在 **__proto__**的内容,则无法设置 **__proto__**的值,因为**__proto__的值在默认在初始化后,是只读的**。

于是就有了第三个方案:

        var a="constructor constructor __proto__ __lookupSetter__ hasOwnProperty prototype";
        //通过JSON.parse初始化对象时,设置__proto__
        var b=JSON.parse("{\"__proto__\":null}");
        var c =a.split(" ");
        for(var t=0;t<c.length;t++){
           // console.log(c[t]);
           // console.log(typeof(b[c[t]]));

            if(!b[c[t]] || typeof(b[c[t]])!=="number"){
                b[c[t]]=1;
            }else{
                b[c[t]]++;
            }
        }
        console.log(b);

这样解决了__proto__不能修改的问题,但是也不完美,因为如果字符串中没有__proto__,但返回的JSON中,还是会有这个属性

后来又尝试了用数据转换,但还是不理想,最后还是用到了面试时,想到的思路,使用Object.defineProperty修改对象属性的特性,允许修改和枚举,为什么最开始没有使用这个方案,是最开始钻牛头尖了,想不使用新特性来做此事,事实证明,还是得用:

    var a="constructor constructor mytest __proto__ __lookupSetter__ __proto__ hasOwnProperty prototype";
    var b={}
    var c =a.split(" ");
    for(var t=0;t<c.length;t++){
        if(!b[c[t]] || typeof(b[c[t]])!=="number"){
            Object.defineProperty(b, c[t], {
                enumerable: true,
                configurable: true,
                writable: true,
                value: 1
            });
        }else{
            b[c[t]]++;
        }
    }
    //在console时,在chrome 控制台中不会显示 __proto__,在jsbin的打印中会显示出来
    console.log(b);

    //但是在遍历的时候会遍历到__proto__
    for (var pro in b) {
        console.log(pro);
        console.log(b[pro])
        console.log('------')


最后,再用ES6简化一下代码:

    var a="constructor constructor mytest __proto__ __lookupSetter__ __proto__ hasOwnProperty prototype";
    var b={};
    a.split(" ").filter(x=>x).forEach(key=>{
      if(!b[key] || typeof(b[key])!=="number"){
        Object.defineProperty(b, key, {
                enumerable: true,
                configurable: true,
                writable: true,
                value: 1
            });
      }else{
        b[key]++
      }


    })
    //在console时,在chrome 控制台中不会显示 __proto__,在jsbin的打印中会显示出来
    console.log(b);

    //但是在遍历的时候会遍历到__proto__
    for (var pro in b) {
        console.log(pro);
        console.log(b[pro]);
        console.log('------');
    }




----------更简单的方式------------

使用 Object.create(null)创建完全的空对象


    var a="constructor constructor mytest __proto__ __lookupSetter__ __proto__ hasOwnProperty prototype";
    //创建没有原型的对象
    //通过这种方式,最简单,但在c
    var b=Object.create(null);
    a.split(" ").filter(x=>x).forEach(key=>{
      if(!b[key]){
        b[key]=1;
      }else{
        b[key]++
      }


    })
    //在console时,在chrome 控制台中不会显示 __proto__,在jsbin的打印中会显示出来
    console.log(b);

    //但是在遍历的时候会遍历到__proto__
    for (var pro in b) {
        console.log(pro);
        console.log(b[pro]);
        console.log('------');
    }

再次更新,使用 map的方式:


   b = (str) => str.split(/\s+/g).reduce((a, e) => a.set(e, a.has(e) ? a.get(e) + 1 : 1), new Map());

    //上面代码等同于:
    b = (str) => str.split(/\s+/g).reduce((a, e) => a.set(e, a.has(e) ? a.get(e) + 1 : 1), new Map());
   var r=b(a);
   console.log(r);

   r.forEach(function (val, key, array) {
       console.log(key+':'+val)
   })

这种方法相对复杂,需要了解一些以下API:

Map的用法:

现在很多时候会将Map来代替 var a={},来创建对象

常用的API

 var m=new Map();
 m.get(key);
 m.set(key,val)
 map.has(key)
 map.delete(key)
 map.forEach() 与 Array.forEach用法相同

数组 API reduce的用法

reduce :累加器

    arr.reduce(callback,[initialValue])

参数

callback

执行数组中每个值的函数，包含四个参数

>previousValue

>上一次调用回调返回的值，或者是提供的初始值（initialValue）

>currentValue

>数组中当前被处理的元素

>index

>当前元素在数组中的索引

>array

>调用 reduce 的数组

initialValue

作为第一次调用 callback 的第一个参数。

示例一:

    var a=[1,2,3,4];


    a.reduce(function(previousValue,currentValue) {
        //此示例中,第一个previousValue 为1,currentValue为2
        //第二次循环中, previousValue为 上一次循环中return 的值,currentValue则3,以此类推
      return previousValue+currentValue;
    })



示例二:

    var a=[1,2,3,4];
    a.reduce(function(previousValue,currentValue) {
        //此示例中,第一个previousValue 为15,currentValue为1
       //第二次循环中, previousValue为 上一次循环中return 的值,currentValue则3,以此类推
       return previousValue+currentValue;
    },15)

与示例一不同,此处设置了 initialValue的值为 15,则在循环开始,则以此初始化为开始,以此类推

示例三,初始值不仅可以是数字,也可以是任何你想传数的格式:

    var a=[1,2,3,4];
    a.reduce(function(previousValue,currentValue) {
        //previousValue为数组,并且将后续的值都添加到这个数组中
        //第一次return 时,将数组return ,后面所有的累加都是在这个数组上不断return 不断类加
       return previousValue.push(currentValue);
    },[])

根据以上示例,你可以对数据做更多处理,代码也更简洁


*总结:这个题,重点不在统计上,而是在使用JSON来返回数据上,并且要考虑到对象的属性,有些是不可修改和枚举的,故如果遇到与内置属性同名的字段,就需要允许其修改,关键点就在 Object.defineProperty的应用和 Object.create(null)的用法:

    Object.defineProperty(b, c[t], {
                    enumerable: true,
                    configurable: true,
                    writable: true,
                    value: 1
                });

    var b=Object.create(null);

最终示例:

http://jsbin.com/xewavet/11/edit?html,js,output

http://jsbin.com/xocasu/7/edit?html,js,console,output

http://jsbin.com/sohepe/4/edit?html,js,output