# URLSearchParams 对象

## 简介

URLSearchParams 对象表示 URL 的查询字符串（比如`?foo=bar`）。它提供一系列方法，用来操作这些键值对。URL 实例对象的`searchParams`属性，就是指向一个 URLSearchParams 实例对象。

URLSearchParams 实例对象可以用`for...of`进行遍历。

```javascript
for (const [key, value] of mySearchParams) {
}
```

## 构造方法

URLSearchParams 可以作为构造函数使用，生成一个实例对象。

```javascript
const params = new URLSearchParams();
```

它可以接受一个查询字符串作为参数，将其转成对应的实例对象。

```javascript
const params = new URLSearchParams('?a=1&b=2');
```

注意，它最多只能去除查询字符串的开头问号`?`，并不能解析完整的网址字符串。

```javascript
const paramsString = "http://example.com/search?query=%40";
const params = new URLSearchParams(paramsString);
```

上面示例中，URLSearchParams 会认为键名是`http://example.com/search?query`，而不是`query`。

它也可以接受表示键值对的对象或数组作为参数。

```javascript
// 参数为数组
const params3 = new URLSearchParams([
  ["foo", "1"],
  ["bar", "2"],
]);

// 参数为对象
const params1 = new URLSearchParams({ foo: "1", bar: "2" });
```

浏览器向服务器发送表单数据时，可以直接使用 URLSearchParams 实例作为表单数据。

```javascript
const params = new URLSearchParams({foo: 1, bar: 2});
fetch('https://example.com/api', {
  method: 'POST',
  body: params
}).then(...)
```

上面示例中，fetch 向服务器发送命令时，可以直接使用 URLSearchParams 实例对象作为数据体。

它还可以接受另一个 URLSearchParams 实例对象作为参数，等于复制了该对象。

```javascript
const params1 = new URLSearchParams('?a=1&b=2');
const params2 = new URLSearchParams(params);
```

上面示例中，`params1`和`params2`是两个一模一样的实例对象，但是修改其中一个，不会影响到另一个。

URLSearchParams会对查询字符串自动编码。

```javascript
const params = new URLSearchParams({'foo': '你好'});
params.toString() // "foo=%E4%BD%A0%E5%A5%BD"
```

上面示例中，`foo`的值是汉字，URLSearchParams 对其自动进行 URL 编码。

键名可以没有键值，这时 URLSearchParams 会认为键值等于空字符串。

```javascript
const params1 = new URLSearchParams("foo&bar=baz");
const params2 = new URLSearchParams("foo=&bar=baz");
```

上面示例中，`foo`是一个空键名，不管它后面有没有等号，URLSearchParams 都会认为它的值是一个空字符串。

## 实例方法

### append()

`append()`用来添加一个查询键值对。如果同名的键值对已经存在，它依然会将新的键值对添加到查询字符串的末尾。

它的第一个参数是键名，第二个参数是键值，下面是用法示例。

```javascript
const params = new URLSearchParams('?a=1&b=2');

params.append('a', 3);
params.toString() // 'a=1&b=2&a=3'
```

上面示例中，键名`a`已经存在，但是`append()`依然会将`a=3`添加在查询字符串的末尾。

### delete()

`delete()`删除给定名字的键值对。

### get()

`get()`返回指定键名所对应的键值。如果存在多个同名键值对，它只返回第一个键值。

```javascript
const params = new URLSearchParams('?a=1&b=2');
params.get('a') // 1
```

对于不存在的键名，它会返回`null`。

注意，`get()`会将键值里面的加号转为空格。

```javascript
const params = new URLSearchParams(`c=a+b`);
params.get('c') // 'a b'
```

上面示例中，`get()`将`a+b`转为`a b`。如果希望避免这种行为，可以先用`encodeURIComponent()`对键值进行转义。

### getAll()

`getAll()`返回一个数组，里面是指定键名所对应的所有键值。

```javascript
const params = new URLSearchParams('?a=1&b=2&a=3');
params.getAll('a') // [ '1', '3' ]
```

### has()

`has()`返回一个布尔值，表示指定键名是否存在。

```javascript
const params = new URLSearchParams('?a=1&b=2');
params.has('a') // true
params.has('c') // false
```

### set()

`set()`用来设置一个键值对。如果相同键名已经存在，则会替换当前值，这是它与`append()`的不同之处。该方法适合用来修改查询字符串。

```javascript
const params = new URLSearchParams('?a=1&b=2');
params.set('a', 3);
params.toString() // 'a=3&b=2'
```

上面示例中，`set()`修改了键`a`。

如果有多个的同名键，`set()`会移除现存所有的键，再添加新的键值对。

```javascript
const params = new URLSearchParams('?foo=1&foo=2');
params.set('foo', 3);
params.toString() // "foo=3"
```

上面示例中，有两个`foo`键，`set()`会将它们都删掉，再添加一个新的`foo`键。

### sort()

`sort()`按照键名（以 Unicode 码点为序）对键值对排序。如果有同名键值对，它们的顺序不变。

```javascript
const params = new URLSearchParams('?a=1&b=2&a=3');
params.sort();
params.toString() // 'a=1&a=3&b=2'
```

### entries()

`entries()`方法返回一个 iterator 对象，用来遍历键名和键值。

```javascript
const params = new URLSearchParams("key1=value1&key2=value2");

for (const [key, value] of params.entries()) {
  console.log(`${key}, ${value}`);
}
// key1, value1
// key2, value2
```

如果直接对 URLSearchParams 实例进行`for...of`遍历，其实内部调用的就是`entries`接口。

```javascript
for (var p of params) {}
// 等同于
for (var p of params.entries()) {}
```

### forEach()

`forEach()`用来依次对每个键值对执行一个回调函数。

它接受两个参数，第一个参数`callback`是回调函数，第二个参数`thisArg`是可选的，用来设置`callback`里面的`this`对象。

```javascript
forEach(callback)
forEach(callback, thisArg)
```

`callback`函数可以接收到以下三个参数。

- value：当前键值。
- key：当前键名。
- searchParams：当前的 URLSearchParams 实例对象。

下面是用法示例。

```javascript
const params = new URLSearchParams("key1=value1&key2=value2");

params.forEach((value, key) => {
  console.log(value, key);
});
// value1 key1
// value2 key2
```

### keys()

`keys()`返回一个 iterator 对象，用来遍历所有键名。

```javascript
const params = new URLSearchParams("key1=value1&key2=value2");

for (const key of params.keys()) {
  console.log(key);
}
// key1
// key2
```

### values()

`values()`返回一个 iterator 对象，用来遍历所有键值。

```javascript
const params = new URLSearchParams("key1=value1&key2=value2");

for (const value of params.values()) {
  console.log(value);
}
// value1
// value2
```

这个方法也可以用来将所有键值，转成一个数组。

```javascritp
Array.from(params.values()) // ['value1', 'value2']
```

### toString()

`toString()`用来将 URLSearchParams 实例对象转成一个字符串。它返回的字符串不带问号，这一点与`window.location.search`不同。

## 实例属性

### size

`size`是一个只读属性，返回键值对的总数。

```javascript
const params = new URLSearchParams("c=4&a=2&b=3&a=1");
params.size; // 4
```

上面示例中，键名`a`在查询字符串里面有两个，`size`不会将它们合并。

如果想统计不重复的键名，可以将使用 Set 结构。

```javascript
[...new Set(params.keys())].length // 3
```

`size`属性可以用来判别，某个网址是否有查询字符串。

```javascript
const url = new URL("https://example.com?foo=1&bar=2");

if (url.searchParams.size) {
  console.log("该 URL 有查询字符串");
}
```

