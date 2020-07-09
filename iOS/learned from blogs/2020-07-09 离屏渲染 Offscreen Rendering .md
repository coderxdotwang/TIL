

>[iOS Rendering 渲染全解析](https://github.com/RickeyBoy/Rickey-iOS-Notes/blob/master/笔记/iOS%20Rendering.md#shouldrasterize-光栅化) by RickeyBoy
>
>[iOS圆角的离屏渲染，你真的弄明白了吗](https://juejin.im/post/5f0339505188252e817c6c02) by 收纳箱

# 2020-07-09 离屏渲染 Offscreen Rendering

## 1、通常渲染流程 VS 离屏渲染流程

通常渲染流程：App（CPU & GPU）渲染内容后 -> 放入`Framebuffer` 帧缓冲器 -> Display 读取显示。

离屏渲染流程：App -> 提前渲染好的内容放入额外创建离屏渲染缓冲区 Offscreen Buffer -> 合适时机再将 `Offscreen Buffer` 中的内容进一步叠加、渲染 -> 结果切换到 `Framebuffer` -> Display 读取显示。

## 2、离屏渲染的效率问题以及为什么需要使用离屏渲染？

### 2.1 离屏渲染主要问题？

>1. **时间消耗：**需要提前对部分内容进行渲染并保存到 `Offscreen Buffer`；需要在必要时刻对 `Offscreen Buffer` 和 `Framebuffer`内容进行切换；
>2. **空间消耗：** `Offscreen Buffer` 需要额外空间，大量离屏渲染可能造成内存压。

### 2.2 为什么使用离屏渲染？

>1. **被动使用：**一些特殊效果需要使用额外的 `Offscreen Buffer` 来保存渲染的中间状态，所以**不得不使用**离屏渲染。
>2. **主动使用：**出于效率目的，可以将内容提前渲染保存在 `Offscreen Buffer` 中，达到**复用**的目的。

一般添加阴影、圆角、组透明等效果后，系统会自动触发离屏渲染。因为需要利用额外内存空间保存中间的渲染结果。常见情形之一如使用 mask 蒙版：通道 1 渲染 layer mask 至 texture -> 通道 2 渲染 layer content 至 texture -> 混合通道-将 mask 应用至 content texture。又比如模糊特效 UIBlurEffectView：通道 1 渲染需要模糊的内容本身 Render content -> 通道 2 内容缩放/捕获 Capture content -> 通道 3 水平模糊 Horizontal blur -> 通道 4 纵向模糊 Vertical blur -> 混合通道-叠加合成。

原本绘制完一个图层就可以丢弃，现在由于需要先保存在 `Offscreen Buffer` 中等待后续合成处理，所以会触发离屏渲染。

主动使用场景例如开启光栅化 *shouldRasterize* 强制将 CALayer 的渲染位图结果 *bitmap* 保存下来以复用提高效率。

> **注意**：当 layer 不能被复用，或者不是静态的，需要频繁修改时开启光栅化反而影响效率。同时离屏渲染缓存内容由*时间限制*（100ms 内没有使用会被丢弃）及*空间限制*（不能超过屏幕像素大小 2.5 倍）。参考：[RickeyBoy 的笔记。](https://github.com/RickeyBoy/Rickey-iOS-Notes/blob/master/笔记/iOS%20Rendering.md#shouldrasterize-光栅化)

## 3、圆角的离屏渲染

### 3.1 怎么调试是否触发了离屏渲染？

打开模拟器 > **Debug** > 勾选 **Color Off-screen Rendered**，启用颜色标记离屏渲染。触发离屏渲染的部分会显示为黄色。

### 3.2 什么情况下触发离屏渲染？

设置 *layer.cornerRadius* 圆角并不一定会触发离屏渲染。默认 cornerRadius 只应用于 *layer* 的 **backgroundColor** 和 **border**，只有同时设置 *layer.masksToBounds* 为 true 时，由于需要对 *layer* 及所有 *sublayers* 的 **contents**（所以**前提条件之一**：*layer.contents != nil* 即设置了内容） 进行裁剪后叠加多个图层，才不得不触发离屏渲染。

除以上圆角（*layer.cornerRadius*）+裁剪（*layer.masksToBounds*）场景外，设置透明度 *layer.opacity* + 组透明 *layer.allowsGroupOpacity*，设置阴影属性 *shadowOffset* 等由于都会作用于 *layer* 及其所有 *sublayers*，从而导致离屏渲染。

### 3.3 如何避免圆角离屏渲染？

1. 直接使用带圆角的图片。 -> 不通用，也不灵活。
2. 添加一层 mask 盖住四个角? -> 不能解决背景为图片或渐变色的情况。
3. 使用贝塞尔曲线？-> layer 布局改动需要重新绘制。
4. 重写 drawRect: 方法在需要应用圆角时手动绘制。 -> 多次调用会有效率问题。

> **Tip**：为 UIButton 设置圆角时可以考虑直接为内部 imageView 设置圆角，可以避免不必要的离屏渲染。 