# Canvas API

## 概述

`<canvas>`元素用于生成图像。它本身就像一个画布，JavaScript 通过操作它的 API，在上面生成图像。它的底层是一个个像素，基本上`<canvas>`是一个可以用 JavaScript 操作的位图（bitmap）。

它与 SVG 图像的区别在于，`<canvas>`是脚本调用各种方法生成图像，SVG 则是一个 XML 文件，通过各种子元素生成图像。

使用 Canvas API 之前，需要在网页里面新建一个`<canvas>`元素。

```html
<canvas id="myCanvas" width="400" height="250">
  您的浏览器不支持 Canvas
</canvas>
```

如果浏览器不支持这个 API，就会显示`<canvas>`标签中间的文字：“您的浏览器不支持 Canvas”。

每个`<canvas>`元素都有一个对应的`CanvasRenderingContext2D`对象（上下文对象）。Canvas API 就定义在这个对象上面。

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');
```

上面代码中，`<canvas>`元素节点对象的`getContext()`方法，返回的就是`CanvasRenderingContext2D`对象。

注意，Canvas API 需要`getContext`方法指定参数`2d`，表示该`<canvas>`节点生成 2D 的平面图像。如果参数是`webgl`，就表示用于生成 3D 的立体图案，这部分属于 WebGL API。

按照用途，Canvas API 分成两大部分：绘制图形和图像处理。

## Canvas API：绘制图形

Canvas 画布提供了一个作图的平面空间，该空间的每个点都有自己的坐标。原点`(0, 0)`位于图像左上角，`x`轴的正向是原点向右，`y`轴的正向是原点向下。

### 路径

以下方法和属性用来绘制路径。

- CanvasRenderingContext2D.beginPath()：开始绘制路径。
- CanvasRenderingContext2D.closePath()：结束路径，返回到当前路径的起始点，会从当前点到起始点绘制一条直线。如果图形已经封闭，或者只有一个点，那么此方法不会产生任何效果。
- CanvasRenderingContext2D.moveTo()：设置路径的起点，即将一个新路径的起始点移动到`(x，y)`坐标。
- CanvasRenderingContext2D.lineTo()：使用直线从当前点连接到`(x, y)`坐标。
- CanvasRenderingContext2D.fill()：在路径内部填充颜色（默认为黑色）。
- CanvasRenderingContext2D.stroke()：路径线条着色（默认为黑色）。
- CanvasRenderingContext2D.fillStyle：指定路径填充的颜色和样式（默认为黑色）。
- CanvasRenderingContext2D.strokeStyle：指定路径线条的颜色和样式（默认为黑色）。

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');

ctx.beginPath();
ctx.moveTo(100, 100);
ctx.lineTo(200, 200);
ctx.lineTo(100, 200);
```

上面代码只是确定了路径的形状，画布上还看不出来，因为没有颜色。所以还需要着色。

```javascript
ctx.fill()
// 或者
ctx.stroke()
```

上面代码中，这两个方法都可以使得路径可见。`fill()`在路径内部填充颜色，使之变成一个实心的图形；`stroke()`只对路径线条着色。

这两个方法默认都是使用黑色，可以使用`fillStyle`和`strokeStyle`属性指定其他颜色。

```javascript
ctx.fillStyle = 'red';
ctx.fill();
// 或者
ctx.strokeStyle = 'red';
ctx.stroke();
```

上面代码将填充和线条的颜色指定为红色。

## 线型

以下的方法和属性控制线条的视觉特征。

- CanvasRenderingContext2D.lineWidth：指定线条的宽度，默认为1.0。
- CanvasRenderingContext2D.lineCap：指定线条末端的样式，有三个可能的值：butt（默认值，末端为矩形）、round（末端为圆形）、square（末端为突出的矩形）。
- CanvasRenderingContext2D.lineJoin：指定线段交点的样式，有三个可能的值：round（交点为扇形）、bevel（交点为三角形底边）、miter（默认值，交点为菱形)。
- CanvasRenderingContext2D.miterLimit：指定交点菱形的长度，默认为10。该属性只在`lineJoin`属性的值等于`miter`时有效。
- CanvasRenderingContext2D.getLineDash()：返回一个数组，表示虚线里面线段和间距的长度。
- CanvasRenderingContext2D.setLineDash()：数组，用于指定虚线里面线段和间距的长度。

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');

