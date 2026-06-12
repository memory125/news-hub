---
name: tech-news-digest
description: "Collects daily tech news from international and Chinese sources, generates Markdown report + HTML webpage. 4 template styles available: cyberpunk-neon (default), newsletter, terminal, minimalist."
version: 2.0.0
author: Hermes Agent
license: MIT
tags: [tech-news, ai-news, markdown, html, dashboard, chinese-tech]
platforms: [linux, macos, windows]
triggers:
  - 科技资讯日报
  - tech news digest
  - daily tech briefing
  - 前沿科技新闻
---

# Tech News Digest v2.0

Collects today's technology news from multiple international and Chinese sources using optimized browser automation with JS-based extraction, generates both a Markdown report and a styled HTML webpage. Includes deduplication, story scoring, and smart source prioritization.

## Relationship to daily-tech-infographic

| Skill | Purpose | Output |
|-------|---------|--------|
| **tech-news-digest** (this skill) | Full news collection pipeline: gather → categorize → score → deduplicate → report | Markdown + HTML webpage |
| **daily-tech-infographic** | Visual infographic generator: takes collected data → creates cyberpunk bento-grid PNG | HTML infographic + PNG screenshot via Playwright |

**Workflow**: Run `tech-news-digest` first to collect and structure the news, then optionally run `daily-tech-infographic` to generate a shareable visual. The infographic skill reuses the same source list but focuses on visual output.

## When to Use

- User asks for daily tech news / 科技资讯日报
- User wants a tech briefing covering AI, biotech, quantum computing
- User requests a styled visual dashboard of today's tech headlines
- Recurring task: "give me today's tech news"

## Data Sources — Optimized 3-Phase Approach

### Phase 1: Core (Always Check, ~2 min)

| Source | URL | Focus | Reliability |
|--------|-----|-------|-------------|
| Hacker News | https://news.ycombinator.com | Community top stories, engagement metrics | ✅ Always works |
| The Verge | https://www.theverge.com | Consumer tech, AI, policy | ✅ Reliable |
| 极客公园 | https://www.geekpark.net/ | Chinese innovation & startups | ✅ Reliable |

### Phase 2: International (Check Most, ~3 min)

| Source | URL | Focus | Reliability |
|--------|-----|-------|-------------|
| TechCrunch | https://techcrunch.com | Startups, funding, IPOs | ✅ Reliable |
| WIRED | https://www.wired.com | Deep tech, security, culture | ⚠️ Sometimes slow |
| MIT Technology Review | https://www.technologyreview.com | AI, biotech, research | ✅ Reliable |
| Ars Technica | https://arstechnica.com | Policy, science, computing | ✅ Reliable |
| VentureBeat | https://venturebeat.com | AI business impact, enterprise | ✅ Use main site (category URLs 404) |
| Engadget | https://www.engadget.com | Consumer gadgets, hardware | ✅ Reliable |

### Phase 3: Spot-Check (Conditional, ~2 min)

**Only check these if relevant topics surface in Phases 1-2:**

| Trigger Topic | Sources to Check | URLs |
|---------------|-----------------|------|
| AI dominates headlines | OpenAI Blog, Anthropic News, Hugging Face Blog | openai.com/blog, anthropic.com/news, huggingface.co/blog |
| Security incidents | Krebs on Security, BleepingComputer | krebsonsecurity.com, bleepingcomputer.com/news/ |
| Academic breakthroughs | Quanta Magazine, arXiv cs.AI | quantamagazine.org, arxiv.org/list/cs.AI/recent |
| M&A / funding news | Reuters Technology, CNBC Tech | reuters.com/technology/, cnbc.com/technology/ |
| Chinese tech coverage | 量子位, 机器之心 | qbitai.com, jiqizhixin.com |

### Source Reliability Summary (Updated 2026-06)

