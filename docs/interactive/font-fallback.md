---
title: 字体加载
order: 0
nav:
  title: 交互
  order: 0
group:
  title: 字体
  order: 1
---

# 字体加载

如果页面中使用了非系统字体，想让这些文字显示出来，我们必须载入自己的字体文件。

外文的字体文件通常都很小，因为大部分国家的文字都是由字母组成，字体文件只需要存储字母的字形即可。然而汉字不是这样：
汉字并不是由字母组成，所以汉字字体文件需要把成千上万个汉字的字形都储存在里面，字体文件很大。

假设一个字体文件大小 8MiB，那么在 4MiB/s 的网速下下载载入字体文件就需要 2s。
如果网页没有加载好字体时什么都不显示，直接白屏，这对于用户体验而言影响较大。当然实际上不是这样。

接下来的示例中，中文我们统一使用**方正准圆**这个字体，字体文件大约 9MiB（Gzip 后大约 5MiB）。
字体定义：

```less | pure
@font-face {
  font-family: 'fzzy';
  src: url('./fzzy.ttf?') format('truetype');
}
```

我们试着加载一个字体：

<code src="@/interactive/font-fallback/try.tsx"></code>

上面这个示例是常见的场景，点击按钮之后加载字体，在字体文件下载期间，使用默认的字体；字体文件下载完成后替换成新的字体。

这个示例是常见的字体加载场景，即使字体没有下载完或者是下载失败了，页面也能使用默认字体正常显示。

## 原理

其实，浏览器在 CSS 中已经为我们提供了相应的属性控制字体显示方式，熟悉字体的同学应该马上就知道了：`font-display` 属性。
这个属性并不是放在元素上的，而是只能放在 `@font-face { }` 块里面，它用来定义**这个字体的加载时的展示方式**。

这个属性的默认值是 `font-display: auto;`，由浏览器自行决定。实际上现在浏览器默认的 `auto` 属性值和 `swap` 属性值的表现是基本一样的。
我们可以当做 `font-display: swap;` 来讲。

这个 `swap` 属性值表示：
**浏览器使用默认字体把文字展示出来，避免过长时间白屏；之后还会继续加载字体文件，如果加载完成那么就用 `font-family` 中优先级更高的字体替换默认字体。**
这样就避免了过长时间白屏（最长也就开头那 100ms），但是这样会导致如果用户网速慢，字体加载完成后替换字体时会引起页面文字闪烁，甚至会引起页面布局重排。

这里给出一个 `font-display: swap;` 实例：

<code src="@/interactive/font-fallback/try-swap.tsx"></code>

## 进阶场景

上面的示例是最常见的场景。再考虑以下几种特殊场景：

- 第一种，我的网页中使用了大量类似于 iconfont 的字体图标，或者是有特殊用途的字体，例如：用字体来绘制界面、绘制科学计算符号、甚至是类似携程这种用字体来反爬虫的；
- 第二种，我的网页只需要用到其他字体，但字体的加载与否无关紧要，例如本网页，代码部分使用了 “Source Code Pro” 字体；
- 第三种，我的页面用到了其他字体，但是页面架构需要稳定，绝对不能发生文字重绘（也就是字体替换）。

相应的，我们可以产生三种不同的需求：

- 第一种场景，字体应作为内容的一部分，字体未加载完成的时候我们认为页面处在 **“未加载完成”** 的状态；
- 第二种场景，字体是用于提升体验的备选资源，可以容忍字体未载入的情况；
- 第三种场景，页面一旦加载，就不能发生字体的变更，即使字体加载完了也不可以。

接下来我们来依次看看如何利用这个 `font-display` 来满足我们的需求。

---

上面说的第一种需求，我们可以使用 `font-display: block;` 这个属性值来实现，它表示：**字体加载期间，一直使用一个空白框来占位，直到字体加载完毕后才显示文本**。这就避免了字体加载期间显示原始字体，而导致了错误的效果。

示例：

<code src="@/interactive/font-fallback/try-block.tsx"></code>

可以看到，使用 `block` 这个属性后，如果用户网速慢，白屏时间会过长。但它可以让文字在字体加载期间不暴露出来，很适用于因为某些需求需要修改字体的场景。

---

上面说的第二种需求，我们可以使用 `font-display: fallback;` 来实现，它表示：**发现使用了某个字体后，浏览器先不显示文字而去加载字体文件，如果无法在约 100ms 内加载完成，那么显示默认字体，并默默继续加载新字体；但如果新字体加载时间较长（大约超 3s），那么浏览器就不再会用新加载的字体替换默认字体了，等到刷新页面后才会使用新字体，这就避免了网速慢的用户加载完字体后发生页面的闪动和重排**。

我们可以使用浏览器 F12 网络菜单中的 “节流” 功能开控制网速，可以试一试 2s 左右加载完字体和 5s 左右加载完字体，显示有什么区别。
示例：

<code src="@/interactive/font-fallback/try-fallback.tsx"></code>

点击按钮后可以看到，即使你的网速很快，点按钮后文字总会闪动一下，这里和 `font-display: swap;` 不同；
我们可以理解为，`font-display: fallback;` 致力于减少页面的闪烁和文字的重绘：所以会先白屏加载字体，加载不出则用默认字体，此时如果新字体加载特别慢，浏览器会避免在用户浏览页面的过程中替换字体。

---

上面说的第三种需求，我们可以用 `font-display: optional;` 来实现，它可以彻底避免页面文字的重绘。
具体来说：**页面加载时如果所需字体存在，那么就使用，否则就直接使用默认字体，并在后台默默加载新字体，加载完成后也不会替换当前字体，直到你刷新页面**。

我们可以使用浏览器 F12 网络菜单中的 “停用缓存” 功能来避免字体文件缓存，在这个选项开启、关闭的情况下分别刷新网页点击按钮，看看显示有什么区别。
示例：

<code src="@/interactive/font-fallback/try-optional.tsx"></code>

可以看到，点击按钮之后文字不会像 `font-display: fallback;` 一样闪动一下；
不开启 F12 调试菜单时，青色的文字如果未使用新字体，那么点击按钮也不会使得任何字体发生改变；刷新页面后，青色文字开始使用新的字体了，此时点击按钮，黑色的文字也会开始时候新字体。
（这个 demo 好像有点问题，建议点 “在独立页面打开” 按钮来体验）

## 医脉同道中的应用

医脉同道在 m 站推广页、小程序 HR 端登录页中使用了特殊字体，但这些项目均是把字体一同打包 + base64 编码到 CSS 里，不存在加载时间，此处暂无用例。
