# Clipboard API 教程

## 简介

Clipboard API 用于读写浏览器的剪贴板，取代传统的`document.execCommand()`方法。这个 API 的最大特点，就是所有操作都是异步的，返回 Promise 对象。

通过`navigator.clipboard`属性，可以获取这个 API。

```javascript
const clipboardObj = navigator.clipboard;
```

这个 API 有明确的安全限制。Chrome 浏览器规定，只有 HTTPS 协议的页面才能使用这个 API，不过开发环境（`localhost`）可以用非加密协议。另外，只有当前页才允许脚本访问剪贴板。

注意，由于 Chrome 仅在页面为当前页时才允许剪贴板访问，因此如果直接把代码粘贴到开发者工具中，可能会报错，因为这时开发者工具的窗口本身就是当前页。

```javascript
(async () => {
  const text = await navigator.clipboard.readText();
  console.log(text);
})();
```

上面的代码放到开发者工具里面运行会报错，因为代码运行的时候，开发者工具窗口是当前页，这个页面不存在`document`相关的一系列对象。一个解决方法就是使用`setTimeout()`延迟剪贴板的访问，在调用函数之前快速点击浏览器的页面窗口，将其变成当前页。

```javascript
setTimeout(async () => {
  const text = await navigator.clipboard.readText();
  console.log(text);
}, 2000);
```

Clipboard API 还有权限问题。由于用户可能把敏感数据（比如密码）放在剪贴板，所以这个 API 的调用需要明确获得用户的许可。它的权限许可用了 Permissions API，其中跟剪贴板相关的有两个权限：`clipboard-write`和`clipboard-read`。Chrome 浏览器规定，当页面是当前页时，`clipboard-write`权限会自动授予脚本，`clipboard-read`权限则必须用户明确同意给予。也就是说，在当前页中，脚本调用 Clipboard API 时，可以在不请求权限的情况下写入剪贴板，但是读取剪贴板时，浏览器会弹出一个对话框，询问用户是否给予脚本读取剪贴板的权限。

## 方法

Clipboard 对象提供了四个方法，用来读写剪贴板。它们都是异步方法，返回 Promise 对象。

### Clipboard.read()

`Clipboard.read()`方法拷贝一份剪贴板数据。该方法获取的数据，可以是文本数据，也可以是二进制数据（比如图片）。

该方法返回一个 Promise 对象。一旦该对象的状态变为 resolved，就可以获得一个包含剪贴板内容的 DataTransfer 对象。

```javascript
const promise = navigator.clipboard.read();
```

读取剪贴板之前，必须先获取`clipboard-read`许可。

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

## Clipboard.readText()

`Clipboard.readText()`拷贝一份剪贴板的文本数据。

```javascript
document.body.addEventListener(
  "click",
  async (e) => {
    const text = await navigator.clipboard.readText()
  }
)
```

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

## Clipboard.write()

`Clipboard.write()`用于将任意数据写入剪贴板。

该方法需要需要获取`clipboard-write`权限许可。当页面在活动选项卡时，可以被自动许可。

```javascript
try {
  const imgURL = '/images/generic/file.png';
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

下面的例子是同时复制多种格式的数据，一种是文本数据，一种是二进制数据，供不同的场合粘贴使用。

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

## Clipboard.writeText()

`Clipboard.writeText()`用于将文本内容写入剪贴板。

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

下面是另一个例子。

```javascript
document.body.addEventListener(
  "click",
  async (e) => {
        await navigator.clipboard.writeText("Yo")
    }
)
```

## copy 事件

复试剪贴板数据时，将触发`copy`事件。

```javascript
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

## paste 事件

粘贴剪贴板数据时，可以利用`paste`事件。

```javascript
document.addEventListener('paste', async (e) => {
  e.preventDefault();
  const text = await navigator.clipboard.readText();
  console.log('Pasted text: ', text);
});
```

## 参考链接

- [Unblocking clipboard access](https://web.dev/async-clipboard/)

