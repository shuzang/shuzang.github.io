---
title: JavaScript学习1-概览
date: 2020-05-22T11:50:00+08:00
categories: [爱编程爱技术的孩子]
slug: JavaScript learning 1 overview
---

JavaScript 是标准 Web 技术的第三层，负责实现动态的行为，比如实现交互式的地图、2D/3D动画、滚动播放的视频等。

JS 的标准是 ECMAScript，2015年发布了该标准的第六版，通常被称为 ECMAScript 6 或 ES6。此后，ECMAScript 每年发表一次新标准，至今最新的为 [ECMASCript2020](https://tc39.es/ecma262/)

## 1. API

**浏览器 API** 内建于浏览器中，JS 可以通过调用浏览器 API 筛选当前计算机环境的相关数据，从而实现复杂的工作，例如

- [文档对象模型 （DOM） API](https://developer.mozilla.org/zh-CN/docs/Web/API/Document_Object_Model) 能动态地操作 HTML 和 CSS。比如当某个页面出现了一个弹窗，或者显示了一些新内容，这就是 DOM 在运行。
- [地理位置 API（Geolocation API）](https://developer.mozilla.org/zh-CN/docs/Web/API/Geolocation) 获取地理信息。
- [画布（Canvas）](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API) 和 [WebGL](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGL_API) API 可以创建生动的 2D/3D 动画。比如 [webglsamples](http://webglsamples.org/)。
- [HTMLMediaElement](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLMediaElement) 和 [WebRTC](https://developer.mozilla.org/zh-CN/docs/Web/API/WebRTC_API) 等影音类 API 可以让我们利用多媒体，比如在网页中直接播放音乐和影片，或用自己的网络摄像头获取录像，然后在其他人的电脑上展示。

第三方 API 则没有默认嵌入浏览器，需要从网上获得它们地代码和信息，比如：

- [Twitter API](https://dev.twitter.com/overview/documentation)、[新浪微博 API](https://open.weibo.com/) 可以在网站上展示最新推文之类
- [谷歌地图 API](https://developers.google.com/maps/)、[高德地图 API](https://lbs.amap.com/) 可以在网站嵌入定制的地图等等。

以合理的方式调用这些 API 是我们在使用 JS 过程中做的最多的事情。

## 2. JS 对页面做了什么

首先明确，HTML 和 CSS 都就位后，JS 才开始运行，因为 JS 主要作用是通过调用 DOM API 动态修改 HTML 和 CSS，如果同时执行，可能引发错误。

每个浏览器标签页是一个运行代码的独立容器，不同标签页中的代码一般不能相互影响。

JS 是解释型语言，代码自上而下运行，实时地返回运行结果，因此要注意代码书写的顺序。

```js
const para = document.querySelector('p');

para.addEventListener('click', updateName);

function updateName() {
  let name = prompt('输入一个新的名字：');
  para.textContent = '玩家1：' + name;
}
```

上例中，首先选定一个文本段落，然后给它附上一个事件监听器，使得在它被点击时，`updateName()` 代码块（code block）便会运行。`updateName()` 向用户请求一个新名字，然后把这个名字插入到段落中以更新显示。如果互换前两行代码，浏览器开发者控制台将返回一个错误： `TypeError: para is undefined`。这意味着 `para` 对象还不存在，所以我们不能为它增添一个事件监听器。

## 3. 向页面添加 JS

CSS 使用 `<style>` 元素向 HTML 嵌入内部样式表，或者使用 `<link>` 元素链接外部样式表。JS 则只有一个元素可用：`<script>`，内部 JavaScript 是直接将代码放在标签中，例如

```html
<script>
document.addEventListener("DOMContentLoaded", function() {
  function createParagraph() {
    let para = document.createElement('p');
    para.textContent = '你点击了这个按钮！';
    document.body.appendChild(para);
  }

  const buttons = document.querySelectorAll('button');

  for(let i = 0; i < buttons.length ; i++) {
    buttons[i].addEventListener('click', createParagraph);
  }
});
</script>
```

外部 JavaScript 是将 JS 代码保存为文件，然后在 `<script> `标签中引用。

```js
function createParagraph() {
  let para = document.createElement('p');
  para.textContent = '你点击了这个按钮！';
  document.body.appendChild(para);
}

const buttons = document.querySelectorAll('button');

for(let i = 0; i < buttons.length ; i++) {
  buttons[i].addEventListener('click', createParagraph);
}
```

将上面的 JS 代码保存到 `script.js` 文件中，放在与 HTML 文件同目录下，然后在 HTML 文件中添加如下代码即可，效果是相同的。

```html
<script src="script.js" async></script>
```

后者是推荐做法，应保存样式与动作分离。

## 4. 脚本调用时机

HTML 代码同样按次序调用，用 JS 管理页面元素时，若 JS 在欲操作的元素前加载，代码将出错。

第三部分中，内部 JS 代码调用时使用了事件监听器监听浏览器的 `DOMContentLoaded` 事件，当 HTML 文档体加载、解释完毕后，事件触发时才会调用代码，从而避免了错误发生。而外部 JS 代码则是声明了 `async` 异步属性来解决该问题，它告知浏览器在遇到 `<script>` 元素时不要中断后续 HTML 内容的加载。

原始的方法是将所有脚本元素放在 HTML 文档体末尾加载，但对 JS 代码比较多的网站性能损耗明显。

延缓加载 JS 有两种关键词：async 和 defer。前者是在加载 JS 代码的同时接着渲染后面的内容，相当于两件事并行完成，无法保证脚本调用的顺序，适用于页面种的脚本间相互独立的情况。如下，script2.js 可能在 jquery 之前执行。

```html
<script async src="js/vendor/jquery.js"></script>
<script async src="js/script2.js"></script>
<script async src="js/script3.js"></script>
```

后者使脚本按照页面中出现的顺序加载，如下，执行顺序为严格的 jquery.js，script2.js，script3.js

```html
<script defer src="js/vendor/jquery.js"></script>
<script defer src="js/script2.js"></script>
<script defer src="js/script3.js"></script>
```

## 5. 注释

与大部分语言相同，JS 的单行注释也是 `//`，多行注释也是 `/* …… */`

