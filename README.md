# Portfolio Creations Rebuilder

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

重建 zhaoliang-portfolio 网站 lib/creations.ts 的完整流水线，适用于内容结构损坏或历史数据回填。

Rebuild lib/creations.ts pipeline for portfolio site: fix nested structures, append history, deploy.

## 安装 / Install

```bash
# 克隆仓库
git clone https://github.com/kylinzhao/hermes-portfolio-creations-rebuilder.git
cd hermes-portfolio-creations-rebuilder

# 复制 skill 到 Hermes 目录
cp -r skills/* ~/.hermes/skills/

# 重启 Hermes Agent 使其加载新 skill
```

## 使用方法

加载 skill 后，Hermes Agent 会自动识别并根据相关对话场景触发。详细用法请参考 [SKILL.md](SKILL.md)。

## 目录结构

```
.
├── SKILL.md    # Skill 定义文件（Hermes Agent 标准格式）
└── README.md   # 本文件
```

## 相关链接

- [Hermes Agent](https://github.com/kylinzhao/hermes)
- [我的其他 Skills](https://github.com/kylinzhao?tab=repositories&q=hermes-)

---

*Powered by Hermes Agent · MIT License*
