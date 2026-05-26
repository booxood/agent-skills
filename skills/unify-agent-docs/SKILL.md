---
name: unify-agent-docs
description: Use this skill when the user wants to unify or consolidate agent entry docs (CLAUDE.md, AGENTS.md, GEMINI.md) in a project so AGENTS.md is the single source of truth and CLAUDE.md/GEMINI.md become thin `@AGENTS.md` references. Triggers include phrases like "统一 CLAUDE.md 和 AGENTS.md"、"合并 agent 文档"、"unify agent docs"、"把 GEMINI.md 也指向 AGENTS.md"、"统一 agent 配置".
---

# Unify Agent Docs

## 目标

把项目根目录下的 agent 入口文档收敛到一种统一形态：

- `AGENTS.md` 是**唯一信息源**，承载所有实际内容（行为准则、项目概览、约定、文档索引…）。
- `CLAUDE.md`、`GEMINI.md` 是 **pointer 文件**，内容只有一行：

  ```
  @AGENTS.md
  ```

这样 Claude Code、Codex、Gemini CLI 等 agent 都读到同一份内容，避免多份漂移。

## 处理范围

**只处理项目根目录下的这三个文件**：`AGENTS.md`、`CLAUDE.md`、`GEMINI.md`。

不要扩展到：
- `.cursorrules`、`.windsurfrules`
- `.codex/` 目录
- 子目录中的同名文件（monorepo 子包的 AGENTS.md 不要动）

## 流程

### 第 1 步：先把 AGENTS.md 落地

判断项目根目录下三个文件的存在情况：

| AGENTS.md | CLAUDE.md | GEMINI.md | 动作 |
|---|---|---|---|
| 存在 | — | — | 直接进第 2 步 |
| 缺失 | 都缺失 | — | **不生成**任何文件。告知用户：本仓库还没有 agent 入口文档，建议先用 `/init` 或人工写一份 AGENTS.md 后再回来跑本 skill。结束。|
| 缺失 | 仅 CLAUDE.md 存在 | 缺失 | `git mv CLAUDE.md AGENTS.md`（非 git 仓库回退为 `mv`），进第 2 步 |
| 缺失 | 缺失 | 仅 GEMINI.md 存在 | `git mv GEMINI.md AGENTS.md`（同上），进第 2 步 |
| 缺失 | 存在 | 存在 | **冲突**：先跑 `diff CLAUDE.md GEMINI.md` 展示给用户，再用 AskUserQuestion 让用户选哪个作为种子升级为 AGENTS.md（或选「我先手动合并好再跑」直接退出）。选中的那个 `git mv` 成 AGENTS.md，另一个留给第 2 步处理。|

### 第 2 步：把每个 pointer 文件收敛为 `@AGENTS.md`

对 `CLAUDE.md` 和 `GEMINI.md` **分别独立**处理。对每个文件按下表判断：

| pointer 文件状态 | 动作 |
|---|---|
| 不存在 | 创建该文件，内容仅为 `@AGENTS.md\n` |
| 已是 `@AGENTS.md`（去除首尾空白后即此字符串） | 跳过，告诉用户「已是目标状态」|
| 内容与 AGENTS.md 字节级相同 | 删除该文件（`git rm` 优先），再创建同名文件内容为 `@AGENTS.md\n` |
| 与 AGENTS.md 内容不同 | **冲突**：不自动改写。跑 `diff <pointer> AGENTS.md` 展示，再用 AskUserQuestion 提供三个选项：①以 AGENTS.md 为准（把该 pointer 覆盖为 `@AGENTS.md`）；②以该 pointer 为准（用其内容覆盖 AGENTS.md）；③跳过此文件，我手动合并。<br>选 ② 时要提醒：其它已经引用化的 pointer 不需要再改，但因为 AGENTS.md 内容变了，它们最终引用到的内容也跟着变了，这是符合预期的。|

### 第 3 步：验证 & 收尾

- 对实际存在的 pointer 文件跑 `cat`，确认内容是 `@AGENTS.md`。
- 确认 `AGENTS.md` 存在且非空。
- 在 git 仓库里跑 `git status` 把变更展示给用户。
- **不要自动 commit**，把 commit 决定权留给用户。

## 不要做的事

- **不要修改 AGENTS.md 的实际内容**，除非用户在第 2 步冲突中明确选了「以某个 pointer 为准」要做覆盖。
- **不要扩展到处理范围外的文件**（`.cursorrules` / `.windsurfrules` / `.codex/` 等）。
- **不要自动 git commit**。
- **不要递归处理子目录**——monorepo 子包里的 AGENTS.md 必须保持原状。
- **不要给 AGENTS.md 增删自己想加的章节**——本 skill 只做指针化，不做内容编辑。
