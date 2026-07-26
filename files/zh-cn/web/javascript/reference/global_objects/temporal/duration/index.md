---
title: Temporal.Duration
slug: Web/JavaScript/Reference/Global_Objects/Temporal/Duration
l10n:
  sourceCommit: 7e14795a6ef2bf5e760c315ce64800dd1cd98c29
---

**`Temporal.Duration`** 对象表示两个时间点之间的差值，可用于日期/时间运算。它基础表示为年、月、周、日、时、分、秒、毫秒、微秒和纳秒数值的组合。

## 描述

### ISO 8601 时间段格式

`Duration` 对象能以 [ISO 8601 时间段格式](https://en.wikipedia.org/wiki/ISO_8601#Durations) 序列化或从其解析（加以 ECMAScript 指定的一些扩展）。字符串有以下的形式（空格仅为方便阅读，实际字符串中不应包含空格）：

```plain
±P nY nM nW nD T nH nM nS
```

- `±` {{optional_inline}}
  - : 一个可选的正负号（`+` 或 `-`），用以表示时间段的正向或负向。默认为正向。
- `P`
  - : 一个 `P` 或 `p` 字母本身，代表英文单词 "period"。
- `nY`, `nM`, `nW`, `nD`, `nH`, `nM`, `nS`
  - : 一个数字，其后跟一个字母, 分别表示年 (`Y`)，月 (`M`)，周 (`W`)，日 (`D`)，时 (`H`)，分 (`M`) 或秒 (`S`) 的数量。除了最末的部分外，每个部分都应用整数。若最末的部分为时间部分（时、分、秒），则可包含 1 至 9 位的小数部分，以句点或逗号开始，如 `PT0.0021S` 或 `PT1.1H`。任意为 0 的部分均可被省略，但至少应有一个部分（即使其值为 0，该情况下意味着时间段为 0）。
- `T`
  - : 一个 `T` 或 `t` 字母本身，用以将日期部分隔开、区分出时间部分，当且仅当其后有至少一个部分时使用。

以下是一些示例：

| ISO 8601           | 含义                                                                  |
| ------------------ | ----- |
| `P1Y1M1DT1H1M1.1S` | 1 年 1 月 1 天 1 时 1 分 1 秒 100 毫秒 |
| `P40D`             | 40 天 |
| `P1Y1D`            | 1 年 1 天 |
| `P3DT4H59M`        | 3 天 4 时 59 分钟 |
| `PT2H30M`          | 2 小时 30 分钟 |
| `P1M`              | 1 月 |
| `PT1M`             | 1 分 |
| `PT0.0021S`        | 2.1 毫秒（2 毫秒又 100 微秒） |
| `PT0S`             | 零（标准表示） |
| `P0D`              | 零 |

> [!NOTE]
> 根据 ISO 8601-1 标准，“周”不能和其他任何单位一起出现，且时间段仅能为正。Temporal 使用的 ISO 8601-2 作为该标准的补充，允许字符串起始使用正负号，并且允许“周”和其他单位一起组合。因此，如若你的时间段序列化为如 `P3W1D`, `+P1M` 或 `-P1M` 这样的字符串，需留意其他程序可能不会接受它。

序列化时，会尽可能尊重所存储的日期/时间部分，[不平衡的](#时间段的平衡)部分保持不变。不过，小于“秒”的（亚秒级）部分会被序列化进“秒”的小数部分，因此如果这些部分的值是不平衡的，则其精确值会丢失。正向时间单位的正号会被省略。为零的时间段总会被序列化为 `PT0S`。

### 日历时间段

*日历时间段* 是指包含了任意[日历](/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Temporal#日历)单位（周、月、年）的时间段。非日历时间段，由于它们无歧义地表示一个定量的时间，则可被任意挪用，无需任何日历信息即可参与日期/时间运算时。而日历时间段不能被随意挪用，因为一月或一年的天数取决于日历系统和参考时间点。因此，对日历时间段进行任何算术运算操作的尝试，均会抛出错误，因为时间段本身不和日历绑定。例如，如果我们处于格里高利历（公历）的五月，则“1 月”意味着“31 天”，但如果处于四月，则“1 月”会变成“30 天”。如要加上或减去日历时间段，需要把它们加至日期上：

```js
const dur1 = Temporal.Duration.from({ years: 1 });
const dur2 = Temporal.Duration.from({ months: 1 });

dur1.add(dur2); // 抛出 RangeError：对于日历时间段的运算，要使用相对于一个起始点的日期运算

const startingPoint = Temporal.PlainDate.from("2021-01-01"); // ISO 8601 日历
startingPoint.add(dur1).add(dur2).since(startingPoint); // "P396D"
```

其余操作，即 `round()`, `total()` 和 `compare()`，需通过 `relativeTo` 选项提供必要的日历（历法）和参考时间。该选项可以是一个 {{jsxref("Temporal.PlainDate")}}, {{jsxref("Temporal.PlainDateTime")}}, {{jsxref("Temporal.ZonedDateTime")}}，或一个可被 {{jsxref("Temporal/ZonedDateTime/from", "Temporal.ZonedDateTime.from()")}} （需提供 `timeZone` 选项或字符串中包含时区标注）或 {{jsxref("Temporal/PlainDate/from", "Temporal.PlainDate.from()")}} 转换的对象或字符串。

需注意，严格来说，从“日”到“时”的转换也是具有歧义的，因为一日的长度可能会因起始点的改变（如夏令时/日光节约时间）而不同。可提供带时区的 `relativeTo` 以说明这样的改变；否则将假定一日为 24 小时。

### 时间段的平衡

同一时间段可以有很多种表示方式：例如，“1 分钟 30 秒”和“90 秒”是等价的。不过，根据语境的不同，某种表示方式可能会比另一种更合适。因此，普遍来说，`Duration` 对象会尽可能保留输入的值，以便在被格式化时以你所预期的方式显示。

时间段的每个部分都有最佳范围；小时数应在 0 至 23 之间，分钟数在 0 至 59 之间，诸如此类。当某个部分溢出其最佳范围时，超出部分可能会被“进位（移入）”相邻的更大的部分。为了进位，需要弄清楚“有多少个 X 在 Y 里面”，对于 [日历单位](#日历时间段) 来说，这个问题没有直接的答案，需要针对具体日历才能解答。此外还需注意，默认情况下，“日”会被直接进位入“月”；“周”只有在显式请求时才会被进位进入。如若尽可能多地进位，最终结果里所有部分都在其最佳区间内，这样就是一个“平衡的”时间段。不平衡的时间段通常是“头重”型，即最大单位是不平衡的（如“27 小时 30 分”）；其余形式，如“23 小时又 270 分钟”，则很少见。

{{jsxref("Temporal/Duration/round", "round()")}} 方法总会将时间段平衡为“头重”型，归入直至 `largestUnit` 选项。通过手动指定一个足够大的 `largestUnit` 选项，可以将时间段充分平衡。类似地，{{jsxref("Temporal/Duration/add", "add()")}} 和 {{jsxref("Temporal/Duration/subtract", "subtract()")}} 方法会将远算结果平衡至输入时间段中最大的单位。

需要注意，由于 ISO 8601 时间段格式将小于秒级的部分表示在单一小数中，因此使用默认格式的序列化无法保留不平衡的小于秒的部分。例如，“1000 毫秒”会被序列化为 `"PT1S"`，随后则会被反序列化为“1 秒”。如需保留小于秒的部分的尺度，则需手动将其作为 JSON 对象序列化（因为 {{jsxref("Temporal/Duration/toJSON", "toJSON()")}} 方法默认会将时间段以 ISO 8601 格式序列化）。

### 时间段的正负性

由于时间段是两个时间点之间的差值，它可为正、负或零。例如，假如你以相对时间表示事件的时长，则负向的时间段或许代表过去的事件，而正向的时间段用于未来的事件。在我们这种“分量组合”的表示法中，符号（正负性）存储在每个部分中：一个负向的时间段的所有部分总是为负（或零），而正向时间段的所有部分总是为正（或零）。正负性混合的部分组成的时间段无效，会被构造函数或 {{jsxref("Temporal/Duration/with", "with()")}} 方法拒绝。而 `add()` 和 `subtract()` 方法会对结果进行平衡处理，以避免正负性混合。

## 构造函数

- {{jsxref("Temporal/Duration/Duration", "Temporal.Duration()")}}
  - : 创建一个新的 `Temporal.Duration` 对象：通过直接提供底层数据。

## 静态方法

- {{jsxref("Temporal/Duration/compare", "Temporal.Duration.compare()")}}
  - : 返回一个数 (-1, 0, or 1) 用以指示第一个时间段之于第二个时间段是较短、相等或较长。
- {{jsxref("Temporal/Duration/from", "Temporal.Duration.from()")}}
  - : 创建一个新的 `Temporal.Duration` 对象：使用另一个 `Temporal.Duration` 对象，或一个具有时间段属性的对象，或一个 ISO 8601 字符串。

## 实例属性

这些属性定义在 `Temporal.Duration.prototype` 上，被所有 `Temporal.Duration` 实例共享.

- {{jsxref("Temporal/Duration/blank", "Temporal.Duration.prototype.blank")}}
  - : 返回布尔值 `true` 时说明时间段为 0；否则返回 `false`。等价于 `duration.sign === 0`。
- {{jsxref("Object/constructor", "Temporal.Duration.prototype.constructor")}}
  - : 创建实例对象所用的构造函数。对于 `Temporal.Duration` 实例，初始值即 {{jsxref("Temporal/Duration/Duration", "Temporal.Duration()")}} 这一构造函数。
- {{jsxref("Temporal/Duration/days", "Temporal.Duration.prototype.days")}}
  - : 返回一个整数，表示时间段中的天数。
- {{jsxref("Temporal/Duration/hours", "Temporal.Duration.prototype.hours")}}
  - : 返回一个整数，表示时间段中的小时数。
- {{jsxref("Temporal/Duration/microseconds", "Temporal.Duration.prototype.microseconds")}}
  - : 返回一个整数，表示时间段中的微秒数。
- {{jsxref("Temporal/Duration/milliseconds", "Temporal.Duration.prototype.milliseconds")}}
  - : 返回一个整数，表示时间段中的毫秒数。
- {{jsxref("Temporal/Duration/minutes", "Temporal.Duration.prototype.minutes")}}
  - : 返回一个整数，表示时间段中的分钟数。
- {{jsxref("Temporal/Duration/months", "Temporal.Duration.prototype.months")}}
  - : 返回一个整数，表示时间段中的月数。
- {{jsxref("Temporal/Duration/nanoseconds", "Temporal.Duration.prototype.nanoseconds")}}
  - : 返回一个整数，表示时间段中的纳秒数。
- {{jsxref("Temporal/Duration/seconds", "Temporal.Duration.prototype.seconds")}}
  - : 返回一个整数，表示时间段中的秒数。
- {{jsxref("Temporal/Duration/sign", "Temporal.Duration.prototype.sign")}}
  - : 返回 `1` 说明时间段为正向, `-1` 则时间段为负向，返回 `0` 则时间段为 0.
- {{jsxref("Temporal/Duration/weeks", "Temporal.Duration.prototype.weeks")}}
  - : 返回一个整数，表示时间段中的周数。
- {{jsxref("Temporal/Duration/years", "Temporal.Duration.prototype.years")}}
  - : 返回一个整数，表示时间段中的年数。
- `Temporal.Duration.prototype[Symbol.toStringTag]`
  - : [`[Symbol.toStringTag]`](/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/toStringTag) 属性的初始值为字符串 `"Temporal.Duration"`。该属性在 {{jsxref("Object.prototype.toString()")}} 中被使用.

## 实例方法

- {{jsxref("Temporal/Duration/abs", "Temporal.Duration.prototype.abs()")}}
  - : 返回一个新的 `Temporal.Duration` 对象，为该时间段的绝对值（所有字段的尺度保持不变，但正负性变为正）。
- {{jsxref("Temporal/Duration/add", "Temporal.Duration.prototype.add()")}}
  - : 返回一个新的 `Temporal.Duration` 对象，为该时间段和给定时间段（以能被 {{jsxref("Temporal/Duration/from", "Temporal.Duration.from()")}} 转换的形式）的和。结果是[平衡的](#时间段的平衡)。
- {{jsxref("Temporal/Duration/negated", "Temporal.Duration.prototype.negated()")}}
  - : 返回一个新的 `Temporal.Duration` 对象，其值与该时间段相反（所有字段的尺度保持不变，但正负性发生反转）。
- {{jsxref("Temporal/Duration/round", "Temporal.Duration.prototype.round()")}}
  - : 返回一个新的 `Temporal.Duration` 对象，时间段会被舍入到传入的最小的单位，并且/或[被平衡到](#时间段的平衡)所传入的最大的单位。
- {{jsxref("Temporal/Duration/subtract", "Temporal.Duration.prototype.subtract()")}}
  - : 返回一个新的 `Temporal.Duration` 对象，其为该时间段和给定时间段（以能被 {{jsxref("Temporal/Duration/from", "Temporal.Duration.from()")}} 转换的形式） 的差值。等价于 [调用 .add()](/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Temporal/Duration/add) 并传入另一个时间段 [调用 .negated() 取反](/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Temporal/Duration/negated) 的值.
- {{jsxref("Temporal/Duration/toJSON", "Temporal.Duration.prototype.toJSON()")}}
  - : 返回一个字符串，表示该时间段，格式和调用 {{jsxref("Temporal/Duration/toString", "toString()")}} 相同，为 [ISO 8601 格式](#iso_8601_时间段格式) 表示该时间段。预期被 {{jsxref("JSON.stringify()")}} 隐式调用。
- {{jsxref("Temporal/Duration/toLocaleString", "Temporal.Duration.prototype.toLocaleString()")}}
  - : 返回一个字符串，为该时间段在考虑了当前语言情况下的表示。若实现支持 [`Intl.DurationFormat` API](/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/DurationFormat)，该方法会将工作交由 `Intl.DurationFormat` 进行。
- {{jsxref("Temporal/Duration/toString", "Temporal.Duration.prototype.toString()")}}
  - : 返回一个字符串，为该时间段在 [ISO 8601 格式](#iso_8601_时间段格式) 下的表示。
- {{jsxref("Temporal/Duration/total", "Temporal.Duration.prototype.total()")}}
  - : 返回一个数字，为该时间段在给定单位下的表示。
- {{jsxref("Temporal/Duration/valueOf", "Temporal.Duration.prototype.valueOf()")}}
  - : 抛出一个 {{jsxref("TypeError")}}，这会防止 `Temporal.Duration` 实例在用于运算或比较操作时被 [隐式转换到原始值](/zh-CN/docs/Web/JavaScript/Guide/Data_structures#原始值强制转换)。
- {{jsxref("Temporal/Duration/with", "Temporal.Duration.prototype.with()")}}
  - : 返回一个新的 `Temporal.Duration` 对象，表示该时间段在某些字段被新值替换后的结果。

## 规范

{{Specifications}}

## 浏览器兼容性

{{Compat}}

## 参见

- {{jsxref("Temporal")}}

