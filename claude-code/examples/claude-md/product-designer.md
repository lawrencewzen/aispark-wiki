> 📚 **AI Spark Wiki** · Claude Code 知识库

---
title: "Product Designer CLAUDE.md Template"
description: "使用 Figma MCP 的设计到代码工作流 CLAUDE.md 配置"
tags: [claude-md, template, design-patterns, mcp]
---

# 产品设计师 CLAUDE.md 模板

针对使用 Figma MCP 的设计到代码工作流优化的 CLAUDE.md 配置。

## 使用方式

将以下内容复制到项目的 `CLAUDE.md` 文件中，并自定义 `[方括号]` 标注的部分。

---

## 模板

```markdown
# 产品设计师项目配置

## 设计系统来源

### Figma 文件
- 设计系统库：[Figma 文件 URL]
- 组件库：[Figma 文件 URL]
- 当前项目：[Figma 文件 URL]

### 访问权限
- Figma MCP 已配置：是/否
- 个人访问令牌：[在 MCP 设置中配置 — 切勿分享实际值]
- Dev Mode 访问：[完整变量检查所需]

## Token 层级

我们的设计 token 遵循三层结构：

### 基础 Token（原始值）
Figma 变量 → CSS 自定义属性
- 颜色：`--blue-600`、`--gray-100` 等
- 间距：`--spacing-1`（4px）、`--spacing-2`（8px）等
- 字体：`--font-size-sm`、`--font-weight-medium` 等
- 圆角：`--radius-sm`、`--radius-md` 等

### 复合 Token
组件中使用的原始值组合
- 按钮内边距：`var(--spacing-2) var(--spacing-4)`
- 卡片阴影：`var(--shadow-md)`
- 输入框边框：`var(--border-width-1) solid var(--border-default)`

### 语义 Token
特定上下文含义
- `--interactive-primary`：主要操作颜色
- `--surface-elevated`：卡片/弹窗背景
- `--text-secondary`：描述性文本
- `--border-error`：表单校验

## 设计规范

### 颜色
- 主色：[#HEX] → `var(--interactive-primary)`
- 副色：[#HEX] → `var(--interactive-secondary)`
- 禁止使用：组件中硬编码的十六进制值
- 必须使用：设计 token 变量

### 间距
- 基础单位：[4px/8px]
- 比例：[4、8、12、16、24、32、48、64、96]
- 禁止使用：魔法数字（如 `padding: 13px`）
- 必须使用：token 倍数（如 `var(--spacing-3)`）

### 字体
- 字体族：[Inter/Roboto/System/自定义]
- 比例：[12、14、16、18、20、24、32、48、64]
- 行高：[1.2、1.5、1.6]，依据文字大小而定
- 字间距：严格遵循 Figma 文字样式

### 响应式断点
与 Figma 画框尺寸保持一致：
- 移动端：[320-767px]
- 平板：[768-1023px]
- 桌面端：[1024px+]
- 最大宽度：[1440px]

## Figma MCP 命令

当我分享 Figma URL 时，可使用以下 MCP 命令：

### 读取设计文件
```
figma_get_file(file_key: string)
→ 返回：文件结构、页面、画框
```

### 提取样式
```
figma_get_styles(file_key: string)
→ 返回：颜色样式、文字样式、效果样式
```

### 获取组件
```
figma_get_component(file_key: string, node_id: string)
→ 返回：组件属性、变体、设计 token
```

### 变量（设计 Token）
```
figma_get_variables(file_key: string)
→ 返回：所有 Figma 变量（原始值）
```

## Code Connect 映射

我们使用 Figma Code Connect 将设计关联到代码。实现组件时，参考以下映射：

| Figma 组件 | 代码路径 | 备注 |
|-----------------|-----------|-------|
| Button/Primary | `components/Button/Primary.tsx` | 使用 `PrimaryButton` 组件 |
| Input/Text | `components/Input/Text.tsx` | 支持错误状态 |
| Card/Default | `components/Card/index.tsx` | Auto Layout → Flexbox |
| Modal/Standard | `components/Modal/Standard.tsx` | 焦点捕获 + 无障碍 |

[在此添加您的映射]

## 实现约束

### 技术栈
- 框架：[React/Vue/Svelte/Angular]
- 样式：[Tailwind/CSS Modules/Styled Components/CSS-in-JS]
- TypeScript：[必须/可选]
- 组件库：[自建 / 使用 MUI/Chakra 等]

### 模式
- 组件结构：[原子设计 / 功能模块 / 扁平结构]
- 状态管理：[useState / Context / Redux / Zustand]
- Props 命名：[严格匹配 Figma 变体名称]

### 质量要求
- 所有 props 需有 TypeScript 类型
- 默认无障碍（至少满足 WCAG AA）
- 响应式行为与 Figma 画框一致
- 包含 Figma 中的悬停/焦点/禁用状态

## 设计交付检查清单

从 Figma 实现组件时，验证以下内容：

### 设计还原
- [ ] 颜色与 Figma 完全一致（使用浏览器 DevTools 取色器）
- [ ] 间距与 Figma 标注一致
- [ ] 字体（大小、字重、行高）一致
- [ ] 圆角一致
- [ ] 阴影/效果一致

### Token
- [ ] 无硬编码颜色（全部使用 `var(--token-name)`）
- [ ] 无魔法数字间距
- [ ] 字体大小使用设计 token
- [ ] 所有值可追溯至设计系统

### 响应式
- [ ] 移动端断点已实现
- [ ] 平板断点已实现
- [ ] 桌面端布局一致
- [ ] 移动端无横向滚动

### 无障碍
- [ ] 键盘导航可用
- [ ] 焦点状态可见
- [ ] 必要位置有 ARIA 标签
- [ ] 颜色对比度通过 WCAG AA
- [ ] 已进行屏幕阅读器测试（若为交互组件）

### 状态
- [ ] 默认状态已实现
- [ ] 悬停状态与 Figma 一致
- [ ] 焦点状态与 Figma 一致
- [ ] 禁用状态与 Figma 一致
- [ ] 错误状态（如适用）
- [ ] 加载状态（如适用）

## 响应偏好

### 组件实现
1. 通过 MCP 读取 Figma 组件
2. 提取所有变体和属性
3. 将设计 token 映射到代码 token
4. 生成带完整类型的 TypeScript 组件
5. 包含 Storybook story（如使用 Storybook）
6. 对照上方检查清单验证

### 设计系统审计
1. 对比代码库与 Figma 作为单一真相来源
2. 报告硬编码值（颜色、间距等）
3. 标记不一致项（缺失变体、错误 token）
4. 提出自动化修复方案

### Token 更新
1. 当 Figma 中的设计 token 变更时，通知受影响组件
2. 若有破坏性变更，生成迁移脚本
3. 验证无硬编码值会被破坏

## 团队规范

### 提交信息
- 格式：`feat(ui): add Button component from Figma`
- 引用 Figma 画框：`feat(ui): implement Modal (Figma frame: abc123)`

### PR 要求
- [ ] 截图对比（代码 vs Figma）
- [ ] 所有检查清单项已验证
- [ ] 响应式行为已测试
- [ ] 无障碍已验证

### 组件文档
每个组件应记录：
- Figma 来源（URL + 画框 ID）
- 最后与设计同步时间
- 与设计的任何偏差（附理由）
```

