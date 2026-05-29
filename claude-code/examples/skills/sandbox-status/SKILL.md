> 📚 **AI Spark Wiki** · Claude Code 知识库

---
name: sandbox-status
description: 展示原生沙盒状态、配置信息及近期违规记录
effort: low
disable-model-invocation: true
---

# 沙盒状态命令

检查原生 Claude Code 沙盒的运行状态、激活配置及安全事件。

## 用法

```
/sandbox-status
```

## 功能说明

1. **检查沙盒可用性**
   - 验证 OS 原语是否已安装（Linux 上的 bubblewrap、macOS 上的 Seatbelt）
   - 展示平台支持状态

2. **展示激活配置**
   - 沙盒模式（自动允许 vs 常规权限）
   - 文件系统策略（允许写入的路径、禁止读取的路径）
   - 网络策略（域名白名单/黑名单）
   - 排除的命令

3. **列出近期沙盒违规记录**
   - 被拦截的文件系统访问尝试
   - 被拦截的网络连接
   - 逃生舱调用（`dangerouslyDisableSandbox`）

## 实现代码

```bash
#!/bin/bash

echo "=== Native Sandbox Status ==="
echo

# 1. Platform Check
echo "Platform:"
case "$OSTYPE" in
  darwin*)
    echo "  ✅ macOS (Seatbelt built-in)"
    ;;
  linux*)
    if which bubblewrap >/dev/null 2>&1; then
      echo "  ✅ Linux (bubblewrap installed)"
      bubblewrap --version 2>/dev/null | head -1
    else
      echo "  ❌ Linux (bubblewrap NOT installed)"
      echo "     Install: sudo apt-get install bubblewrap socat"
    fi
    if which socat >/dev/null 2>&1; then
      echo "  ✅ socat installed"
    else
      echo "  ❌ socat NOT installed"
    fi
    ;;
  *)
    echo "  ❌ Unsupported platform: $OSTYPE"
    ;;
esac
echo

# 2. Configuration
echo "Configuration (from settings.json):"
if [ -f .claude/settings.json ]; then
  CONFIG=".claude/settings.json"
elif [ -f ~/.claude/settings.json ]; then
  CONFIG="~/.claude/settings.json"
else
  echo "  ⚠️  No settings.json found"
  CONFIG=""
fi

if [ -n "$CONFIG" ]; then
  echo "  Source: $CONFIG"

  # Auto-allow mode
  AUTO_ALLOW=$(jq -r '.sandbox.autoAllowMode // "not set"' "$CONFIG" 2>/dev/null)
  echo "  Auto-allow: $AUTO_ALLOW"

  # Allowed write paths
  WRITE_PATHS=$(jq -r '.sandbox.filesystem.allowedWritePaths[]? // empty' "$CONFIG" 2>/dev/null | tr '\n' ', ')
  echo "  Allowed writes: ${WRITE_PATHS:-not set}"

  # Denied read paths
  DENIED_READS=$(jq -r '.sandbox.filesystem.deniedReadPaths[]? // empty' "$CONFIG" 2>/dev/null | tr '\n' ', ')
  echo "  Denied reads: ${DENIED_READS:-not set}"

  # Network policy
  NET_POLICY=$(jq -r '.sandbox.network.policy // "not set"' "$CONFIG" 2>/dev/null)
  echo "  Network policy: $NET_POLICY"

  # Allowed domains
  DOMAINS=$(jq -r '.sandbox.network.allowedDomains[]? // empty' "$CONFIG" 2>/dev/null | head -3 | tr '\n' ', ')
  DOMAINS_COUNT=$(jq -r '.sandbox.network.allowedDomains | length' "$CONFIG" 2>/dev/null)
  if [ -n "$DOMAINS" ]; then
    echo "  Allowed domains: $DOMAINS... ($DOMAINS_COUNT total)"
  else
    echo "  Allowed domains: not set"
  fi

  # Excluded commands
  EXCLUDED=$(jq -r '.sandbox.excludedCommands[]? // empty' "$CONFIG" 2>/dev/null | tr '\n' ', ')
  echo "  Excluded commands: ${EXCLUDED:-not set}"
fi
echo

# 3. Recent Violations (placeholder - actual implementation would read Claude Code logs)
echo "Recent sandbox violations:"
echo "  ℹ️  Log inspection not yet implemented"
echo "  Tip: Check Claude Code session logs for sandbox violation notifications"
echo

# 4. Open-Source Runtime
echo "Open-Source Runtime:"
if which npx >/dev/null 2>&1; then
  echo "  ✅ npx available - can use @anthropic-ai/sandbox-runtime"
  echo "  Usage: npx @anthropic-ai/sandbox-runtime <command>"
else
  echo "  ⚠️  npx not found (install Node.js)"
fi
echo

# 5. Documentation
echo "Documentation:"
echo "  Guide: guide/sandbox-native.md"
echo "  Official: https://code.claude.com/docs/en/sandboxing"
echo "  Runtime: https://github.com/anthropic-experimental/sandbox-runtime"
```

## 示例输出

```
=== Native Sandbox Status ===

Platform:
  ✅ macOS (Seatbelt built-in)

Configuration (from settings.json):
  Source: .claude/settings.json
  Auto-allow: true
  Allowed writes: ${CWD}, /tmp
  Denied reads: ${HOME}/.ssh, ${HOME}/.aws, ${HOME}/.kube
  Network policy: deny
  Allowed domains: api.anthropic.com, registry.npmjs.com, github.com... (9 total)
  Excluded commands: docker, kubectl, podman

Recent sandbox violations:
  ℹ️  Log inspection not yet implemented
  Tip: Check Claude Code session logs for sandbox violation notifications

Open-Source Runtime:
  ✅ npx available - can use @anthropic-ai/sandbox-runtime
  Usage: npx @anthropic-ai/sandbox-runtime <command>

Documentation:
  Guide: guide/sandbox-native.md
  Official: https://code.claude.com/docs/en/sandboxing
  Runtime: https://github.com/anthropic-experimental/sandbox-runtime
```

## 适用场景

- **部署前**：在运行自主工作流之前验证沙盒配置
- **调试**：排查某些命令被拦截的原因
- **安全审查**：检查已允许的域名和文件系统访问权限
- **新人入职**：帮助新团队成员理解项目的沙盒策略

## 另请参阅

- [原生沙盒指南](../../guide/security/sandbox-native.md) - 完整技术参考
- [沙盒验证钩子](../hooks/bash/sandbox-validation.sh) - 命令前验证
- [沙盒配置示例](../config/sandbox-native.json) - 生产就绪配置
