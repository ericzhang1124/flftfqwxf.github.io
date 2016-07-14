---
title: es6 class总结
date: 2016-07-14 15:58:49
tags: es6 class
---
ES6中,创建类将更接近其他语言,使用class关键词,其本质可以理解为语法糖。所有有class关键词定义的类,在ES5中使用function同样可以定义。

下面总结一下class的特点及与function定义类的不同:

语法:

    class Bar {
      // class内的内容强制要求为严格模式
      //如果在class中，没有定义constructor，则会默认创建一个空的 constructor函数
      constructor(){
        //正常情况下，由于严格模式要求，变量必须显示声明
        //此写法会报错，ReferenceError: myname is not defined
        //但由于浏览器目前对ES6的语法支持不完善，在使用 babel等进行转义后，并不会报错，
        myname=4
        console.log('this is constructor'+myname)
      }
      doStuff() {
        console.log('stuff');
      }

      pro(){
        this.x=1;
        this.y=2;
      }
    }

    var b = new Bar();
    console.log(b)
    b.pro()
    console.log(b.hasOwnProperty('x'))

    //类表达式
    const myclass=class m {

      getName(){
       return m.name
      }

    }

    var c=new myclass()
    console.log(c.getName())








1) 类和模块的内部,默认为严格模式,比如在严格模式,变量必须显示声明,否则会报错,ReferenceError: myname is not defined,示例中【myname】处,但由于浏览器目前对ES6的语法支持不完善，在使用 babel等进行转换为ES5后，并不会报错。
2) constructor 构建函数,在 new myclass()时,会调用一次,一个类只能拥有一个名为 constructor 的方法，否则会抛出 SyntaxError 异常。
3) constructor 默认返回实例对象（即this），完全可以指定返回另外一个对象

    class Foo {
      constructor() {
        return Object.create(null);
      }
    }

    new Foo() instanceof Foo

4) 如果class中没有显示声明constructor,则会默认生成一个空的constructor函数
5) ES5中,new一个实例,默认其指向的constructor为,定义的function(),故:

    class Cat {
        constructor(){
            this.x=1
        }
    }

    //与上面代码等同
    function (){
        this.x=1
    }
5) 使用 class 定义的属性或方法,都是在其prototype上定义,只有通过[this]定义的属性才会在实例上,同时由于所有方法默认都定义在原型上,所以可以通过修改原型上的方法来为实例添加或修改方法:

    class a {
        func(){
            this.x=1
            this.y=2
        }

    }

    function a() {

    }
    a.prototype.func=function(){

    }
6) class与 function定义的类不同, function定义时,会有变量提升,而class不会,即class的定义一定要在实例化之前,否则会报错

7) function和class皆可以通过__proto__来修改原型的方法,达到修改实例的目录,但不推荐使用,推荐使用继承的方式,下面会讲到,  [在线示例](http://jsbin.com/yodolum/12/edit?html,js,console)

    class Bar {
      constructor(){

      }
      getName(){
        return 'myName'
      }
    }

    var a=new Bar()
    var b=new Bar()
    console.log('a:'+a.getName())
    console.log('b:'+b.getName())

    //修改原型上的方法，两个实例上调用都会被修改
    a.__proto__.getName=function(){

      return 'getName is modify'
    }


    console.log('a:'+a.getName())
    console.log('b:'+b.getName())


8) class和 function 上,除了 通过[this]定义在实例上的属性外,其他都是不可枚举的,至于哪些是不可枚举的属性(不可枚举的属性的判断及使用,在其他文章中讨论),[在线示例](http://jsbin.com/jekifo/2/edit?html,js,console)

    class cat {
      constructor(){
        this.x=1
        this.y=2
      }
    }
    const b=new cat()
    console.log(Object.keys(b))//["x", "y"]

    function fish(){

      this.fx=1;
      this.fy=2
    }
    fish.fh=5
    fish.prototype.a=5
    console.log(Object.keys(new fish()))//["fx", "fy"]


### class的继承

 通过使用 **extend** 关键词,[在线示例](http://jsbin.com/rexumaz/2/edit?html,js,console)

     class animal {

      constructor(sex,age,color,weight){
        this.sex=sex;
        this.age=age;
        this.color=color;
        this.weight=weight;
      }
      getSex(){
        return this.sex;
      }

    }

     class Cat extends animal{
      constructor(sex,age,color,weight,category){
        super(sex,age,color,weight)
        this.category=category
      }
       getPro(){

         return Object.keys(this)
       }
    }

    const cat = new Cat('male',4,'black','4kg','china')

    console.log(cat.getSex())
    console.log(cat.getPro())

**ES5的继承，实质是先创造子类的实例对象this，然后再将父类的方法添加到this上面（Parent.apply(this)）。ES6的继承机制完全不同，实质是先创造父类的实例对象this（所以必须先调用super方法），然后再用子类的构造函数修改this。**

如果子类没有定义constructor方法，这个方法会被默认添加，代码如下。也就是说，不管有没有显式定义，任何一个子类都有constructor方法。

    constructor(...args) {
      super(...args);
    }


super的用法和注意事项,[在线示例](http://jsbin.com/dacicoj/9/edit?html,js,console):

    class a{
      constructor(){
        console.log('this is a');
        this.x='x';
        this.y='y';
      }
      get(pro){
        return this[pro];
      }
    }
    class b extends a{

      constructor(){
        //直接调用时，相当于父类的constructor调用
        super();
        //作为对象调用时，可以引用父类实例的属性和方法，也可以引用父类的静态方法
        //注意:使用super无法引用父级通过[this]设置的值，
        console.log(super.x)//undefine
        //只能引用父类非 [this]下的方法或属性
        //而在class中，只能定义函数，无法定义变量值，
        //故只能引用父有的方法
         console.log(super.get('x'))//x
         console.log(this.get())
      }
      get(){
        return 'sub get'
      }
    }
    const c=new b()
    //console.log(c.get())

1) class只能定义方法,不能定义属性,定义属性只能通过[this],区别在于 class定义的方法在子类的中,可以通过 super.functionName()的方式来引用,而通过 【this】定义的则不能
2) 直接调用时[super(...args)]，相当于父类的constructor调用
3) 作为对象调用时，可以引用父类实例的属性和方法，也可以引用父类的静态方法
4) 注意:使用super无法引用父级通过[this]设置的值
5) 如果子类定义了与父类同名的方法名,可以用 super.xxx和 this.xxx来区分父类和子类的同名方法
6) 判断一个类是否继承另一个类,Object.getPrototypeOf方法可以用来从子类上获取父类。

    Object.getPrototypeOf(ColorPoint) === Point
    // true

### class 继承不同类型:

使用class继承和使用es5继承的区别:
使用class继承:
先

    function r(){
      console.log('this is function')
    }

    class a extends r {
      constructor(){
        super()
      }
    }
    const b=new a()

    //继承原生类在 babel模式下会报错，在javascript模式下，CHROME正常
    class u extends Date {
      constructor(){
        super()
       console.log(super.getFullYear())
      }
    }

    const subDate=new u()