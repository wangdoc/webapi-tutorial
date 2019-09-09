# Service Worker

## 含义

Service Worker 首先是一个运行在后台的 Worker 线程，然后它会长期运行，充当一个服务，很适合那些不需要网页或用户互动的功能。它的最常见用途就是拦截和处理网络请求。

Service Worker 是一个后台运行的脚本，充当一个代理服务器，拦截用户发出的网络请求，比如加载脚本和图片。Service Worker 可以修改用户的请求，或者直接向用户发出回应，不用联系服务器，这使得用户可以在离线情况下使用网络应用。它还可以在本地缓存资源文件，直接从缓存加载文件，因此可以加快访问速度。

```javascript
if ('serviceWorker' in navigator) {
  window.addEventListener('load', function() {
    navigator.serviceWorker.register('/service-worker.js');
  });
}
```

上面代码确认浏览器支持 Service Worker 以后，会注册一个 Service Worker。

为了节省内存，Service worker 在不使用的时候是休眠的。它也不会保存数据，所以重新启动的时候，为了拿到数据，最好把数据放在 IndexedDb 里面。

Service Worker 是事件驱动的。

下面是拦截请求的例子。

```javascript
self.addEventListener('fetch', (event) => {
  event.waitUntil(
    if (event.request.url.includes('/product') {
      let productId = event.data.productId
      let productCount = getProductData(productId)
      indexedDB.open('store', 1, (db) => {
        let productStore = db.createObjectStore('products', { keyPath: 'id' })
        productStore.put({ id: productId, count: ++productCount })
      })
    })
  )
})
```

Service Worker 不能直接操作 DOM。

## 使用步骤

### 登记

使用 service worker 的第一步，就是告诉浏览器，需要注册一个 service worker 脚本。

```javascript
navigator.serviceWorker.register('sw.js'.then(() => {
  console.info('注册成功')
}).catch((err) => {
  console.error('注册失败')
})
```

上面代码的`sw.js`就是需要浏览器注册的 service worker 脚本。注意，这个脚本必须与当前网址同域，service worker 不支持跨与脚本。另外，`sw.js`必须是从 HTTPS 协议加载的。

默认情况下，Service worker 只对根目录`/`生效，如果要改变生效范围，可以运行下面的代码。

```javascript
navigator.serviceWorker.register(
  '/service-worker.js',
  { scope: '/products/fashion' }
)
```

### 安装

一旦登记成功，接下来都是 service worker 脚本的工作。下面的代码都是写在 service worker 脚本里面的。

登记后，就会触发`install`事件。service worker 脚本需要监听这个事件。

```javascript
self.addEventListener('install', event => {

  event.waitUntil(() => console.info('安装完成'))
})
```

`event.waitUntil()`方法为事件完成后指定回调函数。

```javascript
self.addEventListener('install', (event) => {
    let CACHE_NAME = 'xyz-cache'
    let urlsToCache = [
        '/',
        '/styles/main.css',
        '/scripts/bundle.js'
    ]
    event.waitUntil(
        caches.open(CACHE_NAME)
        .then (cache => cache.addAll(urlsToCache))
    )
})
```

### 激活

安装完成后，service worker 就会等待激活。

```javascript
self.addEventListener('activate', (event) => {
    let cacheWhitelist = ['products-v2']

    event.waitUntil(
        caches.keys().then (cacheNames => {
            return Promise.all(
                cacheNames.map( cacheName => {
                    if (cacheWhitelist.indexOf(cacheName) === -1) {
                        return caches.delete(cacheName)
                    }
                })
            )
        })
    )
})
```

## Service Worker 与网页的通信

```javascript
self.addEventListener('activate', (event) => {
    event.waitUntil(
        self.clients.matchAll().then ( (client) => {
            client.postMessage({
                msg: 'Hey, from service worker! I\'m listening to your fetch requests.',
                source: 'service-worker'
            })
        })
    )
})
```

上面代码中，Service Worker 监听`activate`事件，然后向客户端发送一条信息。

客户端需要部署消息监听代码。

```javascript
this.addEventListener('message', (data) => {
    if (data.source == 'service-worker') {
        console.log(data.msg)
    }
})
```

## 参考链接

- [Service Workers](https://frontendian.co/service-workers), by Ryan Miller
