# Request API

浏览器原生提供 Request() 构造函数，用来构造发给服务器的 HTTP 请求。它生成的 Response 实例，可以作为`fetch()`的参数。

注意，构造一个 Request 对象，只是构造出一个数据结构，本身并不会发出 HTTP 请求，只有将它传入`fetch()`方法才会真的发出请求。

## 构造方法

Request 作为构造函数的语法如下，返回一个 Request 实例对象。

```javascript
new Request(url: String, [init: Object]): Request
```

它的第一个参数是请求的 URL 字符串，第二个参数是一个可选的配置对象，用来构造 HTTP 请求，该对象的类型描述如下。

```javascript
{
  body: Object
  cache: String
  credentials: String
  headers: Object
  integrity: String
  keepalive: Boolean
  method: String
  mode: String
  redirect:	String
  referrer:	String
  referrerPolicy: String
  requestMode: String
  requestCredentials: String
  signal: AbortSignal
}
```

第二个参数配置对象的各个属性的含义如下。

- `body`：HTTP 请求的数据体，必须是 Blob、BufferSource、FormData、String、URLSearchParams 类型之一。
- `cache`：请求的缓存模式。
- `credentials`：请求所用的凭证，可以设为 omit、same-origini、include。Chrome 47 之前，默认值为 same-origin；Chrome 47 之后，默认值为 include。
- `headers`：一个代表 HTTP 请求数据头的对象，类型为 Headers 对象实例。
- `integrity`：请求的资源的资源完整度验证值，比如`sha256-BpfBw7ivV8q2jLiT13fxDYAe2tJllusRSZ273h2nFSE=`。
- `method`：HTTP 方法，一般为`GET`、`POST`、`DELETE`，默认是`GET`。
- `mode`：请求模式，比如 cors、no-cors、navigate，默认为 cors。
- `redirect`：请求所用的模式，可以设为 error、follow、manual，默认为 follow。
- `referrer`：请求的来源，默认为 about:client。

下面是两个示例。

```javascript
// 示例一
const request = new Request('flowers.jpg');

// 示例二
const myInit = {
  method: "GET",
  headers: {
    "Content-Type": "image/jpeg",
  },
  mode: "cors",
  cache: "default",
};

const request = new Request('flowers.jpg', myInit);
```

`Request()`还有另一种语法，第一个参数是另一个 Request 对象，第二个参数还是一个配置对象。它返回一个新的 Request 对象，相当于对第一个参数 Request 对象进行修改。

```javascript
new Request(request: Request, [init: Object]): Request
```

## 实例属性

Request 实例对象的属性，大部分就是它的构造函数第二个参数配置对象的属性。

（1）`body`

`body`属性返回 HTTP 请求的数据体，它的值是一个 ReadableStream 对象或 null（`GET`或`HEAD`请求时没有数据体）。

```javascript
const request = new Request('/myEndpoint', {
  method: "POST",
  body: "Hello world",
});

request.body; // ReadableStream 对象
```

注意，Firefox 不支持该属性。

（2）`bodyused`

`bodyUsed`属性是一个布尔值，表示`body`是否已经被读取了。

（3）`cache`

`cache`属性是一个只读字符串，表示请求的缓存模式，可能的值有 default、force-cache、no-cache、no-store、only-if-cached、reload。

（4）`credentials`

`credentials`属性是一个只读字符串，表示跨域请求时是否携带其他域的 cookie。可能的值有 omit（不携带）、 include（携带）、same-origin（只携带同源 cookie）。

（5）`destination`

`destination`属性是一个字符串，表示请求内容的类型，可能的值有 ''、'audio'、'audioworklet'、'document'、'embed'、'font'、'frame'、'iframe'、'image'、'manifest'、'object'、'paintworklet'、 'report'、'script'、'sharedworker'、'style'、'track'、'video'、'worker'、'xslt' 等。

（6）`headers`

`headers`属性是一个只读的 Headers 实例对象，表示请求的数据头。

（7）`integrity`

`integrity`属性表示所请求资源的完整度的验证值。

（8）`method`

`method`属性是一个只读字符串，表示请求的方法（GET、POST 等）。

（9）`mode`

`mode`属性是一个只读字符串，用来验证是否可以有效地发出跨域请求，可能的值有 same-origin、no-cors、cors。

（10）`redirect`

`redirect`属性是一个只读字符串，表示重定向时的处理模式，可能的值有 follow、error、manual。

（11）`referrer`

`referrer`属性是一个只读字符串，表示请求的引荐 URL。

（12）`referrerPolicy`

`referrerPolicy`属性是一个只读字符串，决定了`referrer`属性是否要包含在请求里面的处理政策。

（13）`signal`

`signal`是一个只读属性，包含与当前请求相对应的中断信号 AbortSignal 对象。

（14）`url`

`url`是一个只读字符串，包含了当前请求的字符串。

```javascript
const myRequest = new Request('flowers.jpg');
const myURL = myRequest.url;
```

## 实例方法

### 取出数据体的方法

- arrayBuffer()：返回一个 Promise 对象，将 Request 的数据体作为 ArrayBuffer 对象返回。
- blob()：返回一个 Promise 对象，将 Request 的数据体作为 Blob 对象返回。
- json()：返回一个 Promise 对象，将 Request 的数据体作为 JSON 对象返回。
- text()：返回一个 Promise 对象，将 Request 的数据体作为字符串返回。
- formData()：返回一个 Promise 对象，将 Request 的数据体作为表单数据 FormData 对象返回。

下面是`json()`方法的一个示例。

```javascript
const obj = { hello: "world" };

const request = new Request("/myEndpoint", {
  method: "POST",
  body: JSON.stringify(obj),
});

request.json().then((data) => {
  // 处理 JSON 数据
});
```

`.formData()`方法返回一个 Promise 对象，最终得到的是一个 FormData 表单对象，里面是用键值对表示的各种表单元素。该方法很少使用，因为需要拦截发给服务器的请求的场景不多，一般用在 Service Worker 拦截和处理网络请求，以修改表单数据，然后再发送到服务器。

```javascript
self.addEventListener('fetch', event => {
  // 拦截表单提交请求
  if (
    event.request.method === 'POST' &&
    event.request.headers.get('Content-Type') === 'application/x-www-form-urlencoded'
  ) {
    event.respondWith(handleFormSubmission(event.request));
  }
});

async function handleFormSubmission(request) {
  const formData = await request.formData();
  formData.append('extra-field', 'extra-value');

  const newRequest = new Request(request.url, {
    method: request.method,
    headers: request.headers,
    body: new URLSearchParams(formData)
  });

  return fetch(newRequest);
}
```

上面示例中，Service Worker 拦截表单请求以后，添加了一个表单成员，再调用`fetch()`向服务器发出修改后的请求。

### clone()

`clone()`用来复制 HTTP 请求对象。

```javascript
const myRequest = new Request('flowers.jpg');
const newRequest = myRequest.clone();
```

