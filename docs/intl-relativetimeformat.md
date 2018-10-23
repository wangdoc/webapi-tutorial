# Intl.RelativeTimeFormat

很多日期库支持显示相对时间，比如“昨天”、“五分钟前”、“两个月之前”等等。由于不同的语言，日期显示的格式和相关词语都不同，造成这些库的体积非常大。

现在，浏览器提供内置的 Intl.RelativeTimeFormat API，可以不使用这些库，直接显示相对时间。

## 基本用法

`Intl.RelativeTimeFormat()`是一个构造函数，接受一个语言代码作为参数，返回一个相对时间的实例对象。如果省略参数，则默认传入当前运行时的语言代码。

```javascript
const rtf = new Intl.RelativeTimeFormat('en');

rtf.format(3.14, 'second') // "in 3.14 seconds"
rtf.format(-15, 'minute') // "15 minutes ago"
rtf.format(8, 'hour') // "in 8 hours"
rtf.format(-2, 'day') // "2 days ago"
rtf.format(3, 'week') // "in 3 weeks"
rtf.format(-5, 'month') // "5 months ago"
rtf.format(2, 'quarter') // "in 2 quarters"
rtf.format(-42, 'year') // "42 years ago"
```

上面代码指定使用英语显示相对时间。

下面是使用西班牙语显示相对时间的例子。

```javascript
const rtf = new Intl.RelativeTimeFormat('es');

rtf.format(3.14, 'second') // "dentro de 3,14 segundos"
rtf.format(-15, 'minute') // "hace 15 minutos"
rtf.format(8, 'hour') // "dentro de 8 horas"
rtf.format(-2, 'day') // "hace 2 días"
rtf.format(3, 'week') // "dentro de 3 semanas"
rtf.format(-5, 'month') // "hace 5 meses"
rtf.format(2, 'quarter') // "dentro de 2 trimestres"
rtf.format(-42, 'year') // "hace 42 años"
```

`Intl.RelativeTimeFormat()`还可以接受一个配置对象，作为第二个参数，用来精确指定相对时间实例的行为。配置对象共有下面这些属性。

- options.style：表示返回字符串的风格，可能的值有`long`（默认值，比如“in 1 month”）、`short`（比如“in 1 mo.”）、`narrow`（比如“in 1 mo.”）。对于一部分语言来说，`narrow`风格和`short`风格是类似的。
- options.localeMatcher：表示匹配语言参数的算法，可能的值有`best fit`（默认值）和`lookup`。
- options.numeric：表示返回字符串是数字显示，还是文字显示，可能的值有`always`（默认值，总是文字显示）和`auto`（自动转换）。

```javascript
// 下面的配置对象，传入的都是默认值
const rtf = new Intl.RelativeTimeFormat('en', {
  localeMatcher: 'best fit', // 其他值：'lookup'
  style: 'long', // 其他值：'short' or 'narrow'
  numeric: 'always', // 其他值：'auto'
});

// Now, let’s try some special cases!

rtf.format(-1, 'day') // "1 day ago"
rtf.format(0, 'day') // "in 0 days"
rtf.format(1, 'day') // "in 1 day"
rtf.format(-1, 'week') // "1 week ago"
rtf.format(0, 'week') // "in 0 weeks"
rtf.format(1, 'week') // "in 1 week"
```

上面代码中，显示的是“1 day ago”，而不是“yesterday”；显示的是“in 0 weeks”，而不是“this week”。这是因为默认情况下，相对时间显示的是数值形式，而不是文字形式。

改变这个行为，可以把配置对象的`numeric`属性改成`auto`。

```javascript
const rtf = new Intl.RelativeTimeFormat('en', { numeric: 'auto' });

rtf.format(-1, 'day') // "yesterday"
rtf.format(0, 'day') // "today"
rtf.format(1, 'day') // "tomorrow"
rtf.format(-1, 'week') // "last week"
rtf.format(0, 'week') // "this week"
rtf.format(1, 'week') // "next week"
```

## Intl.RelativeTimeFormat.prototype.format()

相对时间实例对象的`format`方法，接受两个参数，依次为时间间隔的数值和单位。其中，“单位”是一个字符串，可以接受以下八个值。

- year
- quarter
- month
- week
- day
- hour
- minute
- second

```javascript
let rtf = new Intl.RelativeTimeFormat('en');
rtf.format(-1, "day") // "yesterday"
rtf.format(2.15, "day") // "in 2.15 days
```

## Intl.RelativeTimeFormat.prototype.formatToParts()

相对时间实例对象的`formatToParts()`方法的参数跟`format()`方法一样，但是返回的是一个数组，用来精确控制相对时间的每个部分。

```javascript
const rtf = new Intl.RelativeTimeFormat('en', { numeric: 'auto' });

rtf.format(-1, 'day') 
// "yesterday"
rtf.formatToParts(-1, 'day');
// [{ type: "literal", value: "yesterday" }]

rtf.format(3, 'week');
// "in 3 weeks"
rtf.formatToParts(3, 'week');
// [
//   { type: 'literal', value: 'in ' },
//   { type: 'integer', value: '3', unit: 'week' },
//   { type: 'literal', value: ' weeks' }
// ]
```

返回数组的每个成员都是一个对象，拥有两个属性。

- type：字符串，表示输出值的类型。
- value：字符串，表示输出的内容。
- unit：如果输出内容表示一个数值（即`type`属性不是`literal`），那么还会有`unit`属性，表示数值的单位。

## 参考链接

- [The Intl.RelativeTimeFormat API](https://developers.google.com/web/updates/2018/10/intl-relativetimeformat), Mathias Bynens
- [Intl.RelativeTimeFormat API Specification](https://github.com/tc39/proposal-intl-relative-time#api), TC39
