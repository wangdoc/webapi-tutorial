# Offline 应用

Web 应用不仅可以在浏览器缓存资源文件（HTML、CSS、JS 脚本、图片等），还可以把应用本身储存到浏览器。

缓存的资源文件必须在线使用，只有先从服务器加载网页，然后才能使用本地缓存；但是，应用一旦储存，就可以离线使用。另外，用户常规性地清除浏览器缓存，并不会清除储存的应用，除非用户显式地卸载或删除它们。

为了开启离线储存，必须创建一个 manifest 文件。该文件列出了所有需要储存的文件。

```html
CACHE MANIFEST
myapp.html
myapp.js
myapp.css
images/background.png
```

Manifest 文件的第一行必须是`CACHE MANIFEST`。然后，每一行列出一个需要储存的文件，它们的位置都是相对于 Manifest 文件的位置。空行会被忽略，以`#`开头的行是注释，也会被忽略。

这个文件的后缀名一般是`.appcache`。它的 MIME 类型必须是`text/cache-manifest`，如果服务器将其设为其他类型，就不会被浏览器缓存。

编写完这个文件以后，要将`<html>`元素的`manifest`属性指向它。浏览器加载这个网页的时候，就会读取这个 Manifest 文件，离线储存这个网页和相关的资源。

```html
<!DOCTYPE HTML>
<html manifest="myapp.appcache">
<head>...</head>
<body>...</body>
</html>
```

如果一个 Web 应用有多个网页需要离线储存，那么每个网页都应该将`manifest`属性指向这个文件。一旦被储存，以后加载该网页的时候，就会从缓存里面加载。这时，只有 Manifest 文件里面列出的文件会被加载，其他文件不会。如果这时浏览器在线，浏览器就会去检查 Manifest 文件是否有新版本，如果有新版本，就会重新储存和更新该文件列出的资源。最方便的办法是在 Manifest 文件里面用注释列出版本号。

```html
CACHE MANIFEST
# MyApp version 1
MyApp.html
MyApp.js
```

如果需要删除离线储存，只要删除 Manifest 文件，让其返回 404 状态码即可。

离线储存更新完成，会触发浏览器的`updateready`事件，可以对这个事件指定监听函数。

```javascript
window.applicationCache.onupdateready = function() {
  var reload = confirm('新版本下载完成。是否需要重新加载？');
  if (reload) location.reload();
}
```

脚本可以注册`online`和`offline`事件的监听函数，通过`navigator.onLine`属性，判断浏览器是否在线从而进行数据同步。

每次浏览器加载一个具有`manifest`属性的网页，浏览器就会触发一个`checking`事件，然后去加载 Manifest 文件。

- 如果应用已经储存，并且 Manifest 文件没有变化，那么触发`noupdate`事件。
- 如果应用已经储存，并且 Manifest 文件有变化，那么触发`downloading`事件，浏览器重新下载所有离线资源。下载过程中，触发`progress`事件，下载结束触发`updateready`事件。
- 如果应用没有储存，下载结束将触发`cached`事件。
- 如果离线，无法检查 Manifest 文件，浏览器会触发一个`error`事件。
- 如果浏览器在线，而且应用已经储存，但是 Manifest 文件返回 404，浏览器触发`obsolete`事件，将储存的应用移除。

所有这些事件都是可以取消的。监听函数可以返回`false`，取消这些事件的默认动作。

`applicationCache. status`属性返回离线储存的状态。

- ApplicationCache.UNCACHED (0)
This application does not have a manifest attribute: it is not cached.
- ApplicationCache.IDLE (1)
The manifest has been checked and this application is cached and up to date.
- ApplicationCache.CHECKING (2)
The browser is checking the manifest file.
- ApplicationCache.DOWNLOADING (3)
The browser is downloading and caching files listed in the manifest.
- ApplicationCache.UPDATEREADY (4)
A new version of the application has been downloaded and cached.
- ApplicationCache.OBSOLETE (5)
The manifest no longer exists and the cache will be deleted.
