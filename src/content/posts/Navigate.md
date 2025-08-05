---
title: Navigate
date: 2025-07-21
category: 'react project'
---

`if (isAuthenticated) navigate("/app", { replace: true });`

好问题！让我用更直观的方式解释 `replace` 到底替换了什么：

---

## 浏览器历史记录就像一个数组

想象浏览器的历史记录是一个数组，用户的每次导航都会改变这个数组：

---

## 1. **正常导航（不使用 replace）**

### 用户操作流程：

```
1. 访问首页：/home
2. 点击登录：/login
3. 登录成功：navigate("/app")
```

### 历史记录变化：

```js
// 初始状态
history = []

// 访问 /home
history = ['/home']

// 导航到 /login
history = ['/home', '/login']

// 导航到 /app（正常导航，添加新条目）
history = ['/home', '/login', '/app']
//                          ↑ 当前位置
```

### 用户按后退按钮：

```js
// 第一次后退
history = ['/home', '/login', '/app']
//                   ↑ 回到 /login

// 第二次后退
history = ['/home', '/login', '/app']
//           ↑ 回到 /home
```

---

## 2. **使用 replace: true**

### 用户操作流程：

```
1. 访问首页：/home
2. 点击登录：/login
3. 登录成功：navigate("/app", { replace: true })
```

### 历史记录变化：

```js
// 初始状态
history = []

// 访问 /home
history = ['/home']

// 导航到 /login
history = ['/home', '/login']

// 导航到 /app（replace: true，替换当前条目）
history = ['/home', '/app']
//                   ↑ 当前位置
//         ↑ /login 被 /app 替换掉了！
```

### 用户按后退按钮：

```js
// 第一次后退
history = ['/home', '/app']
//           ↑ 直接回到 /home
// /login 页面不存在了，跳过了！
```

---

## 3. **直观对比**

| 导航方式      | 历史记录                | 后退行为           |
| ------------- | ----------------------- | ------------------ |
| 正常导航      | `[/home, /login, /app]` | app → login → home |
| replace: true | `[/home, /app]`         | app → home         |

---

## 4. **replace 替换的是什么？**

**替换的是当前历史记录中的最后一个条目**：

```js
// 替换前（在 /login 页面）
history = ['/home', '/login']
//                   ↑ 当前位置

// 执行 navigate("/app", { replace: true })
history = ['/home', '/app']
//                   ↑ /login 被 /app 替换了
```

---

## 5. **形象比喻**

想象历史记录是一摞纸：

### 正常导航：

```
[第3张纸: /app]     ← 在最上面放一张新纸
[第2张纸: /login]
[第1张纸: /home]
```

### replace: true：

```
[第2张纸: /app]      ← 把最上面的纸换掉
[第1张纸: /home]     ← /login 的纸被扔掉了
```

---

## 总结

`replace: true` 替换的是：

- **当前历史记录栈中的最后一个条目**
- **用户当前所在的页面记录**
- **让用户无法通过后退按钮回到被替换的页面**

这样用户就不会回到登录页面，而是直接回到登录前的页面！
