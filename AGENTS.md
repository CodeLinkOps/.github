# CodeLinkOps — Agent 工作约束

给在 CodeLinkOps 任一仓库里干活的 AI agent（Claude Code / Codex / Cursor / 任何 MCP 客户端）。
完整规范与理由见 [README.md](README.md)；这份是**可直接抄进各仓库 `AGENTS.md`** 的精简版。

Claude Code 用户：仓库根目录放一个 `CLAUDE.md`，内容只需一行 `@AGENTS.md`。

---

## 硬约束

1. **一切改动走 PR。** 不许直接 push 到 `dev` / `prod`，不许 force push 到这两个分支。
2. **工作分支命名** `<type>/<issue号>-<描述>`，type ∈ `feat` `fix` `chore` `docs`。例：`feat/128-milestone-rollup`。
3. **每个 commit 必须带 agent 署名 trailer**，二选一：
   - `Co-Authored-By: Claude Opus 5 <noreply@anthropic.com>`（Claude Code 默认会加）
   - `Agent: <工具>/<模型>`，例 `Agent: codex/gpt-5`

   CI 会检查。带 `break-glass` 标签的 PR 放行。
4. **PR 标题用 Conventional Commits**，正文必须有 `Closes #<issue号>`（跨仓库写 `owner/repo#n`）。
5. **一个 PR 对应一个 issue。** 顺手发现的别的问题，另开 issue、另开 PR。
6. **没有 `DESIGN.md` 不要开始写实现。** 先补设计文档，或者问人要。
7. **密钥不进仓库。** 新增环境变量必须同时更新 `.env.example`。
8. **回滚用 `git revert` 开 PR**，不改历史。

## 技术选型（偏离要在 DESIGN.md 写明理由）

- 后端：**Rust**
- Web：**React**（Vue 需理由）
- App：**Flutter**（严格；不用 React Native、不写原生双份）
- 脚本：shell 或 Rust；Python 仅限数据 / ML
- Node 包管理器：**pnpm**
- 版本钉死：`rust-toolchain.toml` / `.nvmrc` + `packageManager` / `.fvmrc`；lockfile 必须提交

## 建 issue 时

- **必须挂里程碑**，否则它在进度统计里不存在。
- **必须至少一个 `type/` 标签和一个 `area/` 标签**（`area/` 决定派给谁）。
  - `type/`：`feat` `fix` `chore` `docs`
  - `area/`：`backend` `web` `app` `infra` `docs`
  - 优先级：`p0` `p1` `p2`；状态：`blocked` `needs-design` `break-glass`
- **粒度 = 一次会话能做完的一个功能点。** 进度按 issue 条数算，不按工作量 —— 把十件事塞进一条，里程碑会长期停在 0% 然后突然跳到 100%。

## 写 AGENTS.md 时（本仓库特有的那部分）

只写**从代码里看不出来的东西**：为什么这么写、踩过什么坑、改动时容易碰坏什么。

不要写目录结构和模块清单 —— agent 自己看得出来，而且那些会过时，过时的说明比没有说明更误导。

## 提交之前

- 构建通过、lint 零警告、测试通过。
- 改了行为就改文档（`README.md` / `DESIGN.md` / `AGENTS.md`）。
- 别提交注释掉的代码和调试输出。

## 你不该自己决定的事

碰到这几类，**停下来问人**，不要自行选择：

- 改动 `prod` 分支、发布、回滚生产
- 引入一个新的语言 / 框架 / 重量级依赖
- 改数据库 schema 中已上线的部分
- 删除任何看起来没被引用的东西 —— 「没有引用」和「引用在另一条路径上」在搜索结果里长得一样
- 修改本文件或 org 规范
