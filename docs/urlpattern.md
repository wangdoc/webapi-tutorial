# URL Pattern API

## 简介

URL Pattern API 基于正则表达式和通配符，对 URL 进行匹配和解析。

它提供一个构造函数`URLPattern()`，用于新建一个 URL 模式实例。

```javascript
const pattern = new URLPattern(input);
```

有了模式实例，就可以知道某个 URL 是否符合该模式。

```javascript
const pattern = new URLPattern({ pathname: "/books" });
console.log(pattern.test("https://example.com/books")); // true
```

上面示例中，模式实例是 包含`/books`路径的 URL，实例方法`test()`用来检测指定网址是否符合该模式，结果为`true`。

URL Pattern 支持多种协议，不仅是 HTTP 协议。

```javascript
const pattern = new URLPattern("data\\:foo*");
```

上面示例中，URL Pattern 新建了一个 Data 协议的模式。

## 构造函数 URLPattern()

### 基本用法 

构造函数`URLPattern()`用于新建一个 URL 模式实例。

```javascript
const pattern = new URLPattern(input);
```

该构造函数的参数`input`是一个模式字符串或者模式对象。

```javascript
new URLPattern("https://example.com/books/:id")
// {
//   hasRegExpGroups: false,
//   hash: "*",
//   hostname: "example.com",
//   password: "*",
//   pathname: "/books/:id",
//   port: "",
//   protocol: "https",
//   search: "*",
//   username: "*",
//   ...
// }
```

上面示例中，参数`https://example.com/books/:id`就是一个模式字符串，执行后返回一个 URLPattern 实例对象，包含模式的各个组成部分。

参数`input`也可以写成一个对象，用属性指定模式 URL 的每个部分。也就是说，模式对象可以有以下属性。

- protocol
- username
- password
- hostname
- port
- pathname
- search
- hash
- baseURL

上面的示例，如果参数改成模式对象，就是下面这样。

```javascript
new URLPattern({
  protocol: 'https',
  hostname: 'example.com',
  pathname: '/books/:id',
})
```

模式字符串或者模式对象之中，没有定义的部分，默认为`*`，表示所有可能的字符，包括零字符的情况。

`URLPattern()`正常情况下将返回一个 URLPattern 实例对象，但是遇到参数无效或语法不正确，则会报错。

```javascript
new URLPattern(123) // 报错
```

上面示例中，参数`123`不是一个有效的 URL 模式，就报错了。

需要注意的是，如果模式字符串为相对路径，那么`URLPattern()`还需要第二个参数，用来指定基准 URL。

```javascript
new URLPattern(input, baseURL)
```

上面代码中，第二个参数`baseURL`就是基准 URL。

```javascript
new URLPattern('/books/:id') // 报错
new URLPattern('/books/:id', 'https://example.com') // 正确
```

上面示例中，第一个参数`/books/:id`是一个相对路径，这时就需要第二个参数`https://example.com`，用来指定基准 URL，否则报错。

但是，如果参数为模式对象，则可以只指定 URL 模式的某个部分。

```javascript
new URLPattern({
  pathname: '/books/:id'
}) // 正确
```

上面示例中，参数是一个模式对象，那么参数允许只指定 URL 的部分模式。

模式对象里面，也可以指定基准 URL。

```javascript
let pattern4 = new URLPattern({
  pathname: "/books/:id",
  baseURL: "https://example.com",
});
```

基准 URL 必须是合法的 URL，不能包含模式。

注意，如果用了模式对象，就不能使用基准 URL 作为第二个参数，这样会报错。

```javascript
new URLPattern({ pathname: "/foo/bar" }, "https://example.com") // 报错
new URLPattern({ pathname: "/foo/bar" }, "https://example.com/baz") // 报错
```

上面示例中，同时使用了模式对象和第二个参数，结果就报错了。

`URLpattern()`还可以加入配置对象参数，用于定制匹配行为。

```javascript
new URLPattern(input, options)
new URLPattern(input, baseURL, options)
```

