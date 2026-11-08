# PinchBridge 3.0.0

Magic Mouse 双指捏合 → macOS 原生无级缩放 + 全局 ⌘Q 防误退 + 双层手掌防误触。

## 功能

- **双指捏合无极缩放**：任意应用丝滑无级缩放（One Euro 自适应滤波，滚动/猛滑零干扰）
- **⌘Q 保护**：全局拦截防误退，可临时关闭（状态持久化）
- **手掌防误触（双层，实验性）**：掌锚点（长驻静止触点自动剔除）+ 校准式排除带（向导 3 步自动测出你的掌区，支持重新校准）
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

零联网、零遥测、全本地。详见 PRIVACY.md。

## 致谢

[ishora](https://github.com/YRDEV1/ishora)、[MagicMouseZoomBridge](https://github.com/EnesSU/MagicMouseZoomBridge)（MIT，技术路线参考）

## 许可证与官方来源

- **版权所有 © 2026 Vynje** — 本软件免费供个人使用
- **禁止倒卖**：严禁出售、收费打包、捆绑转售；再分发须保留出处声明并链接官方仓库
- **唯一官方仓库**：https://github.com/Vynje/PinchBridge
- **完整许可证**：[LICENSE.md](https://github.com/Vynje/PinchBridge/blob/main/LICENSE.md)
- **上游致谢**：[ishora](https://github.com/YRDEV1/ishora) 与 [MagicMouseZoomBridge](https://github.com/EnesSU/MagicMouseZoomBridge)（MIT）
