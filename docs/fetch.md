# Fetch API

## 基本用法

Ajax 操作使用的`XMLHttpRequest`对象，已经有十多年的历史了，它的API设计并不是很好，输入、输出、状态都在同一个接口管理，容易写出非常混乱的代码。Fetch API 是一种新规范，用来取代`XMLHttpRequest`对象。

它主要有两个特点，一是接口合理化，Ajax 将所有不同性质的接口都放在 XHR 对象上，而 Fetch 将它们分散在几个不同的对象上，设计更合理；二是 Fetch 操作返回`Promise`对象，避免了嵌套的回调函数。

下面的代码检查浏览器是否部署了 Fetch API。

```javascript
if ('fetch' in window){
  // 支持
} else {
  // 不支持
}
```

下面是一个 Fetch API 的简单例子。

```javascript
fetch(url)
.then(function (response) {
  return response.json();
})
.then(function (jsonData) {
  console.log(jsonData);
})
.catch(function () {
  console.log('出错了');
});
```

上面代码首先向`url`发出请求，得到回应后，将其转为 JSON 格式，输出到控制台。如果出错，则输出一条提示信息。这里需要注意，`fetch`方法返回的是一个 Promise 对象。

作为比较，`XMLHttpRequest`的写法如下。

```javascript
var xhr = new XMLHttpRequest();
xhr.open('GET', url);
xhr.responseType = 'json';
xhr.onload = function() {
  console.log(xhr.response);
};
xhr.onerror = function() {
  console.log('出错了');
};
xhr.send();
```

## stream 数据流

除了返回`Promise`对象，Fetch API 还有一个特点，就是数据传送是以数据流（stream）的形式进行的。对于大文件，数据是一段一段得到的。

```javascript
response.text().then(function (responseText) {
  console.log(responseText);
}
```

上面代码中，`text()`就是一个数据流读取器。

Fetch API 提供以下五个数据流读取器。

- `.text()`：返回字符串
- `.json()`：返回JSON对象
- `.formData()`：返回`FormData`对象
- `.blob()`：返回`blob`对象
- `.arrayBuffer()`：返回二进制数组`ArrayBuffer`对象

数据流只能读取一次，一旦读取，数据流就空了。再次读取就不会得到结果。解决方法是在读取之前，先使用`.clone()`方法，复制一份一模一样的副本。

```javascript
var url = 'LargeFile.txt';
var progress = 0;
var contentLength = 0;

fetch(url).then(function (response) {
  // 本次请求总的数据长度
  contentLength = response.headers.get('Content-Length');
  var getStream = function (reader) {
    // ...
  };
  return getStream(response.body.getReader());
})
.catch(function (error) {
  console.log(error);
});
```

上面代码中，`response.body.getReader()`返回的就是数据流之中的一段。处理数据流的`getStream`函数代码如下。

```javascript
var progress = 0;
var contentLength = 0;

var getStream = function (reader) {
  return reader.read().then(function (result) {
    // 如果数据已经读取完毕，直接返回
    if (result.done) {
      return;
    }

    // 取出本段数据（二进制格式）
    var chunk = result.value;

    var text = '';
    // 假定数据是UTF-8编码，前三字节是数据头，
    // 而且每个字符占据一个字节（即都为英文字符）
    for (var i = 3; i < chunk.byteLength; i++) {
      text += String.fromCharCode(chunk[i]);
    }

    // 将本段数据追加到网页之中
    document.getElementById('content').innerHTML += text;

    // 计算当前进度
    progress += chunk.byteLength;
    console.log(((progress / contentLength) * 100) + '%');

    // 递归处理下一段数据
    return getStream(reader);
  };
};
```

上面这样的数据流处理，可以提高网站性能表现，减少内存占用，对于请求大文件或者网速慢的场景相当有用。传统的`XMLHTTPRequest`对象不支持数据流，所有的数据必须放在缓存里，等到全部拿到后，再一次性吐出来。

## fetch()

`fetch`方法的第一个参数可以是 URL 字符串，也可以是后文要讲到的`Request`对象实例。`Fetch`方法返回一个`Promise`对象，并将一个`response`对象传给回调函数。

`response`对象有一个`ok`属性，如果返回的状态码在200到299之间（即请求成功），这个属性为`true`，否则为`false`。因此，判断请求是否成功的代码可以写成下面这样。

```javascript
fetch('./api/some.json').then(function (response) {
  if (response.ok) {
    response.json().then(function (data) {
      console.log(data);
    });
  } else {
    console.log('请求失败，状态码为', response.status);
  }
}, function(err) {
  console.log('出错：', err);
});
```

`response`对象除了`json`方法，还包含了服务器 HTTP 回应的元数据。

```javascript
fetch('users.json').then(function(response) {
  console.log(response.headers.get('Content-Type'));
  console.log(response.headers.get('Date'));
  console.log(response.status);
  console.log(response.statusText);
  console.log(response.type);
  console.log(response.url);
});
```

