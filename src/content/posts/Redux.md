---
title: Redux
date: 2025-07-30
category: 'React'
---

![Screenshot 2025-12-28 at 12.10.47 pm](assets/Screenshot%202025-12-28%20at%2012.10.47%E2%80%AFpm.png)

Store 存储数据

Action修改数据

通过dispatch来修改store数据

Action是一个普通的js对象，包含type和payload 属性

![Screenshot 2025-12-28 at 12.15.24 pm](assets/Screenshot%202025-12-28%20at%2012.15.24%E2%80%AFpm.png)

Reducer

reducer是一个纯函数，将curState 和 action结合起来生成新的state

![Screenshot 2025-12-28 at 12.16.27 pm](assets/Screenshot%202025-12-28%20at%2012.16.27%E2%80%AFpm.png)

`newState = reducer(curState, action)`

reducer返回一个新的state而不是修改原有state

```js
function reducer(curState, action) {
  state.name = action.payload.name // 错误
  return { ...curState, name: action.payload.name } // 正确
}
```

`store.dispatch(action)` 中的action派发给reducer

参数一：现在的state

参数二：dispatch传入的action

返回值：作为store之后存储的state

每调用一次`store.dispatch` 就会触发一次reducer

```js
function reducer(curState = initState, action) {
  // 有数据更新返回新的state

  // 没有数据更新，返回state，第一次数据更新，curState为undefined，返回默认值
  return state
}
```

```js
class App extends PureComponent {
  constructor() {
    super()
    this.state = {
      counter: 0,
    }
  }
  componentDidMount() {
    // subscribe 会在store变化的时候调用。只要 dispatch，就会触发 subscribe。
    store.subscribe(() => {
      const state = store.getState()
      this.setState({ counter: state.counter })
    })
  }
  render() {
    const { counter } = this.state
  }
}
```

`connect` 来自`react-redux`

connect接受两个函数作为参数，返回一个高阶组件

`connect`的返回值是一个高阶函数

![Screenshot 2025-12-29 at 12.10.41 pm](assets/Screenshot%202025-12-29%20at%2012.10.41%E2%80%AFpm.png)

dispatch 的==解耦操作==

![Screenshot 2025-12-29 at 4.07.38 pm](assets/Screenshot%202025-12-29%20at%204.07.38%E2%80%AFpm.png)

通过component来请求数据

component不应该用请求的数据，这样不符合逻辑，因为请求数据应该在redux中完成，而不是在不使用这个数据的组件中完成

![Screenshot 2025-12-29 at 11.45.05 pm](assets/Screenshot%202025-12-29%20at%2011.45.05%E2%80%AFpm.png)

![Screenshot 2025-12-29 at 11.47.20 pm](assets/Screenshot%202025-12-29%20at%2011.47.20%E2%80%AFpm.png)

# redux-thunk

异步处理：

dispatch一个函数，需要用到redux-thunk 这个插件才成dispatch(foo: function)

`foo(dispatch, getState)`

foo接受两个参数，dispatch和getState，可以在then的回掉里面用

![Screenshot 2025-12-30 at 12.26.45 am](assets/Screenshot%202025-12-30%20at%2012.26.45%E2%80%AFam.png)

```js
function fetchHomeMultidataAction(){
	function foo(){
	...
  }
  return foo
}
```

![Screenshot 2025-12-30 at 12.27.54 am](assets/Screenshot%202025-12-30%20at%2012.27.54%E2%80%AFam.png)

![Screenshot 2025-12-30 at 10.21.06 pm](assets/Screenshot%202025-12-30%20at%2010.21.06%E2%80%AFpm.png)

![Screenshot 2025-12-30 at 10.21.18 pm](assets/Screenshot%202025-12-30%20at%2010.21.18%E2%80%AFpm.png)

### combineReducer的原理（了解）

![Screenshot 2025-12-31 at 12.09.18 pm](assets/Screenshot%202025-12-31%20at%2012.09.18%E2%80%AFpm.png)

# redux toolkit

![Screenshot 2025-12-31 at 6.54.15 pm](assets/Screenshot%202025-12-31%20at%206.54.15%E2%80%AFpm.png)

![Screenshot 2025-12-31 at 7.18.39 pm](assets/Screenshot%202025-12-31%20at%207.18.39%E2%80%AFpm.png)
