# Page Lifecycle API

Android、iOS 和最新的 Windows 系统上，操作系统可以随时启动和停止应用程序。系统资源不足的时候，这样可以释放资源，提高系统性能。

虽然，浏览器对网页提供一些生命周期相关事件（比如`load`、`unload`和`visibilitychange`），但是缺乏系统性的 API，而且没有考虑操作系统主动杀死进程的情况。

新的 Page Lifecycle API 尝试解决这个问题。

- 标准化网页的生命周期状态的概念。
- 定义新的系统启动状态，允许浏览器限制隐藏或非活动选项卡。
- 创建新的 API 和事件，允许开发者响应这些状态的转换。

有了这个 API，开发者就可以预测网页下一步的状态，从而进行各种针对性的处理。Chrome 68 支持这个 API，对于老式浏览器可以使用谷歌开发的兼容库 [PageLifecycle.js](https://github.com/GoogleChromeLabs/page-lifecycle)。

## 简介

网页的生命周期分成不同的阶段，每个时刻只可能处于某一个阶段。

- Active：网页可见，且处于输入焦点。
- Passive：网页可见，但没有处于输入焦点。UI 更新和动画仍然在执行。
- Hidden：网页不可见，当尚未冻结。
- Frozen：网页冻结，定时器和回调函数不会执行，不过正在运行的任务会执行完。
- Terminated：网页开始被浏览器卸载并从内存中清除，一般由用户手动触发。任何新任务都不可以在此状态下启动，并且如果运行时间太长，正在进行的任务可能会被终止。
- Discarded：浏览器卸载网页，一般是在用户没有介入的情况下，由浏览器或操作系统强制执行。任何类型的新任务或 JavaScript 代码，都不能在此状态下运行，因为这时通常处在在资源限制的状况下，无法启动新进程。

跟上面这些阶段相配合的，还有一系列事件。

- focus 事件：页面获得焦点，将从 Passive 状态变为 Active 状态。
- blur 事件：页面失去焦点，将从 Active 状态变为 Passive 状态。
- visibilitychange 事件：用户前往前页面、切换选项卡、关闭选项卡、最小化或关闭浏览器时，都会发生该时间。经过这个事件，网页可能处于 Passive 和 Hidden 状态。
- freeze 事件：网页被冻结，可能从 Hidden 状态变为 Frozen 状态。
- resume 时间：网页从冻结状态恢复，可能从 Froze 状态变为 Active / Passive / Hidden 状态。 
- pageshow 事件：用户加载网页时触发，可能是全新的页面加载，也可能是从缓存中获取的页面，跟网页是否可见无关。如果是从缓存中获取，则该事件的`persisted`属性为`true`，否则为`false`。
- pagehide 事件：用户离开当前网页，进入另一个网页时触发，跟网页是否可见无关。如果浏览器能够将当前页面添加到缓存以供稍后重用，则事件的`persisted`属性为`true`。 如果为`true`，则页面进入 Frozen 状态，否则进入 Terminatied 状态。
- beforeunload 事件：窗口，文档即将卸载时触发。该事件发生时，文档仍然可见，此时事件仍可取消。经过这个事件，网页进入 Terminated 状态。
- unload 事件：页面正在卸载时触发。经过这个事件，网页进入 Terminated 状态。

状态和事件有下面几种可能的搭配。

- Active 状态 + blur 事件 = Passive 状态
- Passive 状态 + focus 事件 = Active 状态
- Passive 状态 + visibilitychange 事件 = Hidden 状态
- Hidden 状态 + visibilitychange 事件 = Passive 状态
- Hidden 状态 + freezen 事件 = Frozen 状态
- Hidden 状态 + pagehide 事件 = Terminated 状态
- Frozen 状态 + resume 事件 = Active 状态 / Passive 状态 / Hidden 状态

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

如果网页处于 Frozen 和 Terminated 状态，由于定时器代码不会执行，只能在事件监听时（监听 freeze 事件 和 pagehide 事件）判断状态。

如果网页处于 Discarded 状态，可以通过`document.wasDiscarded`属性判断（见下文）。

## Frozen 状态的监听

这个 API 出现之前，开发者无法知道浏览器何时冻结网页（即 Frozen 状态）。现在，通过监听`freeze`事件和`resume`事件，可以了解网页何时进入/离开`Frozen`状态。

```javascript
document.addEventListener('freeze', (event) => {
  // The page is now frozen.
});

document.addEventListener('resume', (event) => {
  // The page has been unfrozen.
});
```

## 网页的卸载

网页的卸载分成下面几种情况。

1. Active / Passive 状态时，用户前往另一个网页，或关闭当前窗口。
1. Hidden 状态时，用户关闭窗口。
1. Hidden 状态时，网页被系统冻结（Frozen），进而可能被系统自动丢弃（Discarded）。

第一种和第二种情况时，会触发`beforeunload`、`pagehide`和`unload`这三个事件，第三种情况会触发`freeze`事件。如果要确保代码在卸载时执行，不宜监听这些事件，因为没法保证所有情况下都会执行。尤其是`beforeunload`和`unload`还会阻止浏览器缓存当前网页，基本上，`beforeunload`只适用一种情况：提醒用户，修改表单后别忘了提交。

这时，可以监听到`visibilitychange`事件，把卸载时要执行的代码提前到变成`Hidden`状态时执行。

## document.wasDiscarded

网页处于 Hidden 状态时，当系统资源紧张，该网页可能会在用户没有感知的情况下，自动丢弃。举例来说，某个选项卡的标题，用户仍然能够看见，但网页已经被丢弃了。

这时，用户再次点击该选项卡的时候，浏览器会重新加载该网页。这时，开发者可以通过判断`document.wasDiscarded`属性，了解先前的网页是否被丢弃了。

```javascript
if (document.wasDiscarded) {
  // 不可见的网页已经被浏览器丢弃了
}
```

## 参考链接

- [Page Lifecycle API](https://developers.google.com/web/updates/2018/07/page-lifecycle-api), Philip Walton
