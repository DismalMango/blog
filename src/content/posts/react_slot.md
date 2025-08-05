---
title: react slot
date: 2025-07-11
category: 'react'
---

下面通过一个小组件演示「槽位（slot）」是如何工作的，以及当我们在条件分支中调用 Hook 时，槽位会如何错乱。

---

## 例子：条件分支中调用 Hook（错误用法）

```jsx
import React, { useState, useEffect } from 'react'

function MyComponent({ flag }) {
  // 槽位 0
  const [count, setCount] = useState(0)

  if (flag) {
    // 当 flag 为 true 时，这里才会“消耗”槽位 1
    // 槽位 1
    useEffect(() => {
      console.log('Effect only when flag=true')
    }, [count])
  }

  // 这里始终调用，React 会把它当作“下一次”槽位
  // 当 flag=true 时，这里是槽位 2；flag=false 时，这里却变成槽位 1！
  // 槽位 2 或 1（不一致）
  const [name, setName] = useState('Alice')

  return (
    <div>
      <p>count: {count}</p>
      <button onClick={() => setCount((c) => c + 1)}>+1</button>
      <p>name: {name}</p>
      <button onClick={() => setName((n) => (n === 'Alice' ? 'Bob' : 'Alice'))}>switch name</button>
    </div>
  )
}
```

**出了什么问题？**

1. **首次渲染**（假设 `flag = true`）：
   - `useState(0)` → 槽位 0
   - `useEffect(...)` → 槽位 1
   - `useState('Alice')` → 槽位 2
2. **第二次渲染**（如果用户把 `flag` 切到 `false`）：
   - `useState(0)` 复用 槽位 0
   - 跳过了 `useEffect`（因为 `flag=false`），**槽位 1** 空缺
   - `useState('Alice')` 这次被当成槽位 1 复用！
   - 结果原本存放在槽位 2（`name` 的状态）被错当成槽位 1 的副作用信息，React 找不到正确的 effect 清理，也会把 `name` 写到错的位置，最终导致渲染和状态极度混乱，甚至抛错。

---

## 正确用法：把条件逻辑放在 effect 内部

要保证每个 Hook 调用的顺序一致，就不要在组件体里根据条件决定“要不要调用某个 Hook”，而是始终调用，再让内部逻辑决定是否执行副作用：

```jsx
import React, { useState, useEffect } from 'react'

function MyComponent({ flag }) {
  // 槽位 0
  const [count, setCount] = useState(0)
  // 槽位 1
  const [name, setName] = useState('Alice')

  // 槽位 2：始终调用，但我们在 effect 内用 flag 判断
  useEffect(() => {
    if (flag) {
      console.log('Effect only when flag=true')
    }
    // 如果需要，也可以在 cleanup 时检查 flag 做不同清理：
    return () => {
      if (flag) {
        console.log('Cleanup when flag was true')
      }
    }
  }, [flag, count])

  return (
    <div>
      <p>count: {count}</p>
      <button onClick={() => setCount((c) => c + 1)}>+1</button>
      <p>name: {name}</p>
      <button onClick={() => setName((n) => (n === 'Alice' ? 'Bob' : 'Alice'))}>switch name</button>
    </div>
  )
}
```

**为什么这次安全？**

- 每次渲染，不管 `flag` 是 `true` 还是 `false`，Hook 调用的顺序都是：
  1. `useState(0)` → 槽位 0
  2. `useState('Alice')` → 槽位 1
  3. `useEffect(...)` → 槽位 2
- React 能稳定地按槽位去读旧状态、写新状态、执行／跳过副作用，槽位永远对上号，不会混乱。

---

**核心要点**：

> “在组件最顶层，无论条件如何分支，都要**固定数量、固定顺序**地调用所有 Hook。条件判断只放在 Hook 的内部逻辑里。”