**✅ Highly Reliable** — rarely blocked, fast: HN, The Verge, TechCrunch, VentureBeat, Ars Technica, MIT TR, Engadget, 极客公园, 量子位
**⚠️ Occasionally Blocked/Slow**: WIRED (slow first load), 36Kr (verification), 虎嗅 (verification), Reuters (CAPTCHA), BleepingComputer (Cloudflare)
**🔧 Workarounds**: When category/sub-page returns 404, try main site URL. For blocked Chinese sites, prioritize 量子位 and 极客公园 over 虎嗅/36Kr. Skip after one failure — don't retry.

## Step 1: Collect News Data (Optimized)

### Primary Method: JS-Based Extraction via browser_console

**After `browser_navigate(url)`, use JavaScript to extract structured data instead of relying on `browser_snapshot` alone.** This is faster, more reliable, and returns consistent JSON.

```javascript
// Universal extraction pattern — works on most news sites
[...document.querySelectorAll('article, .story-link, a[href*="/articles/"], a.storylink')].slice(0, 30).map(el => ({
  title: el.querySelector('h1,h2,h3,h4,.title,.headline,a')?.textContent?.trim() || '',
  summary: el.querySelector('.summary,.description,.excerpt,p')?.textContent?.trim().substring(0, 200) || '',
  url: el.tagName === 'A' ? el.href : (el.querySelector('a')?.href || ''),
  time: el.querySelector('time,[class*="date"],[class*="time"]')?.textContent?.trim() || ''
}))
```

### Site-Specific Extraction Patterns

**Hacker News:**
```javascript
// HN scores are NOT in dedicated <span id="score-..."> elements — they appear inline as "XXX points by USER Y hours ago | hide | Zcomments"
[...document.querySelectorAll('.athing')].slice(0, 30).map(el => {
  const link = el.querySelector('.titleline a');
  // Find the sibling row with score info (next element after this one's parent)
  const metaRow = el.parentElement.nextElementSibling;
  const metaText = metaRow?.textContent || '';
  const ptsMatch = metaText.match(/(\d+)\s*points/);
  const commMatch = metaText.match(/(\d+)\s*comments?/);
  return {
    title: link?.textContent?.trim() || '',
    url: link?.href || '',
    points: ptsMatch ? parseInt(ptsMatch[1]) : 0,
    comments: commMatch ? parseInt(commMatch[1]) : 0
  };
})
```

**The Verge:**
```javascript
// The Verge uses heading elements inside generic containers — query h3 links directly
JSON.stringify([...document.querySelectorAll('h3 a')].slice(0, 25).map(a => ({
  title: a.textContent?.trim() || '',
  url: a.href || ''
})))
```

**Tip:** When JS extraction fails on any site, `browser_navigate` already returns a compact snapshot with all headlines. Use that as the primary data source — JS extraction is a supplement for engagement metrics (HN points, comment counts), not the only path.

### Fallback Methods (in order of preference)

1. `browser_snapshot(full=false)` — compact interactive elements
2. `browser_vision(question="list all headlines and their summaries")` — visual extraction when DOM is unclear
3. Skip source if blocked after one attempt

### Collection Targets

| Phase | Min Sources | Target | Time Budget |
|-------|-------------|--------|-------------|
| Phase 1 (Core) | 3/3 | 3 | ~2 min |
| Phase 2 (International) | 4/6 | 5-6 | ~3 min |
| Phase 3 (Spot-check) | 0/∞ | 2-3 if triggered | ~2 min |
| **Total** | **7+** | **10-12** | **~7 min** |

## Step 2: Categorize, Score & Deduplicate

### Categories

| Category | Emoji | Focus Areas |
|----------|-------|-------------|
| AI · Artificial Intelligence | 🤖 | LLMs, agents, foundation models, applications |
| AI Policy & Regulation | ⚖️ | Regulations, legislation, ethics, antitrust |
| Biotech & Science | 🧬 | Biotech, brain-computer interfaces, health tech |
| Quantum & Academic Research | ⚛️ | Quantum computing, academic research, science |
| Hot Tech & Consumer | 🔥 | Security, hardware, gaming, developer tools |
| Chinese Tech Frontier | 🇨🇳 | Chinese startups, funding, domestic AI scene |