ctx.beginPath();
ctx.moveTo(100, 100);
ctx.lineTo(200, 200);
ctx.lineTo(100, 200);

ctx.lineWidth = 3;
ctx.lineCap = 'round';
ctx.lineJoin = 'round';
ctx.setLineDash([15, 5]);
ctx.stroke();
```

上面代码中，线条的宽度为3，线条的末端和交点都改成圆角，并且设置为虚线。

### 矩形

以下方法用来绘制矩形。

- CanvasRenderingContext2D.rect()：绘制矩形路径。
- CanvasRenderingContext2D.fillRect()：填充一个矩形。
- CanvasRenderingContext2D.strokeRect()：绘制矩形边框。
- CanvasRenderingContext2D.clearRect()：指定矩形区域的像素都变成透明。

上面四个方法的格式都一样，都接受四个参数，分别是矩形左上角的横坐标和纵坐标、矩形的宽和高。

`CanvasRenderingContext2D.rect()`方法用于绘制矩形路径。

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');

ctx.rect(10, 10, 100, 100);
ctx.fill();
```

上面代码绘制一个正方形，左上角坐标为`(10, 10)`，宽和高都为100。

`CanvasRenderingContext2D.fillRect()`用来向一个矩形区域填充颜色。

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');

ctx.fillStyle = 'green';
ctx.fillRect(10, 10, 100, 100);
```

上面代码绘制一个绿色的正方形，左上角坐标为`(10, 10)`，宽和高都为100。

`CanvasRenderingContext2D.strokeRect()`用来绘制一个矩形区域的边框。

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');

ctx.strokeStyle = 'green';
ctx.strokeRect(10, 10, 100, 100);
```

上面代码绘制一个绿色的空心正方形，左上角坐标为`(10, 10)`，宽和高都为100。

`CanvasRenderingContext2D.clearRect()`用于擦除指定矩形区域的像素颜色，等同于把早先的绘制效果都去除。

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');

ctx.fillRect(10, 10, 100, 100);
ctx.clearRect(15, 15, 90, 90);
```

上面代码先绘制一个 100 x 100 的正方形，然后在它的内部擦除 90 x 90 的区域，等同于形成了一个5像素宽度的边框。

### 弧线

以下方法用于绘制弧形。

- `CanvasRenderingContext2D.arc()`：通过指定圆心和半径绘制弧形。
- `CanvasRenderingContext2D.arcTo()`：通过指定两根切线和半径绘制弧形。

`CanvasRenderingContext2D.arc()`主要用来绘制圆形或扇形。

```javascript
// 格式
ctx.arc(x, y, radius, startAngle, endAngle, anticlockwise)

// 实例
ctx.arc(5, 5, 5, 0, 2 * Math.PI, true)
```

`arc()`方法的`x`和`y`参数是圆心坐标，`radius`是半径，`startAngle`和`endAngle`则是扇形的起始角度和终止角度（以弧度表示），`anticlockwise`表示做图时应该逆时针画（`true`）还是顺时针画（`false`），这个参数用来控制扇形的方向（比如上半圆还是下半圆）。

下面是绘制实心圆形的例子。

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');

ctx.beginPath();
ctx.arc(60, 60, 50, 0, Math.PI * 2, true); 
ctx.fill();
```

上面代码绘制了一个半径50，起始角度为0，终止角度为 2 * PI 的完整的圆。

绘制空心半圆的例子。

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');

ctx.beginPath();
ctx.moveTo(50, 20);
ctx.arc(100, 20, 50, 0, Math.PI, false);
ctx.stroke();
```

`CanvasRenderingContext2D.arcTo()`方法主要用来绘制圆弧，需要给出两个点的坐标，当前点与第一个点形成一条直线，第一个点与第二个点形成另一条直线，然后画出与这两根直线相切的弧线。

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');

ctx.beginPath();
ctx.moveTo(0, 0);
ctx.arcTo(50, 50, 100, 0, 25);
ctx.lineTo(100, 0);
ctx.stroke();
```