上面代码中，`response`对象有很多属性，其中的`response.type`属性比较特别，表示HTTP回应的类型，它有以下三个值。

- basic：正常的同域请求
- cors：CORS 机制下的跨域请求
- opaque：非 CORS 机制下的跨域请求，这时无法读取返回的数据，也无法判断是否请求成功

如果需要在 CORS 机制下发出跨域请求，需要指明状态。

```javascript
fetch('http://some-site.com/cors-enabled/some.json', {mode: 'cors'})
  .then(function(response) {
    return response.text();
  })
  .then(function(text) {
    console.log('Request successful', text);
  })
  .catch(function(error) {
    log('Request failed', error)
  });
```

除了指定模式，fetch 方法的第二个参数还可以用来配置其他值，比如指定 cookie 连同 HTTP 请求一起发出。

```javascript
fetch(url, {
  credentials: 'include'
})
```

发出 POST 请求的写法如下。

```javascript
fetch('http://www.example.org/submit.php', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded'
  },
  body: 'firstName=Nikhil&favColor=blue&password=easytoguess'
}).then(function(res) {
  if (res.ok) {
    console.log('Perfect! Your settings are saved.');
  } else if (res.status == 401) {
    console.log('Oops! You are not authorized.');
  }
}, function(e) {
  console.log('Error submitting form!');
});
```

## Headers

Fetch API 引入三个新的对象（也是构造函数）：`Headers`, `Request`和`Response`。其中，`Headers`对象用来构造/读取 HTTP 数据包的头信息。

```javascript
var content = 'Hello World';
var headers = new Headers();
headers.append('Accept', 'application/json');
headers.append('Content-Type', 'text/plain');
headers.append('Content-Length', content.length.toString());
headers.append('X-Custom-Header', 'ProcessThisImmediately');
```

`Headers`对象的实例，除了使用`append`方法添加属性，也可以直接通过构造函数一次性生成。

```javascript
var reqHeaders = new Headers({
  'Content-Type': 'text/plain',
  'Content-Length': content.length.toString(),
  'X-Custom-Header': 'ProcessThisImmediately',
});
```

Headers 对象实例还提供了一些工具方法。

```javascript
reqHeaders.has('Content-Type') // true
reqHeaders.has('Set-Cookie') // false
reqHeaders.set('Content-Type', 'text/html')
reqHeaders.append('X-Custom-Header', 'AnotherValue')

reqHeaders.get('Content-Length') // 11
reqHeaders.getAll('X-Custom-Header') // ["ProcessThisImmediately", "AnotherValue"]

reqHeaders.delete('X-Custom-Header')
reqHeaders.getAll('X-Custom-Header') // []
```

生成 Header 实例以后，可以将它作为第二个参数，传入`Request`方法。

```javascript
var headers = new Headers();
headers.append('Accept', 'application/json');
var request = new Request(URL, {headers: headers});

fetch(request).then(function(response) {
  console.log(response.headers);
});
```

同样地，Headers 实例可以用来构造 Response 方法。

```javascript
var headers = new Headers({
  'Content-Type': 'application/json',
  'Cache-Control': 'max-age=3600'
});

var response = new Response(
  JSON.stringify({photos: {photo: []}}),
  {'status': 200, headers: headers}
);

response.json().then(function(json) {
  insertPhotos(json);
});
```

上面代码中，构造了一个 HTTP 回应。目前，浏览器构造 HTTP 回应没有太大用处，但是随着 Service Worker 的部署，不久浏览器就可以向 Service Worker 发出 HTTP 回应。

## Request对象

`Request`对象用来构造 HTTP 请求。

```javascript
var req = new Request('/index.html');
req.method // "GET"
req.url // "http://example.com/index.html"
```

`Request`对象的第二个参数，表示配置对象。

```javascript
var uploadReq = new Request('/uploadImage', {
  method: 'POST',
  headers: {
    'Content-Type': 'image/png',
  },
  body: 'image data'
});
```

上面代码指定`Request`对象使用 POST 方法发出，并指定 HTTP 头信息和信息体。

下面是另一个例子。

```javascript
var req = new Request(URL, {method: 'GET', cache: 'reload'});
fetch(req).then(function(response) {
  return response.json();
}).then(function(json) {
  someOperator(json);
});
```

上面代码中，指定请求方法为 GET，并且要求浏览器不得缓存 response。

`Request`对象实例有两个属性是只读的，不能手动设置。一个是`referrer`属性，表示请求的来源，由浏览器设置，有可能是空字符串。另一个是`context`属性，表示请求发出的上下文，如果值是`image`，表示是从`<img>`标签发出，如果值是`worker`，表示是从`worker`脚本发出，如果是`fetch`，表示是从`fetch`函数发出的。

`Request`对象实例的`mode`属性，用来设置是否跨域，合法的值有以下三种：same-origin、no-cors（默认值）、cors。当设置为`same-origin`时，只能向同域的 URL 发出请求，否则会报错。

