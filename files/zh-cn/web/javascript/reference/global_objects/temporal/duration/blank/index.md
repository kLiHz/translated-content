---
title: Temporal.Duration.prototype.blank
short-title: blank
slug: Web/JavaScript/Reference/Global_Objects/Temporal/Duration/blank
l10n:
  sourceCommit: 7e14795a6ef2bf5e760c315ce64800dd1cd98c29
---

{{jsxref("Temporal.Duration")}} 实例的 **`blank`** 访问器属性返回一个布尔值，时间段代表零时返回 `true`，否则返回 `false`。等价于 `duration.sign === 0`。

## Examples

### 使用 blank

```js
const d1 = Temporal.Duration.from({ hours: 1, minutes: 30 });
const d2 = Temporal.Duration.from({ hours: -1, minutes: -30 });
const d3 = Temporal.Duration.from({ hours: 0 });

console.log(d1.blank); // false
console.log(d2.blank); // false
console.log(d3.blank); // true
```

## 规范

{{Specifications}}

## 浏览器兼容性

{{Compat}}

## 参见

- {{jsxref("Temporal.Duration")}}
- {{jsxref("Temporal/Duration/sign", "Temporal.Duration.prototype.sign")}}
