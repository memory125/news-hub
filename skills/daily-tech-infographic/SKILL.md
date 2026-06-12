---
name: daily-tech-infographic
description: "每日前沿科技资讯日报 — 自动采集全球科技新闻，生成 HTML/CSS cyberpunk-neon 风格信息图页面并导出为 PNG 图片。"
version: 1.0.0
author: Hermes Agent
license: MIT
tags: [tech-news, infographic, daily-briefing, html-css, screenshot]
platforms: [linux, macos, windows]
triggers:
  - "每日科技资讯"
  - "前沿科技日报"
  - "生成科技信息图"
  - "daily tech briefing"
---

# 每日前沿科技资讯日报

自动采集全球科技新闻，生成 HTML/CSS cyberpunk-neon 风格的信息图页面并导出为 PNG 图片。

## 流程概述

```
1. 采集新闻 → 2. 分析内容 → 3. 结构化 → 4. 生成HTML → 5. 截图PNG
```

## Step 1: 创建输出目录

```bash
DATE=$(date +%Y-%m-%d)
mkdir -p "$HOME/Desktop/infographic/科技资讯日报-$DATE"
```

## Step 2: 采集前沿科技新闻

搜索以下来源获取当日最新科技新闻（重点关注 AI、生物科技、量子计算、热点科技）：

**国际来源：**
- Google News (Technology section)
- Hacker News (front page)
- TechCrunch
- The Verge
- WIRED
- MIT Technology Review
- Ars Technica

**中文来源：**
- 36氪 (36kr.com)
- 虎嗅网
- 极客公园

**搜索关键词：** AI, Artificial Intelligence, Quantum Computing, Biotech, Brain-Computer Interface, Tech News, 前沿科技, 人工智能

将采集到的新闻内容保存到 `source.md`：

```markdown
# 前沿科技资讯日报 - {DATE}

## AI · Artificial Intelligence
...

## 生物科技 · Biotechnology
...

## 量子计算 · Quantum Computing
...

## 热点科技 · Hot Tech
...

## 中国科技前沿 · China Tech Frontier
...
```

## Step 3: 内容分析与结构化

### 分析框架 (analysis.md)

```yaml
---
title: "前沿科技资讯日报 - {DATE}"
topic: "technology-news-overview"
data_type: "overview/summary with multiple domains"
complexity: "complex"
point_count: <number of key points>
source_language: "zh"
user_language: "zh"
primary_domain: "AI / Technology News"
secondary_domains: ["Biotech", "Quantum Computing", "Hot Tech"]
...
```

### 结构化内容 (structured-content.md)

按以下板块整理新闻：

1. **Overview** — 一句话概括当日科技要闻
2. **Learning Objectives** — 读者将了解的核心要点
3. **Key Insights & Trends** — 趋势分析（3-5条）
4. **Content Sections** — 每个板块包含：
   - Section icon (emoji)
   - Headline (中文标题)
   - Key points (2-5条，每条含 icon + 简短描述)
   - Source attribution

## Step 4: 生成 HTML/CSS 信息图页面

参考模板见 `templates/infographic.html`。核心设计规格：

### 布局 — Bento Grid
```css
.bento-grid {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  gap: 1rem;
}
/* AI HERO: span 5 | Policy: span 4 | Biotech: span 3 */
/* Quantum: span 3 | Hot Tech: span 6 | China Tech: span 3 */
```

### 风格 — Cyberpunk Neon
| 板块 | 霓虹色 | CSS Variable |
|------|--------|-------------|
| AI · 人工智能 | Cyan | `--neon-cyan: #00f0ff` |
| AI政策与监管 | Pink | `--neon-pink: #ff00aa` |
| 生物科技 | Green | `--neon-green: #39ff14` |
| 量子计算 | Purple | `--neon-purple: #bf5fff` |
| 热点科技 | Orange | `--neon-orange: #ff8c00` |
| 中国科技前沿 | Red | `--neon-red: #ff3344` |

### 字体
- 中文：Noto Sans SC (Google Fonts)
- 英文/数字：Orbitron (monospace, Google Fonts)

### HTML 模板结构

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>前沿科技资讯日报 - {DATE}</title>
  <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+SC:wght@300;400;500;700;900&family=Orbitron:wght@400;500;700;900&display=swap" rel="stylesheet">
  <style>
    /* See template for full CSS */
  </style>
