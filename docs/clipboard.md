# 剪贴板操作 Clipboard API 教程

## 简介

浏览器允许 JavaScript 脚本读写剪贴板，自动复制或粘贴内容。

一般来说，脚本不应该改动用户的剪贴板，以免不符合用户的预期。但是，有些时候这样做确实能够带来方便，比如“一键复制”功能，用户点击一下按钮，指定的内容就自动进入剪贴板。

目前，一共有三种方法可以实现剪贴板操作。

- `Document.execCommand()`方法
- 异步的 Clipboard API
- `copy`事件和`paste`事件

本文逐一介绍这三种方法。

## Document.execCommand() 方法

`Document.execCommand()`是操作剪贴板的传统方法，各种浏览器都支持。

它支持复制、剪切和粘贴这三个操作。

- `document.execCommand('copy')`（复制）
- `document.execCommand('cut')`（剪切）
- `document.execCommand('paste')`（粘贴）

（1）复制操作

复制时，先选中文本，然后调用`document.execCommand('copy')`，选中的文本就会进入剪贴板。

```javascript
const inputElement = document.querySelector('#input');
inputElement.select();
document.execCommand('copy');
```

上面示例中，脚本先选中输入框`inputElement`里面的文字（`inputElement.select()`），然后`document.execCommand('copy')`将其复制到剪贴板。

注意，复制操作最好放在事件监听函数里面，由用户触发（比如用户点击按钮）。如果脚本自主执行，某些浏览器可能会报错。

（2）粘贴操作

粘贴时，调用`document.execCommand('paste')`，就会将剪贴板里面的内容，输出到当前的焦点元素中。

```javascript
const pasteText = document.querySelector('#output');
pasteText.focus();
document.execCommand('paste');
```

（3）缺点

`Document.execCommand()`方法虽然方便，但是有一些缺点。

首先，它只能将选中的内容复制到剪贴板，无法向剪贴板任意写入内容。

其次，它是同步操作，如果复制/粘贴大量数据，页面会出现卡顿。有些浏览器还会跳出提示框，要求用户许可，这时在用户做出选择前，页面会失去响应。

为了解决这些问题，浏览器厂商提出了异步的 Clipboard API。

## 异步 Clipboard API

Clipboard API 是下一代的剪贴板操作方法，比传统的`document.execCommand()`方法更强大、更合理。

它的所有操作都是异步的，返回 Promise 对象，不会造成页面卡顿。而且，它可以将任意内容（比如图片）放入剪贴板。

`navigator.clipboard`属性返回 Clipboard 对象，所有操作都通过这个对象进行。

```javascript
const clipboardObj = navigator.clipboard;
```

如果`navigator.clipboard`属性返回`undefined`，就说明当前浏览器不支持这个 API。

由于用户可能把敏感数据（比如密码）放在剪贴板，允许脚本任意读取会产生安全风险，所以这个 API 的安全限制比较多。

首先，Chrome 浏览器规定，只有 HTTPS 协议的页面才能使用这个 API。不过，开发环境（`localhost`）允许使用非加密协议。

其次，调用时需要明确获得用户的许可。权限的具体实现使用了 Permissions API，跟剪贴板相关的有两个权限：`clipboard-write`（写权限）和`clipboard-read`（读权限）。“写权限”自动授予脚本，而“读权限”必须用户明确同意给予。也就是说，写入剪贴板，脚本可以自动完成，但是读取剪贴板时，浏览器会弹出一个对话框，询问用户是否同意读取。

