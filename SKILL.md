---
name: portfolio-creations-rebuilder
description: 重建 zhaoliang-portfolio 网站 lib/creations.ts 的完整流水线：修腐败嵌套结构、转为多日数组、追加历史数据、重构 timeline 页面、推送 GitHub + Vercel 自动部署。适用于网站内容因多次替换 entry 导致结构损坏、或需要一次性发布大量历史数据的场景。
tags: [portfolio, nextjs, github, vercel, typescript, automation]
---

# Portfolio Creations 重建流水线

## 背景

`lib/creations.ts` 是 zhaoliang-portfolio 网站的内容数据源。若 deploy 脚本（trending / moon video）使用**替换策略**（每次只写入单个日期的 entry）重复运行，会导致：

1. 历史 entry 被新 entry **覆盖**，只剩最新一天
2. 多次替换后，`lib/creations.ts` 中同一 slug 下出现**嵌套残缺结构**（缺少闭合 `}`）
3. TypeScript 编译不过或页面渲染异常

## 症状识别

```bash
# 检查是否有嵌套结构
grep -n "slug:" ~/.hermes/scripts/deploy_trending_to_github_pages.sh | head -3
# 输出中 entry_start/entry_end 逻辑里 brace_depth 计算容易出错

# 症状：lib/creations.ts 里同一 slug 出现两次
grep -c 'slug: "video-storylab"' /tmp/zhaoliang-portfolio/lib/creations.ts
# 应该只有 1 个
```

## 核心原则

**每个 slug 只能有一个 entry**。历史多日展示必须用 `const DAYS: CreationSection[] = [...]` 数组实现，不要在 `creations[]` 数组里塞多个同名 slug entry。

### GitHub Trending 的新增展示原则（2026-05-10）

对 `github-trending-notes`，不要再做“单条 entry 反复覆盖当天 Top5”的方案。正确目标应是：

1. **每天一条 record**：每个日期都是一条独立记录，而不是把 `sections` 永远改写成“今天的 Top5”。
2. **优先消费已落盘产物**：当天目录里若存在这些文件，应把它们视为展示源，而不是只抽一段摘要文本：
   - `top5-trending-xhs.md`
   - `top5-trending-xhs.html`
   - `top5-card-images.zip`
   - `cards/**/00-cover.png`
   - `cards/**/01-*.png` ... `05-*.png`
3. **最适合的页面形态是“时间线 + 图片画廊”混合模式**：
   - 列表层：日期、当日一句总结、Top5 项目名、cover 缩略图
   - 展开层：Top5 文字摘要 + 6 张卡片图 gallery + ZIP/MD/HTML 链接
4. **不要继续把图片信息丢掉**。如果本地已经落了 PNG/HTML/ZIP，就应在数据模型中显式保留并传到页面层。

建议把 GitHub Trending 的每日数据建模为独立对象，例如：

```ts
interface TrendingDayRecord {
  date: string;
  title: string;
  summary: string;
  top5: string[];
  coverImage?: string;
  images: { label: string; src: string }[];
  markdownPath?: string;
  htmlPath?: string;
  zipPath?: string;
}
```

然后由 `github-trending-notes` 的单个 slug 引用 `TrendingDayRecord[]`，页面层再渲染为 timeline + gallery。

## 流水线步骤

### Step 1：摸清历史数据

```bash
# GitHub Trending 历史（需要 top5-trending-xhs.md）
ls -d ~/.hermes/output/trending-xhs/*/ | while read d; do
  date=$(basename "$d" | sed 's/-top5-xhs//')
  if [[ -f "$d/top5-trending-xhs.md" ]]; then
    echo "$date: OK"
  else
    echo "$date: MISSING"
  fi
done

# 月球背面历史（需要 story.md）
ls -d ~/.hermes/serial_video/moon_reversal_series/episodes/*/ | while read d; do
  date=$(basename "$d")
  if [[ -f "$d/story.md" ]]; then
    echo "$date: OK"
  else
    echo "$date: MISSING"
  fi
done
```

### Step 2：解析 story.md 格式

月球背面 `story.md` 使用**小写字段名**：
```
day number: 1
one-line logline: xxx
whether today is a reversal of yesterday: no
how today reverses yesterday: xxx
```

**注意**：deploy 脚本里用的解析是正则（`## 基本信息`、`Day`**、`Logline`**、`是否反转`**），但实际文件是**小写无星号**格式。重建脚本必须匹配实际格式。

### Step 3：克隆仓库

```bash
# HTTPS 克隆（SSH key 可能需要验证）
git clone https://github.com/YOUR_GITHUB_USERNAME/zhaoliang-portfolio.git /tmp/portfolio-work
cd /tmp/portfolio-work
```

### Step 4：运行重建脚本

主脚本：`~/.hermes/scripts/rebuild_portfolio_creations.py`

输入：
- `~/.hermes/output/trending-xhs/{date}-top5-xhs/top5-trending-xhs.md`
- `~/.hermes/output/trending-xhs/{date}-top5-xhs/cards/C-再放大一档/*.png`
- `~/.hermes/output/trending-xhs/{date}-top5-xhs/top5-trending-xhs.html`
- `~/.hermes/output/trending-xhs/{date}-top5-xhs/top5-card-images.zip`
- `~/.hermes/serial_video/moon_reversal_series/episodes/{date}/story.md`
- `~/.hermes/serial_video/moon_reversal_series/episodes/{date}/final.mp4`

