# Flutter 编辑器中 Run / Debug / Profile 的区别

在 VS Code、Cursor 等基于 Dart 扩展的编辑器里，`main()` 上方出现的 **Run**、**Debug**、**Profile** 是 **CodeLens** 快捷入口，对应三种不同的启动方式。下面从「是否附加调试器」和「Flutter 构建模式」两方面说明。

## 一句话对照

| 按钮 | 典型用途 | 是否附加调试器 | Flutter 模式（默认） |
|------|----------|----------------|----------------------|
| **Run** | 日常跑起来看 UI、走流程 | 一般**不**附加调试器（断点不生效） | **debug** |
| **Debug** | 查 bug：断点、单步、看变量 | **附加**调试器 | **debug** |
| **Profile** | 看性能：帧率、CPU、内存等 | 仍可通过 DevTools 连接，但以分析为主 | **profile** |

> 说明：具体参数以你项目里的 `launch.json` 为准；未自定义时，扩展的默认行为与上表一致。

---

## Run（运行）

- **做什么**：启动应用，等同于常见 IDE 里的「启动但不调试」（Start Without Debugging）。
- **特点**：
  - 仍是 **debug 构建**（断言、热重载等仍按 debug 行为），但**通常不会在断点处停下**，也不走完整的调试会话。
  - 适合：只想快速看界面、验证流程，不需要打断点。
- **命令层面**：对应 `flutter run` 一类启动，且不挂接调试器（与 Debug 的差别在这里）。

---

## Debug（调试）

- **做什么**：启动应用并**附加 Dart/Flutter 调试器**。
- **特点**：
  - 断点、单步进入/跳过、查看调用栈与局部变量、表达式求值等。
  - 同样是 **debug 构建**，支持热重载 / 热重启（与设备、项目配置有关）。
- **适用**：定位逻辑错误、状态不对、异步流程等需要逐步跟踪的场景。

---

## Profile（性能剖析）

- **做什么**：以 **profile 模式**运行应用（通常对应 `flutter run --profile`）。
- **特点**：
  - 介于 debug 与 release 之间：会做更多优化，性能比 debug 更接近真实用户环境，同时仍保留足够的性能剖析能力。
  - 适合配合 **Flutter DevTools**（帧时间线、CPU 采样、内存等）做性能分析。
  - **不适合**用「debug 模式下的性能」去判断上线表现；也不应依赖 profile 代替 release 做最终体验验收。
- **不适用**：指望它和 Debug 一样在所有调试场景里都顺手（例如部分调试体验与 debug 构建有差异）。

---

## 和 Release 的关系

这三种入口里，**默认都不会是 release 模式**。若要做「与线上最接近」的测试，需在终端或配置里使用 `flutter run --release` 或打 release 包安装验证；这和 CodeLens 上的 Debug/Run/Profile 是不同维度的问题。

---

## 如何选择

1. **改功能、跟流程** → 多用 **Run** 或 **Debug**（需要断点就用 Debug）。
2. **卡顿、掉帧、启动慢、内存偏高** → 用 **Profile** + DevTools。
3. **上线前最终体验与体积** → **Release** 构建或安装 release 包。

---

## 参考

- Flutter 构建模式：「调试 / 分析 / 发布」（debug / profile / release）的官方说明见 Flutter 文档中的 *Build modes*。
- 编辑器行为：由 **Dart** 扩展的 CodeLens 与 `launch.json` 中的 `flutterMode`、`noDebug` 等配置共同决定；若本机行为与上文不一致，请先检查 `.vscode/launch.json`。
