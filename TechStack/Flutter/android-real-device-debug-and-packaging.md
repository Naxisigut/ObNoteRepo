# Flutter 安卓真机调试与打包速查

本文整理了开发阶段使用安卓真机调试，以及打包安卓安装产物（APK/AAB）的常见流程与判断标准。

## 1. 安卓真机调试

### 1.1 基础准备（Windows）
- 手机开启 `开发者选项` 与 `USB 调试`
- 电脑安装好 Flutter、Android Studio（含 SDK / platform-tools）
- 安装手机厂商 USB 驱动（如小米/华为/三星等）

先执行检查：

```bash
flutter doctor -v
flutter devices
adb devices
```

### 1.2 多设备时运行到指定真机
当连接了多台设备（真机 + 模拟器）时，使用设备 ID 指定运行目标：

```bash
flutter devices
flutter run -d <deviceId>
```

示例：

```bash
flutter run -d du8l8h9prc7prg4p
```

说明：
- `emulator-xxxx` 通常是模拟器
- 其他序列号大多是真机

### 1.3 真机首次运行耗时
常见耗时范围：
- 首次运行（需下载依赖/安装 SDK）：`3~15 分钟`，网络慢时更久
- 后续运行（缓存已就绪）：通常 `20 秒~2 分钟`
- 热重载：`1~5 秒`
- 热重启：`5~20 秒`

若 `assembleDebug` 长时间无输出（如 15~20 分钟以上）才需要重点排查网络、SDK 和 Gradle 状态。

## 2. 打包安卓产物

### 2.1 常用命令

Debug 包（开发测试）：

```bash
flutter build apk --debug
```

Release APK（按 ABI 拆分）：

```bash
flutter build apk --release --split-per-abi
```

Release AAB（上架推荐）：

```bash
flutter build appbundle --release
```

### 2.2 是否必须签名
- `flutter run` / `flutter build apk --debug`：通常无需手动配置发布签名（会使用 debug 签名）
- `release` 构建：建议配置正式 keystore；上架商店必须使用正式签名
- 仅用于本地查看 release 体积时，很多项目默认也可先构建成功（取决于 `build.gradle` 配置）

## 3. 包体积判断

### 3.1 Debug 包 200MB 是否正常
通常正常。原因包括：
- Debug 构建包含调试信息
- 可能包含多 ABI（fat APK）
- Flutter 引擎与运行时本身占一定体积

### 3.2 更接近真实发布体积的看法
建议看 release 产物，尤其是拆分 ABI 或 AAB：

```bash
flutter build apk --release --split-per-abi --analyze-size
flutter build appbundle --release --analyze-size
```

## 4. 为什么会生成多个 APK

当使用 `--split-per-abi` 时，按 CPU 架构拆分会生成多个 APK（这是预期行为）：
- `armeabi-v7a`：32 位 ARM 设备
- `arm64-v8a`：64 位 ARM 设备（多数真机）
- `x86_64`：常见于模拟器

优点：每台设备仅安装对应架构的包，体积更小。

## 5. 常见选择建议
- 日常联调：`flutter run` 或 `flutter build apk --debug`
- 提供接近线上测试：`flutter build apk --release`（或 split 版本）
- 上架发布：`flutter build appbundle --release` + 正式签名