</head>
<body>
  <div class="scanline"></div>
  <div class="container">
    <header class="header">...</header>
    <div class="bento-grid">
      <!-- Cards for each section -->
    </div>
    <footer class="footer">...</footer>
  </div>
</body>
</html>
```

保存为 `index.html`。

## Step 5: 浏览器验证

用 browser_navigate 打开 HTML 文件，用 browser_vision 检查视觉效果：

- [ ] 中文文字正确渲染（无方块/乱码）
- [ ] 霓虹发光效果正常
- [ ] 所有卡片完整显示
- [ ] 布局平衡美观

## Step 6: 导出 PNG 图片

使用 Playwright 截图：

```python
#!/usr/bin/env python3
from playwright.sync_api import sync_playwright

html_path = r"{OUTPUT_DIR}/index.html"
output_path = r"{OUTPUT_DIR}/infographic.png"

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page(viewport={"width": 1400, "height": 900})
    page.goto(f"file://{html_path}")
    page.wait_for_timeout(2000)  # Wait for fonts/animations
    page.screenshot(path=output_path, full_page=True)
    browser.close()
```

### Playwright 安装（首次运行）

```bash
# Install playwright Python package
uv pip install --system playwright

# Install Chromium browser
python -m playwright install --with-deps chromium
```

## Step 7: 清理临时文件

保留 `index.html` + `infographic.png`，可选删除 `prompts/`, `source.md`, `analysis.md`, `structured-content.md`。

## 输出规格

| 属性 | 值 |
|------|-----|
| **PNG 分辨率** | ~1400 × 1100 px (full page) |
| **文件大小** | ~300-500 KB |
| **格式** | PNG (无损, RGB) |
| **HTML** | 单文件自包含 (内联 CSS) |

## 依赖

- Python 3.12+ with `playwright` package
- Chromium browser (via playwright install)
- Google Fonts CDN access (Noto Sans SC + Orbitron)
- Internet access for news gathering (web_search / browser_navigate)

## Pitfalls

1. **Playwright 首次安装较慢** — Chromium 下载约 150MB，需提前安装好
2. **中文字体加载** — Google Fonts 在部分地区可能慢，确保网络通畅
3. **截图等待时间** — `page.wait_for_timeout(2000)` 确保字体和动画加载完成
4. **文件路径编码** — 中文目录名在 file:// URL 中需正确 percent-encode。**实际方案**: 浏览器无法直接打开含中文的 file:// URL。将最终 HTML 复制到 ASCII-only 文件名 (如 `tech-daily-YYYY-MM-DD.html`) 供浏览器验证，原始中文路径版本保留归档。
5. **新闻来源变化** — Google News / Hacker News 页面结构可能变动，搜索关键词要灵活调整
6. **write_file 流超时 (~8K token limit)** — 单文件 HTML 超过 ~8K tokens 时 stream 会超时。**修复**: 将内容拆成多个小文件 (CSS/head → body chunk 1 → body chunk 2 → footer)，每个 < 8K，然后用 `cat file1 file2 file3 > final.html` 合并。
7. **MSYS bash heredoc append 失败** — `cat >> file << 'EOF'` 中的 `>>` 被误解析为后台操作符 `&`。**修复**: 用 write_file 写独立文件，再用 `cat f1 f2 > combined` 在 terminal 中合并。
8. **浏览器验证 checklist** (必须执行):
   - [ ] 中文文字正确渲染（无方块/乱码）— browser_vision 确认
   - [ ] 霓虹发光效果正常 — 各卡片边框颜色与 glow 可见
   - [ ] Bento Grid 布局完整 — JS 检查: `document.querySelectorAll('.card').length` 应等于预期卡片数
   - [ ] Footer + Sources 存在 — JS 检查: `!!document.querySelector('footer')` 应为 true
   - [ ] 页面总高度合理 — JS 检查: `document.body.scrollHeight` 应 > 2000px (内容完整)

## Verification Checklist

- [ ] 今日科技新闻已采集（至少覆盖 AI、生物科技两个领域）
- [ ] HTML 页面在浏览器中正常渲染
- [ ] PNG 图片完整（无截断/空白区域）
- [ ] 中文文字清晰可读
- [ ] 霓虹效果正常显示
- [ ] 文件大小 < 1MB