上面代码中，`arcTo()`有5个参数，前两个参数是第一个点的坐标，第三个参数和第四个参数是第二个点的坐标，第五个参数是半径。然后，`(0, 0)`与`(50, 50)`形成一条直线，然后`(50, 50)`与`(100, 0)`形成第二条直线。弧线就是与这两根直线相切的部分。

### 绘制文本

`context.fillText(string, x, y)`用来绘制文本，它的三个参数分别为文本内容、起点的`x`坐标、`y`坐标。使用之前，需用`font`属性设置字体、大小、样式（写法类似与 CSS 的`font`属性）。与此类似的还有`strokeText`方法，用来添加空心字。

```javascript
// 设置字体
ctx.font = "Bold 20px Arial";
// 设置对齐方式
ctx.textAlign = "left";
// 设置填充颜色
ctx.fillStyle = "#008600";
// 设置字体内容，以及在画布上的位置
ctx.fillText("Hello!", 10, 50);
// 绘制空心字
ctx.strokeText("Hello!", 10, 100);
```

`fillText`方法不支持文本断行，即所有文本出现在一行内。所以，如果要生成多行文本，只有调用多次`fillText`方法。

### 设置渐变色

`context.createLinearGradient`方法用来设置渐变色。

```javascript
var myGradient = ctx.createLinearGradient(0, 0, 0, 160); 
myGradient.addColorStop(0, "#BABABA"); 
myGradient.addColorStop(1, "#636363");
```

`createLinearGradient`方法的参数是`(x1, y1, x2, y2)`，其中`x1`和`y1`是起点坐标，`x2`和`y2`是终点坐标。通过不同的坐标值，可以生成从上至下、从左到右的渐变等等。

使用方法如下：

```javascript
ctx.fillStyle = myGradient;
ctx.fillRect(10,10,200,100);
```

### 设置阴影

一系列与阴影相关的方法，可以用来设置阴影。

```javascript
ctx.shadowOffsetX = 10; // 设置水平位移
ctx.shadowOffsetY = 10; // 设置垂直位移
ctx.shadowBlur = 5; // 设置模糊度
ctx.shadowColor = "rgba(0,0,0,0.5)"; // 设置阴影颜色

ctx.fillStyle = "#CC0000";
ctx.fillRect(10,10,200,100);
```

## 图像处理方法

### drawImage()

Canvas API 允许将图像文件插入画布，做法是读取图片后，使用`drawImage`方法在画布内进行重绘。

```javascript
var canvas = document.querySelector('#canvas');
var ctx = canvas.getContext('2d');

var img = new Image();
img.src = 'image.png';
ctx.drawImage(img, 0, 0); // 将图像放置在画布，后两个参数是图像左上角的坐标
```

上面代码将一个 PNG 图像载入画布。`drawImage()`方法接受三个参数，第一个参数是图像文件的 DOM 元素（即`<img>`节点），第二个和第三个参数是图像左上角在画布中的坐标，上例中的`(0, 0)`就表示将图像左上角放置在画布的左上角。

由于图像的载入需要时间，`drawImage`方法只能在图像完全载入后才能调用，因此上面的代码需要改写。

```javascript
var image = new Image();

image.onload = function() {
  var canvas = document.createElement('canvas');
  canvas.width = image.width;
  canvas.height = image.height;
  canvas.getContext('2d').drawImage(image, 0, 0);
  // 插入页面底部
  document.body.appendChild(image);
  return canvas;
}

image.src = 'image.png';
```

### getImageData()，putImageData()

`getImageData`方法可以用来读取 Canvas 的内容，返回一个对象，包含了每个像素的信息。

```javascript
var imageData = context.getImageData(0, 0, canvas.width, canvas.height);
```

`imageData`对象有一个`data`属性，它的值是一个一维数组。该数组的值，依次是每个像素的红、绿、蓝、alpha 通道值，因此该数组的长度等于`图像的像素宽度 x 图像的像素高度 x 4`，每个值的范围是 0～255。这个数组不仅可读，而且可写，因此通过操作这个数组的值，就可以达到操作图像的目的。

