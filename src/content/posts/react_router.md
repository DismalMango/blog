---
title: react router
date: 2025-09-05
category: 'react-project'
---

# NavLink vs. Link

好 👌，这个问题在 **React Router v6** 里特别常见。

---

## 1. 共同点

- `Link` 和 `NavLink` 都是 **React Router 提供的导航组件**。

- 作用类似于 HTML 的 `<a href="...">`，但不会整页刷新，而是执行单页应用 (SPA) 的路由切换。

- 都通过 `to` 指定跳转地址：

  ```jsx
  <Link to="/about">About</Link>
  <NavLink to="/about">About</NavLink>
  ```

---

## 2. 不同点

### 🔹 `Link`

- 只是一个**普通的路由跳转**。
- 用于单纯的导航，不会关心当前路由是否匹配。

例子：

```jsx
<Link to="/home">Home</Link>
```

---

### 🔹 `NavLink`

- 是 `Link` 的增强版，用于 **导航菜单**。
- 它可以检测当前路由是否与 `to` 匹配，然后自动添加 **状态样式**。
- 常见场景：高亮显示当前选中的菜单项。

例子：

```jsx
<NavLink to="/home" className={({ isActive }) => (isActive ? 'active' : 'inactive')}>
  Home
</NavLink>
```

这里的 `isActive` 是 React Router 传入的参数：

- 当前路径匹配 `/home` → `isActive = true`，className = `"active"`
- 不匹配 → `"inactive"`

---

## 3. 总结对比表

| 特性                     | `Link`   | `NavLink`                 |
| ------------------------ | -------- | ------------------------- |
| 路由跳转                 | ✅       | ✅                        |
| 检查当前路由是否激活     | ❌       | ✅                        |
| 支持自动添加 active 状态 | ❌       | ✅                        |
| 适合场景                 | 普通跳转 | 导航菜单 / 需要高亮的链接 |

---

✅ **一句话总结**

- 用 `Link`：只是跳转。
- 用 `NavLink`：还能知道自己是不是“当前页面”，常用于导航栏高亮
