# Request API

浏览器原生提供 Request() 构造函数，用来构造发给服务器的 HTTP 请求。它生成的 Response 实例，可以作为`fetch()`的参数。

## 构造方法

Request 作为构造函数的语法如下，返回一个 Request 实例对象。

```javascript
new Request(url: String, [init: Object]): Request
```

它的第一个参数是请求的 URL 字符串，第二个参数是一个可选的对象，用来构造 HTTP 请求，它的类型如下。

```javascript
{
  body : Object	// 发送的数据体，必须是 Blob、BufferSource、FormData、String、URLSearchParams 类型之一
  cache : String
  credentials :	String
  headers :	Object
  integrity :	String
  keepalive :	Boolean
  method :	String // HTTP 方法，必须是 'GET'、'POST'、'DELETE' 等值
  mode :	String
  redirect :	String // 必须是 'follow'、'error'、'manual' 之一
  referrer :	String
  referrerPolicy :	String
  requestMode :	String
  requestCredentials :	String
  signal :	AbortSignal
}
```

它的第一个参数也可以是另一个 Request 对象，通过第二个参数对其进行修改。

```javascript
new Request(request: Request, [init: Object]): Request
```

## 实例属性

- bodyUsed : Boolean  readonly
- cache : String  readonly。Will be one of: 'default', 'force-cache', 'no-cache', 'no-store', 'only-if-cached', 'reload'.
- credentials : String  readonly。Will be one of: 'include', 'omit', 'same-origin'.
- destination : String  readonly。Will be one of: '', 'audio', 'audioworklet', 'document', 'embed', 'font', 'frame', 'iframe', 'image', 'manifest', 'object', 'paintworklet', 'report', 'script', 'sharedworker', 'style', 'track', 'video', 'worker', 'xslt'.
- headers : Headers  readonly。
- integrity : String  readonly。
- isHistoryNavigation : Boolean  readonly。
- isReloadNavigation : Boolean  readonly。
- keepalive : Boolean  readonly。
- method : String  readonly。
- mode : String  readonly。Will be one of: 'same-origin', 'no-cors', 'cors'.
- redirect : String  readonly。
- referrer : String  readonly。
- referrerPolicy : String  readonly。
- signal : AbortSignal  readonly。
- url : String  readonly。

## 实例方法

### 取出数据体

- arrayBuffer()：返回一个 Promise 对象，将 Request 的数据体作为 ArrayBuffer 对象返回。
- blob()：返回一个 Promise 对象，将 Request 的数据体作为 Blob 对象返回。
- json()：返回一个 Promise 对象，将 Request 的数据体作为 JSON 对象返回。
- text()：返回一个 Promise 对象，将 Request 的数据体作为字符串返回。
- formData()：返回一个 Promise 对象，将 Request 的数据体作为表单数据 FormData 对象返回。

`.formData()`方法返回一个 Promise 对象，最终得到的是一个 FormData 表单对象，里面是用键值对表示的各种表单元素。

该方法很少使用，因为需要拦截发给服务器的请求的场景不多，一般用在 Service Worker。它需要拦截和处理网络请求，修改表单数据，然后将修改后的表单数据发送到服务器。

下面是 Service Worker 拦截表单提交请求，并使用`.formData()`方修改表单数据的例子。

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

