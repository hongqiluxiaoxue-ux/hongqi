# Obsidian 知识库整理方法（LLM Wiki 标准）

> 作者：方向东 · 建立日期：2026-06-06  
> 参考架构：[Karpathy LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9Billboard/442a6bf555914893e9f)  
> 适用工具：Claude Code（claude.ai/code）+ Obsidian

---

## 核心思想

传统 RAG 方式每次提问都要从原始文档中重新检索，没有积累。  
**LLM Wiki 模式不同**：LLM 主动维护一个持久化的 Wiki，每次新增资料都更新 Wiki 条目，知识不断复利增长。

```
原始资料（raw）→ LLM 整理 → Wiki 条目（持久化）→ 查询时直接读取
```

你负责：提供资料来源、提问、把控方向  
Claude 负责：摘要、交叉引用、整理、维护 Wiki

---

## 三层文件夹结构

```
<知识库根目录>/
├── raw/          ← 原始资料（只读，不修改）
│   └── ...       ← 论文、笔记、PDF、网页剪藏等
├── Wiki/         ← Claude 维护的结构化知识条目
│   ├── 00-导航/
│   │   ├── index.md    ← 全库总目录
│   │   ├── log.md      ← 操作日志（追加式）
│   │   └── SCHEMA.md   ← 本库的维护规范
│   ├── 01-主题A/
│   ├── 02-主题B/
│   └── ...
└── outputs/      ← 对话总结、生成文章、汇报材料
    └── README.md
```

**规则：**
- `raw/`：用户放入，Claude 只读不改
- `Wiki/`：Claude 全权维护，用户只读浏览
- `outputs/`：每次对话生成的重要内容存放处

---

## 七步标准流程

### 第一步：探查现状

```
列出所有文件（含大小）→ 识别主题分布 → 检测重复和垃圾
```

- 相同大小文件 → 计算 MD5 哈希 → 确认内容相同后删除
- 垃圾文件：macOS `._` 资源分叉文件、0字节/极小存根文件

### 第二步：建立三层目录

按上方结构创建 `raw/`、`Wiki/`（含编号子文件夹）、`outputs/`

### 第三步：迁移与清理

- 将现有原始资料移入 `raw/`
- 删除 macOS 垃圾文件（`._` 前缀）
- 删除确认的重复文件（保留版本更完整的）
- 将旧版索引重命名为 `raw/📚 旧版知识库索引.md`

### 第四步：规划 Wiki 分类

- 根据内容划分 3—8 个编号主题文件夹
- 每个主题 2—6 个 Wiki 条目
- 条目是**综合整理**，不是原文复制

### 第五步：撰写 Wiki 条目

每个条目格式见下方「Wiki 条目模板」

### 第六步：建立三个导航文件

| 文件 | 作用 |
|------|------|
| `Wiki/00-导航/index.md` | 全部条目的分类目录表格，含一行简介 |
| `Wiki/00-导航/log.md` | 操作日志，格式：`## [YYYY-MM-DD] 类型 \| 描述` |
| `Wiki/00-导航/SCHEMA.md` | 三层架构说明 + 分类体系 + 格式规范 + 操作规范 |

### 第七步：outputs 目录

创建 `outputs/README.md`，说明命名规范：`YYYY-MM-DD_主题描述.md`

---

## Wiki 条目模板

```markdown
---
tags: [主题标签1, 主题标签2]
aliases: [别名1, 英文名]
sources: [raw/对应原始文件路径]
updated: YYYY-MM-DD
---

# 条目标题

## 核心概念

（定义、来源、一句话总结）

## 主要内容

（可用表格、列表、代码块组织）

## 关键引用

> "重要原文引用"  
> ——出处

## 相关研究与案例

（具体案例分析，不能只有理论）

## 参见

- [[相关条目A]] · [[相关条目B]]
```

---

## 重复文件处理规则

| 情况 | 处理方式 |
|------|---------|
| MD5 哈希完全相同 | 删除其中一个，保留文件名更清晰的 |
| 大小相近但不同 | 读取内容对比，保留更完整版本，差异整合到 Wiki |
| 带版本号的历史版本（v0/v1/v2） | 移入 `raw/_历史版本/` 归档，不删除 |
| macOS `._` 前缀文件 | 直接删除（系统垃圾文件） |
| 0.3KB 以下的存根文件 | 读取确认是存根后删除 |

---

## 新增资料时的操作（Ingest 流程）

1. 将新资料放入 `raw/` 对应子文件夹
2. 告诉 Claude："请整合这篇资料到 Wiki"
3. Claude 读取资料 → 更新/新建相关 Wiki 条目 → 更新 `index.md`
4. Claude 在 `log.md` 追加一条记录
5. （可选）将新知识点生成的分析存入 `outputs/`

---

## 定期维护（Lint 流程）

定期告诉 Claude："请检查 Wiki 健康状态"，Claude 会：

- 检查孤立页面（无入链的条目）
- 发现矛盾内容（新旧资料之间的冲突）
- 补充缺失的交叉引用
- 建议应新建但尚未建立的条目

---

## 与 Claude Code 配合使用

### 在新电脑上调用此方法

1. 在 Obsidian 库根目录下创建 `CLAUDE.md`，写入：

```markdown
# 知识库维护规范

本知识库采用 LLM Wiki 三层架构（raw / Wiki / outputs）。
整理规范请参考：https://github.com/hongqiluxiaoxue-ux/hongqi/tree/master/obsidian-wiki-method/
具体操作请遵循 Wiki/00-导航/SCHEMA.md 中的规定。
```

2. 打开 Claude Code，进入知识库目录，Claude 会自动读取 `CLAUDE.md` 并按规范操作

### 同步到 GitHub

```powershell
# 首次推送
git init
git checkout -b obsidian-wiki
git add -A
git commit -m "更新：描述改动内容"
git remote add origin https://github.com/你的用户名/仓库名.git
git push -u origin obsidian-wiki

# 后续同步
git add -A
git commit -m "更新：描述改动内容"
git push
```

---

## 参考案例

**obsidian-设计**（天津城建大学·方向东，2026-06-06）

| 项目 | 内容 |
|------|------|
| 路径 | `D:\2026Claude-hong\教学\obsidian-设计\` |
| GitHub | `hongqiluxiaoxue-ux/hongqi`，分支 `obsidian-wiki` |
| Wiki 条目数 | 24 个 |
| 主题分类数 | 6 个 |
| 清理文件数 | 6 个（3个macOS垃圾 + 3个重复文件）|
| 原始资料 | 80 个文件 |

Wiki 6个主题：
1. 人工智能基础与历史（7条）
2. CMU 智能教育体系（4条）
3. 产教融合与设计教育（4条）
4. 设计思维与设计管理（3条）
5. 体验式学习（1条）
6. 研究项目追踪（2条）
