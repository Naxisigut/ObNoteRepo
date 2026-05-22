# Dart `const` 规范化与 Flutter `Widget` 复用边界

本文整理自项目内关于 **`const` 规范化（canonicalization）**、**跨文件是否同一实例**、以及 **`Widget` 与纯数据对象复用边界** 的讨论，便于日后查阅。

---

## 1. 什么叫规范化实例

对 **`const` 表达式**，若在编译期能完全求值，Dart 会使用规范化实例（canonical instance）。因此当常量表达式完全一致时：

```dart
identical(const EdgeInsets.all(8), const EdgeInsets.all(8)); // true
```

这不是值相等，而是 **identity 相同**（同一规范实例）。

---

## 2. 规范化大致规则（何时合并为同一实例）

一般会发生规范化的条件包括：

- 使用 **`const` 构造**（或 `const` 字面量）。
- **所有实参**都是编译期常量。
- **类型与参数**在常量意义上等价。

由此：两处一模一样的 `const SomeClass(...)`，通常会指向同一个规范实例。

**写法位置通常不影响这一点**：无论在 `static`、文件顶层、`build` 内局部 `const`，还是不同 `.dart` 文件，只要处于同一 Isolate 且 `const` 表达式等价，通常仍是 `identical` 的同一对象。

---

## 3. 何时不会共用实例

- **未使用 `const`**：普通构造每次执行通常产生新实例（除非你自己缓存）。
- **参数含运行时值**：表达式无法成为 `const`，不会走常量规范化。
- **不同 Isolate**：对象空间隔离，不会跨 Isolate 共享同一实例。

---

## 4. 与 Flutter 的关系：`const Widget` 复用本身通常是安全的

Flutter 用 `Widget` 描述配置、由 `Element` 承载运行时位置。  
**同一个不可变 `Widget` 实例（尤其 `const`）被多处使用在实践中通常是可行且常见的**，并非天然错误。

像 `const SizedBox.shrink()`、`const Icon(...)`、`const Text('...')` 这类不可变 widget 被多处复用很常见。  
你在价格输入框看到的“前缀未聚焦不显示，聚焦后显示”，根因是 `InputDecoration.prefix/suffix` 的显示机制，不是 `const` 规范化导致的多处复用冲突。

真正需要注意的边界：

- 不要复用同一个 `GlobalKey` 到多个节点。
- 不要把可变状态对象（如 `TextEditingController`、`FocusNode`）错误共享到不该共享的位置。
- 对纯不可变 widget，`const` 复用一般没有问题。

---

## 5. `Widget` 与 `TextStyle` 的复用差异

| | `Widget`（如 `Padding`） | `TextStyle`（非 `Widget`） |
|--|-------------------------|-----------------------------|
| 角色 | 声明 UI 配置 | 不可变样式数据 |
| 复用 `const` 实例 | 通常可行 | 通常可行 |
| 常见风险来源 | `GlobalKey` 误用、状态对象误共享 | 主要是业务语义而非框架机制 |

`TextStyle` 被规范化成同一实例时，本质是多处读取同一份不可变数据；  
而 widget 复用时也仍会在树中形成各自位置的 `Element`，并不等同于“一个 element 被挂两次”。

一句话：  
- **`const` 规范化让对象“可共享”，不等于会破坏 Flutter 树结构。**  
- **输入前后缀显示异常，应优先看 `InputDecoration` 规则，不要先归因到 `const`。**

---

## 6. 实务建议（价格前缀场景）

- `static const TextStyle`、`EdgeInsets` 等数据对象可放心复用。
- 对 `InputDecoration.prefix/suffix`：要预期它们在“空且未聚焦”时可能不显示。
- 需要“始终可见”的货币符号时，优先放在 `TextField` 外层 `Row`，或使用 `prefixIcon` 并调 `prefixIconConstraints`。

---

## 参考

- Dart 语言规范（常量表达式与常量求值）：[Language specification](https://dart.dev/resources/language/spec)
- Flutter：`Widget` / `Element` 更新模型 — https://docs.flutter.dev/resources/inside-flutter
- Flutter API：`InputDecoration` — https://api.flutter.dev/flutter/material/InputDecoration-class.html
