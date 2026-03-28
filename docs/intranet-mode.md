# Intranet Mode Configuration Guide

## Overview

When using Cline in an intranet environment, the first conversation may experience long waits due to network requests to external services (PostHog analytics, remote config, auth token refresh, etc.). Intranet mode disables these network requests to improve user experience.

## Enabling Intranet Mode

### Method 1: Environment Variable (Recommended)

Set the environment variable before launching VS Code:

```bash
export CLINE_INTRANET_MODE=true
code
```

### Method 2: VS Code Settings

Add to your VS Code `settings.json`:

```json
{
  "terminal.integrated.env.osx": {
    "CLINE_INTRANET_MODE": "true"
  },
  "terminal.integrated.env.linux": {
    "CLINE_INTRANET_MODE": "true"
  },
  "terminal.integrated.env.windows": {
    "CLINE_INTRANET_MODE": "true"
  }
}
```

### Method 3: Pre-built Intranet Release

Download the intranet release `.vsix` from [GitHub Releases](https://github.com/BobbyNie/cline/releases) (tagged `intranet-v*`). This build has `CLINE_INTRANET_MODE=true` embedded in the CI environment, so the extension automatically operates in intranet mode.

To install:
1. Download `cline-intranet-*.vsix`
2. In VS Code, run `Extensions: Install from VSIX...`
3. Select the downloaded file

**Note:** `CLINE_INTRANET_MODE` is a **runtime** environment variable, not a build-time flag. For Method 1 and 2, the env var is read each time the extension starts. The pre-built release (Method 3) works because the CI workflow sets the env var at package time.

## What Intranet Mode Disables

When intranet mode is enabled, the following external network requests are skipped:

| Feature | Description | Guard Location |
|---------|-------------|----------------|
| **PostHog Analytics** | Telemetry and usage analytics initialization | `src/common.ts` |
| **Remote Configuration** | Periodic fetching of remote config | `src/core/storage/remote-config/fetch.ts` |
| **Remote Config Timer** | The 1-hour polling timer for remote config | `src/core/controller/index.ts` |
| **Banner Service** | External banner API fetches | `src/core/controller/index.ts` |
| **Auth Token Refresh** | Cline account auth token refresh | `src/core/controller/index.ts` |
| **MCP Marketplace** | External marketplace API fetches | `src/core/controller/index.ts` (on-demand) |

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

| Variable | Value | Description |
|----------|-------|-------------|
| `CLINE_INTRANET_MODE` | `true` | Enable intranet mode, disables all non-essential network requests |
| `CLINE_TELEMETRY_DISABLED` | `true` | Disable telemetry only, other network features remain active |
| `HTTP_PROXY` | `http://proxy:port` | HTTP proxy address |
| `HTTPS_PROXY` | `http://proxy:port` | HTTPS proxy address |

## Implementation Details

The intranet mode check flows through the configuration system:

1. `process.env.CLINE_INTRANET_MODE` is read by `ClineEnv.getEnvironment()` in `src/config.ts`
2. The result is stored in `EnvironmentConfig.isIntranetMode` and `EnvironmentConfig.telemetryDisabled`
3. Each network-touching module checks `ClineEnv.config().isIntranetMode` and returns early if true

### Key Files Modified

- `src/shared/config-types.ts` - Added `isIntranetMode` and `telemetryDisabled` to `EnvironmentConfig`
- `src/config.ts` - Added `intranetFlags` getter, updated all `getEnvironment()` return paths
- `src/common.ts` - Skip PostHog init in intranet mode
- `src/core/controller/index.ts` - Skip BannerService, auth token refresh, remote config timer, MCP marketplace
- `src/core/storage/remote-config/fetch.ts` - Skip remote config fetch
