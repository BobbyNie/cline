@.clinerules/general.md
@.clinerules/network.md
@.clinerules/cli.md

## 重要：本仓库为内网模式运行

**本仓库 (`BobbyNie/cline`) 是 cline/cline 的 fork，默认以内网模式运行。**

内网模式现在是**默认行为**（不需要设置环境变量）。设置 `CLINE_INTRANET_MODE=false` 可以关闭内网模式。

所有修改和上游同步工作必须遵守以下原则：

1. **保留内网模式代码** — 内网模式相关代码守护（PostHog 跳过、远程配置跳过、BannerService 跳过、Auth token refresh 跳过、MCP Marketplace 跳过）不得被同步覆盖或删除
2. **上游同步时注意冲突** — 同步 cline/cline 上游更新时，如果上游修改了 `src/config.ts`、`src/common.ts`、`src/core/controller/index.ts`、`src/core/storage/remote-config/fetch.ts`、`src/shared/config-types.ts` 这些文件，必须手动检查并保留内网模式的改动
3. **内网相关文件清单**：
   - `src/config.ts` — `intranetFlags` getter（默认 `isIntranetMode=true`，需显式设 `"false"` 关闭）
   - `src/common.ts` — PostHog 初始化的内网模式检查
   - `src/core/controller/index.ts` — BannerService、Auth token refresh、Remote config timer、MCP Marketplace 的内网模式守护
   - `src/core/storage/remote-config/fetch.ts` — 远程配置获取的内网模式检查
   - `.github/workflows/intranet-build.yml` — 内网构建和发布工作流（默认内网模式）
   - `.github/workflows/test.yml` — 上游测试工作流（需设置 `CLINE_INTRANET_MODE=false`）
   - `docs/intranet-mode.md` — 内网模式文档
4. **Release 流程** — 每次 push 到 main 自动触发 CI + 发布，使用 `intranet-v*` 标签，构建 `.vsix` 文件
5. **测试** — 内网模式相关测试在 `src/__tests__/config.test.ts` 中；marketplace 测试需禁用内网模式
6. **CI 注意** — `test.yml` 的所有 job 必须设置 `CLINE_INTRANET_MODE=false`，否则 marketplace 等测试会失败
