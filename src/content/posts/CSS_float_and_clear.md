---
title: CSS float and clear
date: 2025-08-24
category: 'CSS'
---

# Clear

### `clear: left`:

- 它只要求 **元素的顶部必须在所有左浮动元素的下方**。

- 对右浮动元素没要求。

### `clear: both`:

- `clear: both` 的作用是：**元素的顶部必须在之前所有浮动元素的下方**。

- 它并不是“强制换行”的意思，而是“避免与浮动元素重叠”。

When applied to non-floating blocks, it moves the [border edge](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_box_model/Introduction_to_the_CSS_box_model#border_area) of the element down until it is below the [margin edge](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_box_model/Introduction_to_the_CSS_box_model#margin_area) of all relevant floats. The non-floated block's top margin collapses.
