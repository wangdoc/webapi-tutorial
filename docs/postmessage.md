# window.postMessage() 方法

## 简介

`window.postMessage()`用于浏览器不同窗口之间的通信，主要包括 iframe 嵌入窗口和新开窗口两种情况。它不要求两个窗口同源，所以有着广泛的应用。

`window.postMessage()`里面的`window`对象，是发送消息的目标窗口。比如，父窗口通过`window.open()`打开子窗口，那么子窗口可以通过`targetWindow = window.opener`获取父窗口。再比如，父窗口通过`iframe`嵌入了子窗口，那么子窗口可以通过`window.parent`获取父窗口。

## 参数和返回值

`window.postMessage()`方法有几种使用形式。

最简单的一种就是直接发送消息。

```javascript
window.postMessage(message)
```

上面写法中的`message`就是发送的消息，可以是字符串，也可以是对象。如果是对象，浏览器会自动将该对象序列化，以字符串形式发送。

由于`window.postMessage()`可以用于任意两个源（协议+域名+端口）之间的通信，为了减少安全隐患，可以使用第二个参数`targetOrigin`，指定目标窗口的源。

```javascript
window.postMessage(message, targetOrigin)
```

上面写法中的`targetOrigin`是一个字符串，表示目标窗口里面的网页的源（origin），比如`https://example.com`。如果对目标窗口不加限制，可以省略这个参数，或者写成`*`。一旦指定了该参数，只有目标窗口符合指定的源（协议+域名+端口），目标窗口才会接收到消息发送事件。

`window.postMessage()`还可以指定第三个参数，用于发送一些可传送物体（transferable object），比如 ArrayBuffer 对象。

```javascript
window.postMessage(message, targetOrigin, transfer)
```

上面写法中的`transfer`就是可传送物体。该物体一旦发送以后，所有权就转移到了目标窗口，当前窗口将无法再使用该物体。这样的设计是为了发送大量数据时，可以提高效率。

`targetOrigin`和`transfer`这两个参数，也可以写在一个对象里面，作为第二个参数。

```javascript
window.postMessage(message, { targetOrigin, transfer })
```

下面是一个跟弹出窗口发消息的例子。

```javascript
const popup = window.open('http://example.com');
popup.postMessage("hello there!", "http://example.com");
```

`window.postMessage()`方法没有返回值。

## message 事件

当前窗口收到其他窗口发送的消息时，会发生 message 事件。通过监听该事件，可以接收对方发送的消息。

```javascript
window.addEventListener(
  "message",
  (event) => {
    if (event.origin !== "http://example.com") return;
    // ...
  },
  false,
);
```

事件的监听函数，可以接收到一个 event 参数对象。该对象有如下属性。

- data：其他窗口发送的消息。
- origin：发送该消息的窗口的源（协议+域名+端口）。
- source：发送该消息的窗口对象的引用，使用该属性可以建立双向通信，下面是一个示例。

```javascript
window.addEventListener("message", (event) => {
  if (event.origin !== "http://example.com:8080") return;
  event.source.postMessage(
    "hi there!",
    event.origin,
  );
});
```

## 实例

父页面是`origin1.com`，它打开了子页面`origin2.com`，并向其发送消息。

```javascript
function sendMessage() {
  const otherWindow = window.open('https://origin2.com/origin2.html');
  const message = 'Hello from Origin 1!';
  const targetOrigin = 'https://origin2.com';
  otherWindow.postMessage(message, targetOrigin);
}
```

子页面`origin2.com`监听父页面发来的消息。

```javascript
window.addEventListener('message', receiveMessage, false);

function receiveMessage(event) {
  if (event.origin === 'https://origin1.com') {
    console.log('Received message: ' + event.data);
  }
}
```

下面是 iframe 嵌入窗口向父窗口`origin1.com`发送消息的例子。

```javascript
function sendMessage() {
  const message = 'Hello from Child Window!';
  window.parent.postMessage(message, 'https://origin1.com');
}
```

