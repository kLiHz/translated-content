---
title: Temporal.Duration.prototype.abs()
short-title: abs()
slug: Web/JavaScript/Reference/Global_Objects/Temporal/Duration/abs
l10n:
  sourceCommit: 7e14795a6ef2bf5e760c315ce64800dd1cd98c29
---

{{jsxref("Temporal.Duration")}} 实例的 **`abs()`** 方法返回一个新的 `Temporal.Duration` 对象，为该时间段的绝对值（所有字段的幅度相同，但正负性变为正）。

## 语法

```js-nolint
abs()
```

### 参数

无。

### 返回值

一个新的 `Temporal.Duration` 对象，为该时间段的绝对值，即，如该时间段已为正则与该时间段相同，若为负则为其 [反向](/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Temporal/Duration/negated)。

## 示例

### 使用 abs()

```js
const d1 = Temporal.Duration.from({ hours: 1, minutes: 30 });
const d2 = Temporal.Duration.from({ hours: -1, minutes: -30 });

console.log(d1.abs().toString()); // "PT1H30M"
console.log(d2.abs().toString()); // "PT1H30M"
```

## 规范

{{Specifications}}

## 浏览器兼容性

{{Compat}}

## 参见

- {{jsxref("Temporal.Duration")}}
- {{jsxref("Temporal/Duration/negated", "Temporal.Duration.prototype.negated()")}}
- {{jsxref("Temporal/Duration/sign", "Temporal.Duration.prototype.sign")}}