`putImageData`方法将一维的像素数组绘制在 Canvas 画布上。

```javascript
context.putImageData(imageData, 0, 0);
```

### toDataURL()

对图像数据做出修改以后，可以使用`toDataURL`方法，将 Canvas 数据重新转化成一般的图像文件形式。

```javascript
function convertCanvasToImage(canvas) {
  var image = new Image();
  image.src = canvas.toDataURL('image/png');
  return image;
}
```

上面的代码将 Canvas 数据，转化成PNG data URI。

### save方法，restore方法

save方法用于保存上下文环境，restore方法用于恢复到上一次保存的上下文环境。

```javascript
ctx.save();

ctx.shadowOffsetX = 10;
ctx.shadowOffsetY = 10;
ctx.shadowBlur = 5;
ctx.shadowColor = 'rgba(0,0,0,0.5)';

ctx.fillStyle = '#CC0000';
ctx.fillRect(10,10,150,100);

ctx.restore();

ctx.fillStyle = '#000000';
ctx.fillRect(180,10,150,100);
```

上面代码先用`save`方法，保存了当前设置，然后绘制了一个有阴影的矩形。接着，使用`restore`方法，恢复了保存前的设置，绘制了一个没有阴影的矩形。

## 图像变换

### 平移、旋转、缩放

```javascript
ctx.translate( x, y )//位移：把图像原点位移到(x， y)的位置
ctx.rotate( deg )//旋转：旋转 deg 度数
ctx.scale( sx, sy )//缩放：在横向进行 sx 倍的缩放，在纵向进行 sy 倍的缩放
```

缩放出现的问题
1.如果有`lineWith`，宽度也会缩放
2.如果起始点不是`(0, 0)`，起始点也会缩放

### 变换矩阵

```javascript
ctx.transform(a, b, c, d, e, f);
/*
a:水平缩放(默认值1)
b:水平倾斜(默认值0)
c:垂直倾斜(默认值0)
d:垂直缩放(默认值1)
e:水平位移(默认值0)
f:垂直位移(默认值0)
*/
```

`context.transform()`可以叠加使用，如果需要重新初始化矩阵变换的值，可以用`context.setTransform(a, b, c, d, e, f)`。它会使得之前设置的`context.transform()`失效，恢复为单位矩阵然后再`transform`。

## 动画

利用 JavaScript，可以在 Canvas 元素上很容易地产生动画效果。

```javascript
var posX = 20,
    posY = 100;

setInterval(function() {
	context.fillStyle = "black";
    context.fillRect(0,0,canvas.width, canvas.height);

	posX += 1;
	posY += 0.25;

	context.beginPath();
	context.fillStyle = "white";

	context.arc(posX, posY, 10, 0, Math.PI*2, true); 
	context.closePath();
	context.fill();
}, 30);
```

上面代码会产生一个小圆点，每隔30毫秒就向右下方移动的效果。`setInterval`函数的一开始，之所以要将画布重新渲染黑色底色，是为了抹去上一步的小圆点。

通过设置圆心坐标，可以产生各种运动轨迹。

先上升后下降。

```javascript
var vx = 10,
    vy = -10,
    gravity = 1;

setInterval(function() {
    posX += vx;
    posY += vy;
    vy += gravity;
	// ...
});
```

上面代码中，`x`坐标始终增大，表示持续向右运动。`y`坐标先变小，然后在重力作用下，不断增大，表示先上升后下降。

小球不断反弹后，逐步趋于静止。

```javascript
var vx = 10,
    vy = -10,
    gravity = 1;

setInterval(function() {
    posX += vx;
    posY += vy;

	if (posY > canvas.height * 0.75) {
          vy *= -0.6;
          vx *= 0.75;
          posY = canvas.height * 0.75;
    }
	
    vy += gravity;
	// ...
});
```

上面代码表示，一旦小球的y坐标处于屏幕下方75%的位置，向x轴移动的速度变为原来的75%，而向y轴反弹上一次反弹高度的40%。

## 像素处理

