---
name: web-to-pdf
version: 1.1.0
description: "网页/博客/文档 URL 转 A4 打印 PDF。当用户需要将网页、博客文章、在线文档转换为可打印的 PDF 文件时使用。支持自动去除侧边栏/导航/广告，输出适合 A4 纸打印的干净 PDF。"
metadata:
  requires:
    bins: ["percollate", "marked"]
---

# web-to-pdf

将网页转换为适合 A4 纸打印的 PDF，供纸质阅读和手写标注使用。

## 工具文件位置

```
~/.claude/skills/web-to-pdf/scripts/
  template.html     # 自定义模板（页码 X/Y，Source URL 中文解码）
  print-clean.css   # 打印样式（A4 / 9pt / table fixed / hyphens:none / pre left-align）
  config.json       # 站点配置（按需扩展）
```

## 执行命令

**网页 URL：**
```bash
PUPPETEER_EXECUTABLE_PATH="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  percollate pdf \
  --template ~/.claude/skills/web-to-pdf/scripts/template.html \
  --style ~/.claude/skills/web-to-pdf/scripts/print-clean.css \
  --output <输出路径.pdf> \
  "<URL>"
```

**本地 Markdown 文件（.md）：**
```bash
MD="<文件路径>"
(echo '<!DOCTYPE html><html><head><meta charset="utf-8"></head><body>'; \
 marked "$MD"; \
 echo '</body></html>') | \
PUPPETEER_EXECUTABLE_PATH="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  percollate pdf - \
  -u "file://$(dirname "$MD")/" \
  --template ~/.claude/skills/web-to-pdf/scripts/template.html \
  --style ~/.claude/skills/web-to-pdf/scripts/print-clean.css \
  --output <输出路径.pdf>
```

## 参数说明

- 输入为 `.md` 文件路径时，走 marked → percollate stdin 链路（需包 charset wrapper 防中文乱码）
- 输入为 URL 时，走 percollate 直接抓取链路
- `--output`：PDF 输出路径，默认用文件名/页面标题命名，放在文件同目录
- 多个 URL 可以合并为一个 PDF，直接在末尾追加即可

## 输出文件命名规则

- 用户未指定输出路径时，从页面 `<title>` 或文件名提取，去除特殊字符，加 `.pdf` 后缀
- 默认输出到用户当前工作目录

## 当前支持的站点优化

- `help.fanruan.com`（帆软 FineBI 文档）：自动去除左侧导航树、顶部 header、底部 footer

## 扩展站点配置

如需为新站点添加针对性元素移除规则，编辑 `~/.claude/skills/web-to-pdf/scripts/config.json`，
在 `sites` 对象中添加条目（key 为域名片段）：

```json
"sites": {
  "example.com": {
    "removeSelectors": [".left-nav", "#site-header"],
    "waitFor": ".main-content"
  }
}
```

注意：percollate 已通过 Mozilla Readability 自动提取正文，大多数站点无需额外配置。

## 触发条件

用户说以下任意一种时使用本 skill：
- 把这个网页/链接/URL 打印出来
- 转成 PDF / 导出 PDF
- 我想打印这篇文章/文档
- web-to-pdf / 网页转PDF
