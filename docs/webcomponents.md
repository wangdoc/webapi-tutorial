# Web Components

## 概述

各种网站往往需要一些相同的模块，比如日历、调色板等等，这种模块就被称为“组件”（component）。Web Components 就是浏览器原生的组件规范。

采用组件开发，有很多优点。

（1）有利于代码复用。组件是模块化编程思想的体现，可以跨平台、跨框架使用，构建、部署和与其他 UI 元素互动都有统一做法。

（2）使用非常容易。加载或卸载组件，只要添加或删除一行代码就可以了。

（3）开发和定制很方便。组件开发不需要使用框架，只要用原生的语法就可以了。开发好的组件往往留出接口，供使用者设置常见属性，比如上面代码的`heading`属性，就是用来设置对话框的标题。

（4）组件提供了 HTML、CSS、JavaScript 封装的方法，实现了与同一页面上其他代码的隔离。

未来的网站开发，可以像搭积木一样，把组件合在一起，就组成了一个网站。这种前景是非常诱人的。

Web Components 不是单一的规范，而是一系列的技术组成，以下是它的四个构成。

- Custom Elements
- Template
- Shadow DOM
- HTML Import

使用时，并不一定上面四种 API 都要用到。其中，Custom Element 和 Shadow DOM 比较重要，Template 和 HTML Import 只起到辅助作用。

## Custom Element

### 简介

HTML 标准定义的网页元素，有时并不符合我们的需要，这时浏览器允许用户自定义网页元素，这就叫做 Custom Element。简单说，它就是用户自定义的网页元素，是 Web components 技术的核心。

举例来说，你可以自定义一个叫做`<my-element>`的网页元素。

```html
<my-element></my-element>
```

注意，自定义网页元素的标签名必须含有连字符`-`，一个或多个连字符都可以。这是因为浏览器内置的的 HTML 元素标签名，都不含有连字符，这样可以做到有效区分。

下面的代码先定义一个自定义元素的类。

```javascript
class MyElement extends HTMLElement {
  constructor() {
    super();
    this.attachShadow( { mode: 'open' } );
    this.shadowRoot.innerHTML = `
      <style>
        /* scoped styles */
      </style>
      <slot></slot>
    `;
  }

  static get observedAttributes() {
    // Return list of attributes to watch.
  }

  attributeChangedCallback( name, oldValue, newValue ) {
    // Run functionality when one of these attributes is changed.
  }

  connectedCallback() {
    // Run functionality when an instance of this element is inserted into the DOM.
  }

  disconnectedCallback() {
    // Run functionality when an instance of this element is removed from the DOM.
  }
}
```

上面代码有几个注意点。

- 自定义元素类的基类是`HTMLElement`。当然也可以根据需要，基于`HTMLElement`的子类，比如`HTMLButtonElement`。
- 构造函数内部定义了 Shadow DOM。所谓`Shadow DOM`指的是，这部分的 HTML 代码和样式，不直接暴露给用户。
- 类可以定义生命周期方法，比如`connectedCallback()`。

然后，`window.customElements.define()`方法，用来登记自定义元素与这个类之间的映射。

```javascript
window.customElements.define('my-element', MyElement);
```

登记以后，页面上的每一个`<my-element>`元素都是一个`MyElement`类的实例。只要浏览器解析到`<my-element>`元素，就会运行`MyElement`的构造函数。

注意，如果没有登记就使用 Custom Element，浏览器会认为这是一个不认识的元素，会当做空的 div 元素处理。

`window.customElements.define()`方法定义了 Custom Element 以后，可以使用`window.customeElements.get()`方法获取该元素的构造方法。这使得除了直接插入 HTML 网页，Custom Element 也能使用脚本插入网页。

```javascript
window.customElements.define(
  'my-element',
  class extends HTMLElement {...}
);
const el = window.customElements.get('my-element');
const myElement = new el();
document.body.appendChild(myElement);
```

