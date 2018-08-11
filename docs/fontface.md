# FontFace API

FontFace API 用来控制字体加载。

这个 API 提供一个构造函数`FontFace()`，返回一个字体对象。

```javascript
new FontFace(family, source, descriptors)
```

`FontFace()`构造函数接受三个参数。

- `family`：字符串，表示字体名，写法与 CSS 的`@font-face`的`font-family`属性相同。
- `source`：字体文件的 URL（必须包括 CSS 的`url()`方法），或者是一个字体的 ArrayBuffer 对象。
- `descriptors`：对象，用来定制字体文件。该参数可选。

```javascript
var fontFace = new FontFace(
  'Roboto',
  'url(https://fonts.example.com/roboto.woff2)'
);

fontFace.family // "Roboto"
```

`FontFace()`返回的是一个字体对象，这个对象包含字体信息。注意，这时字体文件还没有开始加载。

字体对象包含以下属性。

- `FontFace.family`：字符串，表示字体的名字，等同于 CSS 的`font-family`属性。
- `FontFace.display`：字符串，指定字体加载期间如何展示，等同于 CSS 的`font-display`属性。它有五个可能的值：`auto`（由浏览器决定）、`block`（字体加载期间，前3秒会显示不出内容，然后只要还没完成加载，就一直显示后备字体）、`fallback`（前100毫秒显示不出内容，后3秒显示后备字体，然后只要字体还没完成加载，就一直显示后备字体）、`optional`（前100毫秒显示不出内容，然后只要字体还没有完成加载，就一直显示后备字体），`swap`（只要字体没有完成加载，就一直显示后备字体）。
- `FontFace.style`：字符串，等同于 CSS 的`font-style`属性。
- `FontFace.weight`：字符串，等同于 CSS 的`font-weight`属性。
- `FontFace.stretch`：字符串，等同于 CSS 的`font-strentch`属性。
- `FontFace.unicodeRange`：字符串，等同于`descriptors`对象的同名属性。
- `FontFace.variant`：字符串，等同于`descriptors`对象的同名属性。
- `FontFace.featureSettings`：字符串，等同于`descriptors`对象的同名属性。
- `FontFace.status`：字符串，表示字体的加载状态，有四个可能的值：`unloaded`、`loading`、`loaded`、`error`。该属性只读。
- `FontFace.loaded`：返回一个 Promise 对象，字体加载成功或失败，会导致该 Promise 状态改变。该属性只读。

字体对象的方法，只有一个`FontFace.load()`，该方法会真正开始加载字体。它返回一个 Promise 对象，状态由字体加载的结果决定。

```javascript
var f = new FontFace('test', 'url(x)');

f.load().then(function () {
  // 网页可以开始使用该字体
});
```