上面代码中，参数`options`就是一个配置对象。

目前，这个配置对象`options`只有`ignoreCase`一个属性，如果设为`true`，将不区分大小写，默认值为`false`，表示区分大小写。

```javascript
new URLPattern(input, {
  ignoreCase: false // 默认值，区分大小写
})
```

请看下面的例子。

```javascript
const pattern = new URLPattern("https://example.com/2022/feb/*");

pattern.test("https://example.com/2022/feb/xc44rsz") // true
pattern.test("https://example.com/2022/Feb/xc44rsz") // false
```

上面示例，默认匹配时，会区分`feb`和`Feb`。

我们可以用`ignoreCase`将其关闭。

```javascript
const pattern = new URLPattern(
  "https://example.com/2022/feb/*",
  {  ignoreCase: true, }
);

pattern.test("https://example.com/2022/feb/xc44rsz") // true
pattern.test("https://example.com/2022/Feb/xc44rsz") // true
```

### 模式写法

模式字符串基本上采用正则表达式的写法，但是不是所有的正则语法都支持，比如先行断言和后行断言就不支持。

（1）普通字符

如果都是普通字符，就表示原样匹配。

```javascript
const p = new URLPattern('https://example.com/abc');
```

上面代码就表示确切匹配路径`https://example.com/abc`。

```javascript
p.test('https://example.com') // false
p.test('https://example.com/a') //false
p.test('https://example.com/abc') // true
p.test('https://example.com/abcd') //false
p.test('https://example.com/abc/') //false
p.test('https://example.com/abc?123') //true
```

上面示例中，URL 必须严格匹配路径`https://example.com/abc`，即使尾部多一个斜杠都不行，但是加上查询字符串是可以的。

（2）`?`

量词字符`?`表示前面的字符串，可以出现0次或1次，即该部分可选。

```javascript
let pattern = new URLPattern({
  protocol: "http{s}?",
});
```

上面示例中，`{s}?`表示字符组`s`可以出现0次或1次。

`?`不包括路径的分隔符`/`。

```javascript
const pattern = new URLPattern("/books/:id?", "https://example.com");

pattern.test("https://example.com/books/123") // true
pattern.test("https://example.com/books") // true
pattern.test("https://example.com/books/") // false
pattern.test("https://example.com/books/123/456") // false
pattern.test("https://example.com/books/123/456/789") // false
pattern.test("https://example.com/books/123/456/") // false
```

上面示例中，`?`不能匹配网址结尾的斜杠。

如果一定要匹配，可以把结尾的斜杠放在`{}`里面。

```javascript
const pattern = new URLPattern({ pathname: "/product{/}?" });

pattern.test({ pathname: "/product" }) // true
pattern.test({ pathname: "/product/" }) // true
```

上面示例中，不管网址有没有结尾的斜杠，`{/}?`都会成功匹配。

（3）`+`

量词字符`+`表示前面的字符串出现1次或多次。

```javascript
const pattern = new URLPattern({
  pathname: "/books/(\\d+)",
})
```

上面示例中，`\\d+`表示1个或多个数字，其中的`\d`是一个内置的字符类，表示0-9的数字，因为放在双引号里面，所以反斜杠前面还要再加一个反斜杠进行转义。

`+`可以包括`/`分隔的路径的多个部分，但不包括路径结尾的斜杠。

```javascript
const pattern = new URLPattern("/books/:id+", "https://example.com");

pattern.test("https://example.com/books/123") // true
pattern.test("https://example.com/books") // false
pattern.test("https://example.com/books/") // false
pattern.test("https://example.com/books/123/456") // true
pattern.test("https://example.com/books/123/456/789") // true
pattern.test("https://example.com/books/123/456/") // false
```

（4）`*`

量词字符`*`表示出现零次或多次。