如果你想扩展现有的 HTML 元素（比如`<button>`）也是可以的。

```javascript
class GreetingElement extends HTMLButtonElement
```

登记的时候，需要提供扩展的元素。

```javascript
customElements.define('hey-there', GreetingElement, { extends: 'button' });
```

使用的时候，为元素加上`is`属性就可以了。

```html
<button is="hey-there" name="World">Howdy</button>
```

### 生命周期方法

Custom Element 提供一些生命周期方法。

```javascript
class MyElement extends HTMLElement {
  constructor() {
    super();
  }

  connectedCallback() {
    // here the element has been inserted into the DOM
  }
}
```

上面代码中，`connectedCallback()`方法就是`MyElement`元素的生命周期方法。每次，该元素插入 DOM，就会自动执行该方法。

- `connectedCallback()`：插入 DOM 时调用。这可能不止一次发生，比如元素被移除后又重新添加。类的设置应该尽量放到这个方法里面执行，因为这时各种属性和子元素都可用。
- `disconnectedCallback()`：移出 DOM 时执行。
- `attributeChangedCallback(attrName, oldVal, newVal)`：添加、删除、更新或替换属性时调用。元素创建或升级时，也会调用。注意：只有加入`observedAttributes`的属性才会执行这个方法。
- `adoptedCallback()`：自定义元素移动到新的 document 时调用，比如执行`document.adoptNode(element)`时。

下面是一个例子。

```javascript
class GreetingElement extends HTMLElement {
  constructor() {
    super();
    this._name = 'Stranger';
  }
  connectedCallback() {
    this.addEventListener('click', e => alert(`Hello, ${this._name}!`));
  }
  attributeChangedCallback(attrName, oldValue, newValue) {
    if (attrName === 'name') {
      if (newValue) {
        this._name = newValue;
      } else {
        this._name = 'Stranger';
      }
    }
  }
}
GreetingElement.observedAttributes = ['name'];
customElements.define('hey-there', GreetingElement);
```

上面代码中，`GreetingElement.observedAttributes`属性用来指定白名单里面的属性，上例是`name`属性。只要这个属性的值发生变化，就会自动调用`attributeChangedCallback`方法。

使用上面这个类的方法如下。

```html
<hey-there>Greeting</hey-there>
<hey-there name="Potch">Personalized Greeting</hey-there>
```

`attributeChangedCallback`方法主要用于外部传入的属性，就像上面例子中`name="Potch"`。

生命周期方法调用的顺序如下：`constructor` -> `attributeChangedCallback` -> `connectedCallback`，即`attributeChangedCallback`早于`connectedCallback`执行。这是因为`attributeChangedCallback`相当于调整配置，应该在插入 DOM 之前完成。

下面的例子能够更明显地看出这一点，在插入 DOM 前修改 Custome Element 的颜色。

```javascript
class MyElement extends HTMLElement {
  constructor() {
    this.container = this.shadowRoot.querySelector('#container');
  }
  attributeChangedCallback(attr, oldVal, newVal) {
    if(attr === 'disabled') {
      if(this.hasAttribute('disabled') {
        this.container.style.background = '#808080';
      } else {
        this.container.style.background = '#ffffff';
      }
    }
  }
}
```

### 自定义属性和方法

Custom Element 允许自定义属性或方法。

```javascript
class MyElement extends HTMLElement {
  ...

  doSomething() {
    // do something in this method
  }
}
```

上面代码中，`doSomething()`就是`MyElement`的自定义方法，使用方法如下。

```javascript
const element = document.querySelector('my-element');
element.doSomething();
```

自定义属性可以使用 JavaScript class 的所有语法，因此也可以设置取值器和赋值器。

```javascript
class MyElement extends HTMLElement {
  ...

  set disabled(isDisabled) {
    if(isDisabled) {
      this.setAttribute('disabled', '');
    }
    else {
      this.removeAttribute('disabled');
    }
  }

  get disabled() {
    return this.hasAttribute('disabled');
  }
}
```

