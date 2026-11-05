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
- **手掌防误触（v3.1 重构为「校准式」）**：掌压触点被并进双指对会造成
  "假捏合乱缩"——现在只按**触点位置**判定：靠掌缘划一条排除带，带内触点
  不计入捏合判定。带的方向与宽度不靠拍脑袋阈值：设置面板点「**手掌
  校准…**」，用你的两个自然手势（握持 3 秒 + 前端捏合 3 秒）实测测出，
  立即生效并保存；设置面板里还有**实时触点预览画布**，掌压触点标红 ✕、
  排除带画阴影，所见即引擎所滤，捏合失灵缩小带宽或重新校准即可。（旧版
  的接触尺寸闸已废弃：真机数据证实 MTTouch.size 不可靠——部分触点
  NaN/乱值、捏合按压时尺寸暴涨与掌缘重叠，可靠判据只剩位置。）

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
  下方安装步骤的解除方式。App 使用同一枚**自签代码签名证书**（PinchBridge
  Dev）签名：后续版本升级会**保留已授权状态**（辅助功能/输入监控授权无需
  重复授予），仅首次安装需要走一次授权向导。
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

运行期间菜单栏出现状态图标即在工作。想让**开机自动运行**：菜单栏图标 →
勾选「**开机自启**」（与系统设置 → 登录项实时同步，默认不勾选）。

---

## 使用说明

### 菜单栏

点击菜单栏图标：

- **状态区**（不可点）：捏合引擎状态（armed · 设备名 / 已暂停）、⌘Q 累计
  拦截次数、Secure Input 当前态（密码框聚焦期间显示「激活（⌘Q 保护暂停）」，
  见已知边界第 1 条）；
- **开关行**：捏合缩放 开/关（暂停检测，进程与 ⌘Q 保护不退出）、⌘Q 保护
  开/关、开机自启（与系统设置 → 登录项实时同步）；
- **授权引导…**（仅两项授权未齐时显示）：随时唤起授权向导窗；
- **设置…**（⌘,）：打开设置面板（见下）；
- **打开日志**：在访达中显示 `~/Library/Logs/PinchBridge.log`；
- **退出**：结束 PinchBridge。

### 设置面板

四组设置，全部**热生效**——改动立即经线程安全配置交换进检测队列，无需
重启 App、**不需要重新授权、不需要重装**（授权绑定的是 App 本体，参数只
是配置）；每组带「恢复默认」按钮。

**捏合灵敏度组**（四参）与 **1€ 滤波组**（三参）：

| 参数 | 默认值 | 面板范围 | 含义 | 手感方向 |
|---|---|---|---|---|
| `entryRatio` | 1.018 | 1.010 – 1.060 | 入场阈值（**张开方向**比值；捏拢方向自动取其倒数 ≈ 0.9823） | 越小越灵敏；越大越防误触 |
| `confirmFrames` | 3 | 2 – 5 | 入场确认：连续 N 帧滤波增量同号才算捏合开始 | 越大越防误触、起步越迟 |
| `bridgeMs` | 100 | 60 – 200 ms | 桥接窗口：捏合中触点瞬时丢失的容忍时长 | 越大越能扛瞬时失触 |
| `maxDeltaPerEvent` | 0.08 | 0.04 – 0.15 | 单个事件的 ln 增量钳制上限（安全阀） | 保护上限，非手感参数 |
| `minCutoff` | 0.9 | 0.3 – 3.0 Hz | 1€ 滤波最低截止，静止抑噪底线 | 越小越稳（迟钝），越大越跟随 |
| `beta` | 6.0 | 0.5 – 10.0 | 速度增益，动态跟手强度 | 越大快捏越跟手；过大放大动态噪声 |
| `dCutoff` | 6.0 | 0.5 – 2.0 Hz | 速度估计自身的低通截止 | 越大越跟随，越小越平滑 |

⚠️ `entryRatio` 始终是 > 1 的张开方向值，捏拢阈值是程序自动取的倒数；
想收紧捏拢触发请把它改**大**。

默认值（beta=6.0 / dCutoff=6.0）由确定性手感契约闸门实测校准：静态噪声
抑制 ≥4×、真实阶跃 63% 响应 ≤5 帧、慢捏轨迹零反向帧、异常间隙零误发。
出厂默认已是稳/跟手的均衡点，建议只微调单项观察。

**手掌防误触组**（防掌压被当成捏合；v3.1 校准式）：

