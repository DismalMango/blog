---
title: Performance Optimization
date: 2025-07-26
category: 'react'
---

# technique

![Screenshot 2025-07-26 at 4.37.00 pm](assets/Screenshot%202025-07-26%20at%204.37.00%E2%80%AFpm.png)

12 atomic blog

Test.js

```react
import { useState } from "react";

function SlowComponent() {
  // If this is too slow on your maching, reduce the `length`
  const words = Array.from({ length: 100_00 }, () => "WORD");
  return (
    <ul>
      {words.map((word, i) => (
        <li key={i}>
          {i}: {word}
        </li>
      ))}
    </ul>
  );
}

function Counter({ children }) {
  const [count, setCount] = useState(0);
  return (
    <div>
      <h1>Slow counter?!?</h1>
      <button onClick={() => setCount((c) => c + 1)}>Increase: {count}</button>
      {children}
    </div>
  );
}

export default function Test() {
  return (
    <div>
      <h1>Slow counter?!?</h1>
      <button onClick={() => setCount((c) => c + 1)}>Increase: {count}</button>
      <SlowComponent />
    </div>
  );

  // the efficiency for following code would be better

  // return (
  //   <div>
  //     <h1>Slow counter?!?</h1>
  //     <Counter>
  //       <SlowComponent />
  //     </Counter>
  //   </div>
  // );
}

```

# Compare

### Case 1

```react
export default function Test() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <h1>Slow counter?!?</h1>
      <button onClick={() => setCount((c) => c + 1)}>Increase: {count}</button>
      <SlowComponent />
    </div>
  );
```

![Screenshot 2025-07-28 at 11.44.53 am](assets/Screenshot%202025-07-28%20at%2011.44.53%E2%80%AFam.png)

### Case 2

```react
export default function Test() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <h1>Slow counter?!?</h1>
      <Counter>
        <SlowComponent />
      </Counter>
    </div>
  );
}
```

![image-20250728121800526](assets/image-20250728121800526.png)

Slowcomponent not be re-render

**Case 2 would be best**

# Why is this case?

Because in case2 `SlowComponent` pass to `counter` as props. It was created before the `Counter` so `Counter` re-render will not result in `SlowComponent` re-render.

# memo function

![Screenshot 2025-07-28 at 1.31.09 pm](assets/Screenshot%202025-07-28%20at%201.31.09%E2%80%AFpm.png)

[python decorator python 装饰器](https://www.runoob.com/w3cnote/python-func-decorators.html)

# issue with memo

![Screenshot 2025-07-28 at 3.37.40 pm](assets/Screenshot%202025-07-28%20at%203.37.40%E2%80%AFpm.png)

![Screenshot 2025-07-28 at 3.40.45 pm](assets/Screenshot%202025-07-28%20at%203.40.45%E2%80%AFpm.png)

![Screenshot 2025-07-28 at 3.43.08 pm](assets/Screenshot%202025-07-28%20at%203.43.08%E2%80%AFpm.png)

# useCallback

```react
  const handleAddPost = useCallback(function handleAddPost(post) {
    setPosts((posts) => [post, ...posts]);
  }, []);
```

# 为什么context changed

![Screenshot 2025-07-28 at 5.17.51 pm](assets/Screenshot%202025-07-28%20at%205.17.51%E2%80%AFpm.png)

![Screenshot 2025-07-28 at 5.19.31 pm](assets/Screenshot%202025-07-28%20at%205.19.31%E2%80%AFpm.png)

App re-render 触发 PostProvider re-render 而PostProvider 中的value为object，每次re-render的object都不一样。所以context changed

# Improve WorldWise

### Infinite http request

```react
  useEffect(
    function () {
      getCity(id);
    },
    [id, getCity]
  );
```

```react
  async function getCity(id) {
    if (id === currentCity.id) return;
    dispatch({ type: "loading" });
    try {
      const res = await fetch(`${BASE_URL}/cities/${id}`);
      const data = await res.json();
      // setCurrentCity(data);
      dispatch({ type: "city/loaded", payload: data });
    } catch {
      // alert("City not found");
      dispatch({ type: "error", payload: "City not found" });
    }
  }
```

`dispatch` 改变state，触发re-render -> recreate `getCity` function -> 触发 `useEffect` -> 触发 `getCity`

# Bundle size optimization

## Lazy Loading

![image-20250729222600220](assets/image-20250729222600220.png)