上面代码中的取值器和赋值器，可用于`<my-input name="name" disabled>`这样的用法。

### window.customElements.whenDefined()

`window.customElements.whenDefined()`方法在一个 Custom Element 被`customElements.define()`方法定义以后执行，用于“升级”一个元素。

```javascript
window.customElements.whenDefined('my-element')
.then(() => {
  // my-element is now defined
})
```

如果某个属性值发生变化时，需要做出反应，可以将它放入`observedAttributes`数组。

```javascript
class MyElement extends HTMLElement {
  static get observedAttributes() {
    return ['disabled'];
  }

  constructor() {
    const shadowRoot = this.attachShadow({mode: 'open'});
    shadowRoot.innerHTML = `
      <style>
        .disabled {
          opacity: 0.4;
        }
      </style>

      <div id="container"></div>
    `;

    this.container = this.shadowRoot('#container');
  }

  attributeChangedCallback(attr, oldVal, newVal) {
    if(attr === 'disabled') {
      if(this.disabled) {
        this.container.classList.add('disabled');
      }
      else {
        this.container.classList.remove('disabled')
      }
    }
  }
}
```

### 回调函数

自定义元素的原型有一些属性，用来指定回调函数，在特定事件发生时触发。

- **createdCallback**：实例生成时触发
- **attachedCallback**：实例插入HTML文档时触发
- **detachedCallback**：实例从HTML文档移除时触发
- **attributeChangedCallback(attrName, oldVal, newVal)**：实例的属性发生改变时（添加、移除、更新）触发

下面是一个例子。

```javascript
var proto = Object.create(HTMLElement.prototype);

proto.createdCallback = function() {
  console.log('created');
  this.innerHTML = 'This is a my-demo element!';
};

proto.attachedCallback = function() {
  console.log('attached');
};

var XFoo = document.registerElement('x-foo', {prototype: proto});
```

利用回调函数，可以方便地在自定义元素中插入HTML语句。

```javascript

var XFooProto = Object.create(HTMLElement.prototype);

XFooProto.createdCallback = function() {
  this.innerHTML = "<b>I'm an x-foo-with-markup!</b>";
};

var XFoo = document.registerElement('x-foo-with-markup',
  {prototype: XFooProto});

```

上面代码定义了createdCallback回调函数，生成实例时，该函数运行，插入如下的HTML语句。

```html
<x-foo-with-markup>
   <b>I'm an x-foo-with-markup!</b>
</x-foo-with-markup>
```

### Custom Element 的子元素

用户使用 Custom Element 时候，可以在内部放置子元素。Custom Element 提供`<slot>`用来引用内部内容。

下面的`<image-gallery>`是一个 Custom Element。用户在里面放置了子元素。

```html
<image-gallery>
  <img src="foo.jpg" slot="image">
  <img src="bar.jpg" slot="image">
</image-gallery>
```

`<image-gallery>`内部的模板如下。

```html
<div id="container">
  <div class="images">
    <slot name="image"></slot>
  </div>
</div>
```

最终合成的代码如下。

```html
<div id="container">
  <div class="images">
    <slot name="image">
      <img src="foo.jpg" slot="image">
      <img src="bar.jpg" slot="image">
    </slot>
  </div>
</div>
```

## `<template>`标签

### 基本用法

`<template>`标签表示组件的 HTML 代码模板。

```html
<template>
  <h1>This won't display!</h1>
  <script>alert("this won't alert!");</script>
</template>
```

`<template>`内部就是正常的 HTML 代码，浏览器不会将这些代码加入 DOM。

下面的代码会将模板内部的代码插入 DOM。

```javascript
let template = document.querySelector('template');
document.body.appendChild(template.content);
```

注意，模板内部的代码只能插入一次，如果第二次执行上面的代码就会报错。

如果需要多次插入模板，可以复制`<template>`内部代码，然后再插入。

```javascript
document.body.appendChild(template.content.cloneNode(true));
```

