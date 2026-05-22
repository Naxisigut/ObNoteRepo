# GetX 与依赖注入（结合本项目）

面向熟悉前端、初次接触 GetX / Dart 的读者，汇总：**`Get.find` 在干什么**、**同类型多实例会怎样**、**依赖注入是什么**、**为什么要这样设计**、**为什么要按类型查找**。

---

## 1. 示例：`IndexPageService(Get.find<ApiClient>())`

```dart
final IndexPageService _pageService = IndexPageService(Get.find<ApiClient>());
```

含义拆开看：

1. **`final ... _pageService`**  
   成员变量（常见于 `State` 中）只初始化一次，类型为 `IndexPageService`，负责首页相关的接口与数据转换等。

2. **`IndexPageService(...)`**  
   构造函数需要 **`ApiClient`**（项目里封装 HTTP 的一层），必须把实例传入。

3. **`Get.find<ApiClient>()`**  
   GetX 在应用启动时把依赖登记到全局容器（本项目在 `InitialBinding` 里 `Get.lazyPut(() => ApiClient(...))`）。  
   **`Get.find<ApiClient>()`** = 按类型 **`ApiClient`** 从容器取出**已注册**的实例（`lazyPut` 时第一次 `find` 会触发创建）。

**一句话：** 首页 Service 不自己 `new` 网络层，而是用全 App 共享、已在绑定里配置好的那一个 `ApiClient`。

### 与直接 `new ApiClient()` 的差别

- 自己 `new`：可能每个页面一套 Dio/拦截器，Token、基址、行为不一致。  
- `Get.find<ApiClient>()`：与 `InitialBinding` 中的顺序、拦截器、登录态一致，**单例语义由注册方式保证**。

### GetX 常用 API 速记

| API | 作用 |
|-----|------|
| `Get.put` | 注册并返回实例 |
| `Get.lazyPut` | 延迟到第一次 `find` 再创建（本项目 `ApiClient` 等） |
| `Get.find<T>()` | 从容器取出类型 `T` 的注册实例 |

---

## 2. GetX 里有两个「同一种类型」的服务会怎样？

GetX 默认以**类型**作为查找键：`Get.find<ApiClient>()` 表示「要一个 `ApiClient`」。

- **只注册一次**（如 `InitialBinding` 里仅一处 `ApiClient`）→ `find` 始终拿到**唯一**实例。  
- **未经设计地注册两次同类型**（且不用 `tag`）→ 行为依赖写法/版本，常见为**后者覆盖前者**或容器只保留一个实现，容易埋坑，**应避免**。  
- **确实需要两个同类型实例**（例如两个不同 `baseUrl` 的 client）→ 使用 **`tag`**：  
  `Get.put(ApiClient(...), tag: 'a')` / `Get.find<ApiClient>(tag: 'a')`，用「类型 + 标签」区分。

本项目架构意图是**全局一个 `ApiClient`**，各 `XxxPageService` 通过 `Get.find<ApiClient>()` 共用。

---

## 3. 依赖注入（DI）是什么？——用前端类比

**依赖**：`IndexPageService` 要能发 HTTP，它**依赖**「会发请求的对象」（此处即 `ApiClient`）。

**注入**：不在 `IndexPageService` 内部写死 `new ApiClient()`，而是由**外部**把已配置好的 `ApiClient` **通过构造函数参数传进来**。这就叫**依赖注入**。

| 前端常见做法 | 类似关系 |
|--------------|----------|
| React：`Context` / `Provider` 提供 `apiClient`，子组件 `useContext` | 顶层注册一次，子处取用 |
| Vue：`provide` / `inject` | 同上 |
| GetX：`InitialBinding` 里 `lazyPut`，各处 `Get.find<T>()` | **Service Locator** 风格的 DI：先注册，再按类型解析 |

**常见收益：**

- **单一配置好的客户端**：拦截器、Token、BaseUrl 只装配一处（如 `AppDio` → `ApiClient`）。  
- **便于替换实现**：测试 Mock、换 Mock 服务时改注册处，而不必改每个 Service 内部的 `new`。  
- **依赖显式**：构造函数需要 `ApiClient`，阅读成本低，不隐藏在类内部的单例里。

---

## 4. 为什么要「做得这么复杂」？

并非移动端独有：只要存在**带状态、需全局共享的底层依赖**（尤其是带 Token 的 HTTP），**集中创建、统一注入**通常比每个页面各自 `new` 更稳。

在本项目中，「步骤感」来自：

1. **启动顺序**：`AuthTokenStore` → `AuthLoginService` → `AuthSessionManager` → `AppDio` → `ApiClient`，适合在 **`InitialBinding` 中按序注册一次**。  
2. **多页面复用**：每个 `XxxPageService` 若各自 `ApiClient()`，拦截器与登录态难以一致。

可理解为：在 **`main` / 根绑定** 阶段往「类型 → 实例」表里写入依赖；各 Service **`find`** 取出同一套配置。

---

## 5. 为什么要「通过类型」查找服务？

- Dart **强类型**：`Get.find<ApiClient>()` 类型写错时，静态检查更易发现问题。  
- 相对**字符串键**（如 `'ApiClient'`），**重构改名**更不容易漏。  
- **默认一个类型对应一个实现**时 API 最简单；需要多实例时再用 **`tag`** 扩展。

---

## 6. 极简对照（给前端同学）

| 概念 | 可类比 |
|------|--------|
| `InitialBinding` 注册 | 根节点或应用启动时创建单例 / 挂到 Context |
| `Get.find<ApiClient>()` | `useContext` / `inject` 取依赖 |
| `IndexPageService(apiClient)` | 参数传入 `api`，而不是在类内耦合全局实现 |

---

## 7. 与本项目文件的对应关系

- 全局注册：`lib/app/bindings/initial_binding.dart`  
- 应用入口调用绑定：`lib/main.dart` 中 `InitialBinding().dependencies()`  
- 页面侧使用示例：`IndexPageService(Get.find<ApiClient>())`（见各 `*_page_service.dart` 或页面 `State`）

更上层的模块划分见 `docs/architecture/architecture-overview.md`。
