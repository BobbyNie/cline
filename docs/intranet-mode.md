# Intranet Mode Configuration Guide

## Overview

This fork (`BobbyNie/cline`) defaults to **intranet mode**, which disables non-essential network requests (PostHog analytics, remote config, auth token refresh, MCP marketplace, etc.) to work seamlessly in air-gapped or restricted network environments.

## Default Behavior

**Intranet mode is enabled by default.** No configuration is needed — the extension automatically operates in intranet mode.

## Disabling Intranet Mode (for Development/Testing)

If you need external network features (e.g., for upstream development or testing):

### Method 1: Environment Variable

```bash
export CLINE_INTRANET_MODE=false
code
```

### Method 2: VS Code Settings

Add to your VS Code `settings.json`:

```json
{
  "terminal.integrated.env.osx": {
    "CLINE_INTRANET_MODE": "false"
  },
  "terminal.integrated.env.linux": {
    "CLINE_INTRANET_MODE": "false"
  },
  "terminal.integrated.env.windows": {
    "CLINE_INTRANET_MODE": "false"
  }
}
```

## What Intranet Mode Disables

When intranet mode is enabled, the following external network requests are skipped:

| Feature | Description | Guard Location |
|---------|-------------|----------------|
| **PostHog Analytics** | Telemetry and usage analytics initialization | `src/common.ts` |
| **Remote Configuration** | Periodic fetching of remote config | `src/core/storage/remote-config/fetch.ts` |
| **Remote Config Timer** | The 1-hour polling timer for remote config | `src/core/controller/index.ts` |
| **Banner Service** | External banner API fetches | `src/core/controller/index.ts` |
| **Auth Token Refresh** | Cline account auth token refresh | `src/core/controller/index.ts` |
| **MCP Marketplace** | External marketplace API fetches | `src/core/controller/index.ts` |

## What Still Works

These features continue to work normally in intranet mode:

- All AI conversation functionality (with local or intranet-accessible models)
- File operations (read, write, edit)
- MCP servers (local or intranet-accessible)
- Command execution and terminal access
- Code search, indexing, and task history
- Local model support (Ollama, LM Studio, etc.)

## Verification

To verify intranet mode is enabled:

1. Open VS Code Developer Tools (`Help > Toggle Developer Tools`)
2. Look for `[RemoteConfig] Intranet mode enabled, skipping remote config fetch` in the console
3. If you see this message, intranet mode is active

## Pre-built Intranet Release

Download the intranet release `.vsix` from [GitHub Releases](https://github.com/BobbyNie/cline/releases) (tagged `intranet-v*`).

To install:
1. Download `cline-intranet-*.vsix`
2. In VS Code, run `Extensions: Install from VSIX...`
3. Select the downloaded file

**Note:** The pre-built release already defaults to intranet mode. No additional configuration needed.

## Related Configuration

### Disable Telemetry Only

If you only want to disable telemetry but keep other network features:

```bash
export CLINE_TELEMETRY_DISABLED=true
```

### Proxy Configuration

If you need to access external services through a proxy:

```bash
export HTTP_PROXY=http://your-proxy:port
export HTTPS_PROXY=http://your-proxy:port
```

## Environment Variables Reference

| Variable | Default | Description |
|----------|---------|-------------|
| `CLINE_INTRANET_MODE` | `true` (default) | Set to `"false"` to disable intranet mode |
| `CLINE_TELEMETRY_DISABLED` | `true` (when intranet mode on) | Set to `"true"` to disable telemetry only |
| `HTTP_PROXY` | — | HTTP proxy address |
| `HTTPS_PROXY` | — | HTTPS proxy address |

## Implementation Details

The intranet mode check flows through the configuration system:

1. `process.env.CLINE_INTRANET_MODE` is read by `ClineEnv.getEnvironment()` in `src/config.ts`
   - Default: `true` (intranet mode enabled)
   - Only `process.env.CLINE_INTRANET_MODE === "false"` disables intranet mode
2. The result is stored in `EnvironmentConfig.isIntranetMode` and `EnvironmentConfig.telemetryDisabled`
3. Each network-touching module checks `ClineEnv.config().isIntranetMode` and returns early if true

### Key Files

- `src/config.ts` - `intranetFlags` getter with default-to-true logic
- `src/common.ts` - Skip PostHog init in intranet mode
- `src/core/controller/index.ts` - Skip BannerService, auth token refresh, remote config timer, MCP marketplace
- `src/core/storage/remote-config/fetch.ts` - Skip remote config fetch
- `.github/workflows/intranet-build.yml` - Intranet CI + auto-release workflow
- `.github/workflows/test.yml` - Upstream tests (sets `CLINE_INTRANET_MODE=false`)