### Story Scoring System

Score each story 1-10 based on these factors:

```
Score = (engagement × 0.4) + (significance × 0.3) + (novelty × 0.2) + (source_authority × 0.1)
```

| Factor | Scale | Examples |
|--------|-------|----------|
| Engagement | 1-10 | HN >500pts=10, >200pts=7, trending=8 |
| Significance | 1-10 | Industry-shifting=10, product update=4, minor fix=1 |
| Novelty | 1-10 | First-of-kind=10, incremental improvement=3 |
| Source Authority | 1-10 | MIT TR/Nature=10, blog post=2 |

**Selection threshold:** Only include stories scoring ≥5. Flag stories ≥8 as "🔴 Breaking/High Impact".

### Deduplication Logic

When the same story appears across multiple sources:
1. **Keep only ONE instance** — prefer the source with highest authority or most detail
2. **Merge engagement metrics** from all sources (e.g., "HN 342pts + covered by TechCrunch, WIRED")
3. **Note cross-source coverage** in the summary: "(also on HN, The Verge)"

## Step 3: Generate Markdown Report

Create at `~/Desktop/前沿科技资讯日报_{DATE}.md`. Structure:

```markdown
# 🔬 前沿科技资讯日报 — YYYY年MM月DD日 WEEKDAY

---

## 🤖 AI · Artificial Intelligence

### 🔴 Breaking / High Impact (Score ≥8)

#### **Headline** *(Source)*
- Summary with key details
- Engagement: HN XXXpts/YYY comments | Score: X/10

### 📰 Industry News

#### **Headline** *(Source)*
- Brief summary

---

## ⚖️ AI Policy & Regulation

| Region | Details | Impact |
|--------|---------|--------|
| **Country** | Policy details | High/Medium/Low |

---

## 🧬 Biotech & Science

#### **Headline** *(Source)*
- Summary

---

## ⚛️ Quantum & Academic Research

#### **Headline** *(Source)*
- Summary

---

## 🔥 Hot Tech & Consumer

### 🔐 Security
### 🖥️ Hardware & Gaming
### 💻 Developer Tools

---

## 🇨🇳 Chinese Tech Frontier

1. **Headline**: Summary *(source)*
2. **Headline**: Summary *(source)*

---

## 📊 Today's Key Themes

> 🎯 Trend 1 — brief description
> 🧠 Trend 2 — brief description

*Sources: [list accessed sources]*
```

## Step 4: Generate HTML Webpage

**Available Templates:** Load with `skill_view(name="tech-news-digest", file_path="templates/{template}.html")`

| Template | File Path | Style | Best For |
|----------|-----------|-------|----------|
| **Cyberpunk Neon** (default) | `templates/cyberpunk-news.html` | Dark bento grid, neon glow | Visual impact, desktop dashboard |
| **Newsletter** | `templates/newsletter.html` | Clean light theme, card-based | Professional reading experience |
| **Terminal** | `templates/terminal.html` | Monospace terminal aesthetic | Developer audience |
| **Minimalist** | `templates/minimalist.html` | Warm paper tones, serif typography | Magazine feel, print-friendly |

### Generation Process (Avoids write_file Timeout)

HTML files exceed the ~8K token limit for single `write_file`. Use this chunked approach:

1. Load template: `skill_view(name="tech-news-digest", file_path="templates/cyberpunk-news.html")`
2. Write HTML in **2-3 chunks** to temp files (2 usually suffices):
   - Chunk 1: `<head>` + CSS + opening `<body>` + header (~6K tokens)
   - Chunk 2: Main content cards / bento grid body + footer + closing tags (~8K tokens)
   - (Optional Chunk 3 if content is very large: split the body further)
3. Merge with terminal: `cat chunk1.html chunk2.html > ~/Desktop/前沿科技资讯日报_{DATE}.html`

### Template Notes

- **cyberpunk**: Bento grid, per-section neon colors (cyan/pink/green/purple/orange/red). Scanning animation + hover effects.
- Replace `${DATE}` and `${WEEKDAY}` in header
- Each category → a `.card` div with actual news content
- Update footer sources list to reflect actually accessed sources

