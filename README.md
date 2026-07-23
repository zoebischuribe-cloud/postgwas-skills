# postgwas-skills

大家好，我目前主要从事生物信息学与 Post-GWAS 分析相关的研究。这个仓库提供了一套可复用的 AI Skills，帮助研究者快速完成 GWAS 后续分析流程。
Hello, I'm lilaoban, working on bioinformatics and Post-GWAS analysis. This repository provides reusable AI Skills to help researchers quickly complete post-GWAS analysis workflows.

## 🎯 适用场景

如果你正在做以下分析，这个仓库适合你：
- 孟德尔随机化（Mendelian Randomization, MR）
- 共定位分析（Colocalization）
- TWAS / S-PrediXcan
- Fine-mapping
- QTL 整合分析（eQTL, sQTL, pQTL, sc-eQTL）
- 双性状共病分析

## ⭐ Star History

[![Star History Chart](https://api.star-history.com/svg?repos=zoebischuribe-cloud/postgwas-skills&type=Date&cache_bust=2026-07-23T14

---

## Installation

`postgwas-skills` 是基于 `SKILL.md` 的可复用指令包。每个 `pgwas-*` 目录是一个可安装单元。

### 1. Codex

```bash
git clone https://github.com/zoebischuribe-cloud/postgwas-skills.git
cd postgwas-skills

# 安装单个技能
mkdir -p ~/.codex/skills
cp -R skills/pgwas-mr ~/.codex/skills/

# 安装所有技能
for d in skills/pgwas-*; do
  cp -R "$d" ~/.codex/skills/
done
```

重启 Codex 后即可使用。

### 2. Claude Code

Claude Code 使用 Subagents 机制：

```bash
mkdir -p ~/.claude/agents
cp skills/pgwas-mr/SKILL.md ~/.claude/agents/pgwas-mr.md
```

然后编辑 `~/.claude/agents/pgwas-mr.md`，确保 frontmatter 格式正确：

```yaml
---
name: pgwas-mr
description: Mendelian Randomization analysis workflow for causal inference. Use when user mentions MR, Mendelian randomization, causal inference, or two-sample MR.
---
```

### 3. OpenClaw / QClaw

将技能目录复制到 OpenClaw skills 目录：

```bash
cp -R skills/pgwas-mr ~/.openclaw/workspace/skills/
```

---

## Skill Index

| Skill | Status | Purpose | Trigger keywords |
|-------|--------|---------|------------------|
| [`pgwas-mr`](skills/pgwas-mr/README.md) | Beta | Two-sample Mendelian Randomization workflow | "MR", "Mendelian randomization", "causal inference", "two-sample MR" |
| [`pgwas-coloc`](skills/pgwas-coloc/README.md) | Beta | Colocalization analysis for shared causal variants | "colocalization", "coloc", "shared causal variant" |

> 更多技能开发中: pgwas-twas, pgwas-finemapping...

---

## 📢 关于作者


如有合作意向，欢迎通过 GitHub Issues 联系。

---

## Contributing

欢迎提交 Issue 和 PR！请遵循以下格式：

1. 新技能目录结构：
```
pgwas-<topic>/
├── README.md
├── SKILL.md
└── references/
    └── *.md
```

2. SKILL.md 必须包含有效的 YAML frontmatter

## License

MIT License
