---
title: Promise 类方法和实例方法
date: 2026-01-20
category: 'js'
---

# 3 个对象方法和 6 个类方法

## 3个对象方法

- then
- catch
- finally

## 6个类方法

### 1. Promise.resolve()

```js
const p1 = Promise.resolve(111)
const p2 = new Promise((resolve, reject) => {
  resolve(111)
})
// p1 相当于 p2
```

### 2. Promise.reject()

```js
const p1 = Promise.reject(222)
const p2 = new Promise((resolve, reject) => {
  reject(222)
})
// p1 相当于 p2
```

### 3. Promise.all()

接收一组 promise ，**当所有 promise 的状态都为 fulfilled 时**，返回结果；

**如果其中一个 promise 的状态变成 rejected** ，则执行 `onRejected`

### 4. Promise.allSettled()

接收一组 promise ，**等待返回所有** promise 执行结果，**无论失败或成功**

_失败的结果也将在 then 中返回，而不是在 catch_

```js
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject('执行失败1')
  }, 1000)
})
const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject('执行失败2')
  }, 2000)
})
Promise.allSettled([p1, p2]).then((res) => {
  console.log(res) // 2s后返回
  // [ {status: 'rejected', reason: '执行失败1' }, {status: 'rejected', reason: '执行失败2'} ]
})
```

### 5. Promise.race()

接收一组 promise ，**只要有一个 promise 执行完成**，**无论拒绝或敲定**，就会返回 promise 的结果

```js
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject('执行失败1')
  }, 1000)
})
const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('执行成功2')
  }, 2000)
})
const p3 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('执行成功3')
  }, 3000)
})

Promise.race([p2, p3, p1])
  .then((res) => {
    console.log(res)
  })
  .catch((err) => {
    console.log(err) // 执行失败1
  })

Promise.race([p3, p2])
  .then((res) => {
    console.log(res) // 执行成功2
  })
  .catch((err) => {})
```

### 6. Promise.any()

接收一组 promise ，**只要有一个 promise 最先执行完成** 且 **状态为 fulfilled** 执行成功，promise 结果返回

```js
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject('执行失败1')
  }, 1000)
})
const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject('执行失败2')
  }, 2000)
})
const p3 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('执行成功3')
  }, 3000)
})
const p4 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('执行成功4')
  }, 4000)
})

Promise.any([p1, p2, p3, p4])
  .then((res) => {
    console.log(res) // 执行成功3
  })
  .catch((err) => {})

Promise.any([p1, p2])
  .then((res) => {
    console.log(res)
  })
  .catch((err) => {
    console.log(err) // AggregateError: All promise were rejected
    console.log(err.errors) // ['执行失败1', '执行失败2']
  })
```

当所有 promise 的返回 **都为 reject** 时，执行 `onRejected`，返回一个由 **AggregateError 实例** 提供的错误信息 `AggregateError: All promise were rejected`
