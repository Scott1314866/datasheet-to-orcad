# Datasheet to OrCAD Skill

将芯片 datasheet PDF 自动转换为 OrCAD Capture 符号库（.olb）和原理图（.dsn）。

## 功能

- 输入：任意芯片 datasheet PDF
- 输出：`.olb`（符号库）+ `.dsn`（原理图），可直接在 OrCAD Capture 中打开
- 5 阶段流水线：PDF 定位 → 双通道提取（MinerU 文本 + DeepSeek 视觉）→ 交叉校验 → 独立复核 → TCL 生成
- 每个引脚带证据（页码 + 来源通道），冲突不猜测、交人工裁决

## 安装

```bash
# 克隆到你的 Qoder skills 目录
cd ~/.qoder-cn/skills
git clone https://github.com/Scott1314866/datasheet-to-orcad.git
```

重启 Qoder 会话，或运行 `/skills reload`。

## 使用

直接告诉 Qoder：

> "帮我把这个 datasheet 转成 OrCAD 符号：`C:/path/to/your-datasheet.pdf`"

或者：

> "用 Module_01 生成 STM32F105 的符号库"

Qoder 会自动识别并加载这个 skill 的上下文。

## 前置条件

| 依赖 | 说明 |
|------|------|
| Python | conda 环境 `module01`（Python 3.12 + pymupdf + requests） |
| Cadence SPB 17.2 | `C:\Cadence\SPB_17.2\tools\bin\tclsh.exe` |
| MinerU API Key | 项目根目录 `MinerU_API_KEY.md` |
| DeepSeek API Key | 环境变量 `DEEPSEEK_API_KEY` 或 Windows 注册表 |

## 项目位置

流水线代码在 `D:\WorkSpace\Claude\Module_01`，这个 skill 只是使用指南，不包含源码。
