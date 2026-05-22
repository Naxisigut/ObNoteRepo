# Flutter：`TextField` 装饰与显示行为速查

本文聚焦 `TextField` / `TextFormField` 的 `InputDecoration`，尤其是你遇到的 **`prefix` 在未聚焦时不显示** 问题。

---

## 先回答你的问题

- 是的，`suffix` 在很多场景下也有类似特性。
- `prefix` / `suffix` 属于输入行内（inline）内容，和可编辑文本、`hint` 共用同一行布局。
- 当输入框 **为空且未聚焦** 时，`hint` 往往优先占位，`prefix` / `suffix` 可能不显示；聚焦或有文本后会显示。

你在 `lib/shared/widgets/product_filter_bar.dart` 里看到的现象就是这个机制触发的。

---

## `prefix` / `suffix` vs `prefixIcon` / `suffixIcon`

| 属性 | 位置 | 常见显示行为 | 适用场景 |
|---|---|---|---|
| `prefix` / `suffix` | 输入文本同一行（靠近文本） | 可能受空值、焦点、hint 状态影响 | 金额符号、单位等“贴着文本”的内容 |
| `prefixText` / `suffixText` | 同上（文本版） | 与 `prefix` / `suffix` 类似 | 固定字符串前后缀 |
| `prefixIcon` / `suffixIcon` | 装饰容器的图标槽位 | 通常稳定显示，不依赖输入值 | 搜索图标、清空按钮、状态图标 |

如果你希望符号“始终可见”，优先考虑：

- 把符号放在 `TextField` 外层 `Row` 中；
- 或改用 `prefixIcon`（并用 `prefixIconConstraints` 控制尺寸，避免过宽）。

---

## 状态与可见性（实战心智模型）

可简单按这 4 个状态理解：

1. **空 + 未聚焦**：`hint` 显示；inline 前后缀（`prefix/suffix`）可能不显示。  
2. **空 + 已聚焦**：`hint` 可能继续显示（除非被 floating label 等行为替代）；inline 前后缀通常出现。  
3. **非空 + 未聚焦**：文本显示；inline 前后缀通常出现。  
4. **非空 + 已聚焦**：文本显示；inline 前后缀通常出现。  

> 说明：不同平台、主题和 `InputDecoration` 组合会有细微差异，但上面足够覆盖绝大多数业务场景。

---

## `hint`、`label`、`floatingLabelBehavior` 的关系

- `hintText`：占位提示，常在“空值状态”出现。
- `labelText`：标签，可在聚焦/有值时浮到上方。
- `floatingLabelBehavior`：控制标签何时浮动（`auto` / `always` / `never`）。

如果使用了 `labelText`，视觉焦点会从“占位提示”转向“浮动标签”，有时能减少和前后缀的冲突感。

---

## 你这个价格输入框的建议

金额场景一般要求货币符号稳定可见，建议优先级如下：

1. **最稳**：`Row(¥ + Expanded(TextField))`（你文件里其实已有注释版本）。
2. **次稳**：`prefixIcon` + 紧凑约束（保证始终显示且视觉接近 inline）。
3. **保持现状**：继续使用 `prefix`，接受“空且未聚焦时可能不显示”。

---

## 示例：用 `prefixIcon` 实现“始终显示且紧凑”

```dart
TextField(
  decoration: const InputDecoration(
    hintText: '最低价',
    border: InputBorder.none,
    isDense: true,
    prefixIcon: Padding(
      padding: EdgeInsets.only(left: 12, right: 4),
      child: Text('¥', style: TextStyle(fontSize: 12)),
    ),
    prefixIconConstraints: BoxConstraints(minWidth: 0, minHeight: 0),
    contentPadding: EdgeInsets.only(right: 12),
  ),
)
```

---

## 参考

- Flutter API: [`InputDecoration`](https://api.flutter.dev/flutter/material/InputDecoration-class.html)
- Flutter API: [`TextField`](https://api.flutter.dev/flutter/material/TextField-class.html)
