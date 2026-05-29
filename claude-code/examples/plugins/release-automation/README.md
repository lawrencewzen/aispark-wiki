> 📚 **AI Spark Wiki** · Claude Code 知识库

# 发布自动化插件

语义化版本管理、变更日志生成与发布管理。

## 安装

```bash
bash install.sh
```

## 组件

- **/release 命令** — 语义化版本升级与标签打标
- **/changelog 命令** — 从提交记录生成发布说明
- **release-notes-generator 技能** — 自动化发布文档生成
- **version-sync 钩子** — 保持各文件间版本号一致

## 快速开始

```bash
# 升级版本并创建发布
/release patch      # v1.0.0 → v1.0.1
/release minor      # v1.0.1 → v1.1.0
/release major      # v1.1.0 → v2.0.0

# 生成变更日志
/changelog 10       # 最近 10 次发布

# 钩子会自动同步各文档中的 VERSION
```

## 功能特性

✓ 语义化版本管理
✓ 自动化变更日志
✓ Git 标签打标
✓ 发布说明生成
✓ 版本号一致性

---

版本管理详见 `guide/workflows/releases-tracking.md`。