上面代码中，`cloneNode()`方法的参数`true`表示复制包含所有子节点。

接受`<template>`插入的元素，叫做宿主元素（host）。在`<template>`之中，可以对宿主元素设置样式。

```html
<template>
<style>
  :host {
    background: #f8f8f8;
  }
  :host(:hover) {
    background: #ccc;
  }
</style>
</template>
```

### document.importNode()

document.importNode方法用于克隆外部文档的DOM节点。

```javascript
var iframe = document.getElementsByTagName("iframe")[0];
var oldNode = iframe.contentWindow.document.getElementById("myNode");
var newNode = document.importNode(oldNode, true);
document.getElementById("container").appendChild(newNode);
```

上面例子是将iframe窗口之中的节点oldNode，克隆进入当前文档。

注意，克隆节点之后，还必须用appendChild方法将其加入当前文档，否则不会显示。换个角度说，这意味着插入外部文档节点之前，必须用document.importNode方法先将这个节点准备好。

document.importNode方法接受两个参数，第一个参数是外部文档的DOM节点，第二个参数是一个布尔值，表示是否连同子节点一起克隆，默认为false。大多数情况下，必须显式地将第二个参数设为true。

## Shadow DOM

所谓 Shadow DOM 指的是，浏览器将模板、样式表、属性、JavaScript 码等，封装成一个独立的 DOM 元素。外部的设置无法影响到其内部，而内部的设置也不会影响到外部，与浏览器处理原生网页元素（比如`<video>`元素）的方式很像。

Shadow DOM 最大的好处有两个，一是可以向用户隐藏细节，直接提供组件，二是可以封装内部样式表，不会影响到外部。

Custom Element 内部有一个 Shadow Root。它就是接入外部 DOM 的根元素。

```javascript
// attachShadow() creates a shadow root.
let shadow = div.attachShadow({ mode: 'open' });
let inner = document.createElement('b');
inner.appendChild(document.createTextNode('Hiding in the shadows'));

// shadow root supports the normal appendChild method.
shadow.appendChild(inner);
div.querySelector('b'); // empty
```

上面代码中，`<div>`包含`<b>`，但是 DOM 方法无法看到它，而且页面的样式也影响不到它。

`mode: 'open'`表示开发者工具里面，可以看到 Custom HTML 内部的 DOM，并与之互动。`mode: closed`将不允许 Custom Element 的使用者与内部代码互动。

Shadow root 内部通过指定`innerHTML`属性或使用`<template>`元素，指定 HTML 代码。

Shadow DOM 内部可以通过向根添加`<style>`（或`<link>`）来设置样式。

```javascript
let style = document.createElement('style');
style.innerText = 'b { font-weight: bolder; color: red; }';
shadowRoot.appendChild(style);

let inner = document.createElement('b');
inner.innerHTML = "I'm bolder in the shadows";
shadowRoot.appendChild(inner);
```

上面代码添加的样式，只会影响 Shadow DOM 内的元素。

Custom Element 的 CSS 样式内部，`:root`表示这个根元素。比如，Custom Element 默认是行内元素，下面代码可以改成块级元素。

```css
:host {
  display: block;
}

:host([disabled]) {
  opacity: 0.5;
}
```

注意，外部样式会覆盖掉`:host`的设置，比如下面的样式会覆盖`:host`。

```css
my-element {
  display: inline-block;
}
```

利用 CSS 的自定义属性，可以为 Custom Element 可以被覆盖的默认样式。下面是外部样式，`my-element`是 Custom Element。

```css
my-element {
  --background-color: #ff0000;
}
```

然后，内部可以指定默认样式，用于用户没有指定颜色的情况。

```css
:host {
  --background-color: #ffffff;
}

#container {
  background-color: var(--background-color);
}
```

下面的例子是为 Shadow DOM 加上独立的模板。

