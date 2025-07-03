---
title: useEffect
date: 2025-07-03
category: 'react project'
---

这句 “Hooks must be called in the same order”（钩子必须以相同的顺序调用）出现在你违反了 React 的「Hooks 规则」（Rules of Hooks）时。它的含义是：

1. **React 通过调用顺序来匹配每一个 useState/useEffect……背后的状态槽位**

   - 第一次渲染时，React 会依次给第 1 个 hook 插槽、第 2 个 hook 插槽……分配 state 或 effect。
   - 下一次渲染时，React 就是按同样的顺序去拿这些插槽里的数据。

2. **如果顺序不一致，就会拿错插槽，从而出错**

   - 例如，你在一个分支里才调用 `useState`：

     ```jsx
     function MyComp({ flag }) {
       if (flag) {
         // 第一次渲染 flag=true，会调用这个 useState，去第 1 个槽
         const [x, setX] = useState(0)
       }
       // flag=false 时，上次已经在第 1 个槽放了 state，
       // 这次没调用，就会导致后续所有槽都往前挪一个，拿不到正确的 state
       const [y, setY] = useState(0)
       // …
     }
     ```

   - 这时 React 会报 “Rendered more hooks than during the previous render” 或 “Rendered fewer hooks…”。

3. **正确做法：**

   - **所有的 hook 都要写在组件最顶层**，不要放在 `if`、`for`、`switch`、早期 `return` 之后或任何循环/条件里。
   - 确保每次渲染，hook 的调用次数和相对顺序都**完全一致**。

---

### 例子

❌ 错误写法（条件调用）：

```jsx
function Comp({ show }) {
  if (show) {
    useEffect(() => { … }, []);
  }
  useState(0);
  // ↓ 这里 React 可能会拿错 useState 对应的插槽
}
```

✔️ 正确写法（无条件调用）：

```jsx
function Comp({ show }) {
  useEffect(() => {
    if (show) {
      // 在 effect 里再判断
    }
  }, [show])

  const [count, setCount] = useState(0)
  // 每次渲染都按同样的顺序执行 useEffect → useState
}
```

总结：**“hook must be at same order”** 就是提醒你，所有的 React Hooks（`useState`/`useEffect`/`useContext`……）必须在组件体内顶层、每次渲染都以完全相同的顺序出现，才能保证状态正确匹配。
