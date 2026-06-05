# 给 Claude Code 的指令：Obsidian 知识库整理

当用户要求整理 Obsidian 知识库时，必须严格遵循以下方法论。

## 核心原则

1. **所有输出用中文**
2. **采用三层架构**：`raw/`（原始资料）→ `Wiki/`（整理条目）→ `outputs/`（生成内容）
3. **Wiki 条目是综合整理**，不是原文复制——将多个原始文件的知识点合并为有结构的页面
4. **raw/ 只读**：永远不修改 raw/ 中的原始文件内容

## 执行步骤

### Step 1：探查
```powershell
# 列出所有文件和大小
Get-ChildItem -Recurse "知识库路径" | Select-Object FullName, Length
```
- 识别主题分布
- 找出重复文件（相同大小 → MD5 对比）
- 找出垃圾文件（`._` 前缀、极小存根）

### Step 2：创建目录结构
```powershell
$dirs = @("raw", "Wiki", "Wiki\00-导航", "Wiki\01-主题A", "outputs")
foreach ($d in $dirs) { New-Item -ItemType Directory -Force "$base\$d" }
```

### Step 3：迁移与清理
- 将现有文件夹移入 `raw/`
- 删除 `._` 前缀的 macOS 垃圾文件
- 删除 MD5 确认的重复文件（保留更完整版本）
- 旧版索引重命名为 `raw/📚 旧版知识库索引.md`

### Step 4：撰写 Wiki 条目
每个条目包含：
```yaml
---
tags: [标签]
aliases: [别名]
sources: [raw/源文件路径]
updated: YYYY-MM-DD
---
```
正文：核心概念 → 主要内容（含表格）→ 关键引用 → 参见（[[双链]]）

### Step 5：建立三个导航文件
- `Wiki/00-导航/index.md`：全部条目的表格目录
- `Wiki/00-导航/log.md`：操作日志（`## [日期] 类型 | 描述`）
- `Wiki/00-导航/SCHEMA.md`：本库维护规范（参考 SCHEMA-template.md）

### Step 6：outputs 目录
创建 `outputs/README.md`，说明命名规范。

## 文件编码注意事项

- 所有新建 Wiki 文件使用 UTF-8 编码
- 用 PowerShell 写文件时：`[System.IO.File]::WriteAllText(path, content, [System.Text.Encoding]::UTF8)`
- 读取现有中文文件时：先尝试 UTF-8，如乱码则尝试 GB18030

## 重复文件判断

```powershell
# 相同大小的文件对比 MD5
Get-FileHash "文件1" -Algorithm MD5
Get-FileHash "文件2" -Algorithm MD5
# Hash 相同 → 删除其中一个
```

## 完成后的验证

```powershell
$wiki = Get-ChildItem -Recurse "Wiki/" -File
$raw = Get-ChildItem -Recurse "raw/" -File
Write-Host "Wiki条目: $($wiki.Count) | 原始资料: $($raw.Count)"
```

## 参考案例

obsidian-设计（2026-06-06）：
- 24个 Wiki 条目，6个主题，80个原始资料文件
- GitHub: hongqiluxiaoxue-ux/hongqi，分支 obsidian-wiki
