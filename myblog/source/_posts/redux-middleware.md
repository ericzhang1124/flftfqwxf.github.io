---
title: redux middleware 中间件
date: 2016-07-09 10:49:05
tags: redux
---
### redux middleware是什么?

简单的说,middleware就是给redux提供额外的外置功能,可以理解为插件,大至有以下特点:
1)它是全局的,middleware是添加到redux的实例上,而一般一个应用只有一个redux store,所以在一个sotre上添加了某个middleware,所有这个应用都会经过这个middleware
2)middleware,可以理解为插件,是可插拔的,定义了一个middleware,如果不使用,就与应用程序没有任何关系

### middleware 如何实现?

写一个 middleware,需要理解以下知识:

1)有一定函数式编程理解
2)ES6箭头函数嵌套



#### 定义中间件:

    var thunkMiddleware = function ({dispatch, getState}) {
        // console.log('Enter thunkMiddleware');
        //此处的next 是 下一个插件,或 store.dispatch
        return function (next) {
            // console.log('Function "next" provided:', next);
            // action 为 调用 store.dispatch时的 action
            //store.dispatch调用时,将会使用这个方法
            return function (action) {
                console.log('this is dispatch')
              //  console.log('Handling action:', action);
                //如果是function则表示为 actioncreator 异步
                if (typeof action === 'function') {
                    return action(dispatch, getState)
                } else {
                    //如果action是一个action对象则调用到 reducer
                    next(action)
                }
            }
        }
    }


#### 初始化实例:

    import {createStore, combineReducers, applyMiddleware} from 'redux'
    //applyMiddleware(thunkMiddleware,logger) 返回一个方法,并以createStore为参数,执行后返回一个store实例
    //其主要作用是把中间件参数值,放入到返回的函数中
    //这是函数式编程的写法,每次调用都返回一个新的函数,并把每次调用的参数都缓存到每个新返回的函数中
    //在这里相当于重写了 createStore方法,之后使用 finalCreateStore来代替createStore实例化store
    //之所以要重写 createStore,根据 applyMiddleware中源码得知,是为了重写dispatch
    const finalCreateStore = applyMiddleware(thunkMiddleware,logger)(createStore)

#### 调用 dispatch:

    var asyncSayActionCreator_1 = function (message) {
        return function (dispatch) {
            setTimeout(function () {
                console.log(new Date(), 'Dispatch action now:')
                dispatch({
                    type: 'SAY',
                    message
                })
            }, 2000)
        }
    }
    //在此处调用的 dispath,不是redux默认的dispatch,而是applyMiddleware 重构后的 dispatch
    store_0.dispatch(asyncSayActionCreator_1('Hi'))

### 中间件的固定格式:

    /**
     * {dispatch,getState} 为createStore后的 store实例能数中的两个
     * next  是 下一个插件的 action,或 store.dispatch
     * action
     * 每个插件都必须执行 next(action) 以继续后续的操作
     */
    const logger=({dispatch,getState})=>next=>action=>{
        console.log('loger:',action.type);
        let result=next(action);
       console.log('loger',getState());

    }
    //此代码与上面等同
    const logger=store=>next=>action=>{
            console.log('loger:',action.type);
            let result=next(action);
           console.log('loger',store.getState());

        }