---

## 定制指南

### React + Tailwind 项目

在"技术栈"中添加：
```markdown
### Tailwind 配置
- 配置文件：`tailwind.config.ts`
- 自定义工具类：`src/styles/utilities.css`
- 设计 token 映射于：`theme.extend` 对象

### 组件模式
```tsx
// components/Button/Primary.tsx
import { cn } from '@/lib/utils';

interface ButtonProps {
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  children: React.ReactNode;
}

export function PrimaryButton({ size = 'md', disabled, children }: ButtonProps) {
  return (
    <button
      className={cn(
        // 基础样式
        'rounded-md font-medium transition-colors',
        // 语义 token
        'bg-interactive-primary text-white',
        'hover:bg-interactive-primary-hover',
        // 状态
        'disabled:opacity-50 disabled:cursor-not-allowed',
        // 尺寸变体
        {
          'px-3 py-1.5 text-sm': size === 'sm',
          'px-4 py-2 text-base': size === 'md',
          'px-6 py-3 text-lg': size === 'lg',
        }
      )}
      disabled={disabled}
    >
      {children}
    </button>
  );
}
```
```

### 使用 Tokens Studio 的设计系统

添加章节：
```markdown
## Token 自动化

### Tokens Studio 配置
- 插件已连接至：[GitHub 仓库 URL]
- 分支：`design-tokens`
- 同步频率：[保存时 / 手动 / CI/CD]

### Token 导出格式
```json
{
  "color": {
    "blue": {
      "600": {
        "value": "#0066CC",
        "type": "color"
      }
    }
  },
  "spacing": {
    "2": {
      "value": "8px",
      "type": "spacing"
    }
  }
}
```

### Style Dictionary 转换
- 配置：`style-dictionary.config.js`
- 输出：`src/tokens/` 目录
- 平台：CSS、Tailwind、iOS、Android
```

### 使用 Storybook 的团队

在"响应偏好"中添加：
```markdown
### Storybook Stories
生成包含所有变体的 story：

```tsx
// Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { PrimaryButton } from './Primary';

const meta: Meta<typeof PrimaryButton> = {
  title: 'Components/Button/Primary',
  component: PrimaryButton,
  parameters: {
    design: {
      type: 'figma',
      url: 'https://www.figma.com/file/[FILE_KEY]?node-id=[NODE_ID]',
    },
  },
};

export default meta;
type Story = StoryObj<typeof PrimaryButton>;

export const Default: Story = {
  args: {
    children: 'Click me',
    size: 'md',
  },
};

export const Small: Story = {
  args: {
    children: 'Small button',
    size: 'sm',
  },
};

// ... Figma 中的所有变体
```
```

---

## 与工作流集成

本 CLAUDE.md 配合设计到代码工作流使用：

- 阅读：[设计到代码工作流](../../guide/workflows/design-to-code.md)
- 理解：Figma Make → Claude MCP → 生产流水线
- 应用：使用本配置维护设计与代码的一致性

---

## 示例提示词

### 从 Figma 实现组件
```
从我们的 Figma 设计系统实现"Card/Product"组件：
[Figma URL]

遵循设计交付检查清单。
使用此 CLAUDE.md 中的 token 规范。
生成包含所有变体的 TypeScript 组件。
```

### 审计设计系统偏差
```
将 src/components 与我们的 Figma 设计系统进行对比审计：
[Figma URL]

报告：
1. 未使用 token 的硬编码值
2. Figma 中存在但代码中缺失的变体
3. 间距/颜色不一致
4. 提出修复方案
```

### 设计变更后更新组件
```
按钮组件在 Figma 中已更新：
[Figma URL → Button 画框]

审查变更，更新代码以匹配。
尽量保持向后兼容。
如需要，同步更新 Storybook stories。
```

---

## 参见

- [设计到代码工作流](../../guide/workflows/design-to-code.md) — 完整 Figma MCP 工作流指南
- [Figma MCP 章节](../../guide/ultimate-guide.md#figma-mcp) — 技术细节
- [处理图片](../../guide/ultimate-guide.md#24-working-with-images) — 视觉分析基础
