# React Router

# `loader` + `useLoaderData`

这是一个很好的问题！让我解释一下使用 `loader` + `useLoaderData` 这种模式的好处：

## 数据获取的两种方式对比

### 方式1：直接在组件中使用（传统方式）

```javascript
// Menu.jsx
function Menu() {
  const [menu, setMenu] = useState([])
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState(null)

  useEffect(() => {
    getMenu()
      .then((data) => setMenu(data))
      .catch((err) => setError(err))
      .finally(() => setLoading(false))
  }, [])

  if (loading) return <Loader />
  if (error) return <Error />

  return <div>{/* 渲染菜单 */}</div>
}
```

### 方式2：使用 loader + useLoaderData（React Router 方式）

```javascript
// 路由配置
{
  path: '/menu',
  element: <Menu />,
  loader: menuLoader,  // 数据获取
}

// Menu.jsx
function Menu() {
  const menu = useLoaderData(); // 直接获取数据
  return <div>{/* 渲染菜单 */}</div>;
}
```

## 使用 loader 的优势：

### 1. **性能优化**

- 数据在组件渲染**之前**就开始获取
- 避免了"先渲染空状态，再加载数据"的闪烁
- 支持并行数据获取

### 2. **更好的用户体验**

- 页面跳转时数据已经在后台加载
- 减少了加载状态的显示时间
- 更流畅的页面切换

### 3. **错误处理**

- 如果数据获取失败，React Router 会自动显示 `errorElement`
- 统一的错误处理机制
- 不需要在每个组件中重复写错误处理逻辑

### 4. **缓存和优化**

- React Router 会缓存 loader 的结果
- 避免重复的数据请求
- 支持数据预取

### 5. **代码简洁**

- 组件逻辑更清晰，专注于渲染
- 数据获取逻辑与UI逻辑分离
- 减少了样板代码

### 6. **SEO 友好**

- 数据在服务端渲染时就可以获取
- 更好的搜索引擎优化

## 总结：

使用 `loader` + `useLoaderData` 是现代 React 应用的最佳实践，它提供了更好的性能、用户体验和代码组织方式。虽然直接在组件中获取数据也能工作，但 loader 模式更加优雅和高效。
