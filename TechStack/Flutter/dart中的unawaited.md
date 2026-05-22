# Dart：`unawaited` 说明

`unawaited` 定义在 **`dart:async`** 里，作用是：**把一个 `Future` 标成「故意不等待」**，告诉分析器（analyzer）你知道这里会触发异步但不写 `await`，这是**刻意的**，不要报 `unawaited_futures` 之类的告警。

## 用法示例

```dart
unawaited(_onSubmitSearch());
```

含义是 **fire-and-forget**：异步函数照样会跑，但当前函数不会因为 `await` 而停顿；调用方也不等待它何时结束、成功与否（除非你另外在 Future 上加 `.catchError` 或 `whenComplete`）。

## 为什么在回调里常见？

例如在 `onTap: () => unawaited(_openImageSourceSheet())`：

- `onTap` 类型是 `void Function()`，不能写成 `async` 返回 `Future`（类型对不上）。
- 你又希望一点击就立刻执行异步逻辑。
- 用 `unawaited` 包一层：**既启动了 Future，又消除了「创建了 Future 却没用 await」的静态检查提示**。

若不用 `unawaited`，有的 `analysis_options` 会要求在顶层 `await` 或显式丢弃，否则会 lint。

## 和裸调用的区别

- `_onSubmitSearch();` —— 若没有 `await`，有的规则会告警「未处理的 Future」。
- `unawaited(_onSubmitSearch());` —— **明确表态**：我不要求在这里等待。

**注意：** 错误不会在 `unawaited` 里自动消失。若 Future 里抛错又没人 `catch`，仍可能变成未捕获异步错误（视全局错误处理而定）。需要时可写成：

```dart
unawaited(_onSubmitSearch().catchError(handleError));
```

## 一句话

`unawaited(future)` 表示「刻意不 await 这个 Future」，满足类型与 analyzer，常用于短回调（`onTap`、`onSubmitted`）里触发异步且不阻塞同步返回。
