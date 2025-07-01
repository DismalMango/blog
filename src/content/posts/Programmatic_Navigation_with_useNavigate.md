---
title: Programmatic Navigation with useNavigate
date: 2025-06-29
category: 'react project'
---

Aim: click map and city compnent would be corresponding change. We can not achieve this by URL. So we use programmatic navigation.

```jsx
const navigate = useNavigate()
```

```jsx
    <div
      className={styles.mapContainer}
      onClick={() => {
        navigate("form");
      }}
    >
```

在 HTML 里，`class`（在 React 中写作 `className`）的值本质上就是一个 **空格分隔的类名列表**。也就是说，你完全可以给一个元素同时指定多个类名，格式是：

```html
<div class="class1 class2 class3">…</div>
```

**浏览器解析规则**

- 浏览器会把 `class="class1 class2"` 当作两个独立的类：`class1` 和 `class2`。
- <font color='red'>元素就会同时应用这两个类对应的所有 CSS 规则。</font >

**在 React + CSS Modules 的场景下**

```jsx
className={`${styles.btn} ${styles[type]}`}
```

- `${styles.btn}` 是基础样式（比如生成的 `"Button_btn__abc123"`）
- `${styles[type]}` 是变体样式（如果 `type = 'primary'`，可能生成 `"Button_primary__def456"`）
- 两者用空格拼接后，就变成了 `"Button_btn__abc123 Button_primary__def456"`
- React 最终会在真实 DOM 上渲染成 `<button class="Button_btn__abc123 Button_primary__def456">…</button>`
- 这样，你的按钮既有 `.btn` 的通用样式，又有 `.primary` 的特殊样式，<font color='red'>两者叠加生效。</font>

所以——**因为 HTML 允许 `class` 属性里写多个以空格分隔的类名**，React 里自然也能通过拼字符串的方式同时传入多个类名。

### 点（`.`） vs 方括号（`[]`）访问对象属性

主要区别在于 **属性名是否“静态”** 以及 **允许的字符范围**：

1. **点号访问（`.`）——静态、简单**

   ```js
   styles.btn
   // 只能写：对象名.固定标识符
   // 标识符必须符合 JS 变量命名规则（不能以数字开头、不能有连字符等）
   // 属性名在代码编写时就确定了，不能用变量替代
   ```

2. **方括号访问（`[]`）——动态、灵活**

   ```js
   styles[type]
   // 方括号里可以放一个**表达式**，结果最终会被转成字符串作为属性名
   // 适合：
   //  - 属性名来自变量（如上例的 type）
   //  - 属性名包含特殊字符（比如 styles['btn-primary']）
   //  - 属性名以数字开头或不符合标识符规则
   ```

---

| 特性            | `obj.prop`              | `obj[expr]`                    |
| --------------- | ----------------------- | ------------------------------ |
| 属性名来源      | 固定写在代码里          | 可以是变量或任意表达式         |
| 命名限制        | 必须符合标识符命名规则  | 任意字符串（包括空格、连字符） |
| 编译时能否校验  | 能（IDE、编译器可检查） | 通常在运行时才知道             |
| 场景示例        | `user.name`             | `user[fieldName]`              |
| CSS Module 场景 | `styles.btn`            | `styles[type]`（动态选择样式） |

因此，当你知道属性名写死就行，用 `styles.btn`；当需要根据变量 `type` 动态选择样式，或属性名不符合 JS 标识符规则，就得用 `styles[type]`。

### back btn 的实现

`navigate(-1);`

`navigate(-1)` 通常是配合 React Router v6 中的 `useNavigate` 钩子一起使用的，它的作用就是模拟浏览器「后退」操作：

1. **用法示例**

   ```jsx
   import { useNavigate } from 'react-router-dom'

   function MyComponent() {
     const navigate = useNavigate()

     return <button onClick={() => navigate(-1)}>返回上一页</button>
   }
   ```

   - 这里的 `navigate` 就是路由提供的一个函数，用于在代码里进行编程式导航。

2. **参数含义**

   - 当你传入一个**负整数**（例如 `-1`）时，等同于浏览器的 `history.back()`，会退回到前一个页面。
   - 传入正整数（如 `navigate(1)`）则会前进相应步数，相当于 `history.forward()`。
   - 传入 `0` 则会“刷新”当前页面。
   - <font color='red'>`navigate(-1)` 就是让浏览器（或 React Router）的 **历史记录栈** 后退一格，跳转到你**上一次真正访问过**的那个 URL：</font>

3. **背后原理**
   React Router 内部维护了一份“历史记录栈”（history stack），`navigate(delta)` 会在这个栈上根据 `delta` 做相对移动：

   - `delta = -1` → 栈指针向后移动一格（后退）。
   - `delta = +1` → 栈指针向前移动一格（前进）。

4. **典型场景**

   - 在「返回」按钮中调用，让用户回到上一步操作的页面。
   - 在流程向导（wizard）中，上一步/下一步的导航。

---

所以，简单来说，`navigate(-1)` 就是“程序化地”让你的应用后退到上一个路由/页面。

### Declarative and imperative

`element={<Navigate replace to="cities" />}` 这种是declarative way

`navigate("form");`这种是imperative

# Navigate component的用法

和` navigate` 函数类似在to props中写地址，可以认为是redirect

`element={<Navigate replace to="cities" />}`

## replace 的作用

现在我来解释 `Navigate` 组件中 `replace` 属性的作用：

在 React Router v6 中，`Navigate` 组件的 `replace` 属性控制导航行为：

### 不加 `replace` 的情况（默认行为）：

```jsx
<Navigate to="cities" />
```

- 会在浏览器历史记录中**添加**一个新的条目
- 用户点击浏览器的"后退"按钮时，会回到之前的页面

### 加上 `replace` 的情况：

```jsx
<Navigate to="cities" replace />
```

- 会**替换**当前的历史记录条目，而不是添加新的
- 用户点击"后退"按钮时，会回到更早的页面

### 为什么在你的代码中需要 `replace`？

在你的路由配置中：

```jsx
<Route path="app" element={<AppLayout />}>
  <Route index element={<Navigate to="cities" />} />
  <Route path="cities" element={<CityList cities={cities} isLoading={isLoading} />} />
  // ... 其他路由
</Route>
```

当用户访问 `/app` 时：

1. 匹配到 `index` 路由
2. 执行 `<Navigate to="cities" />`
3. 重定向到 `/app/cities`

**如果不加 `replace`**：

- 浏览器历史记录会是：`/app` → `/app/cities`
- 用户点击后退时，会回到 `/app`，然后又会被重定向到 `/app/cities`
- 这会造成无限重定向循环，用户无法正常使用后退功能

**加上 `replace`**：

- 浏览器历史记录中 `/app` 被 `/app/cities` 替换
- 用户点击后退时，会直接回到 `/app` 之前的页面
- 避免了重定向循环问题

所以 `replace` 在这里的作用是**防止重定向循环**，确保用户能够正常使用浏览器的后退功能。
