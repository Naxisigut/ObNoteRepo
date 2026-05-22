# Flutter：`TextInputType` 简介

`TextInputType` 定义在 `package:flutter/services.dart`（通过 `material.dart` 等上层库间接导出）。它描述**系统软键盘（以及部分桌面平台输入法）应以何种形态呈现**，供 `TextField` / `TextFormField` / `EditableText` 的 `keyboardType` 使用。

它 **不会自动校验或过滤字符**：用户仍可能通过粘贴、外接键盘输入不符合预期的内容。需要限制输入时，应配合 `inputFormatters`（如 `FilteringTextInputFormatter`）或提交前校验。

---

## 基本用法

```dart
import 'package:flutter/material.dart';

TextField(
  keyboardType: TextInputType.emailAddress,
)
```

---

## 常用内置类型

| 类型 | 典型用途 | 说明 |
|------|----------|------|
| `TextInputType.text` | 普通单行文本 | 默认键盘 |
| `TextInputType.multiline` | 多行备注、评论 | 通常配合 `maxLines: null` 或大于 1 |
| `TextInputType.number` | 整数键盘 | 具体布局因平台而异 |
| `TextInputType.phone` | 电话号码 | 数字为主 + 拨号相关符号 |
| `TextInputType.datetime` | 日期时间（提示性） | 仍建议用日期选择器做复杂场景 |
| `TextInputType.emailAddress` | 邮箱 | 键盘常含 `@`、`.` |
| `TextInputType.url` | URL | 键盘常含 `/`、`:` |
| `TextInputType.visiblePassword` | 可见密码（较少单独用） | 常见是与 `obscureText` 组合时的键盘提示 |
| `TextInputType.name` | 人名（平台提示） | 部分平台会做联想/拼写优化 |
| `TextInputType.none` | 不弹出键盘 | 多用于自定义输入场景 |

---

## `numberWithOptions`：带符号与小数

金额、度量等需要 **小数** 或 **负数** 时，不要用裸的 `TextInputType.number`，一般用：

```dart
TextField(
  keyboardType: const TextInputType.numberWithOptions(decimal: true),
)
```

可选参数：

- `signed: true`：允许负数（键盘可能出现 `-`）。
- `decimal: true`：允许小数点。

本项目筛选最低价/最高价示例：`product_filter_bar.dart` 中 `_PriceField` 使用 `TextInputType.numberWithOptions(decimal: true)`。

---

## 平台差异（简要）

- **iOS / Android**：`keyboardType` 对软键盘布局影响最明显。
- **桌面 / Web**：物理键盘为主时，键盘类型多为**提示**，体验不如移动端显著。
- 最终呈现以系统输入法为准，**不可假设**某一类型在所有设备上完全一致。

---

## 与相关 API 的配合

- **`inputFormatters`**：限制可输入字符（数字-only、长度、正则等）。
- **`textInputAction`**：回车键行为（下一项、完成、搜索等）。
- **`keyboardAppearance`**（Cupertino）：明暗键盘外观。
- **`enableSuggestions` / `autocorrect`**：是否联想与纠错，可与邮箱、验证码等场景配合关闭。

---

## 参考

- Flutter API：`TextInputType` — https://api.flutter.dev/flutter/services/TextInputType-class.html