```html
<div id="nameTag">张三</div>

<template id="nameTagTemplate">
  <style>
    .outer {
      border: 2px solid brown;
    }
  </style>

  <div class="outer">
    <div class="boilerplate">
      Hi! My name is
    </div>
    <div class="name">
      Bob
    </div>
  </div>
</template>
```

上面代码是一个`div`元素和模板。接下来，就是要把模板应用到`div`元素上。

## HTML Import

### 基本操作

长久以来，网页可以加载外部的样式表、脚本、图片、多媒体，却无法方便地加载其他网页，iframe和ajax都只能提供部分的解决方案，且有很大的局限。HTML Import就是为了解决加载外部网页这个问题，而提出来的。

下面代码用于测试当前浏览器是否支持HTML Import。

```javascript

function supportsImports() {
  return 'import' in document.createElement('link');
}

if (supportsImports()) {
  // 支持
} else {
  // 不支持
}

```

HTML Import用于将外部的HTML文档加载进当前文档。我们可以将组件的HTML、CSS、JavaScript封装在一个文件里，然后使用下面的代码插入需要使用该组件的网页。

```html

<link rel="import" href="dialog.html">

```

上面代码在网页中插入一个对话框组件，该组建封装在`dialog.html`文件。注意，dialog.html文件中的样式和JavaScript脚本，都对所插入的整个网页有效。

假定A网页通过HTML Import加载了B网页，即B是一个组件，那么B网页的样式表和脚本，对A网页也有效（准确得说，只有style标签中的样式对A网页有效，link标签加载的样式表对A网页无效）。所以可以把多个样式表和脚本，都放在B网页中，都从那里加载。这对大型的框架，是很方便的加载方法。

如果B与A不在同一个域，那么A所在的域必须打开CORS。

```html

<!-- example.com必须打开CORS -->
<link rel="import" href="http://example.com/elements.html">

```

除了用link标签，也可以用JavaScript调用link元素，完成HTML Import。

```javascript

var link = document.createElement('link');
link.rel = 'import';
link.href = 'file.html'
link.onload = function(e) {...};
link.onerror = function(e) {...};
document.head.appendChild(link);

```

HTML Import加载成功时，会在link元素上触发load事件，加载失败时（比如404错误）会触发error事件，可以对这两个事件指定回调函数。

```html

<script async>
  function handleLoad(e) {
    console.log('Loaded import: ' + e.target.href);
  }
  function handleError(e) {
    console.log('Error loading import: ' + e.target.href);
  }
</script>

<link rel="import" href="file.html"
      onload="handleLoad(event)" onerror="handleError(event)">

```

上面代码中，handleLoad和handleError函数的定义，必须在link元素的前面。因为浏览器元素遇到link元素时，立刻解析并加载外部网页（同步操作），如果这时没有对这两个函数定义，就会报错。

HTML Import是同步加载，会阻塞当前网页的渲染，这主要是为了样式表的考虑，因为外部网页的样式表对当前网页也有效。如果想避免这一点，可以为link元素加上async属性。当然，这也意味着，如果外部网页定义了组件，就不能立即使用了，必须等HTML Import完成，才能使用。

```html

<link rel="import" href="/path/to/import_that_takes_5secs.html" async>

```

但是，HTML Import不会阻塞当前网页的解析和脚本执行（即阻塞渲染）。这意味着在加载的同时，主页面的脚本会继续执行。

最后，HTML Import支持多重加载，即被加载的网页同时又加载其他网页。如果这些网页都重复加载同一个外部脚本，浏览器只会抓取并执行一次该脚本。比如，A网页加载了B网页，它们各自都需要加载jQuery，浏览器只会加载一次jQuery。

### 脚本的执行

外部网页的内容，并不会自动显示在当前网页中，它只是储存在浏览器中，等到被调用的时候才加载进入当前网页。为了加载网页网页，必须用DOM操作获取加载的内容。具体来说，就是使用link元素的import属性，来获取加载的内容。这一点与iframe完全不同。

```javascript

var content = document.querySelector('link[rel="import"]').import;

```

