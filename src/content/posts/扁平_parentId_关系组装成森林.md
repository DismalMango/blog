---
title: 扁平 parentId 关系组装成森林
date: 2026-02-11
category: 'leetcode'
---

```
算法：
const A = [
    { id: 2, parentId: 1 },
    { id: 1 },
    { id: 3, parentId: 2 },
    { id: 5, parentId: 4 },
    { id: 4 },
];
// const B = [
//   {id: 1, child: [{id: 2, parentId: 1, child: [{id: 3, parentId: 2}]}]},
//   {id: 4, child: [{id: 5, parentId: 4}]},
// ]
把A变成B
```
