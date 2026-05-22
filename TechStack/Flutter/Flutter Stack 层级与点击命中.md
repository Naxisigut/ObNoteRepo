# Flutter Stack 层级与点击命中

## 核心结论

`Stack` **没有** CSS 式的 `z-index` 属性。子组件的：

- **绘制顺序**
- **点击命中顺序**（重叠区域谁接到事件）

都由 `children` **在列表中的顺序**决定：

| 位置 | 效果 |
|------|------|
| 靠前（index 小） | 先绘制，在**下层** |
| 靠后（index 大） | 后绘制，在**上层**，重叠处**优先接收点击** |

因此：需要可点击的浮层（顶栏、搜索条、按钮）时，应把它放在 `children` **最后**，或按层级排序后再构建。

## 常见误区

### 1. 以为留了空白就不会挡点击

`CustomScrollView` / `ListView` 即使用 `SliverToBoxAdapter(SizedBox(height: headerH))` 在顶部留白，滚动组件仍会铺满父级区域并参与命中测试。若 `Stack` 里列表在顶栏**之后**声明，顶栏区域的点击会被列表抢走。

**本项目案例**：`lib/pages/store/store_page.dart` 门店页顶栏搜索框点不动，即因 `RefreshIndicator` + `CustomScrollView` 叠在 `_StoreTopBar` 之上。修复方式：顶栏 `Positioned` 放到 `Stack.children` 末尾。

### 2. 以为 `Material(elevation: …)` 能改 Stack 内层级

`elevation` 主要影响 Material 阴影及部分绘制，**不能**用来决定与普通 `Stack` 兄弟之间的谁在上、谁接点击。

### 3. 以为可以给 Stack 子组件写 `zIndex`

标准 `Stack` 子 widget **没有** `zIndex` 参数。要「指明层级」只能通过**调整 children 顺序**（或构建前按数字排序）。

## 推荐做法

### 做法 A：直接调整 `children` 顺序（最简单）

```dart
Stack(
  children: [
    // 底层：可滚动正文
    RefreshIndicator(
      child: CustomScrollView(
        slivers: [
          SliverToBoxAdapter(child: SizedBox(height: headerH)),
          // ...
        ],
      ),
    ),
    // 顶层：固定顶栏（可点击）
    Positioned(
      left: 0,
      right: 0,
      top: 0,
      child: topBar,
    ),
  ],
)
```

### 做法 B：用 `zOrder` 排序后再 `build`（逻辑更清晰）

```dart
final layers = <({int z, Widget child})>[
  (z: 0, child: scrollBody),
  (z: 1, child: topBar),
]..sort((a, b) => a.z.compareTo(b.z));

Stack(children: layers.map((e) => e.child).toList());
```

适合层数多、顺序会随状态变化的页面。

### 做法 C：嵌套 Stack 分层

```dart
Stack(
  children: [
    scrollLayer,
    Stack(children: [floatingChrome]), // 整层浮在正文上
  ],
)
```

适合「底图一层 + 浮层一层」的固定结构。

### 做法 D：全局浮层用 `Overlay`

Dialog、全屏 Loading、SnackBar 等跨页浮层，用 `Overlay.of(context).insert(...)`，而不是和业务页 `Stack` 抢顺序。

## 与点击区域相关的补充

1. **`HitTestBehavior`**：透明区域也要响应点击时，给 `GestureDetector` 设 `behavior: HitTestBehavior.opaque`（如 `SearchRow` 拉满宽度时）。
2. **`IgnorePointer` / `AbsorbPointer`**：显式让下层不参与命中，或吸收事件（慎用，易误伤子组件）。
3. **`Positioned.fill` + 透明层**：若必须在下层挡事件，可在上层铺透明 `Listener`/`GestureDetector`（一般不如直接调整 Stack 顺序清晰）。

## 调试清单

重叠区域点击无反应时，按顺序检查：

1. `Stack.children` 里，可点击 widget 是否在**更靠后**的位置？
2. 下层是否是 `ScrollView` / `ListView` 等铺满全屏的组件？
3. 上层是否 `IgnorePointer(ignoring: true)`？
4. 可点击区域是否实际有尺寸（`width: double.infinity`、`behavior: opaque` 等）？

## 相关文件

- 门店页修复示例：`lib/pages/store/store_page.dart`
- 可复用搜索条：`lib/shared/widgets/search_row.dart`
- 带返回的顶栏搜索：`lib/shared/widgets/custom_nav_scaffold.dart`（`SearchRow` 放在 `Positioned(left/right)` 内，宽度由 Positioned 约束）