发生以下情况时，link.import属性为null。

- 浏览器不支持HTML Import
- link元素没有声明`rel="import"`
- link元素没有被加入DOM
- link元素已经从DOM中移除
- 对方域名没有打开CORS

下面代码用于从加载的外部网页选取id为template的元素，然后将其克隆后加入当前网页的DOM。

```javascript

var el = linkElement.import.querySelector('#template');

document.body.appendChild(el.cloneNode(true));

```

当前网页可以获取外部网页，反过来也一样，外部网页中的脚本，不仅可以获取本身的DOM，还可以获取link元素所在的当前网页的DOM。

```javascript

// 以下代码位于被加载（import）的外部网页

// importDoc指向被加载的DOM
var importDoc = document.currentScript.ownerDocument;

// mainDoc指向主文档的DOM
var mainDoc = document;

// 将子页面的样式表添加主文档
var styles = importDoc.querySelector('link[rel="stylesheet"]');
mainDoc.head.appendChild(styles.cloneNode(true));

```

上面代码将所加载的外部网页的样式表，添加进当前网页。

被加载的外部网页的脚本是直接在当前网页的上下文执行，因为它的`window.document`指的是当前网页的document，而且它定义的函数可以被当前网页的脚本直接引用。

### Web Component的封装

对于Web Component来说，HTML Import的一个重要应用是在所加载的网页中，自动登记Custom Element。

```html

<script>
  // 定义并登记<say-hi>
  var proto = Object.create(HTMLElement.prototype);

  proto.createdCallback = function() {
    this.innerHTML = 'Hello, <b>' +
                     (this.getAttribute('name') || '?') + '</b>';
  };

  document.registerElement('say-hi', {prototype: proto});
</script>

<template id="t">
  <style>
    ::content > * {
      color: red;
    }
  </style>
  <span>I'm a shadow-element using Shadow DOM!</span>
  <content></content>
</template>

<script>
  (function() {
    var importDoc = document.currentScript.ownerDocument; //指向被加载的网页

    // 定义并登记<shadow-element>
    var proto2 = Object.create(HTMLElement.prototype);

    proto2.createdCallback = function() {
      var template = importDoc.querySelector('#t');
      var clone = document.importNode(template.content, true);
      var root = this.createShadowRoot();
      root.appendChild(clone);
    };

    document.registerElement('shadow-element', {prototype: proto2});
  })();
</script>

```

上面代码定义并登记了两个元素：\<say-hi\>和\<shadow-element\>。在主页面使用这两个元素，非常简单。

```html

<head>
  <link rel="import" href="elements.html">
</head>
<body>
  <say-hi name="Eric"></say-hi>
  <shadow-element>
    <div>( I'm in the light dom )</div>
  </shadow-element>
</body>

```

不难想到，这意味着HTML Import使得Web Component变得可分享了，其他人只要拷贝`elements.html`，就可以在自己的页面中使用了。

## Polymer.js