### Color Coding (cyberpunk template)

| Section | Neon Color | CSS Class |
|---------|-----------|-----------|
| AI · 人工智能 | Cyan (#00f0ff) | `.card-hero` |
| AI Policy & Regulation | Pink (#ff00aa) | `.card-policy` |
| Biotech & Science | Green (#39ff14) | `.card-bio` |
| Quantum & Academic | Purple (#bf5fff) | `.card-quantum` |
| Hot Tech & Consumer | Orange (#ff8c00) | `.card-hot` |
| Chinese Tech Frontier | Red (#ff3344) | `.card-china` |

## Step 5: Verify Output

1. Open HTML in browser: `browser_navigate("file:///path/to/file.html")`
2. Run JS verification checks via `browser_console`:
   ```javascript
   // Check all cards rendered
   document.querySelectorAll('.card').length
   // Check footer exists
   !!document.querySelector('footer')
   // Check page height (should be > 2000px for full content)
   document.body.scrollHeight
   ```
3. Use `browser_vision(question="verify Chinese text renders correctly and neon glow effects are visible")`

## Automated Daily Briefing (Cron Job)

Schedule a recurring daily collection:

```
cronjob(action='create',
  name='daily-tech-news',
  prompt='Run the tech-news-digest skill to collect today\'s top tech news. Output both Markdown and HTML.',
  schedule='0 8 * * *',  # Daily at 8 AM
  skills=['tech-news-digest'],
  enabled_toolsets=['browser', 'file', 'terminal']
)
```

## Output Files

| File | Location | Purpose |
|------|----------|---------|
| Markdown report | `~/Desktop/前沿科技资讯日报_{DATE}.md` | Text-based daily briefing |
| HTML webpage | `~/Desktop/前沿科技资讯日报_{DATE}.html` | Styled visual dashboard (single file) |

## Pitfalls & Lessons Learned

1. **write_file stream timeout (~8K token limit)** — Large HTML files MUST be split into chunks written separately, then merged with `cat`. In practice 2 chunks often suffice (head+CSS + body content). Never attempt a single write for the full HTML.
2. **MSYS bash heredoc append fails** — `>>` gets misinterpreted as background operator. Use `write_file` for individual chunks, then `cat f1 f2 > combined` in terminal.
3. **Chinese file paths in file:// URLs** — Browsers can't open Chinese-character paths via `file://`. Copy final HTML to ASCII-only filename (e.g., `tech-daily-YYYY-MM-DD.html`) for browser verification. Keep original Chinese path as archive.
4. **Category page 404s** — When sub-category URL returns 404, immediately fall back to main site (e.g., `venturebeat.com` instead of `/category/...`).
5. **Cloudflare / bot detection** — Google News, BleepingComputer, some Chinese sites block automated access. Skip after ONE failure — don't retry. Target coverage compensates for individual failures.
6. **browser_snapshot insufficient for data extraction** — Prefer `browser_console(expression=JS)` for structured JSON extraction. Use snapshot only as fallback or for navigation structure.
7. **WIRED slow first load** — WIRED may timeout on initial navigate. Retry once, then skip if still failing.
8. **Chinese source reliability ranking**: 极客公园 ≥ 量子位 >> 机器之心 > 虎嗅 ≈ 36Kr (verification pages). Always check reliable sources first to guarantee minimum coverage.
9. **Ref IDs change between snapshots** — After any navigation/click/scroll, MUST call `browser_snapshot` again before clicking. Stale refs produce "Unknown ref" errors.
10. **Deduplication is critical** — Same story often appears on 3-5 sources. Always merge into single entry with combined engagement metrics to avoid bloated reports.
11. **Parallel delegation timeouts** — Spawning 3+ subagents for browser navigation simultaneously causes all to hit the 600s timeout. Prefer sequential `browser_navigate` calls in the main session, or limit parallel delegates to 2 max with explicit `timeout` awareness.
