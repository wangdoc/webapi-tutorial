# Web Audio API

Web Audio API 用于操作声音。这个 API 可以让网页发出声音。

## 基本用法

浏览器原生提供`AudioContext`对象，该对象用于生成一个声音的上下文，与扬声器相连。

```javascript
const audioContext = new AudioContext();
```

然后，获取音源文件，将其在内存中解码，就可以播放声音了。

```javascript
const context = new AudioContext();

fetch('sound.mp4')
  .then(response => response.arrayBuffer())
  .then(arrayBuffer => context.decodeAudioData(arrayBuffer))
  .then(audioBuffer => {
    // 播放声音
    const source = context.createBufferSource();
    source.buffer = audioBuffer;
    source.connect(context.destination);
    source.start();
  });
```

## context.createBuffer()

`context.createBuffer()`方法生成一个内存的操作视图，用于存放数据。

```javascript
const buffer = audioContext.createBuffer(channels, signalLength, sampleRate);
```

`createBuffer`方法接受三个参数。

- channels：整数，表示声道。创建单声道的声音，该值为 1。
- signalLength：整数，表示声音数组的长度。
- sampleRate：浮点数，表示取样率，即一秒取样多少次。

`signalLength`和`sampleRate`这两个参数决定了声音的长度。比如，如果取样率是`1/3000`（每秒取样3000次），声音数组长度是6000，那么播放的声音是2秒长度。

接着，使用`buffer.getChannelData`方法取出一个声道。

```javascript
const data = buffer.getChannelData(0)
```

上面代码中，`buffer.getChannelData`的参数`0`表示取出第一个声道。

下一步，将声音数组放入这个声道。

```javascript
const data = buffer.getChannelData(0)

// singal 是一个声音数组
// singalLengal 是该数组的长度
for (let i = 0; i < signalLength; i += 1) {
  data[i] = signal[i]
}
```

最后，使用`context.createBufferSource`方法生成一个声音节点。

```javascript
// 生成一个声音节点
const node = audioContext.createBufferSource();
// 将声音数组的内存对象，放入这个节点
node.buffer = buffer;
// 将声音上下文与节点连接
node.connect(audioContext.destination);
// 开始播放声音
node.start(audioContext.currentTime);
```

默认情况下，播放一次后就将停止播放。如果需要循环播放，可以将节点对象的`looping`属性设为`true`。

```javascript
node.looping = true;
```

## 过滤器

Web Audio API 原生提供了一些过滤器（filter），用来处理声音。

首先，使用`context.createBiquadFilter`方法建立过滤器实例。

```javascript
const filter = audioContext.createBiquadFilter();
```

然后，通过`filter.type`属性指定过滤器的类型。

```javascript
filter.type = 'lowpass';
```

目前，过滤器有以下这些类型。

- lowpass
- highpass
- bandpass
- lowshelf
- highshelf
- peaking
- notch
- allpass

然后指定过滤器的频率（frequency）属性。

```javascript
filter.frequency.value = frequency
```

最后，过滤器实例连接节点实例，就可以生效了。

```javascript
sourceNode.connect(filter);
```