![](https://www.wangbase.com/blogimg/asset/202101/bg2021012004.jpg)

另外，需要注意的是，脚本读取的总是当前页面的剪贴板。这带来的一个问题是，如果把相关的代码粘贴到开发者工具中直接运行，可能会报错，因为这时的当前页面是开发者工具的窗口，而不是网页页面。

```javascript
(async () => {
  const text = await navigator.clipboard.readText();
  console.log(text);
})();
```

如果你把上面的代码，粘贴到开发者工具里面运行，就会报错。因为代码运行的时候，开发者工具窗口是当前页，这个页面不存在 Clipboard API 依赖的 DOM 接口。一个解决方法就是，相关代码放到`setTimeout()`里面延迟运行，在调用函数之前快速点击浏览器的页面窗口，将其变成当前页。

```javascript
setTimeout(async () => {
  const text = await navigator.clipboard.readText();
  console.log(text);
}, 2000);
```

上面代码粘贴到开发者工具运行后，快速点击一下网页的页面窗口，使其变为当前页，这样就不会报错了。

## Clipboard 对象

Clipboard 对象提供了四个方法，用来读写剪贴板。它们都是异步方法，返回 Promise 对象。

### Clipboard.readText()

`Clipboard.readText()`方法用于复制剪贴板里面的文本数据。

```javascript
document.body.addEventListener(
  'click',
  async (e) => {
    const text = await navigator.clipboard.readText();
    console.log(text);
  }
)
```

上面示例中，用户点击页面后，就会输出剪贴板里面的文本。注意，浏览器这时会跳出一个对话框，询问用户是否同意脚本读取剪贴板。

如果用户不同意，脚本就会报错。这时，可以使用`try...catch`结构，处理报错。

```javascript
async function getClipboardContents() {
  try {
    const text = await navigator.clipboard.readText();
    console.log('Pasted content: ', text);
  } catch (err) {
    console.error('Failed to read clipboard contents: ', err);
  }
}
```

### Clipboard.read()

`Clipboard.read()`方法用于复制剪贴板里面的数据，可以是文本数据，也可以是二进制数据（比如图片）。该方法需要用户明确给予许可。

该方法返回一个 Promise 对象。一旦该对象的状态变为 resolved，就可以获得一个数组，每个数组成员都是 ClipboardItem 对象的实例。

```javascript
async function getClipboardContents() {
  try {
    const clipboardItems = await navigator.clipboard.read();
    for (const clipboardItem of clipboardItems) {
      for (const type of clipboardItem.types) {
        const blob = await clipboardItem.getType(type);
        console.log(URL.createObjectURL(blob));
      }
    }
  } catch (err) {
    console.error(err.name, err.message);
  }
}
```

ClipboardItem 对象表示一个单独的剪贴项，每个剪贴项都拥有`ClipboardItem.types`属性和`ClipboardItem.getType()`方法。

`ClipboardItem.types`属性返回一个数组，里面的成员是该剪贴项可用的 MIME 类型，比如某个剪贴项可以用 HTML 格式粘贴，也可以用纯文本格式粘贴，那么它就有两个 MIME 类型（`text/html`和`text/plain`）。

`ClipboardItem.getType(type)`方法用于读取剪贴项的数据，返回一个 Promise 对象。该方法接受剪贴项的 MIME 类型作为参数，返回该类型的数据，该参数是必需的，否则会报错。

### Clipboard.writeText()

`Clipboard.writeText()`方法用于将文本内容写入剪贴板。

```javascript
document.body.addEventListener(
  'click',
  async (e) => {
    await navigator.clipboard.writeText('Yo')
  }
)
```

上面示例是用户在网页点击后，脚本向剪贴板写入文本数据。

该方法不需要用户许可，但是最好也放在`try...catch`里面防止报错。

```javascript
async function copyPageUrl() {
  try {
    await navigator.clipboard.writeText(location.href);
    console.log('Page URL copied to clipboard');
  } catch (err) {
    console.error('Failed to copy: ', err);
  }
}
```

### Clipboard.write()

`Clipboard.write()`方法用于将任意数据写入剪贴板，可以是文本数据，也可以是二进制数据。

该方法接受一个 ClipboardItem 实例作为参数，表示写入剪贴板的数据。

```javascript
try {
  const imgURL = 'https://dummyimage.com/300.png';
  const data = await fetch(imgURL);
  const blob = await data.blob();
  await navigator.clipboard.write([
    new ClipboardItem({
      [blob.type]: blob
    })
  ]);
  console.log('Image copied.');
} catch (err) {
  console.error(err.name, err.message);
}
```

上面示例中，脚本向剪贴板写入了一张图片。注意，Chrome 浏览器目前只支持写入 PNG 格式的图片。

`ClipboardItem()`是浏览器原生提供的构造函数，用来生成`ClipboardItem`实例，它接受一个对象作为参数，该对象的键名是数据的 MIME 类型，键值就是数据本身。

下面的例子是将同一个剪贴项的多种格式的值，写入剪贴板，一种是文本数据，另一种是二进制数据，供不同的场合粘贴使用。

```javascript
function copy() {
  const image = await fetch('kitten.png');
  const text = new Blob(['Cute sleeping kitten'], {type: 'text/plain'});
  const item = new ClipboardItem({
    'text/plain': text,
    'image/png': image
  });
  await navigator.clipboard.write([item]);
}
```

## copy 事件，cut 事件

用户向剪贴板放入数据时，将触发`copy`事件。

下面的示例是将用户放入剪贴板的文本，转为大写。

```javascript
const source = document.querySelector('.source');

source.addEventListener('copy', (event) => {
  const selection = document.getSelection();
  event.clipboardData.setData('text/plain', selection.toString().toUpperCase());
  event.preventDefault();
});
```

上面示例中，事件对象的`clipboardData`属性包含了剪贴板数据。它是一个对象，有以下属性和方法。

- `Event.clipboardData.setData(type, data)`：修改剪贴板数据，需要指定数据类型。
- `Event.clipboardData.getData(type)`：获取剪贴板数据，需要指定数据类型。
- `Event.clipboardData.clearData([type])`：清除剪贴板数据，可以指定数据类型。如果不指定类型，将清除所有类型的数据。
- `Event.clipboardData.items`：一个类似数组的对象，包含了所有剪贴项，不过通常只有一个剪贴项。

下面的示例是拦截用户的复制操作，将指定内容放入剪贴板。

```javascript
const clipboardItems = [];

document.addEventListener('copy', async (e) => {
  e.preventDefault();
  try {
    let clipboardItems = [];
    for (const item of e.clipboardData.items) {
      if (!item.type.startsWith('image/')) {
        continue;
      }
      clipboardItems.push(
        new ClipboardItem({
          [item.type]: item,
        })
      );
      await navigator.clipboard.write(clipboardItems);
      console.log('Image copied.');
    }
  } catch (err) {
    console.error(err.name, err.message);
  }
});
```

上面示例中，先使用`e.preventDefault()`取消了剪贴板的默认操作，然后由脚本接管复制操作。

`cut`事件则是在用户进行剪切操作时触发，它的处理跟`copy`事件完全一样，也是从`Event.clipboardData`属性拿到剪切的数据。

## paste 事件

用户使用剪贴板数据，进行粘贴操作时，会触发`paste`事件。

下面的示例是拦截粘贴操作，由脚本将剪贴板里面的数据取出来。

```javascript
document.addEventListener('paste', async (e) => {
  e.preventDefault();
  const text = await navigator.clipboard.readText();
  console.log('Pasted text: ', text);
});
```

## 参考链接

- [Unblocking clipboard access](https://web.dev/async-clipboard/)
- [Interact with the clipboard](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Interact_with_the_clipboard)
- [Multi-MIME Type Copying with the Async Clipboard API](https://blog.tomayac.com/2020/03/20/multi-mime-type-copying-with-the-async-clipboard-api/)

