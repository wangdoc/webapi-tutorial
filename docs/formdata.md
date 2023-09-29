# FormData 对象

## 简介

FormData 代表表单数据，是浏览器的原生对象。

它可以当作构造函数使用，构造一个表单实例。

```javascript
const formData = new FormData();
```

上面示例中，`FormData()`当作构造函数使用，返回一个空的表单实例对象。

它也可以接受一个表单的 DOM 节点当作参数，将表单的所有元素及其值，转换成一个个键值对，包含在返回的实例对象里面。

```javascript
const formData = new FormData(form);
```

上面示例中，`FormData()`的参数`form`就是一个表单的 DOM 节点对象。

下面是用法示例。

```javascript
const form = document.querySelector('#subscription');

try {
  let response = await fetch('subscribe.php', {
    method: 'POST',
    body: new FormData(form),
  });

  const result = await response.json();

  console.log(result);
} catch (error) {
  console.log(error);
}
```

浏览器向服务器发送 FormData 对象时，不管是用户点击 Submit 按钮发送，还是使用脚本发送，都会自动将其编码，并以`Content-Type: multipart/form-data`的形式发送。

## 实例方法

### append()

`append()`用于添加一个键值对，即添加一个表单元素。它有两种使用形式。

```javascript
FormData.append(name, value)
FormData.append(name, value, file)
```

它的第一个参数是键名，第二个参数是键值。

如果键名已经存在，它会为其添加新的键值。

### delete()

`delete()`用于删除指定的键值对，它的参数为键名。

```javascript
FormData.delete(name);
```

### entries()

`entries()`返回一个迭代器，用于遍历所有键值对。

```javascript
FormData.entries()
```

下面是一个用法示例。

```javascript
const form = document.querySelector('#subscription');
const formData = new FormData(form);
const values = [...formData.entries()];
console.log(values);
```

### get()

`get()`用于获取指定键名的键值，它的参数为键名。

```javascript
FormData.get(name)
```

如果该键名有多个键值，只返回第一个键值。

### getAll()

`getAll()`用于获取指定键名的所有键值，它的参数为键名，返回值为一个数组。

```javascript
FormData.getAll(name)
```

### has()

`has()`返回一个布尔值，表示是否存在指定键名，它的参数为键名。

```javascript
FormData.has(name)
```

### keys()

`key()`返回一个键名的迭代器，用于遍历所有键名。

```javascript
FormData.keys()
```

### set()

`set()`用于为指定键名设置新的键值。它有两种使用形式。

```javascript
FormData.set(name, value);
FormData.set(name, value, filename);
```

它的第一个参数为键名，第二个参数为键值。如果指定键名不存在，它会添加该键名。

### value()

`value()`返回一个键值的迭代器，用于遍历所有键值。

```javascript
FormData.values()
```

