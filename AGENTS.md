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
9. **产出一律用中文**：文档、注释、commit 消息、issue 与 PR。代码标识符、日志字段名、第三方引用（报错原文、命令输出、外部文档摘录）保持原样。

## 技术选型（偏离要在 DESIGN.md 写明理由）

- 后端：**Rust**
- Web：**React**（Vue 需理由）
- App：**Flutter**（严格；不用 React Native、不写原生双份）
- 脚本：shell 或 Rust；Python 仅限数据 / ML
- Node 包管理器：**pnpm**
- 运行形态：**Kubernetes**（生产与预发；本地开发用 compose 随意）
- 镜像仓库：**私有 GHCR**，`ghcr.io/codelinkops/<仓库名>`
- 版本钉死：`rust-toolchain.toml` / `.nvmrc` + `packageManager` / `.fvmrc`；lockfile 必须提交

## 部署

**打 tag 触发的是「构建 + 开 PR」，不是「部署」。集群里跑什么，由 Git 里写着什么决定。**

在 `prod` 分支打 `v1.2.3` 之后：CI 构建镜像推 GHCR（tag 为 `v1.2.3` 和 `sha-<短sha>`）
→ CI 开一个只改 image tag 那一行的 PR → 人审、合并 → 同步器把集群拉到位。
`dev` 分支合并可以自动更新 dev 环境；**prod 永远要人点一下。**

- **镜像 tag 不可变**：推上去就不重推，不用 `latest`。重推同一个 tag 会让「集群里跑的是哪份代码」查不出来，而且 rollout 不触发 —— 新 pod 是新版、老 pod 是旧版。
- 镜像必须打 `org.opencontainers.image.source` 标签指向仓库，否则包和仓库关联不上。
- **清单里 image tag 必须加引号**：`tag: "1.0"` 是字符串，`tag: 1.0` 是浮点数，会变成 `1`。
- 每个环境一个目录（`deploy/dev/`、`deploy/prod/`），不要一份清单加条件判断。
- 副本数由 HPA 管的话，清单里**不要写 `replicas`** —— 会和 HPA 互相覆盖。
- **回滚 = `git revert` 那个改 tag 的 PR 再合并。** 不用 `helm rollback` / `argocd rollback` / `kubectl rollout undo` —— 那些绕过 Git，同步器会改回去，表现成「回滚了又变回去」。
- **不要手改生产集群**（`kubectl apply` / `edit` / `helm upgrade`）。只读的 `logs` / `describe` / `events` / `get` 随便用。
- **「CI 绿了」和「PR 合了」都不算部署完成**，同步是异步的。判据是 Synced + Healthy。在那之前说「已触发」，别说「已上线」。
- 清理镜像前先查集群在用哪些 tag，**必须算上 `initContainers`** —— 漏了它会删掉只在 init 阶段用的镜像，然后下次 pod 起不来。
- 密钥不进 Git，包括部署清单。用 sealed-secrets / external-secrets 或集群侧手工建。

## 建 issue 时

- **必须挂里程碑**，否则它在进度统计里不存在。
- **必须至少一个 `type/` 标签和一个 `area/` 标签**（`area/` 决定派给谁）。
  - `type/`：`feat` `fix` `chore` `docs`
  - `area/`：`backend` `web` `app` `infra` `docs`
  - 优先级：`p0` `p1` `p2`；状态：`blocked` `needs-design` `break-glass`
- **粒度 = 一次会话能做完的一个功能点。** 进度按 issue 条数算，不按工作量 —— 把十件事塞进一条，里程碑会长期停在 0% 然后突然跳到 100%。

## 收到「跟上 org 规范」的 issue 时

org 规范改了而你所在的仓库还没跟上时，`spec-drift` 会开一条带 `spec-drift`
标签的 issue，附上 compare 链接。**这条 issue 优先做** —— 在同步之前，
你读到的 `AGENTS.md` 是旧规则，照着它干活会做出与规范冲突的东西，
而不会有任何征兆。

同步的三步写在那条 issue 里：抄条款进本仓库 `AGENTS.md` →
`DESIGN.md` 跟着调章节（**注意检查交叉引用**）→ 用 issue 里给的内容覆盖
`.org-spec.json`。跟上之后它会自动关闭。

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
