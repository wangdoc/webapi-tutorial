# Intl segmenter API

## 简介

Intl.Segmenter 是浏览器内置的用于文本分词的 API。

使用时，先用`Intl.Segmenter()`新建一个分词器对象。

```javascript
const segmenter = new Intl.Segmenter(
  'en',
  { granularity: 'word' }
);
```

`Intl.Segmenter()`接受两个参数，第一个是所要分词的语言简称（上例是`en`），第二个参数是一个配置对象，有以下两个属性。

- `localeMatcher`：指定分词算法，有两个可能的值，一个是`lookup`，表示采用特定的算法（BCP 47），另一个是`best fit`（默认值），表示采用操作系统或浏览器现有的尽可能适用的算法。
- `granularity`：表示分词的颗粒度，有三个可能的值：grapheme（字符，这是默认值），word（词语），sentence（句子）。

拿到分词器对象以后，就可以进行分词了。

```javascript
const segmenter = new Intl.Segmenter(
  'en',
  { granularity: 'word' }
);

const segments = segmenter.segment('This has four words!');

Array.from(segments).map((segment) => segment.segment);
// ['This', ' ', 'has', ' ', 'four', ' ', 'words', '!']
```

上面示例中，变量`segmenter`是分词器对象，可以对英语进行分词，颗粒度是词语。所以，“This has four words!”被分成了8个部分，包括4个词语、3个空格和1个标点符号。

分词器对象的`segment()`方法是实际的分词方法，它的参数是需要分词的文本，返回值是一个具有迭代器接口的分词结果对象。`Array.from()`将这个分词结果对象转成数组，也可以采用`[...segments]`的写法。

下面的例子是过滤掉非词语字符。

```javascript
const segments = segmenter.segment('This has four words!');

Array.from(segments)
  .filter((segment) => segment.isWordLike)
  .map((segment) => segment.segment);
// ['This', 'has', 'four', 'words']
```

上面示例中，`Array.from()`将分词结果对象转成一个数组，变量`segment`是数组的每个成员，它也是一个对象。该对象的`isWordLike`属性是一个布尔值，表示当前值是否为一个真正的词，而该对象的`segment`属性（上例的`segment.segment`）则是真正的分词结果。

Intl Segmenter 支持各种语言，下面是日语分词的例子。

```javascript
const segmenter = new Intl.Segmenter('ja', { granularity: 'word' });
const segments = segmenter.segment('これは日本語のテキストです');

Array.from(segments).map((segment) => segment.segment);
// ['これ', 'は', '日本語', 'の', 'テキスト', 'です']
```

下面是法语的例子。

```javascript
const segmenterFr = new Intl.Segmenter('fr', { granularity: 'word' });
const string1 = 'Que ma joie demeure';

const iterator1 = segmenterFr.segment(string1)[Symbol.iterator]();

iterator1.next().value.segment // 'Que'
iterator1.next().value.segment // ' '
```

## 静态方法

### Intl.Segmenter.supportedLocalesOf()

`Intl.Segmenter.supportedLocalesOf()`返回一个数组，用来检测当前环境是否支持指定语言的分词。

```javascript
const locales1 = ['ban', 'id-u-co-pinyin', 'de-ID'];
const options1 = { localeMatcher: 'lookup', granularity: 'string' };

Intl.Segmenter.supportedLocalesOf(locales1, options1)
// ["id-u-co-pinyin", "de-ID"]
```

它接受两个参数，第一个参数是一个数组，数组成员是需要检测的语言简称；第二个参数是配置对象，跟构造方法的第二个参数是一致的，可以省略。

上面示例中，需要检测的三种语言分别是巴厘岛语（ban）、印度尼西亚语（id-u-co-pinyin）、德语（de-ID）。结果显示只支持前两者，不支持巴厘岛语。

## 实例方法

### resolvedOptions()

实例对象的`resolvedOptions()`方法，用于获取构造该实例时的参数。

```javascript
const segmenter1 = new Intl.Segmenter('fr-FR');
const options1 = segmenter1.resolvedOptions();

options1.locale // "fr-FR"
options1.granularity // "grapheme"
```

