# PinchBridge

把 **Magic Mouse 双指捏合** 翻译成 **macOS 原生缩放手势事件**，并顺手提供
**全局 ⌘Q 防误触** 的菜单栏小工具。

Magic Mouse（一代/二代）硬件不产生捏合手势，系统层收不到 magnification
事件，所以 Preview、Safari、照片等应用里没法用捏合缩放——PinchBridge 在
触面层读双指触点、算间距变化，合成系统级缩放手势，让这些应用照常缩放。
同时它常驻拦截纯 ⌘Q 键击，防止手滑把未保存的文档直接退出。

> **状态透明**：本项目仅提供 arm64 构建，且**未经 Apple 公证**。首次打开
> 需要右键 → 打开（见下方[安装](#安装)），介意请勿使用。

---

## 功能

### ① Magic Mouse 双指捏合 → 原生缩放

- **只读触面、只发事件**：经 MultitouchSupport 框架只读触点帧，输出端只
  投递手势事件，不拦截、不吞掉任何输入。
- **默认只挂 Magic Mouse**：内建触控板 / Magic Trackpad 的原生捏合不受
  影响（不会被重复翻译成双重缩放）。
- **捏合与横扫不打架**：双指横扫（滚动）靠质心位移消歧被压制，不会误触
  发缩放；真捏合是双指围绕质心对称聚拢/张开，天然区分。
- **丝滑无台阶**：1€ Filter 滤波 + 逐帧连续发射（Casiez 2012 原式），
  没有死区积攒后一次跳一格的顿挫；单事件增量带钳制安全阀，异常帧不会发
  出巨幅缩放。
- **瞬时失触不断线**：捏合中触点瞬时丢失有桥接窗口兜底，Mouse 单指瞬间
  失触不会把缩放会话掐断。

### ② 全局 ⌘Q 防误触

- 拦截**纯 ⌘Q**（不含 Shift/Ctrl/Option/Fn 修饰）的按下与抬起，防止手滑
  退出正在编辑的应用。
- ⌘S、⌘W、⌘⇧Q（退出所有窗口）等其余组合**全部放行**；菜单栏/Dock 里的
  鼠标退出操作不走键盘事件流，同样不受影响。
- 内置看门狗：事件 tap 被系统超时禁用时自动重新启用；连续收不到物理键盘
  事件会打告警，不会静默失效。

---

## 系统要求

| 项目 | 要求 |
|---|---|
| 机型 | Apple Silicon（**仅 arm64 构建**，Intel Mac 不可用） |
| 系统 | macOS 14.0 或更高 |
| 鼠标 | Magic Mouse（一代 / 二代） |
| 权限 | 辅助功能 + 输入监控（首次启动向导会引导，见下） |

诚实声明：

- **未经 Apple 公证（notarization）**：首次打开会被 Gatekeeper 拦。见
  下方安装步骤的解除方式。
- **识别判据来自社区逆向**：Magic Mouse 一代/二代的设备家族号判据
  （112/113）来自上游开源项目的社区逆向成果；Magic Mouse 3 等其他版本
  **未验证**。

---

## 安装

### 第一步：DMG 拖拽安装

1. 从 Releases 页下载最新 `.dmg`，打开后把 **PinchBridge** 拖入
   「应用程序」文件夹；
2. 因未公证，首次打开任选其一：
   - 在「应用程序」里**右键点 PinchBridge → 打开**，再点一次「打开」；
   - 或终端执行
     `xattr -d com.apple.quarantine /Applications/PinchBridge.app`。

### 第二步：首次启动 → 授权向导

捏合发事件需要「**辅助功能**」授权，⌘Q 拦截收物理键盘事件需要「**输入
监控**」授权。App 首次启动自带**授权向导**：

1. 向导里有两个按钮，分别**一键直达** 系统设置 → 隐私与安全性 →
   「辅助功能」/「输入监控」面板；
2. 在对应面板点 **＋** 添加（或直接拖入）`应用程序` 里的
   **PinchBridge**；
3. 回到向导，它**实时检测**两项授权状态；两授权全绿后**自动激活**保护，
   **无需手动重启** App。

之后每次开机 App 随登录自启，菜单栏出现状态图标即在工作。

---

## 使用说明

### 菜单栏

点击菜单栏图标：

- **状态区**：显示辅助功能 / 输入监控两项授权是否就绪；密码框等安全输入
  聚焦期间显示「⌘Q 保护暂停」提示（见已知边界第 1 条）；
- **设置…**：打开设置面板（下方七参数）；
- **退出**：结束 PinchBridge。

### 设置面板：七参数

所有手感参数集中在设置面板，一一对应检测/滤波链路。**改参数不需要重新
授权、不需要重装**（授权绑定的是 App 本体，参数只是配置）；保存后 App
以新参数重启捏合引擎即可生效。

| 参数 | 默认值 | 允许范围 | 含义 | 手感方向 |
|---|---|---|---|---|
| `minCutoff` | 0.9 | 0.01 – 20 | 1€ 滤波最低截止频率（Hz），静止时的抑噪底线 | 越小越稳、越迟钝；越大越跟手、越易抖 |
| `beta` | 6.0 | 0 – 50 | 速度增益：滤波截止随捏合速度抬升的强度，动态跟手 | 越大快捏越跟手；过大动态噪声也放大 |
| `dCutoff` | 6.0 | 0.05 – 20 | 速度估计自身的低通截止（Hz），速度信号平滑度 | 越大阶跃起压越快；越小越平滑迟缓 |
| `entryRatio` | 1.018 | 1.0001 – 1.5 | 入场阈值（**张开方向**比值；捏拢方向自动取其倒数 ≈ 0.9823） | 越大越难触发、越防误触；越小越灵敏 |
| `confirmFrames` | 3 | 1 – 10（整数） | 入场确认：连续 N 帧滤波增量同号才算捏合开始 | 越大越防误触、起步越迟；越小越快 |
| `bridgeMs` | 100 | 0 – 1000 | 桥接窗口（毫秒）：捏合中触点数 ≠2 的容忍时长，同指回归则续会话；**0 = 禁用桥接**（立即结束） | 越大越能扛瞬时失触；越大误连悬会话的风险也越高 |
| `maxDeltaPerEvent` | 0.08 | 0.001 – 1 | 单个事件的 ln 增量钳制上限（安全阀，异常帧不会发出巨幅缩放） | 正常链路不应触发；是保护上限不是手感参数 |

⚠️ 注意 `entryRatio` 的方向：**始终填 > 1 的张开方向值**，捏拢触发阈值是
程序自动取的倒数。想收紧捏拢触发请把 `entryRatio` 改**大**；填 0.98 这类
小于 1 的值不会生效（会回落默认）。

默认值（beta=6.0 / dCutoff=6.0）由确定性手感契约闸门实测校准：静态噪声
抑制 ≥4×、真实阶跃 63% 响应 ≤5 帧、慢捏轨迹零反向帧、异常间隙零误发。
出厂默认已经是稳/跟手的均衡点，建议只微调单项观察。

---

## 已知边界（诚实清单）

1. **Secure Input 期 ⌘Q 保护暂停**：密码框等安全输入框聚焦时（系统激活
   Secure Event Input），macOS 不向第三方事件 tap 投递键盘事件，⌘Q 保护
   **必然失聪**——此期按 ⌘Q 会真的退出应用。这是苹果系统级限制，用户态
   无 root、无 DriverKit 资格就堵不死，本项目不做任何补救承诺；App 会以
   2Hz 轮询检测并把「保护暂停 / 保护恢复」显示在菜单栏状态里。捏合功能
   **不受影响**（只读触面、只发手势事件，不碰键盘流）。
2. **捏合与双指横扫并存**：平移式横扫被质心闸门压制；但**对称聚拢/张开
   式横扫**（横扫同时双指间距漂移超过入场阈值约 ±1.8%、质心基本不动）
   仍会进入缩放会话，且会话开始后不再回闸，会持续到抬指。
3. **私有 API 风险**：MultitouchSupport 框架接口与手势 CGEvent 字段均为
   未公开 SPI，macOS 大版本更新可能失效（失效形态：无声、无事件）。
4. **Magic Mouse 3 未验证**：设备识别依赖社区逆向的家族号（112 = 一代、
   113 = 二代）；其他/更新型号未验证，可能默认挂不上。
5. **大捏合到触面边缘**：手指被推到边缘后继续捏，质心被迫移动，可能被
   横扫闸门压制——抬指重新落指即可恢复。
6. **更新/重装后需重新授权**：授权按签名绑定，换新版本的二进制后若发现
   失效，去两个授权面板把旧条目删掉、重新添加一次即可（向导会再次引导）。

---

## 卸载（三步）

1. 菜单栏图标 → **退出** PinchBridge；
2. 「应用程序」里把 **PinchBridge** 拖到废纸篓；
3. 系统设置 → 隐私与安全性 → **辅助功能** 和 **输入监控** 两个列表里，
   分别把 PinchBridge 条目**手动移除**（卸载不会自动清掉授权记录）。

可选清理：用户资源库（`~/Library`）下 PinchBridge 的配置与日志目录。

---

## 致谢

本项目站在两个 MIT 开源项目的肩膀上：

- [ishora](https://github.com/YRDEV1/ishora)（MIT）——MultitouchSupport
  私有框架触面读取实现与 Magic Mouse 家族号判据、质心横扫消歧思路的
  Technique 来源；
- [MagicMouseZoomBridge](https://github.com/EnesSU/MagicMouseZoomBridge)
  （MIT）——手势 CGEvent 私有字段定义与事件构造写法（含相邻事件时间戳
  严格递增的关键发现）的 Technique 来源。

完整版权与 MIT 许可文本见 [ACKNOWLEDGMENTS.md](ACKNOWLEDGMENTS.md)
（闭源分发二进制的许可合规声明）。

## 隐私声明

**零联网、零遥测、全本地**：不发任何网络请求，不收集任何数据。授权用途
逐条说明见 [PRIVACY.md](PRIVACY.md)。

---

## English TL;DR

**PinchBridge** is a small menu bar app for Mac that does two things:

1. **Magic Mouse pinch → native macOS zoom.** The Magic Mouse hardware
   can't produce a pinch gesture, so apps like Preview / Safari never see
   one. PinchBridge reads two-finger contacts at the multitouch layer and
   synthesizes native magnification gesture events, with 1€-filter
   smoothing (no staircase jumps), swipe disambiguation, and a bridging
   window against momentary touch loss.
2. **Global ⌘Q guard.** It swallows plain ⌘Q key presses system-wide so
   you don't accidentally quit an app mid-edit. Every other shortcut
   (⌘S, ⌘W, ⌘⇧Q, …) passes through untouched.

**Requirements:** Apple Silicon (arm64-only build), macOS 14+, Magic
Mouse (Gen 1/2). **Not Apple-notarized** — on first launch, right-click
the app → Open, or run `xattr -d com.apple.quarantine
/Applications/PinchBridge.app`.

**Setup:** drag the DMG into Applications, launch once, and the built-in
wizard deep-links you to the Accessibility and Input Monitoring panes;
once both permissions turn green, protection activates automatically —
no restart needed.

**Tuning:** 7 parameters in the settings panel (1€ filter cutoffs, entry
threshold, confirm frames, bridge window, per-event clamp). Changing
them never invalidates your permissions.

**Honest limits:** during password fields (Secure Input) the ⌘Q guard is
paused — an Apple OS-level restriction; symmetric spread/squeeze while
side-swiping can still trigger zoom; uses private APIs that a future
macOS may break; Magic Mouse 3 unverified.

**Uninstall:** quit the app, delete it from Applications, and remove its
entries from the Accessibility and Input Monitoring lists in System
Settings.

**Privacy:** no network access, no telemetry, everything stays on your
machine. See [PRIVACY.md](PRIVACY.md).
