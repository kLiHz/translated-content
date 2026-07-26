---
title: Temporal.Duration.prototype.add()
short-title: add()
slug: Web/JavaScript/Reference/Global_Objects/Temporal/Duration/add
l10n:
  sourceCommit: 7e14795a6ef2bf5e760c315ce64800dd1cd98c29
---

{{jsxref("Temporal.Duration")}} 实例的 **`add()`** 方法返回一个新的 `Temporal.Duration` 对象，为该时间段与给定时间段的和。结果是[平衡的](/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Temporal/Duration#时间段的平衡)。


## 语法

```js-nolint
add(other)
```

### 参数

- `other`
  - : 一个字符串，或一个对象，或一个 {{jsxref("Temporal.Duration")}} 实例，代表一个将加于该时间段的时间段。其会被转换为一个 `Temporal.Duration` 对象，与 {{jsxref("Temporal/Duration/from", "Temporal.Duration.from()")}} 使用相同的算法。

### 返回值

一个新的 `Temporal.Duration` 对象，代表该时间段与 `other` 的和。

### 异常

- {{jsxref("RangeError")}}
  - : 在遇到以下情况之一时抛出:
    - 当 `this` 亦或是 `other` 是一个 [日历时间段](/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Temporal/Duration#日历时间段s) （具有非零的 `years`, `months` 或 `weeks`），因为日历时间段在无历法和时间参考时是有歧义的。
    - 当 `this` 与 `other` 之和，溢出可表示的时间段的最大值，或下溢于可表示的最小值，即 ±2<sup>53</sup> 秒。

## 说明

非日历时间段能够无歧义地表示一个定量的时间。在内部，`this` 和 `other` 二者都会被转换为纳秒（假设一日为 24 小时），然后相加在一起。相加的结果随后会被转换回一个 `Temporal.Duration` 对象，因此结果总是[平衡的或“头重”的](/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Temporal/Duration#时间段的平衡)，即可能的最大单位为“日（`days`）”。

如要使用日历时间段进行加法或减法，可以将两个时间段加于某起始点，再在两个瞬间之间进行比较；即，`dur1 + dur2` 等价于 `(start + dur1 + dur2) - start`。

若要将一个时间段加于某个日期或时间，则应使用对应日期或时间对象的 `add()` 方法。

## 示例

### 使用 add()

```js
const d1 = Temporal.Duration.from({ hours: 1, minutes: 30 });
const d2 = Temporal.Duration.from({ hours: -1, minutes: -20 });

const d3 = d1.add(d2);
console.log(d3.toString()); // "PT10M"
```

### 日历时间段相加

```js
const d1 = Temporal.Duration.from({ days: 1 });
const d2 = Temporal.Duration.from({ months: 1 });

d1.add(d2); // RangeError: 对于日历时间段的运算，应基于一个起始点，使用日期进行运算

const start = Temporal.PlainDateTime.from("2022-01-01T00:00"); // ISO 8601 日历
const result = start.add(d1).add(d2).since(start);
console.log(result.toString()); // "P32D"
```

## 规范

{{Specifications}}

## 浏览器兼容性

{{Compat}}

## 参见

- {{jsxref("Temporal.Duration")}}
- {{jsxref("Temporal/Duration/subtract", "Temporal.Duration.prototype.subtract()")}}
