---
title: Temporal.Duration.compare()
short-title: compare()
slug: Web/JavaScript/Reference/Global_Objects/Temporal/Duration/compare
l10n:
  sourceCommit: 7e14795a6ef2bf5e760c315ce64800dd1cd98c29
---

**`Temporal.Duration.compare()`** 为静态方法，返回一个数字 (-1, 0, 或 1)，用以指示第一个时间段是短于、等于或长于第二个时间段。

## 语法

```js-nolint
Temporal.Duration.compare(duration1, duration2)
Temporal.Duration.compare(duration1, duration2, options)
```

### 参数

- `duration1`
  - : 一个字符串，或一个对象，或一个 {{jsxref("Temporal.Duration")}} 实例，代表参与比较的第一个时间段。它会被转换为 `Temporal.Duration` 对象，使用与 {{jsxref("Temporal/Duration/from", "Temporal.Duration.from()")}} 相同的算法。
- `duration2`
  - : 参与比较的第二个时间段，会被转换为 `Temporal.Duration` 对象，使用的算法与 `duration1` 相同。
- `options` {{optional_inline}}
  - : 一个对象，具有如下属性:
    - `relativeTo` {{optional_inline}}
      - : 一个带时区的、或单纯的日期（时间），提供用于解算 [日历时间段](/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Temporal/Duration#日历时间段) 的日期和日历信息（点击链接获取关于该选项的大概解释）。当 `duration1` 或 `duration2` 二者任一为日历时间段时该参数为必要参数（除非两个时间段的每个部分均相等，此时将返回 `0`，不进行计算）。

### 返回值

返回 `-1` 说明 `duration1` 短于 `duration2`，返回 `0` 说明二者相等，而返回 `1` 说明 `duration1` 长于 `duration2`。

### 异常

- {{jsxref("RangeError")}}
  - : 当 `duration1` 或 `duration2` 二者任一为 [日历时间段](/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Temporal/Duration#日历时间段) (拥有非零的 `years`, `months` 或 `weeks`) 且未提供 `relativeTo` 时抛出。

## 说明

若 `relativeTo` 为带时区的日期时间，且 `duration1` 或 `duration2` 二者任一为日历时间，则会将两个时间段加于起始点后，对得到的两个瞬间进行比较并得出结果。否则，则会将两个时间段转换至纳秒（假设一日为 24 小时，并在必要时使用 `relativeTo` 的日历）再对结果进行比较。

## 示例

### 使用 Temporal.Duration.compare()

```js
const d1 = Temporal.Duration.from({ hours: 1, minutes: 30 });
const d2 = Temporal.Duration.from({ minutes: 100 });
console.log(Temporal.Duration.compare(d1, d2)); // -1

const d3 = Temporal.Duration.from({ hours: 2 });
const d4 = Temporal.Duration.from({ minutes: 110 });
console.log(Temporal.Duration.compare(d3, d4)); // 1

const d5 = Temporal.Duration.from({ hours: 1, minutes: 30 });
const d6 = Temporal.Duration.from({ seconds: 5400 });
console.log(Temporal.Duration.compare(d5, d6)); // 0
```

### 比较日历时间段

```js
const d1 = Temporal.Duration.from({ days: 31 });
const d2 = Temporal.Duration.from({ months: 1 });

console.log(
  Temporal.Duration.compare(d1, d2, {
    relativeTo: Temporal.PlainDate.from("2021-01-01"), // ISO 8601 日历
  }),
); // 0

console.log(
  Temporal.Duration.compare(d1, d2, {
    relativeTo: Temporal.PlainDate.from("2021-02-01"), // ISO 8601 日历
  }),
); // 1; 二月有 28 天
```

### 使用带时区的 relativeTo

使用带时区的 `relativeTo` 时，甚至会将夏令时间（夏时制；日照节约时间）的改变纳入考虑。在 `2024-11-03`，美利坚合众国从夏时制切换至标准时间，因此那一日会有 25 小时，因为时钟会向过去调一小时。

```js
const d1 = Temporal.Duration.from({ days: 1 });
const d2 = Temporal.Duration.from({ hours: 24 });

console.log(
  Temporal.Duration.compare(d1, d2, {
    relativeTo: Temporal.ZonedDateTime.from(
      "2024-11-03T01:00-04:00[America/New_York]",
    ),
  }),
); // 1
```

### 对时间段的数组进行排序

该 `compare()` 函数的目的是为了用作传入 {{jsxref("Array.prototype.sort()")}} 及相关函数的比较器。

```js
const durations = [
  Temporal.Duration.from({ hours: 1 }),
  Temporal.Duration.from({ hours: 2 }),
  Temporal.Duration.from({ hours: 1, minutes: 30 }),
  Temporal.Duration.from({ hours: 1, minutes: 45 }),
];

durations.sort(Temporal.Duration.compare);
console.log(durations.map((d) => d.toString()));
// [ 'PT1H', 'PT1H30M', 'PT1H45M', 'PT2H' ]
```

要传入可选参数，则像这样:

```js
durations.sort((a, b) =>
  Temporal.Duration.compare(a, b, {
    relativeTo: Temporal.Now.zonedDateTimeISO(),
  }),
);
```

## 规范

{{Specifications}}

## 浏览器兼容性

{{Compat}}

## 参见

- {{jsxref("Temporal.Duration")}}
- {{jsxref("Temporal/Duration/subtract", "Temporal.Duration.prototype.subtract()")}}
