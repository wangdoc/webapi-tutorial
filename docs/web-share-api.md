# Web Share API

## 概述

网页内容如果要分享到其他应用，通常要自己实现分享接口，逐一给出目标应用的连接方式。这样很麻烦，也对网页性能有一定影响。Web Share API 就是为了解决这个问题而提出的，允许网页调用操作系统的分享接口，实质是 Web App 与本机的应用程序交换信息的一种方式。

这个 API 不仅可以改善网页性能，而且不限制分享目标的数量和类型。社交媒体应用、电子邮件、即时消息、以及本地系统安装的、且接受分享的应用，都会出现在系统的分享弹窗，这对手机网页尤其有用。另外，使用这个接口只需要一个分享按钮，而传统的网页分享有多个分享目标，就有多少个分享按钮。

目前，桌面的 Safari 浏览器，手机的安卓 Chrome 浏览器和 iOS Safari 浏览器，支持这个 API。

这个 API 要求网站必须启用 HTTPS 协议，但是本地 Localhost 开发可以使用 HTTP 协议。另外，这个 API 不能直接调用，只能用来响应用户的操作（比如`click`事件）。

## 接口细节

该接口部署在`navigator.share`，可以用下面的代码检查本机是否支持该接口。

```javascript
if (navigator.share) {
  // 支持
} else {
  // 不支持
}
```

`navigator.share`是一个函数方法，接受一个配置对象作为参数。

```javascript
navigator.share({
  title: 'WebShare API Demo',
  url: 'https://codepen.io/ayoisaiah/pen/YbNazJ',
  text: '我正在看《Web Share API》'
})
```

配置对象有三个属性，都是可选的，但至少必须指定一个。

- `title`：分享文档的标题。
- `url`：分享的 URL。
- `text`：分享的内容。

一般来说，`url`是当前网页的网址，`title`是当前网页的标题，可以采用下面的写法获取。

```javascript
const title = document.title;
const url = document.querySelector('link[rel=canonical]') ?
  document.querySelector('link[rel=canonical]').href :
  document.location.href;
```

`navigator.share`的返回值是一个 Promise 对象。这个方法调用之后，会立刻弹出系统的分享弹窗，用户操作完毕之后，Promise 对象就会变为`resolved`状态。

```javascript
navigator.share({
  title: 'WebShare API Demo',
  url: 'https://codepen.io/ayoisaiah/pen/YbNazJ'
}).then(() => {
  console.log('Thanks for sharing!');
}).catch((error) => {
  console.error('Sharing error', error);
});
```

由于返回值是 Promise 对象，所以也可以使用`await`命令。

```javascript
shareButton.addEventListener('click', async () => {
  try {
    await navigator.share({ title: 'Example Page', url: '' });
    console.log('Data was shared successfully');
  } catch (err) {
    console.error('Share failed:', err.message);
  }
});
```

## 分享文件

这个 API 还可以分享文件，先使用`navigator.canShare()`方法，判断一下目标文件是否可以分享。因为不是所有文件都允许分享的，目前图像，视频，音频和文本文件可以分享2。

```javascript
if (navigator.canShare && navigator.canShare({ files: filesArray })) {
  // ...
}
```

上面代码中，`navigator.canShare()`方法的参数对象，就是`navigator.share()`方法的参数对象。这里的关键是`files`属性，它的值是一个`FileList`实例对象。

`navigator.canShare()`方法返回一个布尔值，如果为`true`，就可以使用`navigator.share()`方法分享文件了。

```javascript
if (navigator.canShare && navigator.canShare({ files: filesArray })) {
  navigator.share({
    files: filesArray,
    title: 'Vacation Pictures',
    text: 'Photos from September 27 to October 14.',
  })
  .then(() => console.log('Share was successful.'))
  .catch((error) => console.log('Sharing failed', error));
}
```



## 参考链接

- [How to Use the Web Share API](https://css-tricks.com/how-to-use-the-web-share-api/), Ayooluwa Isaiah
- [Web Share API - Level 1](https://wicg.github.io/web-share/), W3C
- [Introducing the Web Share API](https://developers.google.com/web/updates/2016/09/navigator-share), Paul Kinlan, Sam Thorogood
- [Share like a native app with the Web Share API](https://web.dev/web-share/), Joe Medley
