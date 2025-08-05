---
title: Lazy Initilization of useState
date: 2025-07-28
category: 'react'
---

# useState 的lazy initialization

这是一个很经典的问题，React 的 `useState` 在接收一个函数作为参数时有**两种不同的行为**，你需要根据**是否包裹在函数体内**来判断。

---

## ✅ 结论总结：

### 情况一：直接传函数 👉 **函数会被当作值来执行**

```jsx
const [count, setCount] = useState(() => {
  console.log('init') // ✅ 只执行一次
  return 0
})
```

> ✅ 用函数包装起来时，它是**惰性初始化（lazy initialization）**。只有第一次渲染才会执行这个函数。

---

### 情况二：不包裹函数 👉 **立即执行函数，结果作为初始值**

```jsx
const [count, setCount] = useState(() => {
  return expensiveComputation() // 🧨 每次渲染都会执行（错误用法）
})
```

如果你这样写：

```jsx
const [count, setCount] = useState(expensiveComputation())
```

> ❌ 这里 `expensiveComputation()` 是**直接执行的结果**，不是“初始化函数”。
> 所以会每次组件函数运行时都执行它，即使它只是用来生成初始值。

---

## 🧠 用途：为什么用函数初始化？

```jsx
const [data, setData] = useState(() => loadFromLocalStorage())
```

这是推荐写法，因为：

- `loadFromLocalStorage()` 是个开销大的操作
- 用函数包裹后，它只会在 **第一次渲染** 时被执行
- 后续 re-render 不会再跑这个函数，提高性能 ✅

---

## 🔍 内部机制简述

React 内部会检查你传给 `useState` 的第一个参数：

- 如果是**函数类型**：认为你要用**惰性初始化**，只执行一次
- 如果是普通值（包括函数调用的返回值）：直接使用该值作为初始 state

---

## ✅ 推荐写法（性能更好）

```jsx
const [state, setState] = useState(() => expensiveInit())
```

---

如果你还想了解 `useMemo`, `useCallback` 和 `useState(() => {})` 的区别，也可以问我继续讲解。

在 React 中，`useState` 的参数既可以是一个“直接值”（比如数字、字符串、对象……），也可以是一个“函数”。当你传入一个函数作为初始状态时，React 会把它当作 **惰性初始化函数（lazy initializer）** 来处理，而不是把这个函数本身当作状态值。

---

## 1. 惰性初始化函数的作用

```jsx
// 直接传值：每次组件渲染，计算表达式都会执行
const [count, setCount] = useState(expensiveComputation())

// 惰性初始化：expensiveComputation 只在初次渲染时执行一次
const [count, setCount] = useState(() => expensiveComputation())
```

- **直接传值**
  `useState(expensiveComputation())` 会在每次组件渲染时都调用一次 `expensiveComputation()`，然后把它的返回值当作初始状态传给 React。（虽然 React 只会在第一次渲染时真正“存”这个值，但调用函数的开销已经发生了。）
- **惰性初始化**
  `useState(() => expensiveComputation())` 传入了一个函数，React 只会在 **第一次** 渲染时执行这个函数，将它的返回值设为初始状态；后续渲染完全不会再次调用它，从而避免重复开销。

---

## 2. 惰性初始化的典型场景

- **昂贵计算**：需要从大数据、复杂算法或本地缓存中读取初始值时
- **从 `localStorage` 恢复状态**：只想在组件首渲染时同步一次
- **避免重复闭包创建**：当初始状态是一个对象或数组，需要一次性生成，且不希望每次渲染都重建

```jsx
function Counter() {
  const [initialValue] = useState(() => {
    const saved = window.localStorage.getItem('count')
    return saved != null ? JSON.parse(saved) : 0
  })

  const [count, setCount] = useState(initialValue)
  // ...
}
```

---

## 3. 如何在状态中存储一个函数

如果你的初衷是把一个函数本身当作状态值（而不是调用它来生成状态），你需要再包一层：

```jsx
// 想要 state 本身就是一个函数（doSomething）
const [doSomething, setDoSomething] = useState(() => () => {
  console.log('Hello!')
})

// 使用时直接调用
doSomething()
```

这样，React 会执行最外层的惰性初始化函数 `() => () => { … }`，并把内层的函数 `() => { … }` 作为状态值保存下来。

---

## 4. 小结

- **传入普通值**：`useState(123)` ⇒ 初始状态就是 `123`
- **传入函数**：`useState(() => 123)` ⇒ React 调用该函数一次，把返回值 `123` 设为初始状态
- **想把函数本身存进去**：`useState(() => yourFunction)` ⇒ 初始状态就是 `yourFunction`

通过惰性初始化，你可以把“只需执行一次的昂贵计算”高效地集成到组件状态里，而不用担心额外的性能损耗。

在 React 源码里，`useState` 对它的第一个参数有这样的处理逻辑：

```js
// 伪代码，简化版
function mountState(initialState) {
  // 如果 initialState 是函数，就执行它，并把返回值当作真正的初始 state
  const state = typeof initialState === 'function' ? initialState() : initialState
  // …将 state 存到 Hook 里，只在初次渲染时用这段逻辑…
}
```

---

### 1. 当你写 `useState(expensiveComputation())`

- **函数体执行流程**
  每次组件渲染（每次 React 调用你的组件函数）都会先走到这一行，就立刻执行 `expensiveComputation()`，无论这个渲染是不是「第一次」。
- **结果**
  虽然后续渲染时 React 并不会把新的返回值再放进 state（state 只在初次渲染时设置），但**每次渲染**都会触发那次开销大的计算。

---

### 2. 当你写 `useState(() => expensiveComputation())`

- **函数体执行流程**
  这里传给 `useState` 的是一个函数（lazy initializer）。React 看到「参数类型是函数」，就**只在第一次渲染**时才调用它一次，之后再渲染就**不会再执行**这个函数了。
- **结果**
  - **首次渲染**：执行一次 `expensiveComputation()`，并把它的返回值当作初始 state。
  - **后续渲染**：完全跳过这段调用，直接使用已有的 state，性能开销为零。

---

### 为什么要这样设计？

1. **性能考虑**
   如果每次渲染都做一次昂贵计算，UI 会卡顿；提供 lazy initializer，可以把开销「推迟到只有一次」。
2. **API 语义**
   把「直接值」和「惰性初始化函数」合二为一的 API，使用时也更简洁，不用写两套。

---

### 小结

- `useState(expensiveComputation())` → **错误用法**，每次渲染都执行 `expensiveComputation()`
- `useState(() => expensiveComputation())` → **正确用法**，只在「组件挂载」时执行一次

> **Tip:** 如果你的初始 state 本身就是一个函数对象（比如你想把一个函数当成 state 存起来），可以再包一层：
>
> ```js
> // 想把 fn 这个函数当成 state 存
> const [fnState] = useState(() => fn)
> ```
