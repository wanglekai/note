## redux 核心原理

[toc]

---

### 基础实现

核心 AIP `createStore`  创建 store 对象

```js
createStore(reducer, preloadedState, enhancer)
// 返回 
{ getState, dispatch, subscribe }
```

```js
function createStore (reducer, preloadedState) {
    var currentState = preloadedState
    // 订阅者函数
    var currentListeners = [];
    // 获取状态
    function getState () {
        return currentState
    }
    // 触发 action
    function dispatch (action) {
        currentState = reducer(currentState, action)
        // 调用订阅者
        for (var i = 0, len = currentListeners.length; i < len; i++) {
            var listenr = currentListeners[i]
            listenr();
        }
    }
    // 订阅状态
    function subscribe (listener) {
        currentListeners.push(listener)
    }
    return {
        getState,
        dispatch,
        subscribe
    }
}
```
```html
<button id="increment">+</button>
<span id="curNum">0</span>
<button id="decrement">-</button>
<script src="./myRedux.js"></script>

<script>
    function reduer(state, action) {
    switch (action.type) {
            case "increment":
        return state + 1;
            case "decrement":
        return state - 1;
            default:
        return state;
        }
    }

    var store = createStore(reduer, 0);
    store.subscribe(function () {
        document.getElementById("curNum").innerText = store.getState();
    });

    document.getElementById("increment").onclick = function () {
        store.dispatch({ type: "increment" });
    };

    document.getElementById("decrement").onclick = function () {
        store.dispatch({ type: "decrement" });
    };
</script>
```

### 参数类型约束

```js
function createStore (reducer, preloadedState) {

    // 约束 reducer 参数类型
    if (typeof reducer !== 'function') throw new Error('reducer 必须是个函数');

    var currentState = preloadedState
    // 订阅者函数
    var currentListeners = [];
    // 获取状态
    function getState () {
        return currentState
    }
    // 触发 action
    function dispatch (action) {
        /*
            action 必须是 对象
            必须具有 type 属性
        */
        if (!isPlainObject(action)) throw new Error('action 必须是个对象');
        if (typeof action.type === 'undefined') throw new Error('action 对象需要有 type 属性');

        currentState = reducer(currentState, aciton)
        // 调用订阅者
        for (var i = 0, len = currentListeners.length; i < len; i++) {
            var listenr = currentListeners[i]
            listenr();
        }
    }
    // 订阅状态
    function subscribe (listener) {
        currentListeners.push(listener)
    }
    return {
        getState,
        dispatch,
        subscribe
    }
}

// 判断 obj 参数是否是对象

function isPlainObject (obj) {
    if (typeof obj !== 'object' || obj === null) return false;

    return Object.getPrototypeOf({}) === Object.getPrototypeOf(obj)
}
```
### Redux源码实现：Enhancer （增强）


```js
function createStore (reducer, preloadedState, enhancer) {

    // 约束 reducer 参数类型
    if (typeof reducer !== 'function') throw new Error('reducer 必须是个函数');

    if (typeof enhancer !== 'undefined') {
        if (typeof enhancer !== 'function') throw new Error('enhancer 必须是函数！')
        return enhancer(createStore)(reducer, preloadedState)
    }

    var currentState = preloadedState
    // 订阅者函数
    var currentListeners = [];
    // 获取状态
    function getState () {
        return currentState
    }
    // 触发 action
    function dispatch (action) {
        /*
            action 必须是 对象
            必须具有 type 属性
        */
        if (!isPlainObject(action)) throw new Error('action 必须是个对象');
        if (typeof action.type === 'undefined') throw new Error('action 对象需要有 type 属性');

        currentState = reducer(currentState, action)
        // 调用订阅者
        for (var i = 0, len = currentListeners.length; i < len; i++) {
            var listenr = currentListeners[i]
            listenr();
        }
    }
    // 订阅状态
    function subscribe (listener) {
        currentListeners.push(listener)
    }
    return {
        getState,
        dispatch,
        subscribe
    }
}

// 判断 obj 参数是否是对象

function isPlainObject (obj) {
    if (typeof obj !== 'object' || obj === null) return false;

    return Object.getPrototypeOf({}) === Object.getPrototypeOf(obj)
}
```

```html
<button id="increment">+</button>
<span id="curNum">0</span>
<button id="decrement">-</button>
<script src="./myRedux.js"></script>

<script>

    function enhancer (createStore) {
        return function (reducer, perloadedState) {
            var store = createStore(reducer, perloadedState)
            var dispatch = store.dispatch;
            
            function _dispatch (action) {
                if (typeof action === 'function') {
                    return action(dispatch)
                }
                dispatch(dispatch)
            }
            return {
                ...store,
                dispatch: _dispatch
            }
        }
    }

    function reduer(state, action) {
        switch (action.type) {
            case "increment":
                return state + 1;
            case "decrement":
                return state - 1;
            default:
                return state;
        }
    }

    var store = createStore(reduer, 0, enhancer);
    store.subscribe(function () {
        document.getElementById("curNum").innerText = store.getState();
    });

    document.getElementById("increment").onclick = function () {
        // store.dispatch({ type: "increment" });
        store.dispatch(function (dispatch) {
            setTimeout(function () {
                dispatch({type: 'increment'})
            }, 1000)
        })
    };

    document.getElementById("decrement").onclick = function () {
        store.dispatch({ type: "decrement" });
    };
</script>
```