输出：
- `/tmp/zhaoliang-portfolio/lib/creations.ts`（干净的多日结构）
- `/tmp/zhaoliang-portfolio/public/creations/github-trending/{date}/...`（按日期归档的静态图片/Markdown/HTML/ZIP）

关键输出字段：
```typescript
export interface CreationSection {
  title: string;
  body: string[];
  gallery?: { src: string; alt: string }[];
  links?: { label: string; href: string }[];
}

const VIDEO_DAYS: CreationSection[] = [
  { title: "Day 13 — 2026-05-09", body: ["logline内容", "反转说明"] },
  ...
];
const TRENDING_DAYS: CreationSection[] = [
  {
    title: "2026-05-10",
    body: ["当天 Top5：...", "Top 1：..."],
    gallery: [
      { src: "/creations/github-trending/2026-05-10/00-cover.png", alt: "2026-05-10 00-cover" },
      ...
    ],
    links: [
      { label: "查看 Markdown", href: "/creations/github-trending/2026-05-10/top5-trending-xhs.md" },
      { label: "查看 HTML", href: "/creations/github-trending/2026-05-10/top5-trending-xhs.html" },
      { label: "下载图片 ZIP", href: "/creations/github-trending/2026-05-10/top5-card-images.zip" },
    ],
  },
  ...
];
```

#### 新规则（2026-05-10 实战）
1. **GitHub Trending 不是只保留“今天 Top5”单页，而是每天一条记录。** `github-trending-notes` 必须用 `sections[]` 承载历史天列表，而不是反复覆盖同一个 sections 文本块。
2. **图片应直接做成静态网站资产。** 已落盘的 `cards/C-再放大一档/*.png` 需要复制到 `public/creations/github-trending/{date}/`，页面里用 `gallery[]` 展示，而不是只保留文案。
3. **同一天目录里若同时存在旧图和新图，只保留与当天 Top5 repo 名匹配的 6 张图（`00-cover` + `01..05-<repo>`）。** 不要把别的日期残留图片一并挂到当天 gallery。
4. **视频连载只纳入真正成功日。** `video-storylab` 重建时，必须同时要求 `story.md` 和 `final.mp4` 都存在；像“当天脚本放弃、只有 story.md 没有 final.mp4”的日期，不应写进公开网站时间线。
5. **若某天只有 Markdown / HTML 没有图片，也保留当天记录，但 `gallery` 留空、只展示可访问链接。** 例如某些日期只生成了 md/html，没有 png，不应因此丢失整天记录。

### Step 5：更新 page.tsx 支持 timeline + gallery

在 `app/creations/[slug]/page.tsx` 中，对多日 slug 渲染成时间线：

```typescript
const MULTI_DAY_SLUGS = ["video-storylab", "github-trending-notes"];

{isMultiDay ? (
  <section>
    <div className="relative">
      <div className="absolute left-4 top-0 bottom-0 w-px bg-border" />
      {entry.sections.map((section) => (
        <details key={section.title} className="group rounded-2xl border border-border bg-card overflow-hidden">
          <summary className="flex items-center justify-between cursor-pointer p-5 hover:bg-muted/30">
            <div>
              <div className="font-semibold">{section.title}</div>
              <div className="text-sm text-muted-foreground mt-1 line-clamp-2">
                {section.body[0]}
              </div>
            </div>
          </summary>
          <div className="px-5 pb-5 border-t border-border">
            {section.body.map((line, i) => (
              <p key={i} className="text-sm text-muted-foreground mt-3">{line}</p>
            ))}
          </div>
        </details>
      ))}
    </div>
  </section>
) : (
  // 原有的 sections 渲染
)}
```

### Step 6：提交推送

```bash
cd /tmp/portfolio-work
git config user.name "Hermes Agent"
git config user.email "hermes@example.com"
git add -A
git commit -m "feat: rebuild creations with full history timeline"
git push
# Vercel 自动部署（约 1-2 分钟）
```

### Step 7：验证

```bash
# 等待 Vercel 部署完成后
open https://zhaoliang-portfolio.vercel.app/creations/video-storylab
open https://zhaoliang-portfolio.vercel.app/creations/github-trending-notes
```

## 关键陷阱

1. **deploy 脚本的 replace 策略**：每日 cron 脚本里替换 entry 的逻辑，不适合累积历史。只修改当天，后续天会覆盖。重建时必须一次性把所有历史打包进 `DAYS[]` 数组。

2. **push 超时**：HTTPS push 在中国大陆可能超时，但后台实际成功。用 `git status` 确认 commit 存在，或用 `git push &` 后台执行再 `wait`。

3. **story.md 格式**：实际文件用小写字段（`day number:`），而旧 deploy 脚本里写的是 `Day**:` 星号格式。解析必须匹配实际文件。

4. **缺失历史数据**：多数日期的 trending md 文件不存在（生成了但没落地，或生成失败）。能发布的只限于有 md 文件的日期。