- **手掌校准…（推荐先做这一步）**：点按钮进入向导，照大字提示做两个自然
  手势——第 1 步「像平时一样握住鼠标不动」3 秒采掌区样本，第 2 步「双指
  在鼠标前端做捏合动作」3 秒采指区样本——随后自动算出**掌区方向**（后缘
  靠手腕 / 前缘靠指尖）与**排除带边界**，立即生效并保存（结果面板显示
  方向 / 边界位置 / 带宽 / 两阶段样本帧数）。校准期间捏合检测暂停，校准
  手势不会误发缩放；某阶段样本不足（< 30 帧）会报「样本不足，请重试」
  ——手指贴住鼠标表面、别悬空即可；
- **排除带宽度滑杆**：0.15 – 0.45（占鼠标表面高度比例，从掌缘端起量）。
  校准后为实测值，也可手动微调：太小会漏掌，太大会吞捏合；
- **开关**：手掌防误触 开/关——关 = 不过滤任何触点（没有掌压误触问题时
  开销最小）；
- **实时触点预览画布**：触点实时映射到鼠标表面矩形，被排除带滤掉的触点
  标红 ✕、排除带画阴影、方向实时标注——**预览 = 引擎实时判定**，亲眼
  看着自己的掌压触点验证校准效果；
- **失效排障（重新校准）**：捏合失灵（真手指被吞，带太宽）→ 缩小带宽或
  「重新校准」；掌压仍乱缩（带没盖住掌区，带太窄或方向不对）→ 加大带宽
  或「重新校准」；换手、换握姿、换鼠标之后掌区位置变了——重新跑一次
  校准即可，无需重装或重新授权；
- 「恢复出厂」= 排除带 30% + 后缘（未校准时的保守默认）；
- 旧版的四级档位与尺寸闸已移除：MTTouch.size 判死（部分触点 NaN/乱值、
  捏合按压时尺寸暴涨与掌缘重叠），判据只剩触点位置 y。旧 tuning.json 里
  残留的 palmSizeGate / palmRatioGate / palmEntryPairCap 字段读入忽略，
  不影响运行。

底部「打开追踪（PINCH_TRACE）」开关（**默认关闭**）：开启后逐帧把触点
间距、接触尺寸、触点在鼠标表面上的位置（手掌排除带调参用）与滤波器状态
写入本机 CSV（仅本机排障用、不上传，隐私说明见
[PRIVACY.md](PRIVACY.md)），会话结束或空闲时生效。

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

1. 菜单栏图标 → **退出** PinchBridge（若开过「开机自启」，先在菜单栏
   关掉，或到 系统设置 → 登录项与扩展 移除 PinchBridge）；
2. 「应用程序」里把 **PinchBridge** 拖到废纸篓；
3. 系统设置 → 隐私与安全性 → **辅助功能** 和 **输入监控** 两个列表里，
   分别把 PinchBridge 条目**手动移除**（卸载不会自动清掉授权记录）。

可选清理（配置与日志的实际位置，确认不再使用后可整目录删除）：

- 配置：`~/Library/Application Support/PinchBridge/tuning.json`（更早版本在
  `~/Library/Application Support/pinch-bridge/`）；
- 日志：`~/Library/Logs/PinchBridge.log`；诊断追踪开启期间生成的逐帧 CSV
  也在 `~/Library/Logs/` 下。

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
   smoothing (no staircase jumps), swipe disambiguation, a bridging
   window against momentary touch loss, and a **palm-rejection layer**
   (a 2-gesture calibration wizard that measures your palm zone, a width
   slider, and a live touch preview that shows exactly what the engine
   filters) so a resting palm never fakes a pinch.
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

**Tuning:** the settings panel edits everything live (no restart, never
invalidates your permissions): pinch sensitivity + 1€ filter parameters,
a calibratable palm-rejection zone (2-gesture wizard + width slider +
live preview), pinch / ⌘Q guard / launch-at-login toggles in the menu
bar, and an optional off-by-default frame trace (local CSV only).

**Honest limits:** during password fields (Secure Input) the ⌘Q guard is
paused — an Apple OS-level restriction; symmetric spread/squeeze while
side-swiping can still trigger zoom; uses private APIs that a future
macOS may break; Magic Mouse 3 unverified.

**Uninstall:** quit the app, delete it from Applications, and remove its
entries from the Accessibility and Input Monitoring lists in System
Settings.

**Privacy:** no network access, no telemetry, everything stays on your
machine. See [PRIVACY.md](PRIVACY.md).
