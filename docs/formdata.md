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

下面是用法示例，通过脚本发送表单数据。

```html
<form id="formElem">
  <input type="text" name="firstName" value="John">
  Picture: <input type="file" name="picture" accept="image/*">
  <input type="submit">
</form>

<script>
  formElem.onsubmit = async (e) => {
    e.preventDefault();

    let response = await fetch('/article/formdata/post/user-avatar', {
      method: 'POST',
      body: new FormData(formElem)
    });

    let result = await response.json();

    alert(result.message);
  };
</script>
```

浏览器向服务器发送表单数据时，不管是用户点击 Submit 按钮发送，还是使用脚本发送，都会自动将其编码，并以`Content-Type: multipart/form-data`的形式发送。

`FormData()`还有第三种用法，如果想把“提交”（Submit）按钮也加入表单的键值对，可以把按钮的 DOM 节点当作`FormData()`的第二个参数。

```javascript
new FormData(form, submitter)
```

上面代码中，`submitter`就是提交按钮的 DOM 节点。这种用法适合表单有多个提交按钮，服务端需要通过按钮的值来判断，用户到底选用了哪个按钮。

```javascript
// 表单有两个提交按钮
// <button name="intent" value="save">Save</button>
// <button name="intent" value="saveAsCopy">Save As Copy</button>

const form = document.getElementById("form");
const submitter = document.querySelector("button[value=save]");

const formData = new FormData(form, submitter);
```

上面示例中，`FormData()`加入了第二个参数，实例对象`formData`就会增加一个键值对，键名为`intent`，键值为`save`。

## 实例方法

### append()

`append()`用于添加一个键值对，即添加一个表单元素。它有两种使用形式。

```javascript
FormData.append(name, value)
FormData.append(name, blob, fileName)
```

它的第一个参数是键名，第二个参数是键值。上面的第二种形式`FormData.append(name, blob, fileName)`，相当于添加一个文件选择器`<input type="file">`，第二个参数`blob`是文件的二进制内容，第三个参数`fileName`是文件名。

如果键名已经存在，它会为其添加新的键值，即同一个键名有多个键值。

下面是一个用法示例。

```javascript
let formData = new FormData();
formData.append('key1', 'value1');
formData.append('key2', 'value2');

for(let [name, value] of formData) {
  console.log(`${name} = ${value}`);
}
// key1 = value1
// key2 = value2
```

下面是添加二进制文件的例子。

```javascript
// HTML 代码如下
// <canvas id="canvasElem" width="100" height="80" style="border:1px solid"></canvas>

let imageBlob = await new Promise(
  resolve => canvasElem.toBlob(resolve, 'image/png')
);

let formData = new FormData();
formData.append('image', imageBlob, 'image.png');

let response = await fetch('/article/formdata/post/image-form', {
  method: 'POST',
  body: formData
});

let result = await response.json();
console.log(result);
```

下面是添加 XML 文件的例子。

```javascript
const content = '<q id="a"><span id="b">hey!</span></q>';
const blob = new Blob([content], { type: "text/xml" });

formData.append('userfile', blob);
```

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

下面是使用`entries()`遍历键值对的例子。

```javascript
formData.append("key1", "value1");
formData.append("key2", "value2");

for (const pair of formData.entries()) {
  console.log(`${pair[0]}, ${pair[1]}`);
}
// key1, value1
// key2, value2
```

### get()

`get()`用于获取指定键名的键值，它的参数为键名。

```javascript
FormData.get(name)
```

如果该键名有多个键值，只返回第一个键值。如果找不到指定键名，则返回`null`。

### getAll()

`getAll()`用于获取指定键名的所有键值，它的参数为键名，返回值为一个数组。如果找不到指定键名，则返回一个空数组。

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

下面是用法示例。

```javascript
const formData = new FormData();
formData.append("key1", "value1");
formData.append("key2", "value2");

for (const key of formData.keys()) {
  console.log(key);
}
// key1
// key2
```

### set()

`set()`用于为指定键名设置新的键值。它有两种使用形式。

```javascript
FormData.set(name, value);
FormData.set(name, blob, fileName);
```

它的第一个参数为键名，第二个参数为键值。上面第二种形式为上传文件，第二个参数`blob`为文件的二进制内容，第三个参数`fileName`为文件名。该方法没有返回值。

如果指定键名不存在，它会添加该键名，否则它会丢弃所有现有的键值，确保一个键名只有一个键值。这是它跟`append()`的主要区别。

### values()

`values()`返回一个键值的迭代器，用于遍历所有键值。

```javascript
FormData.values()
```

下面是用法示例。

```javascript
const formData = new FormData();
formData.append("key1", "value1");
formData.append("key2", "value2");

for (const value of formData.values()) {
  console.log(value);
}
// value1
// value2
```

