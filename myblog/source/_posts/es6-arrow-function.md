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

### 箭头函数的this指针:

很多文章说,箭头函数的this是固定的,并且函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象,这个一直是似懂非懂。下面做一下疏理。

如下代码:

    function foo(r) {

      setTimeout(() => {
        console.log('=>'+this);
      }, 100);
    }
    foo.call({r:1});
    foo()
    new foo()
    function foo2(r) {
      setTimeout(function() {
        console.log('not arrow:'+this);
      }, 100);
      }
    foo2.call({r:1})
    new foo2()

运行会发现,foo的运行结果 this是变化的, 而 foo2运行时的this,反而是不变的。
原因是什么??


关键在于【定义时】和【运行时】,如何简单区分【定义时】和【运行时】,以一个方法为例:

    function a(){
      b+c
    }

首先,【编译时】会解析a方法是否有语法错误,在这个方法中,但是如果不执行 a(),并不会报错,说明b和c在此时并没有被初始化,即不在【定义时】,
而当执行a() 时,是方法a的【运行时】,但对于a内部的b,c才开始【定义时】,故会检查到b,c不存在报错

将上面代码作下注释:

    function foo(r) {

      //setTimeout的定义是在foo执行时，即如果没有调用foo方法，即setTimeout根本没有定义，此点一定要记清楚
      setTimeout(() => {
        console.log('=>'+this);
      }, 100);
    }

    //foo调用并指定了通过call指定this为{r:1}
    //运行到内部setTimeout开始定义，故此时箭头函数内的this就是 {r:1}
    foo.call({r:1});
    //直接调用 foo执行内部this指向window,setTimeout开始定义时，也就指向了window
    foo()
    //运行foo时，this指向foo内部
    new foo()

    function foo2(r) {

      setTimeout(function() {
        console.log('not arrow:'+this);
      }, 100);
      }

    foo2.call({r:1})
    new foo2()


根据上面注释,说明一定要先确认什么时候才是定义时,什么时候是运行时,foo()是foo本身的运行时,但内部的setTimeout是在foo()调用之后才开始初始化定义的,故foo执行时的this是什么就会影响到setTimeout的箭头函数的this。

####再看两个示例:

此示例说明,箭头函数的this是不变的:

    var adder = {
      base : 1,

      add : function(a) {
        var f = v => v + this.base;
        return f(a);
      },

      addThruCall: function(a) {
        var f = v => v + this.base;
        var b = {
          base : 2
        };
        //这里的b改变了 this但并不会影响到 f函数
        return f.call(b, a);
      }
    };
    console.log(adder.add(1));         // 输出 2
    console.log(adder.addThruCall(1)); // 仍然输出 2（而不是3）


在上面示例中,adder.addThruCall(1)执行时,f才开始初始化定义,并且此时的this指向adder对象,而箭头函数的初化后定义后this指向即固定,所以即使在后面使用【f.call(b, a)】修改this也无效

简单示例:

    var f = () => {
      console.log(this)
    };

    f.call({r:1})//this 依然为window


严格模式下,箭头函数的this 规则将被忽略:


    var f = () => {'use strict'; return this};
    console.log('arror:'+f())

    var f2 = function ()
    {'use strict'; return this};
    console.log('no arrow:'+f2())

