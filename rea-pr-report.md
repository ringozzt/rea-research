# REA（morluto/rea）近期 PR 与版本调研
查询时间：2026-10-19 约 01:55 北京时间（cursor-github 实时数据）。以下时间均为北京时间（UTC+8）。

## 0. 仓库确认
- **morluto/rea**：https://github.com/morluto/rea ，"Reverse engineer anything with agents"，MIT 协议，官网 https://rea.tools 。
  - Star **319,197**（10-17 查询时是 288,316；#4778 "celebrate 300,000 GitHub stars" 已合并），Fork 80,957，open issue+PR 合计 1,231。
  - 创建于 2026-04-14，最后 push 是 10-19 01:43。topic 里有 dsh、dsh-plugin、cordis、claude-code、codex、mcp。
  - **补充核实**：之前把它列为"是否基于 DSH 不确定"。现在可以确认，它是一个**多客户端的 MCP 逆向工具链，支持 DSH 作为客户端之一**（`rea setup --client dsh`，会写 Cordis patch 和 skill），并不是基于 DSH 构建的。
- 用户自己的仓库：**ringozzt/rea-research**（https://github.com/ringozzt/rea-research ，62 星，31 Fork，创建于 10-11，最后 push 10-15）："REA 深度技术调研：让 AI coding agent 无源码逆向软件的 MCP 工具链"。这次没有读它的内容。

## 1. 节奏与规模
- 近 30 天（09-19 起）**合并了 1,507 个 PR**（search API 的 total_count）。
- 版本由 release-please 自动发布，发得非常密：6.0.0（10-08）、6.1.0（10-09）、6.2.0（10-10）、6.3.0（10-10）、6.4.0（10-11）、6.5.0（10-14）、6.6.0（10-18 11:40，**打了 tag 但没发布到 npm**）、6.7.0（10-18 15:45）；6.8.0 的发布 PR #5004 还没合并。
- 活跃贡献者很多：gensokyo-dev、N0zoM1z0（看起来是核心维护者）、kaoru0822-kitauji（一人一天合并十几个 fix）、Adolanium、ShuaixinHuang 等。

## 2. 改进主题（引用 PR / 版本说明）
### 新功能
- **Kuna 静态分析后端**（6.6.0）：#3832 #3854 #4071 #4095；只在 Linux 上用 Kuna 1.833 实测过，Windows / PE / 通用 Mach-O 还不支持。
- **Ghidra 支持 Windows ARM PE**（#3965，对应 issue #2186）；Hopper Linux 嵌入运行时的修复和校验（#5041）。
- **保留分析结果的查询**：`inspect_analysis_view` 可以分页查 JS 事实、Android manifest、Dart AOT、WASM、EVM 等摘要，不用重新跑分析（6.5.0 / 6.6.0：#2122 #4265 等）。
- **一次解码多个 APK 语言包**（#4804，对应 #4777）。
- **离线检查**：SQLite 快照、包和信号录制、WASM、Rust 依赖元数据、Electron fuse（6.5.0）。
- **打开的功能 PR**（还没合并）：流式 HTTP 的 MCP 服务（#4955）、radare2 一次性命令（#3521）、EVM / Solidity 编译产物检查（#3024）、ADB 屏幕视图层级（#5049）、环境清单导出和回放（#4954 #4999 #4975）。

### 性能
- #4754：Ghidra 在后端汇总 binary overview，避免把完整过程表和字符串表送到 Node 再计数（原来 6 次调用会引发 6 次完整读取）。
- #5017：Hopper 字符串字面量搜索复用已解码的记录；实测：暖查询中位数 14.849 → 12.628 ms，但整任务更慢（89.384 → 93.630 秒），PR **明确不宣称整体加速**。
- #4990 #4962 #4791：apktool、analysis-view、Android 类页序列化这些局部复用和过滤优化。

### 可靠性 / Evidence 正确性（近期最大的主题）
- 大量 PR 保证"取消、失败或部分清理时，已完成的观察结果还在"（6.5.0 / 6.6.0 亮点；#4766 #4762 #4764 #4771 等）。
- Evidence 身份和覆盖率字段更严格：#4442、#4761、#4764。
- JavaScript 恢复边界情况：#4725 #4759 #4757 #4755 #4922 #4017 #3803。
- Electron / 浏览器隔离：#4389（19 条评论）、#4387、#4668、#4731、#1588 #1590。
- Flutter / IDA：#4765 #4766 #4645 #4286。

### DSH / Claude Code / Codex 集成
- **DSH**：打开的 #4419 要把 skill 装到 `$DSH_HOME/skills`（默认 `~/.dsh/skills`），而不是共享的 `~/.agents/skills`，避免污染 kimi、pi、codex、grok 等其他客户端。6.5.0 起已支持 DeepSeek Harness 作为 setup 客户端（版本说明原文）。
- **Claude Code**：#4944 等，只接受严格 JSON 配置（含 BOM），拒绝注释和尾逗号。打开的 #5045 对 Claude Desktop 做同样处理。
- **Codex**：版本说明里提到了入门改进（#1613）、保留 TOML 注释（#1204）。打开的 #4937 用无损方式读有符号的 64 位 TOML 整数。
- 其他客户端：打开的 #2492 增加 CodeBuddy；版本说明里还提到 Qoder、Trae、ZCode、Kimi Code、Factory Droid、WorkBuddy。

### 安全 / 沙箱
- 近期版本说明和已抽样的 PR 正文里，**没有**以"security"为主题的改动。浏览器和 Electron 相关修复（#1590 等）更多是证据隔离，不是权限收紧。

### 破坏性变更（迁移成本）
- 6.6.0：Evidence / 快照必须显式带上可空的 `relationship_coverage`；旧清单要重新生成，**不做自动迁移**。
- 6.5.0：`attached` 可以为 null；source map 的源名可以为 null；`--filter-output` 输出 `null` 而不是 `{}`。

## 3. 效果评估（只写有证据的）
- **基准数字**：#5017 有实验收据（experiments/analysis-efficiency/hopper-string-records.md），但作者承认没有整体加速。其他抽样到的 PR 没有给出百分比或加速倍数。
- **Issue 关闭**：很多 PR 写了 `Fixes #xxxx`（比如 #4754 修 #4746，#5017 修 #5010，#4804 修 #4777），但本次没有统计近 30 天一共关了多少 issue。
- **回滚**：近几版 changelog 里搜不到 "revert"。
- **用户反应**：抽样到的 PR 评论数一般是 2–6；#4389（Electron）有 19 条评论；reactions 大多是 1–3。没有找到有代表性的用户表扬或差评。
- **版本质量信号**：6.6.0 打了 tag 但没发到 npm；6.7.0 一天后就出了，主要修配置语义。说明发布节奏快，同时有小回潮和补丁。

## 4. 未核实 / 局限
- 近 30 天有 1,507 个合并 PR，这次只抽样了约 50 个搜索结果 + 3 个详细正文 + 4 个版本说明；主题归类是归纳，不是全量统计。
- ringozzt/rea-research 的内容没读。
- 没有用 GitHub 的"issue 关闭趋势"或外部评测（如博客）做交叉验证。
