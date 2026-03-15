---
name: nano-pdf
description: 使用 nano-pdf CLI 通过自然语言指令编辑 PDF 文件。
homepage: https://pypi.org/project/nano-pdf/
metadata: {"clawdbot":{"emoji":"📄","requires":{"bins":["nano-pdf"]},"install":[{"id":"uv","kind":"uv","package":"nano-pdf","bins":["nano-pdf"],"label":"Install nano-pdf (uv)"}]}}
---

# nano-pdf

使用 `nano-pdf` 通过自然语言指令对 PDF 的特定页面进行编辑。

## 快速开始

```bash
nano-pdf edit deck.pdf 1 "Change the title to 'Q3 Results' and fix the typo in the subtitle"
```

注意事项：
- 页码根据工具版本/配置的不同，可能使用 0-based 或 1-based；如果结果看起来偏移了一页，请使用另一种方式重试。
- 在发送输出 PDF 之前，始终要对其进行合理性检查。
