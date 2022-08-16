## 问题描述

在故事板中使用滚动视图 `UIScrollView` 时如果只配置上下左右四边的约束，或者设置宽度约束等，会有约束错误提醒：

> Scrollable Content Size Ambiguity（有歧义，不明确）：
>
> Has ambiguous scrollable content height / width

因为不能确定 Content Size 有多大。

## 解决方法

为滚动视图添加子视图 ContentView，子视图设置四面约束后，最后**设置 宽度/高度 等于 Frame Layout Guide**，需要向哪个方向滑动就将对应的约束优先级调低（eg：水平滑动 - 宽度）。如果需要动态设置 ContentSize 宽度/高度可以改约束的 constant。

## 参考

1. [ios - UIScrollView Scrollable Content Size Ambiguity - Stack Overflow](https://stackoverflow.com/questions/19036228/uiscrollview-scrollable-content-size-ambiguity/27227174#27227174)