通过`getImageData`方法和`putImageData`方法，可以处理每个像素，进而操作图像内容。

假定`filter`是一个处理像素的函数，那么整个对Canvas的处理流程，可以用下面的代码表示。

```javascript
if (canvas.width > 0 && canvas.height > 0) {
	var imageData = context.getImageData(0, 0, canvas.width, canvas.height);
  filter(imageData);
  context.putImageData(imageData, 0, 0);
}
```

以下是几种常见的处理方法。

### 灰度效果

灰度图（grayscale）就是取红、绿、蓝三个像素值的算术平均值，这实际上将图像转成了黑白形式。假定`d[i]`是像素数组中一个象素的红色值，则`d[i+1]`为绿色值，`d[i+2]`为蓝色值，`d[i+3]`就是 alpha 通道值。转成灰度的算法，就是将红、绿、蓝三个值相加后除以3，再将结果写回数组。

```javascript
grayscale = function (pixels) {
  var d = pixels.data;
  for (var i = 0; i < d.length; i += 4) {
    var r = d[i];
    var g = d[i + 1];
    var b = d[i + 2];
    d[i] = d[i + 1] = d[i + 2] = (r+g+b)/3;
  }
  return pixels;
};
```

### 复古效果

复古效果（sepia）则是将红、绿、蓝三个像素，分别取这三个值的某种加权平均值，使得图像有一种古旧的效果。

```javascript
sepia = function (pixels) {
    var d = pixels.data;
    for (var i = 0; i < d.length; i += 4) {
      var r = d[i];
      var g = d[i + 1];
      var b = d[i + 2];
      d[i]     = (r * 0.393)+(g * 0.769)+(b * 0.189); // red
      d[i + 1] = (r * 0.349)+(g * 0.686)+(b * 0.168); // green
      d[i + 2] = (r * 0.272)+(g * 0.534)+(b * 0.131); // blue
    }
    return pixels;
};
```

### 红色蒙版效果

红色蒙版指的是，让图像呈现一种偏红的效果。算法是将红色通道设为红、绿、蓝三个值的平均值，而将绿色通道和蓝色通道都设为0。

```javascript
var red = function (pixels) {
  var d = pixels.data;
  for (var i = 0; i < d.length; i += 4) {
    var r = d[i];
    var g = d[i + 1];
    var b = d[i + 2];
    d[i] = (r+g+b)/3;        // 红色通道取平均值
    d[i + 1] = d[i + 2] = 0; // 绿色通道和蓝色通道都设为0
  }
  return pixels;
};
```

### 亮度效果

亮度效果（brightness）是指让图像变得更亮或更暗。算法将红色通道、绿色通道、蓝色通道，同时加上一个正值或负值。

```javascript
var brightness = function (pixels, delta) {
  var d = pixels.data;
  for (var i = 0; i < d.length; i += 4) {
    d[i] += delta;     // red
    d[i + 1] += delta; // green
    d[i + 2] += delta; // blue   
  }
  return pixels;
};
```

### 反转效果

反转效果（invert）是指图片呈现一种色彩颠倒的效果。算法为红、绿、蓝通道都取各自的相反值（255-原值）。

```javascript
invert = function (pixels) {
  var d = pixels.data;
  for (var i = 0; i < d.length; i += 4) {
		d[i] = 255 - d[i];
		d[i+1] = 255 - d[i + 1];
		d[i+2] = 255 - d[i + 2];
	}
	return pixels;
};
```

## 参考链接

- David Walsh, [JavaScript Canvas Image Conversion](http://davidwalsh.name/convert-canvas-image)
- Matt West, [Getting Started With The Canvas API](http://blog.teamtreehouse.com/getting-started-with-the-canvas-api)
- John Robinson, [How You Can Do Cool Image Effects Using HTML5 Canvas](http://www.storminthecastle.com/2013/04/06/how-you-can-do-cool-image-effects-using-html5-canvas/)
- Ivaylo Gerchev, [HTML5 Canvas Tutorial: An Introduction](http://www.sitepoint.com/html5-canvas-tutorial-introduction/)
- Donovan Hutchinson, [Particles in canvas](http://hop.ie/blog/particles/)