```javascript
var arbitraryUrl = document.getElementById("url-input").value;
fetch(arbitraryUrl, { mode: "same-origin" }).then(function(res) {
  console.log("Response succeeded?", res.ok);
}, function(e) {
  console.log("Please enter a same-origin URL!");
});
```

上面代码中，如果用户输入的URL不是同域的，将会报错，否则就会发出请求。

如果`mode`属性为`no-cors`，就与默认的浏览器行为没有不同，类似`<script>`标签加载外部脚本文件、`<img>`标签加载外部图片。如果`mode`属性为`cors`，就可以向部署了`CORS`机制的服务器，发出跨域请求。

```javascript
var u = new URLSearchParams();
u.append('method', 'flickr.interestingness.getList');
u.append('api_key', '<insert api key here>');
u.append('format', 'json');
u.append('nojsoncallback', '1');

var apiCall = fetch('https://api.flickr.com/services/rest?' + u);

apiCall.then(function(response) {
  return response.json().then(function(json) {
    // photo is a list of photos.
    return json.photos.photo;
  });
}).then(function(photos) {
  photos.forEach(function(photo) {
    console.log(photo.title);
  });
});
```

上面代码是向 Flickr API 发出图片请求的例子。

`Request`对象的一个很有用的功能，是在其他`Request`实例的基础上，生成新的`Request`实例。

```javascript
var postReq = new Request(req, {method: 'POST'});
```

## Response

fetch方法返回Response对象实例，它有以下属性。

- `status`：整数值，表示状态码（比如200）
- `statusText`：字符串，表示状态信息，默认是“OK”
- `ok`：布尔值，表示状态码是否在200-299的范围内
- `headers`：Headers对象，表示HTTP回应的头信息
- `url`：字符串，表示HTTP请求的网址
- `type`：字符串，合法的值有五个basic、cors、default、error、opaque。basic表示正常的同域请求；cors表示CORS机制的跨域请求；`error`表示网络出错，无法取得信息，`status`属性为0，`headers`属性为空，并且导致`fetch`函数返回`Promise`对象被拒绝；`opaque`表示非 CORS 机制的跨域请求，受到严格限制。

`Response`对象还有两个静态方法。

- Response.error() 返回一个type属性为error的Response对象实例
- Response.redirect(url, status) 返回的Response对象实例会重定向到另一个URL

```javascript
fetch('https://example.com', init)
.then(function (response) {
// Check that the response is a 200
  if (response.status === 200) {
    alert("Content type: " + response.headers.get('Content-Type'));
  }
});
```

## body属性

`Request`对象和`Response`对象都有`body`属性，表示请求的内容。`body`属性可能是以下的数据类型。

- ArrayBuffer
- ArrayBufferView (Uint8Array等)
- Blob/File
- string
- URLSearchParams
- FormData

```javascript
var form = new FormData(document.getElementById('login-form'));
fetch("/login", {
  method: "POST",
  body: form
})
```

上面代码中，`Request`对象的`body`属性为表单数据。

`Request`对象和`Response`对象都提供以下方法，用来读取`body`。

- arrayBuffer()
- blob()
- json()
- text()
- formData()

注意，上面这些方法都只能使用一次，第二次使用就会报错，也就是说，`body`属性只能读取一次。`Request`对象和`Response`对象都有`bodyUsed`属性，返回一个布尔值，表示`body`是否被读取过。

```javascript
var res = new Response('one time use');
console.log(res.bodyUsed); // false
res.text().then(function(v) {
  console.log(res.bodyUsed); // true
});
console.log(res.bodyUsed); // true

res.text().catch(function(e) {
  console.log('Tried to read already consumed Response');
});
```

上面代码中，第二次通过`text`方法读取`Response`对象实例的`body`时，就会报错。

这是因为`body`属性是一个stream对象，数据只能单向传送一次。这样的设计是为了允许 JavaScript 处理视频、音频这样的大型文件。

如果希望多次使用`body`属性，可以使用`Response`对象和`Request`对象的`clone`方法。它必须在`body`还没有读取前调用，返回一个新的`body`，也就是说，需要使用几次`body`，就要调用几次`clone`方法。

```javascript
addEventListener('fetch', function(evt) {
  var sheep = new Response("Dolly");
  console.log(sheep.bodyUsed); // false
  var clone = sheep.clone();
  console.log(clone.bodyUsed); // false

  clone.text();
  console.log(sheep.bodyUsed); // false
  console.log(clone.bodyUsed); // true

  evt.respondWith(cache.add(sheep.clone()).then(function(e) {
    return sheep;
  });
});
```

## reader 的实例

```javascript
const fetchUrl = async function (url) {
  const response = await fetch(url);
  const reader = response.body.getReader();
  const chunk1 = await reader.read();
  console.log(chunk1);
}
```

## 参考链接

- [A Guide to Faster Web App I/O and Data Operations with Streams](https://www.sitepen.com/blog/2017/10/02/a-guide-to-faster-web-app-io-and-data-operations-with-streams/), by Umar Hansa
