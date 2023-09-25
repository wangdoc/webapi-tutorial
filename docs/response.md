# Response API

浏览器原生提供`Response()`构造函数，用来构造服务器响应。

`fetch()`方法返回的就是一个 Response 对象。

## 构造方法

`Response()`作为构造方法调用时，返回 Response 实例。

```javascript
// 定义
new Response([body:Object, [init : Object]]): Response

// 用法
new Response()
new Response(body)
new Response(body, options)
```

它带有两个参数，都是可选的。

第一个参数`body`代表服务器返回的数据体，必须是下面类型之一：ArrayBuffer、ArrayBufferView、Blob、FormData、ReadableStream、String、URLSearchParams。

第二个参数`init`是一个对象，代表服务器返回的数据头，类型描述如下。

```javascript
{
  status: Number
  statusText: String
  headers: Object
}
```

下面是一个例子。

```javascript
const myBlob = new Blob();
const myOptions = { status: 200, statusText: "OK" };
const myResponse = new Response(myBlob, myOptions);
```

注意，如果返回 JSON 数据，必须将其转成字符串返回。

```javascript
const data = {
  hello: "world",
};

const json = JSON.stringify(data, null, 2);

const result = new Response(json, {
  headers: {
    "content-type": "application/json;charset=UTF-8",
  },
});
```

上面示例中，构造一个返回 JSON 数据的 Response 对象，就必须用`JSON.stringify()`方法，将第一个参数转为字符串。

## 实例属性

### body，bodyUsed

`body`属性代表数据体，是一个只读的 ReadableStream 对象。

```javascript
const res = await fetch('/fireworks.ogv');
const reader = res.body.getReader();

let result;
while (!(result = await reader.read()).done) {
  console.log('chunk size:', result.value.byteLength);
}
```

上面示例中，先建立一个 body 的读取器，然后每次读取一段数据，输出这段数据的字段长度。

注意，`body`是一个 Stream 对象，只能读取一次。取出所有数据以后，第二次就读不到了。

`bodyUsed`属性是一个只读的布尔值，表示`body`属性是否已经读取。

### headers

`headers`属性代表服务器返回的数据头，是一个只读的 Headers 对象。

```javascript
const res = await fetch('/flowers.jpg');
console.log(...res.headers);
```

上面示例中，发出请求后，展开打印`res.headers`属性，即服务器回应的所有消息头。

### ok

`ok`属性是一个布尔值，表示服务器返回的状态码是否成功（200到299），该属性只读。

```javascript
const res1 = await fetch('https://httpbin.org/status/200');
console.log(res1.ok); // true

const res2 = await fetch('https://httpbin.org/status/404');
console.log(res2.ok); // false
```

### redirected

`redirected`是一个布尔值，表示服务器返回的状态码是否跳转类型（301，302等），该属性只读。

```javascript
const res1 = await fetch('https://httpbin.org/status/200');
console.log(res1.redirected); // false

const res2 = await fetch('https://httpbin.org/status/301');
console.log(res2.redirected); // true
```

### status，statusText

`status`属性是一个数值，代表服务器返回的状态码，该属性只读。

```javascript
const res1 = await fetch('https://httpbin.org/status/200');
console.log(res1.status); // 200

const res2 = await fetch('https://httpbin.org/status/404');
console.log(res2.status); // 404
```

`statusText`属性是一个字符串，代表服务器返回的状态码的文字描述。比如，状态码200的`statusText`一般是`OK`，也可能为空。

### type

`type`属性是一个只读字符串，表示服务器回应的类型，它的值有下面几种：basic、cors、default、error、opaque、opaqueredirect。

### url

`url`属性是一个字符串，代表服务器路径，该属性只读。如果请求是重定向的，该属性就是重定向后的 URL。

## 实例方法

### 数据读取

以下方法可以获取服务器响应的消息体，根据返回数据的不同类型，调用相应方法。

- .json()：返回一个 Promise 对象，最终得到一个解析后的 JSON 对象。
- .text()：返回一个 Promise 对象，最终得到一个字符串。
- .blob()：返回一个 Promise 对象，最终得到一个二进制 Blob 对象，代表某个文件整体的原始数据。
- .arrayBuffer()：返回一个 Promise 对象，最终得到一个 ArrayBuffer 对象，代表一段固定长度的二进制数据。
- .formData()：返回一个 Promise 对象，最终得到一个 FormData 对象，里面是键值对形式的表单提交数据。

下面是从服务器获取 JSON 数据的一个例子，使用`.json()`方法，其他几个方法的用法都大同小异。

```javascript
async function getRedditPosts() {
  try {
    const response = await fetch('https://www.reddit.com/r/all/top.json?limit=10');
    const data = await response.json();
    const posts = data.data.children.map(child => child.data);
    console.log(posts.map(post => post.title));
  } catch (error) {
    console.error(error);
  }
}
```

下面是从服务器获取二进制文件的例子，使用`.blob()`方法。

```javascript
async function displayImageAsync() {
  try {
    const response = await fetch('https://www.example.com/image.jpg');
    const blob = await response.blob();
    const url = URL.createObjectURL(blob);
    const img = document.createElement('img');
    img.src = url;
    document.body.appendChild(img);
  } catch (error) {
    console.error(error);
  }
}
```

下面是从服务器获取音频文件，直接解压播放的例子，使用`.arrayBuffer()`方法。

```javascript
async function playAudioAsync() {
  try {
    const response = await fetch('https://www.example.com/audio.mp3');
    const arrayBuffer = await response.arrayBuffer();
    const audioBuffer = await new AudioContext().decodeAudioData(arrayBuffer);
    const source = new AudioBufferSourceNode(new AudioContext(), { buffer: audioBuffer });
    source.connect(new AudioContext().destination);
    source.start(0);
  } catch (error) {
    console.error(error);
  }
}
```

### clone()

`clone()`方法用来复制 Response 对象。

```javascript
const res1 = await fetch('/flowers.jpg');
const res2 = res1.clone();
```

复制以后，读取一个对象的数据，不会影响到另一个对象。

## 静态方法

### Response.json()

`Response.json()`返回一个 Response 实例，该实例对象的数据体就是作为参数的 JSON 数据，数据头的`Content-Type`字段自动设为`application/json`。

```javascript
Response.json(data)
Response.json(data, options)
```

`Response.json()`基本上就是`Response()`构造函数的变体。

下面是示例。

```javascript
const jsonResponse1 = Response.json({ my: "data" });

const jsonResponse2 = Response.json(
  { some: "data", more: "information" },
  { status: 307, statusText: "Temporary Redirect" },
);
```

### Response.error()

`Response.error()`用来构造一个表示报错的服务器回应，主要用在 Service worker，表示拒绝发送。

```javascript
self.addEventListener('fetch', (event) => {
  const url = new URL(event.request.url);
  if (url.pathname === '/flowers.jpg') {
    event.respondWith(Response.error());
  }
});
```

### Response.redirect()

`Response.redirect()`用来构造一个表示跳转的服务器回应，主要用在 Service worker，表示跳转到其他网址。

```javascript
Response.redirect(url)
Response.redirect(url, status)
```

这个方法的第一个参数`url`是所要跳转的目标网址，第二个参数是状态码，一般是301或302（默认值）。

```javascript
Response.redirect("https://www.example.com", 302);
```

