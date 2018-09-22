# Geolocation API

Geolocation API 用于获取用户的地理位置。

由于该功能涉及用户隐私，所以浏览器会提示用户，是否同意给出地理位置，用户可能会拒绝。另外，这个 API 只能在 HTTPS 环境使用。

浏览器通过`navigator.geolocation`属性提供该 API。

## Geolocation 对象

`navigator.geolocation`属性返回一个 Geolocation 对象。该对象具有以下三个方法。

- `Geolocation.getCurrentPosition()`：返回一个 Position 对象，表示用户的当前位置。
- `Geolocation.watchPosition()`：指定一个监听函数，每当用户的位置发生变化，就执行该监听函数。
- `Geolocation.clearWatch()`：取消`watchPosition()`方法指定的监听函数。

### Geolocation.getCurrentPosition()

`Geolocation.getCurrentPosition()`方法用于获取用户的位置。

```javascript
navigator.geolocation.getCurrentPosition(success, error, options)
```

该方法接受三个参数。

- `success`：用户同意给出位置时的回调函数，它的参数是一个 Position 对象。
- `error`：用户拒绝给出位置时的回调函数，它的参数是一个 PositionError 对象。该参数可选。
- `options`：参数对象，该参数可选。

Position 对象有两个属性。

- `Position.coords`：返回一个 Coordinates 对象，表示当前位置的坐标。
- `Position.timestamp`：返回一个对象，代表当前时间戳。

PositionError 对象主要有两个属性。

- `PositionError.code`：整数，表示发生错误的原因。`1`表示无权限，有可能是用户拒绝授权；`2`表示无法获得位置，可能设备有故障；`3`表示超时。
- `PositionError.message`：字符串，表示错误的描述。

参数对象`option`可以指定三个属性。

- `enableHighAccuracy`：布尔值，是否返回高精度结果。如果设为`true`，可能导致响应时间变慢或（移动设备的）功耗增加；反之，如果设为`false`，设备可以更快速地响应。默认值为`false`。
- `timeout`：正整数，表示等待查询的最长时间，单位为毫秒。默认值为`Infinity`。
- `maximumAge`：正整数，表示可接受的缓存最长时间，单位为毫秒。如果设为`0`，表示不返回缓存值，必须查询当前的实际位置；如果设为`Infinity`，必须返回缓存值，不管缓存了多少时间。默认值为`0`。

下面是一个例子。

```javascript
var options = {
  enableHighAccuracy: true,
  timeout: 5000,
  maximumAge: 0
};

function success(pos) {
  var crd = pos.coords;

  console.log(`经度：${crd.latitude}`);
  console.log(`纬度：${crd.longitude}`);
  console.log(`误差：${crd.accuracy} 米`);
}

function error(err) {
  console.warn(`ERROR(${err.code}): ${err.message}`);
}

navigator.geolocation.getCurrentPosition(success, error, options);
```

### Geolocation.watchPosition()

`Geolocation.watchPosition()`对象指定一个监听函数，每当用户的位置发生变化，就是自动执行这个函数。

```javascript
navigator.geolocation.watchPosition(success[, error[, options]])
```

该方法接受三个参数。

- `success`：监听成功的回调函数，该函数的参数为一个 Position 对象。
- `error`：该参数可选，表示监听失败的回调函数，该函数的参数是一个 PositionError 对象。
- `options`：该参数可选，表示监听的参数配置对象。

该方法返回一个整数值，表示监听函数的编号。该整数用来供`Geolocation.clearWatch()`方法取消监听。

下面是一个例子。

```javascript
var id;

var target = {
  latitude : 0,
  longitude: 0
};

var options = {
  enableHighAccuracy: false,
  timeout: 5000,
  maximumAge: 0
};

function success(pos) {
  var crd = pos.coords;

  if (target.latitude === crd.latitude && target.longitude === crd.longitude) {
    console.log('恭喜，你已经到达了指定位置。');
    navigator.geolocation.clearWatch(id);
  }
}

function error(err) {
  console.warn('ERROR(' + err.code + '): ' + err.message);
}

id = navigator.geolocation.watchPosition(success, error, options);
```

### Geolocation.clearWatch()

`Geolocation.clearWatch()`方法用来取消`watchPosition()`方法指定的监听函数。它的参数是`watchPosition()`返回的监听函数的编号。

```javascript
navigator.geolocation.clearWatch(id);
```

使用方法的例子见上一节。

## Coordinates 对象

Coordinates 对象是地理位置的坐标接口，`Position.coords`属性返回的就是这个对象。

它有以下属性，全部为只读属性。

- `Coordinates.latitude`：浮点数，表示纬度。
- `Coordinates.longitude`：浮点数，表示经度。
- `Coordinates.altitude`：浮点数，表示海拔（单位：米）。如果不可得，返回`null`。
- `Coordinates.accuracy`：浮点数，表示经度和纬度的精度（单位：米）。
- `Coordinates.altitudeAccuracy`：浮点数，表示海拔的精度（单位：米）。返回`null`。
- `Coordinates.speed`：浮点数，表示设备的速度（单位：米/秒）。如果不可得，返回`null`。
- `Coordinates.heading`：浮点数，表示设备前进的方向（单位：度）。方向按照顺时针，北方是0度，东方是90度，西方是270度。如果`Coordinates.speed`为0，`heading`属性返回`NaN`。如果设备无法提供方向信息，该属性返回`null`。

下面是一个例子。

```javascript
navigator.geolocation.getCurrentPosition( function (position) {
  let lat = position.coords.latitude;
  let long = position.coords.longitude;
  console.log(`纬度：${lat.toFixed(2)}`);
  console.log(`经度：${long.toFixed(2)}`);
});
```

## 参考链接

- [Geolocation API](https://developer.mozilla.org/en-US/docs/Web/API/Geolocation_API), by MDN
