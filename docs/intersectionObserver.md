# IntersectionObserver

网页开发时，常常需要了解某个元素是否进入了“视口”（viewport），即用户能不能看到它。

![](http://www.ruanyifeng.com/blogimg/asset/2016/bg2016110201.gif)

上图的绿色方块不断滚动，顶部会提示它的可见性。

传统的实现方法是，监听到`scroll`事件后，调用目标元素（绿色方块）的[`getBoundingClientRect()`](https://developer.mozilla.org/en/docs/Web/API/Element/getBoundingClientRect)方法，得到它对应于视口左上角的坐标，再判断是否在视口之内。这种方法的缺点是，由于`scroll`事件密集发生，计算量很大，容易造成[性能问题](http://www.ruanyifeng.com/blog/2015/09/web-page-performance-in-depth.html)。

[IntersectionObserver API](https://wicg.github.io/IntersectionObserver/)，可以自动“观察”元素是否可见，Chrome 51+ 已经支持。由于可见（visible）的本质是，目标元素与视口产生一个交叉区，所以这个 API 叫做“交叉观察器”（intersection oberserver）。

## 简介

IntersectionObserver API 的用法，简单来说就是两行。

```javascript
var observer = new IntersectionObserver(callback, options);
observer.observe(target);
```

上面代码中，`IntersectionObserver`是浏览器原生提供的构造函数，接受两个参数：`callback`是可见性变化时的回调函数，`option`是配置对象（该参数可选）。

`IntersectionObserver()`的返回值是一个观察器实例。实例的`observe()`方法可以指定观察哪个 DOM 节点。

```javascript
// 开始观察
observer.observe(document.getElementById('example'));

// 停止观察
observer.unobserve(element);

// 关闭观察器
observer.disconnect();
```

上面代码中，`observe()`的参数是一个 DOM 节点对象。如果要观察多个节点，就要多次调用这个方法。

```javascript
observer.observe(elementA);
observer.observe(elementB);
```

注意，IntersectionObserver API 是异步的，不随着目标元素的滚动同步触发。规格写明，`IntersectionObserver`的实现，应该采用`requestIdleCallback()`，即只有线程空闲下来，才会执行观察器。这意味着，这个观察器的优先级非常低，只在其他任务执行完，浏览器有了空闲才会执行。

## IntersectionObserver.observe()

IntersectionObserver 实例的`observe()`方法用来启动对一个 DOM 元素的观察。该方法接受两个参数：回调函数`callback`和配置对象`options`。

### callback 参数

目标元素的可见性变化时，就会调用观察器的回调函数`callback`。

`callback`会触发两次。一次是目标元素刚刚进入视口（开始可见），另一次是完全离开视口（开始不可见）。

```javascript
var observer = new IntersectionObserver(
  (entries, observer) => {
    console.log(entries);
  }
);
```

上面代码中，回调函数采用的是[箭头函数](http://es6.ruanyifeng.com/#docs/function#箭头函数)的写法。`callback`函数的参数（`entries`）是一个数组，每个成员都是一个[`IntersectionObserverEntry`](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry)对象（详见下文）。举例来说，如果同时有两个被观察的对象的可见性发生变化，`entries`数组就会有两个成员。

### IntersectionObserverEntry 对象

`IntersectionObserverEntry`对象提供目标元素的信息，一共有六个属性。

```javascript
{
  time: 3893.92,
  rootBounds: ClientRect {
    bottom: 920,
    height: 1024,
    left: 0,
    right: 1024,
    top: 0,
    width: 920
  },
  boundingClientRect: ClientRect {
     // ...
  },
  intersectionRect: ClientRect {
    // ...
  },
  intersectionRatio: 0.54,
  target: element
}
```

每个属性的含义如下。

> - `time`：可见性发生变化的时间，是一个高精度时间戳，单位为毫秒
> - `target`：被观察的目标元素，是一个 DOM 节点对象
> - `rootBounds`：容器元素的矩形区域的信息，`getBoundingClientRect()`方法的返回值，如果没有容器元素（即直接相对于视口滚动），则返回`null`
> - `boundingClientRect`：目标元素的矩形区域的信息
> - `intersectionRect`：目标元素与视口（或容器元素）的交叉区域的信息
> - `intersectionRatio`：目标元素的可见比例，即`intersectionRect`占`boundingClientRect`的比例，完全可见时为`1`，完全不可见时小于等于`0`

![](http://www.ruanyifeng.com/blogimg/asset/2016/bg2016110202.png)

上图中，灰色的水平方框代表视口，深红色的区域代表四个被观察的目标元素。它们各自的`intersectionRatio`图中都已经注明。

我写了一个 [Demo](http://jsbin.com/canuze/edit?js,console,output)，演示`IntersectionObserverEntry`对象。注意，这个 Demo 只能在 Chrome 51+ 运行。

### Option 对象

`IntersectionObserver`构造函数的第二个参数是一个配置对象。它可以设置以下属性。

**（1）threshold 属性**

`threshold`属性决定了什么时候触发回调函数，即元素进入视口（或者容器元素）多少比例时，执行回调函数。它是一个数组，每个成员都是一个门槛值，默认为`[0]`，即交叉比例（`intersectionRatio`）达到`0`时触发回调函数。

如果`threshold`属性是0.5，当元素进入视口50%时，触发回调函数。如果值为`[0.3, 0.6]`，则当元素进入30％和60％是触发回调函数。

```javascript
new IntersectionObserver(
  entries => {/* … */},
  {
    threshold: [0, 0.25, 0.5, 0.75, 1]
  }
);
```

用户可以自定义这个数组。比如，上例的`[0, 0.25, 0.5, 0.75, 1]`就表示当目标元素 0%、25%、50%、75%、100% 可见时，会触发回调函数。

![](http://www.ruanyifeng.com/blogimg/asset/2016/bg2016110202.gif)

**（2）root 属性，rootMargin 属性**

`IntersectionObserver`不仅可以观察元素相对于视口的可见性，还可以观察元素相对于其所在容器的可见性。容器内滚动也会影响目标元素的可见性，参见本文开始时的那张示意图。

IntersectionObserver API 支持容器内滚动。`root`属性指定目标元素所在的容器节点。注意，容器元素必须是目标元素的祖先节点。

```javascript
var opts = {
  root: document.querySelector('.container'),
  rootMargin: '0px 0px -200px 0px'
};

var observer = new IntersectionObserver(
  callback,
  opts
);
```

上面代码中，除了`root`属性，还有[`rootMargin`](https://wicg.github.io/IntersectionObserver/#dom-intersectionobserverinit-rootmargin)属性。该属性用来扩展或缩小`rootBounds`这个矩形的大小，从而影响`intersectionRect`交叉区域的大小。它的写法类似于 CSS 的`margin`属性，比如`0px 0px 0px 0px`，依次表示 top、right、bottom 和 left 四个方向的值。

上例的`0px 0px -200px 0px`，表示容器的下边缘向上收缩200像素，导致页面向下滚动时，目标元素的顶部进入可视区域200像素以后，才会触发回调函数。

这样设置以后，不管是窗口滚动或者容器内滚动，只要目标元素可见性变化，都会触发观察器。

## 实例

### 惰性加载（lazy load）

有时，我们希望某些静态资源（比如图片），只有用户向下滚动，它们进入视口时才加载，这样可以节省带宽，提高网页性能。这就叫做“惰性加载”。

有了 IntersectionObserver API，实现起来就很容易了。图像的 HTML 代码可以写成下面这样。

```html
<img src="placeholder.png" data-src="img-1.jpg">
<img src="placeholder.png" data-src="img-2.jpg">
<img src="placeholder.png" data-src="img-3.jpg">
```

上面代码中，图像默认显示一个占位符，`data-src`属性是惰性加载的真正图像。

```javascript
function query(selector) {
  return Array.from(document.querySelectorAll(selector));
}

var observer = new IntersectionObserver(
  function(entries) {
    entries.forEach(function(entry) {
      entry.target.src = entry.target.dataset.src;
      observer.unobserve(entry.target);
    });
  }
);

query('.lazy-loaded').forEach(function (item) {
  observer.observe(item);
});
```

上面代码中，只有图像开始可见时，才会加载真正的图像文件。

### 无限滚动

无限滚动（infinite scroll）指的是，随着网页滚动到底部，不断加载新的内容到页面，它的实现也很简单。

```javascript
var intersectionObserver = new IntersectionObserver(
  function (entries) {
    // 如果不可见，就返回
    if (entries[0].intersectionRatio <= 0) return;
    loadItems(10);
    console.log('Loaded new items');
  }
);

// 开始观察
intersectionObserver.observe(
  document.querySelector('.scrollerFooter')
);
```

无限滚动时，最好像上例那样，页面底部有一个页尾栏（又称[sentinels](sentinels)，上例是`.scrollerFooter`）。一旦页尾栏可见，就表示用户到达了页面底部，从而加载新的条目放在页尾栏前面。否则就需要每一次页面加入新内容时，都调用`observe()`方法，对新增内容的底部建立观察。

### 视频自动播放

下面是一个视频元素，希望它完全进入视口的时候自动播放，离开视口的时候自动暂停。

```html
<video src="foo.mp4" controls=""></video>
```

下面是 JS 代码。

```javascript
let video = document.querySelector('video');
let isPaused = false;

let observer = new IntersectionObserver((entries, observer) => {
  entries.forEach(entry => {
    if (entry.intersectionRatio != 1  && !video.paused) {
      video.pause();
      isPaused = true;
    } else if (isPaused) {
      video.play();
      isPaused=false;
    }
  });
}, {threshold: 1});

observer.observe(video);
```

上面代码中，`IntersectionObserver()`的第二个参数是配置对象，它的`threshold`属性等于`1`，即目标元素完全可见时触发回调函数。

## 参考链接

- [IntersectionObserver’s Coming into View](https://developers.google.com/web/updates/2016/04/intersectionobserver)
- [Intersection Observers Explained](https://github.com/WICG/IntersectionObserver/blob/gh-pages/explainer.md)
- [A Few Functional Uses for Intersection Observer to Know When an Element is in View](https://css-tricks.com/a-few-functional-uses-for-intersection-observer-to-know-when-an-element-is-in-view/), Preethi

