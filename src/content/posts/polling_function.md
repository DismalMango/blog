---
title: polling function
date: 2026-01-20
category: 'airbrb'
---

# 实时通知系统实现详解

## 🎯 系统架构

这是一个基于 **轮询 + 快照对比** 的实时通知系统，监控两种场景：

1. **房东视角**：监听新的订单请求（pending 状态）
2. **房客视角**：监听订单状态变化（accepted/declined）

---

## 📊 核心数据流

```
┌─────────────────────────────────────────────────────────┐
│                     每 5 秒轮询                          │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│  Promise.allSettled 并发请求                             │
│  ├─ fetchMyBookingsAsGuest()  (我作为房客的订单)         │
│  └─ fetchMyBookingsAsHost()   (我作为房东的订单)         │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│  detectNewNotifications() 快照对比算法                   │
│  ├─ 对比 previousGuestSnapshotRef (上次房客订单快照)    │
│  └─ 对比 previousHostSnapshotRef  (上次房东订单快照)    │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│  检测到变化 → 生成通知 → 入队                            │
│  ├─ 房东：新订单 (new pending booking)                   │
│  └─ 房客：状态变更 (pending → accepted/declined)        │
└─────────────────────────────────────────────────────────┘
```

---

## 🔑 关键技术实现

### 1. **初始化标志位机制**

```javascript
const isSnapshotInitializedRef = useRef(false);

const detectNewNotifications = useCallback(
  (latestGuestBookings = [], latestHostBookings = []) => {
    // 第一次运行时只初始化快照，不生成通知
    if (!isSnapshotInitializedRef.current) {
      previousGuestSnapshotRef.current = latestGuestBookings;
      previousHostSnapshotRef.current = latestHostBookings;
      isSnapshotInitializedRef.current = true;
      return; // 关键：避免首次加载时误报通知
    }
    // ...后续对比逻辑
  },
  [...]
);
```

**作用**：防止用户登录时把所有历史订单都当作"新通知"弹出。

---

### 2. **房东通知检测（新订单）**

```javascript
// 使用 Set 存储上次快照的订单 ID
const previousHostIds = new Set(previousHostSnapshotRef.current.map((booking) => booking.id))

latestHostBookings.forEach((booking) => {
  const status = normalizeStatus(booking?.status)

  // 条件：
  // 1. 不在上次快照中（新订单）
  // 2. 状态是 pending（待处理）
  if (!previousHostIds.has(booking.id) && status === 'pending') {
    notificationsToAdd.push(buildHostNotification(booking))
  }
})

// 更新快照
previousHostSnapshotRef.current = latestHostBookings
```

**检测逻辑**：

- 使用 `Set` 数据结构，O(1) 时间复杂度快速查找
- 只有 **全新的 pending 订单** 才会触发通知
- 避免重复通知同一个订单

---

### 3. **房客通知检测（状态变更）**

```javascript
// 使用 Map 存储上次快照，key 是 booking.id，value 是完整 booking 对象
const previousGuestById = new Map(
  previousGuestSnapshotRef.current.map((booking) => [booking.id, booking]),
)

latestGuestBookings.forEach((booking) => {
  const currentStatus = normalizeStatus(booking?.status)
  const previousStatus = normalizeStatus(previousGuestById.get(booking.id)?.status)

  // 检测状态变化
  const statusChanged = previousGuestById.has(booking.id) && currentStatus !== previousStatus

  // 只关心 accepted 和 declined 状态
  if (statusChanged && (currentStatus === 'accepted' || currentStatus === 'declined')) {
    notificationsToAdd.push(buildGuestNotification(booking, currentStatus))
  }
})

previousGuestSnapshotRef.current = latestGuestBookings
```

**检测逻辑**：

- 使用 `Map` 快速查找上次状态
- 必须是 **已存在的订单** 才检测状态变化
- 只通知 `accepted` 或 `declined`（不通知 pending）

---

### 4. **通知去重机制**

```javascript
const enqueueNotifications = useCallback((items) => {
  if (!Array.isArray(items) || items.length === 0) {
    return
  }

  setNotifications((prev) => {
    // 使用 Set 记录已存在的通知 ID
    const existingIds = new Set(prev.map((notification) => notification.id))

    // 过滤掉重复的通知
    const uniqueItems = items.filter((item) => !existingIds.has(item.id))

    // 新通知在前，老通知在后（时间倒序）
    return uniqueItems.length ? [...uniqueItems, ...prev] : prev
  })
}, [])
```

