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
