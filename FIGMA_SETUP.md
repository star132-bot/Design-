# Figma × Claude Code 配置指南

目标：让你电脑上的 Claude Code 连上 Figma，并装好官方 Skills，实现
「定规范 → AI 在 Figma 里设计 → 你调整 → AI 生成代码」的流程。

> 资料来源：Figma 官方仓库 [figma/mcp-server-guide](https://github.com/figma/mcp-server-guide)（2026-10 核对）。
> 目前这些 Skills 是 Beta 功能，名称可能会变，以官方仓库为准。

---

## 交给本地 Claude 的提示词

在本仓库目录里打开 Claude Code，把下面这段直接发给它：

```text
请阅读本仓库的 FIGMA_SETUP.md，按「安装步骤」帮我配置 Figma：
1. 安装 Figma 官方插件（含 MCP 和 Skills）
2. 需要我在浏览器里授权时，停下来告诉我该点什么
3. 验证 MCP 已连接、Skills 已加载，把结果列给我
4. 完成后告诉我每个核心 Skill 什么时候用
不要安装社区 Skills，除非我单独要求。
```

---

## 安装步骤

### 前提

- Figma 账号（已有学生认证即可）
- 已安装 Claude Code
  - macOS / Linux：`curl -fsSL https://claude.ai/install.sh | bash`
  - Windows（PowerShell）：`irm https://claude.ai/install.ps1 | iex`

### 第 1 步：安装 Figma 官方插件

这一个插件同时包含 **Figma MCP 连接** 和 **全部官方 Skills**，不需要单独装。

```bash
claude plugin install figma@claude-plugins-official
```

### 第 2 步：授权

1. 如果 Claude Code 正在运行，先退出再重新打开
2. 输入 `/plugin` 回车，找到 **figma**，回车
3. 浏览器会打开 Figma 授权页，点 **Allow access**

### 第 3 步：验证

- 输入 `/mcp`：能看到 `figma` 显示为已连接
- 让 Claude 列出已加载的 Skills，应能看到下文「核心 Skills」里的名字
- 发一个 Figma 画板链接（右键画板 → Copy link），让 Claude 读取并描述它

### 备用：手动只添加 MCP（不含 Skills）

插件装不上时才用：

```bash
claude mcp add --transport http figma https://mcp.figma.com/mcp
```

然后在 `/mcp` 里完成登录授权。

---

## Skills 清单

### 核心（插件自带，必用）

| Skill | 作用 | 什么时候用 |
|---|---|---|
| `figma-use` | 基础 Skill，教 AI 正确操作 Figma 画布（建图层、组件、变量、自动布局）。其他写入类 Skill 都依赖它 | 任何让 AI 在 Figma 里动手的任务（自动加载） |
| `figma-generate-design` | 用你已有的组件和变量拼出整页或弹窗、侧栏等，**不写死数值** | 让 AI 在 Figma 里设计新页面 |
| `figma-generate-library` | 搭建设计系统：变量、组件、变体、明暗主题 | 建立或补全你的组件库；哪怕只做一个组件也会用到 |
| `figma-design-to-code` | 把 Figma 设计稿实现成代码 | 设计定稿后转代码 |
| `figma-code-connect` | 把 Figma 组件和代码组件对应起来 | 组件库在 Figma 和代码里都有之后 |
| `figma-create-new-file` | 新建 Figma Design、FigJam 或 Slides 文件 | 开新项目 |

> 生成设计规则文件（`CLAUDE.md` / `AGENTS.md`）不是一个 Skill，而是 MCP 工具
> `create_design_system_rules`。对 Claude 说「用 create_design_system_rules 为这个项目生成设计规则」即可。

### 按需（插件自带，用到时自动加载）

| Skill | 作用 |
|---|---|
| `figma-use-figjam` | 操作 FigJam 白板 |
| `figma-use-slides` | 操作 Figma Slides |
| `figma-generate-diagram` | 生成流程图、架构图、时序图等 |
| `figma-use-motion` | 在 Figma 里给图层加动画 |
| `figma-implement-motion` | 把 Figma 里的动效实现成代码 |
| `figma-shaders` | 创建着色器效果 |
| `figma-generative-plugins` | 创建可复用的生成式 Figma 插件 |
| `figma-swiftui` | SwiftUI 与 Figma 互转（做 iOS 时） |

### 官方额外工作流（不在插件里，需手动添加）

| Skill | 作用 |
|---|---|
| `generate-project-plan` | 根据需求文档生成 FigJam 项目规划板 |
| `video-interaction-mapper` | 把界面录屏转成带注释的 Figma 故事板 |

需要时，让 Claude 从官方仓库的 `workflow-skills/` 目录复制到本项目的 `.claude/skills/`。

### 社区 Skills（谨慎安装）

Figma 博客推荐过一些，例如 `/make-a-type-scale`（生成字号体系）、`/image-audit`（检查图片可导出性）。
安装前先读它的 `SKILL.md`，只装来源可信的。

---

## 注意事项

- **写入画布只支持远程服务器**（`https://mcp.figma.com/mcp`，插件默认用的就是它）。
  Figma 桌面版的本地 MCP（`127.0.0.1:3845`）只能读取，不能让 AI 在画布里创建内容。
- 写入画布在 Beta 期间免费，Figma 表示以后会改为按用量付费。
- Skills 只教 AI **怎么操作**，不决定**设计好不好**。效果取决于你在 Figma 里定好的
  变量（颜色、字体、间距）和组件。规范越清楚，AI 产出越统一。
- 云端会话不会自动拥有本地插件。云端要用 Figma 时，在
  https://claude.ai/customize/connectors 连接 Figma，然后新开会话。

---

## 下一步

1. 按上面装好并验证
2. 在 Figma 里建好颜色、字体、间距变量
3. 运行 `create_design_system_rules`，把生成的规则文件提交到本仓库
4. 让 AI 用 `figma-generate-library` 做第一个组件（建议从按钮开始，做全所有状态）
