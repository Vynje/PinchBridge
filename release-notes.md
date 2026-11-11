# PinchBridge 3.0.0

Magic Mouse 双指捏合 → macOS 原生无级缩放 + 全局 ⌘Q 防误退 + 双层手掌防误触。

## 功能

- **双指捏合无级缩放**：预览、Safari、照片等支持原生捏合的应用里丝滑无级缩放（One Euro 自适应滤波；平移式滚动/横扫经质心消歧，不误触发缩放）
- **⌘Q 保护**：全局拦截防误退，可临时关闭（状态持久化）
- **手掌防误触（双层，实验性）**：掌锚点（长驻静止触点自动剔除）+ 校准式排除带（向导 2 步自然手势、手动推进，自动测出你的掌区，支持重新校准）
- **手势绑定框架（实验性，默认无绑定）**：gesture.json 自定义手势→动作（进阶用户）

## 安装要求

- Apple Silicon、macOS 14+、Magic Mouse
- 未公证：首次打开**右键 → 打开**，或终端执行 `xattr -d com.apple.quarantine /Applications/PinchBridge.app`
- 首次启动按向导授予「辅助功能 + 输入监控」两项权限；**同证书升级自动保留授权**

## 校验

```
shasum -a 256 -c PinchBridge.dmg.sha256
```

## 隐私

零联网、零遥测、全本地。详见 [PRIVACY.md](https://github.com/Vynje/PinchBridge/blob/main/PRIVACY.md)。

## 致谢

[ishora](https://github.com/YRDEV1/ishora)、[MagicMouseZoomBridge](https://github.com/EnesSU/MagicMouseZoomBridge)（MIT，技术路线参考）

## 许可证与官方来源

免费使用；版权与再分发条款见 [LICENSE.md](https://github.com/Vynje/PinchBridge/blob/main/LICENSE.md)，官方安装包经本仓库 [Releases 页](https://github.com/Vynje/PinchBridge/releases/latest) 发布。

# PinchBridge 3.0.1

v3.0.0 的纯修复版：根治启动后空闲时吃满一个 CPU 核的耗电问题（macOS「耗电过多」提示的直接原因）。不含新功能。

## 核心修复：空闲 CPU 由约 96–99.6% 降到约 0%

- **现象**：启动后即使不做任何操作，PinchBridge 也会持续占满一个 CPU 核（空闲时进程 CPU 实测 96.2–99.6%，单核口径），macOS 因此在电池菜单把它列为「耗电过多」。
- **根因**：负责采集 Magic Mouse 触面数据的专用线程，它的 run loop 上没有挂任何事件源，等待函数 `CFRunLoopRun()` 立即返回，驻留循环退化成不停空转的忙等。
- **修复**：给这条线程的 run loop 挂上一个「停机门铃」事件源，空闲时真正阻塞在内核等消息、不再空转；退出序列同步改为「门铃信号 + 唤醒 + 停止」三重保证，各种退出时序下都能确定退出。触面设备挂接、手势回调注册的调用顺序与语义零改动。
- **实测**：修复验证包空闲 CPU 降到 0.0–0.1%（3 次启动共 12 个采样拍）；装机版同一进程启动后 13–24 秒与约 2 分钟两段共 12 个采样拍全部为 0.0%。帧线程采样由「修复前 12 秒 9729/9729 个样本全部在用户态忙等」变为「修复后全部阻塞在内核等待」（修复验证包 12 秒 9107/9107、装机版 10 秒 7602/7602）。
- **说明**：以上为无人触摸的空闲态读数。升级后系统电池菜单里的「耗电过多」提示可能不会立即消失（它可能按近期用量累计，旧记录不一定马上消退）；以活动监视器里 PinchBridge 的空闲 %CPU 接近 0 为准。

## 其他优化

- **授权状态轮询退避**：内部状态对账轮询在稳定态（双授权通过、⌘Q 就绪、触面已挂接）由每秒一次降为每 5 秒一次；任一条件变化立即恢复每秒一次，打开菜单或设置窗时仍即时刷新。个别仅靠轮询镜像的状态（如登录项开关回显）延迟最长约 6 秒。
- **缩放事件构造减半**：缩放事件投递不再额外构造探针事件（此前每次缩放要多构造一个只用来读光标位置的探针事件），单次事件构造成本实测由约 300ns 降到约 155ns。
- **触点预览节流改用单调时钟**：设置窗实时触点预览的 30Hz 节流计时不再依赖墙钟时间，系统对时或手动改时间不再造成预览连发或冻结。

## 升级说明

- 从 3.0.0 升级：下载并打开 DMG，把 PinchBridge.app 拖入「应用程序」文件夹覆盖旧版即可。
- **授权自动保持**：本版本与 3.0.0 的签名身份相同（证书签名锚未变），覆盖安装后「辅助功能 + 输入监控」两项系统授权自动保留，无需重新授权（装机实测：覆盖安装后首次启动即通过双授权预检，未弹授权引导）。万一系统弹出授权提示，按提示重新授权即可恢复。
- **本版本不含新功能**，是纯修复版本。

## 安装要求

- Apple Silicon、macOS 14+、Magic Mouse
- 未公证：首次打开**右键 → 打开**，或终端执行 `xattr -d com.apple.quarantine /Applications/PinchBridge.app`
- 首次启动按向导授予「辅助功能 + 输入监控」两项权限；**同证书升级自动保留授权**

## 校验

```
shasum -a 256 -c PinchBridge.dmg.sha256
```

## 隐私

零联网、零遥测、全本地。详见 [PRIVACY.md](https://github.com/Vynje/PinchBridge/blob/main/PRIVACY.md)。

## 致谢

[ishora](https://github.com/YRDEV1/ishora)、[MagicMouseZoomBridge](https://github.com/EnesSU/MagicMouseZoomBridge)（MIT，技术路线参考）

## 许可证与官方来源

免费使用；版权与再分发条款见 [LICENSE.md](https://github.com/Vynje/PinchBridge/blob/main/LICENSE.md)，官方安装包经本仓库 [Releases 页](https://github.com/Vynje/PinchBridge/releases/latest) 发布。
