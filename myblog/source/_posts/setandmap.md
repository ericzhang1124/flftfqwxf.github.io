---
title: javascript 数据结构 Set Map
date: 2016-07-13 17:37:10
tags: es6 Map Set
---
在es6中,多了两种数据结构 Set和Map

先说Set,Set是类似数组的数据结构,有以下特点:

1) Set只有键值,没有键名
2) Set的值是唯一的,如果重复则只存一个
3) Set中 NaN被认为是不相同的值
4) 可以将数组或类似数组的集合转换成Set
5) IE兼容性不好
6) Set不能使用索引来获取值,只能通过遍历,如forEach,keys等

    //set 类似数组的数据结构, set没有键名，只有值
    //set 不能像数组一样通过索引获取到值，只能遍历
    //与数组不同的是，它的值必须是唯一的，如果有重复，则只存储一个
    var set = new Set([1, 2, 3, 4, 4]);

    //常用API
    set.add({a:3})
    console.log([...set])
    //判断是否存在
    console.log(set.has(1))

    set.clear()
    //set.delete()
    var divs=new Set(document.querySelectorAll(".e"))

    //在jsbin不会输出，但是在单写在网页中会输出
    divs.forEach(val=>{console.log(val.innerHTML)})

###Map

Object和MAP的比较:

Object和Map类似的一点是,它们都允许你按键存取一个值,都可以删除键,还可以检测一个键是否绑定了值.因此,一直以来,我们都把对象当成Map来使用,不过,现在有了Map,下面的区别解释了为什么使用Map更好点.

>一个对象通常都有自己的原型,所以一个对象总有一个"prototype"键.不过,现在可以使用map = Object.create(null)来创建一个没有原型的对象.
一个对象的键只能是字符串,但一个Map的键可以是任意值.
你可以很容易的得到一个Map的键值对个数,而只能跟踪一个对象的键值对个数

1) JSON对象的键名只能是字符串,如果键名是对象则会被强制 toString(),如:

    var data = {};
    var element = document.getElementById("myDiv");

    data[element] = metadata;
    data["[Object HTMLDivElement]"] // metadata

上面代码原意是将一个DOM节点作为对象data的键，但是由于对象只接受字符串作为键名，所以element被自动转为字符串[Object HTMLDivElement]。

2) 如果对同一个键多次赋值，后面的值将覆盖前面的值。

3) 只有对同一个对象的引用，Map结构才将其视为同一个键。这一点要非常小心。

    var map = new Map();

    map.set(['a'], 555);
    map.get(['a']) // undefined

4) 常用API与Set一致

set与map的区别:

set是只有键值的数据结构,而map则是必须包括 键名和键值
set的值是唯一的,map如果重复则后面覆盖前面

###与其他数据结构的互相转换
（1）Map转为数组

前面已经提过，Map转为数组最方便的方法，就是使用扩展运算符（...）。

    let myMap = new Map().set(true, 7).set({foo: 3}, ['abc']);
    [...myMap]
    // [ [ true, 7 ], [ { foo: 3 }, [ 'abc' ] ] ]

（2）数组转为Map

将数组转入Map构造函数，就可以转为Map。

    new Map([[true, 7], [{foo: 3}, ['abc']]])
    // Map {true => 7, Object {foo: 3} => ['abc']}

（3）Map转为对象

如果所有Map的键都是字符串，它可以转为对象。

    function strMapToObj(strMap) {
      let obj = Object.create(null);
      for (let [k,v] of strMap) {
        obj[k] = v;
      }
      return obj;
    }

    let myMap = new Map().set('yes', true).set('no', false);
    strMapToObj(myMap)
    // { yes: true, no: false }
    （4）对象转为Map

    function objToStrMap(obj) {
      let strMap = new Map();
      for (let k of Object.keys(obj)) {
        strMap.set(k, obj[k]);
      }
      return strMap;
    }

    objToStrMap({yes: true, no: false})
    // [ [ 'yes', true ], [ 'no', false ] ]


（5）Map转为JSON

Map转为JSON要区分两种情况。一种情况是，Map的键名都是字符串，这时可以选择转为对象JSON。

    function strMapToJson(strMap) {
      return JSON.stringify(strMapToObj(strMap));
    }

    let myMap = new Map().set('yes', true).set('no', false);
    strMapToJson(myMap)
    // '{"yes":true,"no":false}'
    另一种情况是，Map的键名有非字符串，这时可以选择转为数组JSON。

    function mapToArrayJson(map) {
      return JSON.stringify([...map]);
    }

    let myMap = new Map().set(true, 7).set({foo: 3}, ['abc']);
    mapToArrayJson(myMap)
    // '[[true,7],[{"foo":3},["abc"]]]'

（6）JSON转为Map

JSON转为Map，正常情况下，所有键名都是字符串。

    function jsonToStrMap(jsonStr) {
      return objToStrMap(JSON.parse(jsonStr));
    }

    jsonToStrMap('{"yes":true,"no":false}')
    // Map {'yes' => true, 'no' => false}

但是，有一种特殊情况，整个JSON就是一个数组，且每个数组成员本身，又是一个有两个成员的数组。这时，它可以一一对应地转为Map。这往往是数组转为JSON的逆操作。

    function jsonToMap(jsonStr) {
      return new Map(JSON.parse(jsonStr));
    }

    jsonToMap('[[true,7],[{"foo":3},["abc"]]]')
    // Map {true => 7, Object {foo: 3} => ['abc']}


参考资源:

http://es6.ruanyifeng.com/#docs/set-map
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Map