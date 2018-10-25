# Page Lifecycle API

Android、iOS 和最新的 Windows 系统上，操作系统可以随时自主地启动和停止后台进程。这是为了及时释放系统资源，提高性能，延长电池的使用时间。

以前的浏览器 API 完全没有考虑到这种情况，导致开发者根本没有办法监听到系统丢弃页面。W3C 新制定了一个 Page Lifecycle API，解决这个问题。

这个 API 主要有两个作用，第一是统一了网页从诞生到卸载的行为模式，第二是创建新的 API 和事件，允许开发者响应网页状态的每一种转换。

有了这个 API，开发者就可以预测网页下一步的状态，从而进行各种针对性的处理。Chrome 68 支持这个 API，对于老式浏览器可以使用谷歌开发的兼容库 [PageLifecycle.js](https://github.com/GoogleChromeLabs/page-lifecycle)。

## 生命周期阶段

网页的生命周期分成六个阶段，每个时刻只可能处于其中一个阶段。

**（1）Active 阶段**

在 Active 阶段，网页处于可见状态，且拥有输入焦点。

**（2）Passive 阶段**

在 Passive 阶段，网页可见，但没有处于输入焦点，无法接受输入。UI 更新和动画仍然在执行。该阶段只可能发生在桌面同时有多个窗口的情况。

**（3）Hidden 阶段**

在 Hidden 阶段，网页不可见，但尚未冻结。UI 界面不再运行。

**（4）Terminated 阶段**

在 Terminated 阶段，网页不可见，由于用户主动关闭窗口，或者在同一个窗口前往其他页面，导致当前页面开始被浏览器卸载并从内存中清除。这个阶段一般由用户手动触发。任何新任务都不可以在此状态下启动，并且如果运行时间太长，正在进行的任务可能会被终止。

**（5）Frozen 阶段**

在 Frozen 阶段，网页冻结，不会再被分配 CPU 计算资源，定时器、回调函数、网络请求、DOM 操作都不会执行，不过正在运行的任务会执行完。通常，Hidden 阶段的页面长时间不打开，就会进入 Frozen 阶段。不过，也有可能，处于可见状态的页面变成 Frozen 阶段。

Frozen 阶段的页面可能会周期性复苏一小段时间，短暂变成 Hidden 状态，允许做一小部分工作。

**（6）Discarded 阶段**

在 Discarded 阶段，浏览器卸载网页，清除该网页的内存占用，以便减少内存占用，一般是在用户没有介入的情况下，由操作系统强制执行。任何类型的新任务或 JavaScript 代码，都不能在此状态下运行，因为这时通常处在在资源限制的状况下，无法启动新进程。

通常情况下，如果 Frozen 状态的网页长时间没有唤醒，就会进入 Discarded 阶段。Passive 阶段的网页也可能直接进入 Discarded 阶段。

如果用户重新访问 Discarded 阶段的网页，浏览器将会重新向服务器发出请求。

## 常见场景

以下是几个常见场景的网页生命周期变化。

（1）用户打开网页后，又长时间去使用其他 App，系统自动丢弃网页。

网页由 Active 变成 Hidden，再变成 Frozen，最后 Discarded。

（2）用户打开网页后，又去使用其他 App，然后从任务管理器里面将浏览器清除。

网页由 Active 变成 Hidden，然后 Terminated。

（3）系统丢弃了某个 Tab 里面的页面后，用户重新打开这个 Tab。

网页由 Discarded 变成 Active。

## 事件

与生命周期各个阶段相配合的，还有一系列事件。其中，只有两个事件是新定义的，其它都是现存的。

生命周期事件将在所有帧（frame）触发，不管是底层的帧，还是内嵌的帧。

**（1）focus 事件**

`focus`事件在页面获得输入焦点时触发。网页从 Passive 阶段变为 Active 阶段。

**（2）blur 事件**

`blur`事件在页面失去输入焦点时触发。网页从 Active 阶段变为 Passive 阶段。

**（3）visibilitychange 事件**

`visibilitychange`事件在网页可见状态发生变化时触发，一般发生在以下几种场景。

- 用户隐藏页面（切换 Tab、最小化浏览器），页面由 Active 阶段变成 Hidden 阶段。
- 用户重新访问隐藏的页面，页面由 Hidden 阶段变成 Active 阶段。
- 用户关闭页面，页面会先进入 Hidden 阶段，然后进入 Terminated 阶段。

可以通过`document.onvisibilitychange`属性指定这个事件的回调函数。

**（4）freeze 事件**

`freeze`事件在网页进入 Frozen 阶段时触发。

可以通过`document.onfreeze`属性指定在进入 Frozen 阶段时调用的回调函数。

```javascript
function handleFreeze(e) {
  // Handle transition to FROZEN
}
document.addEventListener('freeze', handleFreeze);

# 或者
document.onfreeze = function() { … }
```

这个事件的监听函数，最长只能运行500毫秒。并且只能复用已经打开的网络连接，不能发起新的网络请求。

注意，从 Frozen 阶段进入 Discarded 阶段，不会触发任何事件，无法指定回调函数，只能在进入 Frozen 阶段时指定回调函数。

**（5）resume 事件**

`resume`事件在网页离开 Frozen 阶段，变为 Active / Passive / Hidden 阶段时触发。

`onresume`属性指定用户重新访问页面，是的页面离开 Frozen 阶段、进入可用阶段时调用的回调函数。

```javascript
function handleResume(e) {
  // handle state transition FROZEN -> ACTIVE
}
document.addEventListener("resume", handleResume);

# 或者
document.onresume = function() { … }
```

**（6）pageshow 事件**

`pageshow`事件在用户加载网页时触发。这时，有可能是全新的页面加载，也可能是从缓存中获取的页面。如果是从缓存中获取，则该事件对象的`event.persisted`属性为`true`，否则为`false`。

这个事件的名字有点误导，它跟页面的可见性其实毫无关系，只能浏览器的 History 记录的变化有关。

**（7）pagehide 事件**

`pagehide`事件在用户离开当前网页、进入另一个网页时触发。它的前提是浏览器的 History y 记录必须发生变化，跟网页是否可见无关。

如果浏览器能够将当前页面添加到缓存以供稍后重用，则事件对象的`event.persisted`属性为`true`。 如果为`true`。如果页面添加到了缓存，则页面进入 Frozen 状态，否则进入 Terminatied 状态。

**（8）beforeunload 事件**

`beforeunload`事件在窗口或文档即将卸载时触发。该事件发生时，文档仍然可见，此时事件仍可取消。经过这个事件，网页进入 Terminated 状态。

**（9）unload 事件**

`unload`事件在页面正在卸载时触发。经过这个事件，网页进入 Terminated 状态。

## 获取当前阶段

如果网页处于 Active、Passive 或 Hidden 阶段，可以通过下面的代码，获得网页当前的状态。

```javascript
const getState = () => {
  if (document.visibilityState === 'hidden') {
    return 'hidden';
  }
  if (document.hasFocus()) {
    return 'active';
  }
  return 'passive';
};
```

如果网页处于 Frozen 和 Terminated 状态，由于定时器代码不会执行，只能通过事件监听判断状态。进入 Frozen 阶段，可以监听`freeze`事件；进入 Terminated 阶段，可以监听`pagehide`事件。

## document.wasDiscarded

如果某个选项卡处于 Frozen 阶段，就随时有可能被系统丢弃，进入 Discarded 阶段。如果后来用户再次点击该选项卡，浏览器会重新加载该页面。

这时，开发者可以通过判断`document.wasDiscarded`属性，了解先前的网页是否被丢弃了。

```javascript
if (document.wasDiscarded) {
  // 该网页已经不是原来的状态了，曾经被浏览器丢弃过
  // 恢复以前的状态
  getPersistedState(self.discardedClientId);
}
```

同时，`window`对象上会新增`clientId`和`discardedClientId`两个属性，用来恢复丢弃前的状态。

## 参考链接

- [Page Lifecycle API](https://developers.google.com/web/updates/2018/07/page-lifecycle-api), Philip Walton
- [Lifecycle API for Web Pages](https://github.com/WICG/page-lifecycle), W3C
- [Page Lifecycle 1 Editor’s Draft](https://wicg.github.io/page-lifecycle/spec.html), W3C
