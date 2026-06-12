# 📰 News Hub Skills Collection

Hermes Agent skills for automated tech news aggregation and daily infographic generation — collecting from international and Chinese sources, then producing styled HTML reports and PNG exports.

---

## Skills Included

### 1. `daily-tech-infographic`

**每日前沿科技资讯日报** — Automatically gathers global technology news and generates a **cyberpunk-neon styled HTML infographic page**, exportable as PNG image.

**Features:**
- Automated collection of cutting-edge tech news from global sources
- Cyberpunk-neon themed HTML/CSS infographic generation
- PNG image export capability
- Daily briefing format suitable for social sharing

**Files:**
| File | Description |
|------|-------------|
| `SKILL.md` | Main skill definition |

---

### 2. `tech-news-digest`

Collects daily tech news from **international and Chinese sources**, generates a Markdown report + HTML webpage with **4 template styles** available: cyberpunk-neon (default), newsletter, terminal, minimalist.

**Features:**
- Multi-source news collection (international + Chinese)
- 4 visual templates to choose from:
  - 🟣 **cyberpunk-neon** — Dark neon-themed layout (default)
  - 📰 **newsletter** — Clean email-style newsletter format
  - 💻 **terminal** — Retro terminal/monospace aesthetic
  - ⬜ **minimalist** — Minimal, whitespace-heavy design
- Markdown report generation alongside HTML output

**Files:**
| File | Description |
|------|-------------|
| `SKILL.md` | Main skill definition (15KB) |
| `templates/cyberpunk-news.html` | Cyberpunk-neon template |
| `templates/newsletter.html` | Newsletter-style template |
| `templates/terminal.html` | Terminal aesthetic template |
| `templates/minimalist.html` | Minimalist design template |

---

## Installation

### Option A: Hermes Skill Manager (Recommended)

```bash
# Install both skills via Hermes CLI
hermes skills install https://github.com/memory125/news-hub --skill daily-tech-infographic
hermes skills install https://github.com/memory125/news-hub --skill tech-news-digest
```

### Option B: Manual Installation

```bash
# Clone this repo and symlink to your Hermes skills directory
git clone https://github.com/memory125/news-hub.git
cd news-hub

# Symlink each skill
ln -s $(pwd)/skills/daily-tech-infographic ~/.hermes/skills/media/daily-tech-infographic
ln -s $(pwd)/skills/tech-news-digest ~/.hermes/skills/productivity/tech-news-digest
```

### Option C: Third-Party Skill Installation

Use the `third-party-skill-installation` Hermes skill to handle installation, including bypassing Skills Guard false positives via clone + symlink.

---

## How They Work Together

These two skills serve **complementary purposes**:

1. **`daily-tech-infographic`** focuses on creating visually striking **single-image infographics** — perfect for social media sharing and quick visual briefings with a cyberpunk-neon aesthetic.

2. **`tech-news-digest`** provides comprehensive **multi-format news reports** with 4 template choices, producing both Markdown and HTML output for deeper reading and archival.

Use `daily-tech-infographic` when you need an eye-catching visual summary. Use `tech-news-digest` when you want a detailed, multi-template report with full article coverage.

---

## Template Styles (tech-news-digest)

| Style | Preview | Best For |
|-------|---------|----------|
| **cyberpunk-neon** 🟣 | Dark background, neon accents, glowing text | Social media, dark mode displays |
| **newsletter** 📰 | Clean white layout, editorial typography | Email distribution, professional sharing |
| **terminal** 💻 | Monospace font, green-on-black, ASCII borders | Developer audiences, retro aesthetic |
| **minimalist** ⬜ | Lots of whitespace, subtle colors, clean lines | Print-friendly, accessibility-focused |

---

## Requirements

- **Hermes Agent** — Web search and browser tools for news collection
- **Python 3** — For template processing and PNG export (infographic)
- **wkhtmltoimage / puppeteer** (optional) — For HTML-to-PNG conversion

---

## License

MIT
