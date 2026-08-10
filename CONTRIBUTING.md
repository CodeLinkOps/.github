# 怎么贡献

完整规范在 [工程规范](README.md)，agent 读 [AGENTS.md](AGENTS.md)。这里是最短版本。

1. **代码由 AI 改、由 AI 提交。** 人提需求、审设计、review PR、点 Merge，并为合并的结果负责。
2. **先有 `DESIGN.md`**，再有代码。功能点逐条建 issue 并挂里程碑。
3. **工作分支** `feat|fix|chore|docs/<issue号>-<描述>`，PR 合到 `dev`；`dev` → `prod` 是一次发布。
4. **PR 标题用 Conventional Commits**，正文写 `Closes #<issue号>`；一个 PR 对应一个 issue。
5. **每个 commit 带 agent 署名** —— `Co-Authored-By: Claude ...` 或 `Agent: <工具>/<模型>`。CI 会检查。
6. **紧急情况可以破例**，打 `break-glass` 标签、在 PR 里说明，**事后补一条 issue**说明这次破例暴露了什么问题。

不确定的时候：停下来问，别猜。
