---
title: es6 箭头函数详解
date: 2016-07-08 17:20:01
tags: javascript es6
---
之前对es6的箭头函数的使用总是生硬和一知半解,今天疏理一下。

格式的本质:

箭头函数的左边是要传递给function的参数
箭头函数的右边是要执行的函数

右侧不同的区别,比较三种不同:

    var a=1;
    var b=a=>a+1;

上面相当于:

    function b(a) {
        return a+1;
    }

右方带括号与不带括号的区别:

    var a2=1;
    var b2=a2=>{a2+1};

上面相当于:

    function b2(a2) {
        a2+1;
    }
即,如果右侧出现{}的代码块,则不会自动return 右侧function最后的值


右方带圆括号的不同:
    var a3=1;
    var b3=a3=>(a3);


上面相当于:

    function b(a) {
        return a;
    }
之所以要使用(),是在如果返回值是一个带{}的对象时,{}被识别成一个代码块
如:

    var a4=1;
    var b4=a4=>({r:1,a4:a4});

错误格式:

    var a4=1;
    var b4=a4=>{r:1,a4:a4};

箭头函数左侧:

如果是没有或多个参数需要传递,则使用():

    var r=()=>5;
    var r1=(a,b)=>a+b;

上面相当于:

    function r1() {
        return 5;
    }
    function r2(a, b) {
        return a + b;
    }

复杂的箭头函数:

    const f1=()=>{return 'first'};
    const f2=()=>{return 'second'};
    const f3=()=>{return 'three'}
    const result=r1=>r2=>r3=>{
     console.log(r1())
     console.log(r2())
     console.log(r3())
    }

    //console.log(result)
    console.log(result(f1)(f2))

上面相当于:

    function result(r1) {
    return function (r2) {
      return function (r3) {
        console.log(r1());
        console.log(r2());
        console.log(r3());
      };
    };
    }

