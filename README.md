# Portfolio Creations Rebuilder | 个人网站内容重建

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## 简介 | Introduction

重建 zhaoliang-portfolio 网站 `lib/creations.ts` 的完整流水线：修复腐败嵌套结构、转为多日数组、追加历史数据、重构 timeline 页面、推送 GitHub + Vercel 自动部署。

Complete pipeline to rebuild the `lib/creations.ts` content file for the zhaoliang-portfolio website: fix corrupted nested structures, convert to multi-day arrays, backfill history, and trigger GitHub + Vercel auto-deployment.

## 问题背景 | Background

若 deploy 脚本（trending / moon video）使用**替换策略**（每次只写入单个日期的 entry）重复运行，会导致：

- 嵌套结构腐败（数组里套数组）
- 历史数据被覆盖而非追加
- Timeline 页面展示异常

This skill fixes the situation where automated deploy scripts using a "replace single entry" strategy cause nested structure corruption, overwritten history, and broken timeline pages.

## 功能 | Features

- **结构修复**：将腐败嵌套结构展开为标准多日数组
- **历史追加**：合并历史数据，不丢失已有 entry
- **Timeline 重构**：按日期重新生成 timeline 展示逻辑
- **一键部署**：修复后自动推送到 GitHub，触发 Vercel 重建
- **验证闭环**：部署后自动打开网站验证

## 核心脚本 | Core Scripts

| 脚本 | 作用 |
|------|------|
| `rebuild_creations.py` | 解析并修复 lib/creations.ts 结构 |
| `deploy.sh` | 推送 GitHub → 触发 Vercel 自动部署 |

## 安装 | Install

```bash
git clone https://github.com/kylinzhao/hermes-portfolio-creations-rebuilder.git
cd hermes-portfolio-creations-rebuilder
cp SKILL.md ~/.hermes/skills/devops/portfolio-creations-rebuilder/SKILL.md
```

## 目录结构 | Structure

```
.
├── SKILL.md    # Skill 定义文件
└── README.md   # 本文件
```

## 相关链接 | Links

- [zhaoliang-portfolio](https://github.com/kylinzhao/zhaoliang-portfolio)
- [Hermes Agent](https://github.com/kylinzhao/hermes)

---

*Powered by Hermes Agent · MIT License*
