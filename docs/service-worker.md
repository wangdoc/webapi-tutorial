# Service Worker

Service Worker 是一个后台运行的脚本，充当一个代理服务器，拦截用户发出的网络请求，比如加载脚本和图片。Service Worker 可以修改用户的请求，或者直接向用户发出回应，不用联系服务器，这使得用户可以在离线情况下使用网络应用。它还可以在本地缓存资源文件，直接从缓存加载文件，因此可以加快访问速度。

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

```javascript
if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('/service-worker.js').then (registration => {
        console.log('Service Worker registered successfully!')
    }, (error) => {
        console.log('Error in registering service worker', error)
    })
}
```

默认情况下，Service worker 只对根目录`/`生效，如果要改变生效范围，可以运行下面的代码。

```javascript
navigator.serviceWorker.register(
  '/service-worker.js',
  { scope: '/products/fashion' }
)
```

### 安装

一旦登记成功，接下来都是 Service worker 脚本的工作。

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

安装完成后，Service worker 就会等待激活。

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