```javascript
const pattern = new URLPattern('https://example.com/{abc}*');

pattern.test('https://example.com') // true
pattern.test('https://example.com/') // true
pattern.test('https://example.com/abc') // true
pattern.test('https://example.com/abc/') // false
pattern.test('https://example.com/ab') // false
pattern.test('https://example.com/abcabc') // true
pattern.test('https://example.com/abc/abc/abc') // false
```

上面示例中，`{abc}*`表示`abc`出现零次或多次，也不包括路径分隔符`/`。

如果`*`前面没有任何字符，就表示所有字符，包括零字符的情况，也包括分隔符`/`。

```javascript
let pattern = new URLPattern({
  search: "*",
  hash: "*",
});
```

上面示例中，`*`表示匹配所有字符，包括零字符。

下面是另一个例子。

```javascript
const pattern = new URLPattern("/*.png", "https://example.com");

pattern.test("https://example.com/image.png") // true
pattern.test("https://example.com/image.png/123") // false
pattern.test("https://example.com/folder/image.png") // true
pattern.test("https://example.com/.png") // true
```

`*`匹配的部分可以从对应部分的数字属性上获取。

```javascript
const pattern = new URLPattern({
  hostname: "example.com",
  pathname: "/foo/*"
});

const result = pattern.exec("/foo/bar", "https://example.com/baz");

result.pathname.input // '/foo/bar'
result.pathname.groups[0] // 'bar'
```

上面示例中，`*`的匹配结果可以从`pathname.groups[0]`获取。

```javascript
const pattern = new URLPattern({ hostname: "*.example.com" });
const result = pattern.exec({ hostname: "cdn.example.com" });

result.hostname.groups[0] // 'cdn'
result.hostname.input // 'cdn.example.com'
```

上面示例中，`*`的匹配结果可以从`hostname.groups[0]`获取。

（5）`{}`

特殊字符`{}`用来定义量词`?`、`+`、`+`的生效范围。

如果`{}`后面没有量词，那就跟没有使用的效果一样。

```javascript
const pattern = new URLPattern('https://example.com/{abc}');

pattern.test('https://example.com/') // false
pattern.test('https://example.com/abc') // true
```

（6）`()`

特殊字符`()`用来定义一个组匹配，匹配结果可以按照出现顺序的编号，从`pathname.groups`对象上获取。

```javascript
const pattern = new URLPattern("/books/(\\d+)", "https://example.com");
pattern.exec("https://example.com/books/123").pathname.groups
// { '0': '123' }
```

上面示例中，`(\\d+)`是一个组匹配，因为它是第一个组匹配，所以匹配结果放在`pathname.groups`的属性`0`。

（7）`|`

特殊字符`|`表示左右两侧的字符，都可以出现，即表示逻辑`OR`。

```javascript
let pattern = new URLPattern({
  port: "(80|443)",
});
```

上面示例中，`(80|443)`表示80或者443都可以。

（8）`:`

特殊字符`:`用来定义一个具名组匹配，后面跟着变量名。

```javascript
let pattern = new URLPattern({
  pathname: "/:path",
});
```

上面示例中，`/:path`表示斜杠后面的部分，都被捕捉放入变量`path`，可以从匹配结果的`pathname.groups`上的对应属性获取。

```javascript
const pattern = new URLPattern({ pathname: "/books/:id" });

pattern.exec("https://example.com/books/123").pathname.groups
// { id: '123' }
```

上面示例中，`pathname.groups`返回一个对象，该对象的属性就是所有捕捉成功的组变量，上例是`id`。


下面是另一个例子。

```javascript
const pattern = new URLPattern({ pathname: "/:product/:user/:action" });
const result = pattern.exec({ pathname: "/store/wanderview/view" });

result.pathname.groups.product // 'store'
result.pathname.groups.user // 'wanderview'
result.pathname.groups.action // 'view'
result.pathname.input // '/store/wanderview/view'
```

上面示例中，`:product`、`:user`、`:action`的匹配结果，都可以从`pathname.groups`的对应属性上获取。

组匹配可以放在模式的前面。

```javascript
const pattern = new URLPattern(
  "/books/:id(\\d+)",
  "https://example.com"
);
```