Web Components是非常新的技术，为了让老式浏览器也能使用，Google推出了一个函数库[Polymer.js](http://www.polymer-project.org/)。这个库不仅可以帮助开发者，定义自己的网页元素，还提供许多预先制作好的组件，可以直接使用。

### 直接使用的组件

Polymer.js提供的组件，可以直接插入网页，比如下面的google-map。。

```html

<script src="components/platform/platform.js"></script>
<link rel="import" href="google-map.html">
<google-map lat="37.790" long="-122.390"></google-map>

```

再比如，在网页中插入一个时钟，可以直接使用下面的标签。

```html

<polymer-ui-clock></polymer-ui-clock>

```

自定义标签与其他标签的用法完全相同，也可以使用CSS指定它的样式。

```css

polymer-ui-clock {
  width: 320px;
  height: 320px;
  display: inline-block;
  background: url("../assets/glass.png") no-repeat;
  background-size: cover;
  border: 4px solid rgba(32, 32, 32, 0.3);
}

```

### 安装

如果使用bower安装，至少需要安装platform和core components这两个核心部分。

```bash

bower install --save Polymer/platform
bower install --save Polymer/polymer

```

你还可以安装所有预先定义的界面组件。

```bash

bower install Polymer/core-elements
bower install Polymer/polymer-ui-elements

```

还可以只安装单个组件。

```bash

bower install Polymer/polymer-ui-accordion

```

这时，组件根目录下的bower.json，会指明该组件的依赖的模块，这些模块会被自动安装。

```javascript

{
  "name": "polymer-ui-accordion",
  "private": true,
  "dependencies": {
    "polymer": "Polymer/polymer#0.2.0",
    "polymer-selector": "Polymer/polymer-selector#0.2.0",
    "polymer-ui-collapsible": "Polymer/polymer-ui-collapsible#0.2.0"
  },
  "version": "0.2.0"
}

```

### 自定义组件

下面是一个最简单的自定义组件的例子。

```html

<link rel="import" href="../bower_components/polymer/polymer.html">
 
<polymer-element name="lorem-element">
  <template>
    <p>Lorem ipsum</p>
  </template>
</polymer-element>

```

上面代码定义了lorem-element组件。它分成三个部分。

**（1）import命令**

import命令表示载入核心模块

**（2）polymer-element标签**

polymer-element标签定义了组件的名称（注意，组件名称中必须包含连字符）。它还可以使用extends属性，表示组件基于某种网页元素。

```html

<polymer-element name="w3c-disclosure" extends="button">

```

**（3）template标签**

template标签定义了网页元素的模板。

### 组件的使用方法

在调用组件的网页中，首先加载polymer.js库和组件文件。

```html

<script src="components/platform/platform.js"></script>
<link rel="import" href="w3c-disclosure.html">

```

然后，分成两种情况。如果组件不基于任何现有的HTML网页元素（即定义的时候没有使用extends属性），则可以直接使用组件。

```html

<lorem-element></lorem-element>

```

这时网页上就会显示一行字“Lorem ipsum”。

如果组件是基于（extends）现有的网页元素，则必须在该种元素上使用is属性指定组件。

```

<button is="w3c-disclosure">Expand section 1</button>

```

## 参考链接

- [The Power of Web Components](https://hacks.mozilla.org/2018/11/the-power-of-web-components/), Potch
- Todd Motto, [Web Components and concepts, ShadowDOM, imports, templates, custom elements](http://toddmotto.com/web-components-concepts-shadow-dom-imports-templates-custom-elements/)
- Dominic Cooney, [Shadow DOM 101](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/)
- Eric Bidelman, [HTML's New Template Tag](http://www.html5rocks.com/en/tutorials/webcomponents/template/)
- Rey Bango, [Using Polymer to Create Web Components](http://code.tutsplus.com/tutorials/using-polymer-to-create-web-components--cms-20475)
- Cédric Trévisan, Building an Accessible Disclosure Button – using Web Components](http://blog.paciellogroup.com/2014/06/accessible-disclosure-button-using-web-components/)
- Eric Bidelman, [Custom Elements: defining new elements in HTML](http://www.html5rocks.com/en/tutorials/webcomponents/customelements/)
- Eric Bidelman, [HTML Imports](http://www.html5rocks.com/en/tutorials/webcomponents/imports/)
- TJ VanToll, [Why Web Components Are Ready For Production](http://developer.telerik.com/featured/web-components-ready-production/)
- Chris Bateman, [A No-Nonsense Guide to Web Components, Part 1: The Specs](http://cbateman.com/blog/a-no-nonsense-guide-to-web-components-part-1-the-specs/)
- [Web Components will replace your frontend framework](https://blog.usejournal.com/web-components-will-replace-your-frontend-framework-3b17a580831c), Danny Moerkerke
- [Custom Elements v1: Reusable Web Components](https://developers.google.com/web/fundamentals/web-components/customelements#extend), Eric Bidelman
