---
title: Bianry Search
date: 2025-08-25
category: 'leetcode'
---

[34. Find First and Last Position of Element in Sorted Array](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)

![image-20250825111554051](assets/image-20250825111554051.png)

![Screenshot 2025-08-25 at 11.16.44 am](assets/Screenshot%202025-08-25%20at%2011.16.44%E2%80%AFam.png)

注意这里为什么==不是L = M==: 在只有一个元素的情况下，在是L = M 的情况下那么会造成 left == right == mid 的死循环。

e.g.

`[0]`

Left, Right, Mid = 0

![Screenshot 2025-08-25 at 11.23.15 am](assets/Screenshot%202025-08-25%20at%2011.23.15%E2%80%AFam.png)

![Screenshot 2025-08-25 at 11.24.03 am](assets/Screenshot%202025-08-25%20at%2011.24.03%E2%80%AFam.png)

==题目中 `>=` 的转换==：

![image-20250825112544502](assets/image-20250825112544502.png)

## bisect_left

`bisect_left(a, x, lo=0, hi=len(a))` 返回在区间 `a[lo:hi]` 中**第一个使得 `a[i] >= x` 的索引** `i`。
也就是把 `x` 插到位置 `i`，还能保持有序，且插在所有等于 `x` 的元素**最左边**。
