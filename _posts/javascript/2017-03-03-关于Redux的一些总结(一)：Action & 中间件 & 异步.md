---
layout: post
title:  "关于Redux的一些总结(一)：Action & 中间件 & 异步"
date:   2017-03-03
desc: "关于Redux的一些总结(一)：Action & 中间件 & 异步"
keywords: "mancoxu,javascript,react,redux"
categories: [Javascript]
tags: [Javascript]
icon: icon-html
---

在[浅说Flux开发](https://github.com/dwqs/blog/issues/14)中，简单介绍了Flux及其开发方式。Flux可以说是一个框架，其有本身的 Dispatcher 接口供开发者；也可以说是一种数据流单向控制的架构设计，围绕单向数据流的核心，其定义了一套行为规范，如下图：

![img](https://camo.githubusercontent.com/9b62f0659ccad9fa71de3b482ce9dce53b304c58/687474703a2f2f66616365626f6f6b2e6769746875622e696f2f666c75782f696d672f666c75782d73696d706c652d66382d6469616772616d2d776974682d636c69656e742d616374696f6e2d31333030772e706e67)

Redux的设计就继承了Flux的架构，并将其完善，提供了多个API供开发者调用。借着react-redux，可以很好的与React结合，开发组件化程度极高的现代Web应用。本文是笔者近半年使用react+redux组合的一些总结，不当之处，敬请谅解。

## Action
-------------------

Action是数据从应用传递到 store/state 的载体，也是开启一次完成数据流的开始。

以添加一个todo的Action为例:

```javascript

{
    type:'add_todo',
    data:'我要去跑步'
}

```

这样就定义了一个添加一条todo的Action，然后就能通过某个行为去触发这个Action，由这个Action携带的数据(data)去更新store(state/reducer)：

```javascript

store.dispatch({
    type:'add_todo',
    data:'your data'
})

```

`type` 是一个常量，Action必备一个字段，用于标识该Action的类型。在项目初期，这样定义Action也能愉快的撸码，但是随着项目的复杂度增加，这种方式会让代码显得冗余，因为如果有多个行为触发同一个Action，则这个Action要写多次；同时，也会造成代码结构不清晰。因而，得更改创建Action的方式：

```javascript

const ADD_TODO = 'add_todo';

let addTodo = (data='default data') => {
    return {
        type: ADD_TODO,
        data: data
    }
}

//触发action
store.dispatch(addTodo());

```

更改之后，代码清晰多了，如果有多个行为触发同一个Action，只要调用一下函数 `addTodo` 就行，并将Action要携带的数据传递给该函数。类似 `addTodo` 这样的函数，称之为 Action Creator。Action Creator 的唯一功能就是返回一个Action供 `dispatch` 进行调用。

但是，这样的Action Creator 返回的Action 并不是一个标准的Action。在Flux的架构中，一个Action要符合 FSA(Flux Standard Action) 规范，需要满足如下条件：

* 是一个纯文本对象

* 只具备 `type` 、`payload`、`error` 和 `meta` 中的一个或者多个属性。`type` 字段不可缺省，其它字段可缺省

* 若 Action 报错，`error` 字段不可缺省，切必须为 true

`payload` 是一个对象，用作Action携带数据的载体。所以，上述的写法可以更改为：


```javascript

let addTodo = (data='default data') => {
    return {
        type: ADD_TODO,
        payload: {
            data
        }
    }
}

```


在 redux 全家桶中，可以利用 redux-actions 来创建符合 FSA 规范的Action：

```javascript

import {creatAction} from 'redux-actions';

let addTodo = creatAction(ADD_TODO)
//same as
let addTodo = creatAction(ADD_TODO,data=>data)

```

可以采用如下一个简单的方式检验一个Action是否符合FSA标准：

```javascript

let isFSA = Object.keys(action).every((item)=>{
   return  ['payload','type','error','meta'].indexOf(item) >  -1
})

```


## 中间件
----------------------

在我看来，Redux提高了两个非常重要的功能，一是 Reducer 拆分，二是中间件。Reducer 拆分可以使组件获取其最小属性(state)，而不需要整个Store。中间件则可以在 Action Creator 返回最终可供 dispatch 调用的 action 之前处理各种事情，如异步API调用、日志记录等，是扩展 Redux 功能的一种推荐方式。

Redux 提供了 `applyMiddleware(...middlewares)` 来将中间件应用到 createStore。applyMiddleware 会返回一个函数，该函数接收原来的 creatStore 作为参数，返回一个应用了 middlewares 的增强后的 creatStore。


```javascript

export default function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, preloadedState, enhancer) => {
    //接收createStore参数
    var store = createStore(reducer, preloadedState, enhancer)
    var dispatch = store.dispatch
    var chain = []

    //传递给中间件的参数
    var middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    }

    //注册中间件调用链
    chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch)

    //返回经middlewares增强后的createStore
    return {
      ...store,
      dispatch
    }
  }
}

```

创建 store 的方式也会因是否使用中间件而略有区别。未应用中间价之前，创建 store 的方式如下：

```javascript

import {createStore} from 'redux';
import reducers from './reducers/index';

export let store = createStore(reducers);

```

应用中间价之后，创建 store 的方式如下：

```javascript

import {createStore，applyMiddleware} from 'redux';
import reducers from './reducers/index';

let createStoreWithMiddleware = applyMiddleware(...middleware)(createStore);
export let store = createStoreWithMiddleware(reducers);


```

那么怎么自定义一个中间件呢？

根据 [redux 文档](http://redux.js.org/docs/api/applyMiddleware.html)，中间件的签名如下：

```javascript

({ getState, dispatch }) => next => action

```

根据上文的 `applyMiddleware` 源码，每个中间件接收 getState & dispatch 作为参数，并返回一个函数，该函数会被传入下一个中间件的 dispatch 方法，并返回一个接收 action 的新函数。

以一个打印 dispatch action 前后的 state 为例，创建一个中间件示例：

```javascript

export default function({getState,dispatch}) {
    return (next) => (action) => {
        console.log('pre state', getState());
        // 调用 middleware 链中下一个 middleware 的 dispatch。
        next(action);
        console.log('after dispatch', getState());
    }
}

```

在创建 store 的文件中调用该中间件：

```javascript

import {createStore,applyMiddleware} from 'redux';

import reducers from './reducers/index';
import log from '../lib/log';

//export let store = createStore(reducers);

//应用中间件log
let createStoreWithLog = applyMiddleware(log)(createStore);
export let store = createStoreWithLog(reducers);

```

可以在控制台看到输出：

![img](https://camo.githubusercontent.com/521eead386c129654543e1c0a3a0967a6b05ebf0/68747470733a2f2f736661756c742d696d6167652e62302e7570616979756e2e636f6d2f3237322f3330362f323732333036383331312d353762633931636339346331355f61727469636c6578)

可以对 store 应用多个中间件：

```javascript

import log from '../lib/log';
import log2 from '../lib/log2';

let createStoreWithLog = applyMiddleware(log,log2)(createStore);
export let store = createStoreWithLog(reducers);

```
log2 也是一个简单的输出：

```javascript

export default function({getState,dispatch}) {
    return (next) => (action) => {
        console.log('我是第二个中间件1');
        next(action);
        console.log('我是第二个中间件2');
    }
}

```
看控制台的输出：

![img](https://camo.githubusercontent.com/8b5c4fafb7c892c8cecac1bfbb4887822a5e8b59/68747470733a2f2f736661756c742d696d6167652e62302e7570616979756e2e636f6d2f3535302f3039372f3535303039373137382d353762633933646562336439635f61727469636c6578)


**应用多个中间件时，中间件调用链中任何一个缺少 `next(action)` 的调用，都会导致 action 执行失败**

## 异步
-------------------

Redux 本身不处理异步行为，需要依赖中间件。结合 [redux-actions](https://github.com/acdlite/redux-actions) 使用，Redux 有两个推荐的异步中间件：

* [redux-thunk](https://github.com/gaearon/redux-thunk)
* [redux-promise](https://github.com/acdlite/redux-promise)

两个中间件的源码都是非常简单的，redux-thunk 的源码如下：

```javascript

function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;

```

从源码可知，action creator 需要返回一个函数给 redux-thunk 进行调用，示例如下：

```javascript

export let addTodoWithThunk = (val) => async (dispatch, getState)=>{
    //请求之前的一些处理

    let value = await Promise.resolve(val + ' thunk');
    dispatch({
        type:CONSTANT.ADD_TO_DO_THUNK,
        payload:{
            value
        }
    });
};

```
效果如下：

![img](https://camo.githubusercontent.com/a6bf47b16581a98b5ca07ea45efc09637ad9db11/68747470733a2f2f7365676d656e746661756c742e636f6d2f696d672f6256436a4a67)

这里之所以不用 createAction，如前文所说，因为 createAction 会返回一个 FSA 规范的 action，该 action 会是一个对象，而不是一个 function：

```javascript

{
    type: "add_to_do_thunk",
    payload: function(){}
}

```

如果要使用 createAction，则要自定义一个异步中间件。

```javascript

export let addTodoWithCustom = createAction(CONSTANT.ADD_TO_DO_CUSTOM, (val) => async (dispatch, getState)=>{
    let value = await Promise.resolve(val + ' custom');
    return {
        value
    };
});

```

在经过中间件处理时，先判断 action.payload 是否是一个函数，是则执行函数，否则交给 next 处理：

```javascript

if(typeof action.payload === 'function'){
    let res = action.payload(dispatch, getState);
} else {
    next(action);
}

```

而 async 函数返回一个 Promise，因而需要作进一步处理：

```javascript

res.then(
    (result) => {
        dispatch({...action, payload: result});
    },
    (error) => {
        dispatch({...action, payload: error, error: true});
    }
);

```
这样就自定义了一个异步中间件，效果如下：

![img](https://camo.githubusercontent.com/aec658cc0848804495096eeddb24e8e2ebe572e1/68747470733a2f2f7365676d656e746661756c742e636f6d2f696d672f6256436a4950)

当然，我们可以对函数执行后的结果是否是Promise作一个判断：

```javascript

function isPromise (val) {
    return val && typeof val.then === 'function';
}

//对执行结果是否是Promise
if (isPromise(res)){
    //处理
} else {
    dispatch({...action, payload: res});
}

```

那么，怎么利用 redux-promise 呢？redux-promise 是能处理符合 FSA 规范的 action 的，其对异步处理的关键源码如下：

```javascript

action.payload.then(
    result => dispatch({ ...action, payload: result }),
    error => {
        dispatch({ ...action, payload: error, error: true });
        return Promise.reject(error);
    }
)

```

因而，返回的 payload 不再是一个函数，而是一个 Promise。而 async 函数执行后就是返回一个 Promise，所以，让上文定义的 async 函数自执行一次就可以：

```javascript

export let addTodoWithPromise = createAction(CONSTANT.ADD_TO_DO_PROMISE, (val) =>
    (async (dispatch, getState)=>{
        let value = await Promise.resolve(val + ' promise');
        return {
            value
        };
    })()
);

```

结果如下图：

![img](https://camo.githubusercontent.com/c6934ee40a473d7c16ae824e08938f208d16c21f/68747470733a2f2f7365676d656e746661756c742e636f6d2f696d672f6256436a4a50)

示例源码：[redux-demo](https://github.com/dwqs/blog/tree/master/redux-demo)