上面示例中，`resolveOptions()`方法返回了一个对象，该对象的`locale`属性对应构造方法的第一个参数，`granularity`属性对应构造方法第二个参数对象的颗粒度属性。

### segment()

实例对象的`segment()`方法进行实际的分词。

```javascript
const segmenterFr = new Intl.Segmenter('fr', { granularity: 'word' });
const string1 = 'Que ma joie demeure';

const segments = segmenterFr.segment(string1);

segments.containing(5)
// {segment: 'ma', index: 4, input: 'Que ma joie demeure', isWordLike: true}
```

`segment()`方法的返回结果是一个具有迭代器接口的分词结果对象，有三种方法进行处理。

（1）使用`Array.from()`或扩展运算符（`...`）将分词结果对象转成数组。

```javascript
const segmenterFr = new Intl.Segmenter('fr', { granularity: 'word' });
const string1 = 'Que ma joie demeure';

const iterator1 = segmenterFr.segment(string1);

Array.from(iterator1).map(segment => {
  if (segment.segment.length > 4) {
    console.log(segment.segment);
  }
})
// demeure
```

上面示例中，`segmenterFr.segment()`返回一个针对`string1`的分词结果对象，该对象具有迭代器接口。`Array.from()`将其转为数组，数组的每个成员是一个分词颗粒对象，该对象的`segment`属性就是分词结果。分词颗粒对象的介绍，详见后文。

（2）使用`for...of`循环，遍历分词结果对象。

```javascript
const segmenterFr = new Intl.Segmenter('fr', { granularity: 'word' });
const string1 = 'Que ma joie demeure';

const iterator1 = segmenterFr.segment(string1);

for (const segment of iterator1) {
  if (segment.segment.length > 4) {
    console.log(segment.segment);
  }
}
// demeure
```

上面示例中，`for...of`默认调用分词结果对象的迭代器接口，获取每一轮的分词颗粒对象。

由于迭代器接口是在`Symbol.iterator`属性上面，所以实际执行的代码如下。

```javascript
const segmenterFr = new Intl.Segmenter('fr', { granularity: 'word' });
const string1 = 'Que ma joie demeure';

const iterator1 = segmenterFr.segment(string1)[Symbol.iterator]();

for (const segment of iterator1) {
  if (segment.segment.length > 4) {
    console.log(segment.segment);
  }
}
// "demeure"
```

`for...of`循环每一轮得到的是一个分词颗粒对象，该对象的`segment`属性就是当前的分词结果，详见下文。

（3）使用`containing()`方法获取某个位置的分词颗粒对象。

```javascript
const segmenterFr = new Intl.Segmenter('fr', { granularity: 'word' });
const string1 = 'Que ma joie demeure';

const segments = segmenterFr.segment(string1);

segments.containing(5)
// {segment: 'ma', index: 4, input: 'Que ma joie demeure', isWordLike: true}
```

`containing()`方法的参数是一个整数，表示原始字符串的指定位置（从0开始计算）。如果省略该参数，则默认为0。

`containing()`的返回值是该位置的分词颗粒对象，如果参数位置超出原始字符串，则返回`undefined`。分词颗粒对象有以下属性。

- segment：指定位置对应的分词结果。
- index：本次分词在原始字符串的开始位置（从0开始）。
- input：进行分词的原始字符串。
- isWordLike：如果分词颗粒度为`word`，该属性返回一个布尔值，表示当前值是否一个真正的词。如果分词颗粒度不为`word`，则返回`undefined`。

```javascript
const input = "Allons-y!";

const segmenter = new Intl.Segmenter("fr", { granularity: "word" });
const segments = segmenter.segment(input);

let current = segments.containing();
// { index: 0, segment: "Allons", isWordLike: true }

current = segments.containing(4);
// { index: 0, segment: "Allons", isWordLike: true }

current = segments.containing(6);
// { index: 6, segment: "-", isWordLike: false }

current = segments.containing(current.index + current.segment.length);
// { index: 7, segment: "y", isWordLike: true }

current = segments.containing(current.index + current.segment.length);
// { index: 8, segment: "!", isWordLike: false }

current = segments.containing(current.index + current.segment.length);
// undefined
```

上面示例中，分词结果中除了空格和标点符号，其他情况下，`isWordLike`都返回`false`。

