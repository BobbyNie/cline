# Intranet Mode Configuration Guide

## Overview

When using Cline in an intranet environment, the first conversation may experience long waits due to network requests to external services. Intranet mode disables these network requests to improve user experience.

## Enabling Intranet Mode

### Method 1: Environment Variable

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

### Method 3: Build from Source with Intranet Mode

```bash
git clone https://github.com/cline/cline.git
cd cline
CLINE_INTRANET_MODE=true npm run compile
```

## What Intranet Mode Disables

When intranet mode is enabled, the following external network requests are skipped:

- **PostHog Analytics**: Telemetry and usage analytics are not initialized
- **Remote Configuration**: Periodic fetching of remote config is disabled
- **Remote Config Timer**: The 1-hour polling timer for remote config is not started

## What Still Works

These features continue to work normally in intranet mode:

- All AI conversation functionality
- File operations (read, write, edit)
- MCP servers (local or intranet-accessible)
- Command execution
- Code search and indexing
- Task history

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