**去重策略**：

- 通知 ID 格式：
  - 房东：`host-${booking.id}`
  - 房客：`guest-${booking.id}-${status}`
- 同一个订单的状态变化会生成不同 ID

---

### 5. **轮询控制与性能优化**

```javascript
useEffect(() => {
  if (!isLoggedIn) {
    return // 未登录时不轮询
  }

  pollingFunction() // 立即执行一次
  const intervalId = setInterval(() => {
    pollingFunction()
  }, 5000) // 每 5 秒轮询一次

  return () => clearInterval(intervalId) // 清理定时器
}, [isLoggedIn, pollingFunction])
```

**优化点**：

- 登出时自动停止轮询
- 使用 `useCallback` 避免 `pollingFunction` 频繁重建
- 组件卸载时清理定时器，防止内存泄漏

---

### 6. **并发请求优化**

```javascript
const pollingFunction = useCallback(async () => {
  if (!isLoggedIn) return

  try {
    // 如果没有房源，跳过房东订单请求
    const hostPromise = myListings.length === 0 ? Promise.resolve([]) : fetchMyBookingsAsHost()

    // 并发请求，互不阻塞
    const [guestResult, hostResult] = await Promise.allSettled([
      fetchMyBookingsAsGuest(),
      hostPromise,
    ])

    // 请求失败时降级使用旧数据
    const latestGuestBookings =
      guestResult.status === 'fulfilled' && Array.isArray(guestResult.value)
        ? guestResult.value
        : myBookingsAsGuestRef.current

    const latestHostBookings =
      hostResult.status === 'fulfilled' && Array.isArray(hostResult.value)
        ? hostResult.value
        : hostBookingRef.current

    // 更新 ref（不触发渲染）
    myBookingsAsGuestRef.current = latestGuestBookings
    hostBookingRef.current = latestHostBookings

    detectNewNotifications(latestGuestBookings, latestHostBookings)
  } catch (err) {
    console.error('Polling failed:', err)
  }
}, [
  detectNewNotifications,
  fetchMyBookingsAsGuest,
  fetchMyBookingsAsHost,
  isLoggedIn,
  myListings.length,
])
```

**关键优化**：

- `Promise.allSettled`：即使一个请求失败，另一个仍会执行
- 降级策略：失败时使用上次成功的数据
- 条件请求：没有房源时不请求房东订单

---

## 🎨 用户体验设计

### 未读计数

```javascript
const unreadCount = notifications.filter((notification) => !notification.read).length
```

### 自动标记已读

```javascript
useEffect(() => {
  if (isPanelOpen) {
    markAllNotificationsRead() // 打开通知面板时自动标记
  }
}, [isPanelOpen, markAllNotificationsRead])

const markAllNotificationsRead = useCallback(() => {
  setNotifications((prev) =>
    prev.map((notification) =>
      notification.read ? notification : { ...notification, read: true },
    ),
  )
}, [])
```

---

## 📝 简历描述建议

```
【实时通知系统】
- 设计轮询 + 快照对比算法，实现订单状态实时监控
  · 使用 useRef 存储快照，避免不必要的重渲染
  · Set/Map 数据结构优化查找性能（O(1) 时间复杂度）

- 智能通知生成策略
  · 房东：检测新的 pending 订单（新订单通知）
  · 房客：检测 pending → accepted/declined 状态变化
  · 首次加载防误报机制（初始化标志位）

- 性能优化
  · Promise.allSettled 并发请求，互不阻塞
  · 请求失败降级处理，使用旧数据兜底
  · 条件请求减少无效 API 调用（无房源时跳过房东订单）

- 通知去重与 UX
  · 基于唯一 ID 的去重机制
  · 自动标记已读（打开面板触发）
  · 未读数 Badge 实时更新
```

---

## 💡 面试可能问到的问题

**Q1: 为什么用轮询而不是 WebSocket？**

- 后端未提供 WebSocket 支持
- 项目规模小，5 秒轮询足够满足需求
- 实现简单，维护成本低

**Q3: 为什么用 useRef 而不是 useState？**

- 快照数据不需要触发 UI 重渲染
- 避免因快照更新导致轮询函数依赖变化
- 性能更优

**Q4: 如何防止重复通知？**

- 使用 Set 记录已存在的通知 ID
- 房客通知 ID 包含状态（`guest-${id}-${status}`）