上面示例中，组匹配`:id`后面跟着模型定义`\\d+`，模式需要放在括号里面。

**（9）特殊字符转义**

如果要将特殊字符当作普通字符使用，必须在其前面加入双重反斜杠进行转义。

```javascript
let pattern1 = new URLPattern({
  pathname: "/a:b",
});

let pattern2 = new URLPattern({
  pathname: "/a\\:b",
});
```

上面示例中，`a:b`表示路径以字符`a`开头，后面的部分都放入变量`b`。而`a\\:b`表示路径本身就是`a:b`就是。

## 实例属性

URLPattern 实例的属性对应`URLPattern()`模式对象参数的各个部分。

```javascript
const pattern = new URLPattern({
  hostname: "{*.}?example.com",
});

pattern.hostname // '{*.}?example.com'
pattern.protocol // '*'
pattern.username // '*'
pattern.password // '*'
pattern.port // ""
pattern.pathname // '*'
pattern.search // '*'
pattern.hash // '*'
```

上面示例中，`pattern`是一个实例对象，它的属性与`URLPattern()`的参数对象的属性一致。

注意，`search`不包括开头的`?`，`hash`不包括开头的`#`，但是`pathname`包括开头的`/`。

下面是另一个例子。

```javascript
const pattern = new URLPattern("https://cdn-*.example.com/*.jpg");

pattern.protocol // 'https'
pattern.hostname // 'cdn-*.example.com'
pattern.pathname // '/*.jpg'
pattern.username // ''
pattern.password // ''
pattern.search // ''
pattern.hash // ''
```

## 实例方法

### exec()

实例的`exec()`方法，把模式用于解析参数网址，返回匹配结果。

`exec()`方法的参数与`new URLPattern()`是一致的。它可以是一个 URL 字符串。

```javascript
pattern.exec("https://store.example.com/books/123");
```

如果第一个参数是相对 URL，那么需要基准 URL，作为第二个参数。

```javascript
pattern.exec("/foo/bar", "https://example.com/baz");
```

`exec()`方法的参数，也可以是一个对象。

```javascript
pattern.exec({
  protocol: "https",
  hostname: "store.example.com",
  pathname: "/books/123",
});
```

如果匹配成功，它返回一个包括匹配结果的对象。如果匹配失败，返回`null`。

```javascript
const pattern = new URLPattern("http{s}?://*.example.com/books/:id");
pattern.exec("https://example.com/books/123") // null
```

上面示例中，匹配失败返回`null`。

匹配成功返回的对象，有一个`inputs`属性，包含传入`pattern.exec()`的参数数组。其他属性的值也是一个对象，该对象的`input`属性对应传入值，`groups`属性包含各个组匹配。

```javascript
const pattern = new URLPattern("http{s}?://*.example.com/books/:id");
let match = pattern.exec("https://store.example.com/books/123");

match.inputs // ['https://store.example.com/books/123']
match.protocol // { input: "https", groups: {} }
match.username // { input: "", groups: {} }
match.password // { input: "", groups: {} }
match.hostname // { input: "store.example.com", groups: { "0": "store" } }
match.port // { input: "", groups: {} }
match.pathname // { input: "/books/123", groups: { "id": "123" } }
match.search // { input: "", groups: {} }
match.hash // { input: "", groups: {} }
```

### test()

实例的`test()`方法，用来检测参数网址是否符合当前模式。

它的参数跟`URLPattern()`是一样的，可以是模式字符串，也可以是模式对象。

```javascript
const pattern = new URLPattern({
  hostname: "example.com",
  pathname: "/foo/*"
 });

pattern.test({
  pathname: "/foo/bar",
  baseURL: "https://example.com/baz",
}) // true

pattern.test("/foo/bar", "https://example.com/baz") // true
```

正常情况下，它返回一个布尔值。但是，如果语法不合法，它也会抛错。

```javascript
pattern.test({ pathname: "/foo/bar" }, "https://example.com/baz") // 报错
```

