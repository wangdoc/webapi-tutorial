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
