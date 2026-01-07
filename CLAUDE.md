# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

StudyNotes 是一个个人学习笔记项目，主要用 Markdown 文档记录游戏开发、编程语言、操作系统等领域的学习内容。

## 项目结构

- `coding/` - 编程开发相关笔记（游戏开发、Unity、Go、C#、Java、Node.js 等）
- `os/` - 操作系统笔记（Linux、Windows、vim 配置等）
- `Learning/` - 英语和其他学习资料

## 工作流程

### 新增笔记
1. 在对应分类目录下创建新的 `.md` 文件
2. 使用中文标题和中文标点符号
3. 文档开头使用以下格式：

```markdown
# 标题名称

## 目录

- [标题名称](#标题名称)
  - [目录](#目录)
  - [子章节](#子章节)
```

### Git 提交
- 使用 `/commit` skill 生成符合 Conventional Commits 规范的中文提交信息
- 使用 `/git-worktree` skill 在平级目录创建独立工作树进行开发

## 语言规范

- 所有内容使用简体中文
- 使用中文标点符号（，。：；？！""）
- 标题使用中文
