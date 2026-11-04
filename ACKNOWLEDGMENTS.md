# 致谢与第三方许可（Acknowledgments & Third-Party Licenses）

PinchBridge 的实现建立在以下两个 MIT 开源项目的成果之上。本应用以闭源
二进制形式分发，其中包含改编自这两个项目的代码；依据 MIT 许可条款，在此
附上原始项目链接、版权声明与许可全文。

## 1. ishora

- 仓库：<https://github.com/YRDEV1/ishora>
- 许可：MIT License
- **Technique 来源**：MultitouchSupport 私有框架的触面读取实现
  （MTDeviceCreateList / MTRegisterContactFrameCallback / MTDeviceStart /
  MTDeviceGetFamilyID / MTDeviceIsBuiltIn 垫片）；Magic Mouse 设备家族号
  判据（112/113，上游注明来自社区逆向）与双指质心位移横扫消歧思路。

```text
MIT License

Copyright (c) 2026 YRDEV1

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## 2. MagicMouseZoomBridge

- 仓库：<https://github.com/EnesSU/MagicMouseZoomBridge>
- 许可：MIT License
- **Technique 来源**：手势 CGEvent 私有字段定义（手势事件 type 29、字段
  110 = IOHID zoom、字段 113 = 缩放幅度、字段 132 = 相位位掩码）；事件
  构造与 `CGEventPost` 投递写法，包括相邻手势事件时间戳必须严格递增、
  否则窗口服务器会把连续缩放合并成一步的关键发现。

```text
MIT License

Copyright (c) 2026 Enes Akkuş

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

以上许可文本逐字抄录自两个上游仓库的 `LICENSE` 文件。在此感谢原作者的
开源工作。
