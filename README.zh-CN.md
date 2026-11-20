# PingPong AI Marketplace

[English](README.md) | 简体中文

用于查询 PingPong 账户数据的 MCP 工具与 Agent Skills。

当前插件包含远程 MCP 连接和三个只读工具 Skill（账户余额、余额流水、交易流水）。不安装 PingPong CLI，不提供付款、出款、换汇执行或发卡工作流。

[MCP 配置](plugins/pingpong-cli/.mcp.json) 默认连接生产环境 `https://mcp.pingpongx.com/mcp`，安装后完成生产环境 OAuth 授权即可使用。如需沙箱，请先将地址替换为管理员提供的沙箱 MCP 端点。环境地址与配置步骤见 [环境配置](SETUP.md#environments)。

## 插件目录

| 插件 | 连接器 | 类型 | 说明 |
| --- | --- | --- | --- |
| `pingpong-cli` | MCP | 工具调用指引 | 面向 Codex、Claude Code、Cursor、Grok 打包的三个账户只读查询 |

目前没有独立的开发集成插件、文档搜索 MCP 或通用财务工作流插件。

### Skills

| 打包后的 Skill | MCP 工具 | 用途 |
| --- | --- | --- |
| `list-account-balances` | `list_account_balances` | 查询账户余额 |
| `list-balance-movements` | `list_balance_movements` | 查询余额流水 |
| `list-transaction-history` | `list_transaction_history` | 查询交易流水 |

业务参数采用 snake_case，例如 `page_no`、`page_size`。查询前查看服务端工具定义和 Skill 说明。Skill 提供调用指引，不注册账号、不签发 token，也不授予权限。

## 连接器

- **MCP：** 生成的 Codex 插件已包含连接配置，需在客户端完成 OAuth 并启用连接。
- **CLI：** 适用于能够执行终端命令的 Agent，是独立安装与登录的另一条路径。安装、环境和命令见 [CLI 使用指南](./docs/cli.zh-CN.md)；CLI 与本插件的安装和凭据相互独立。

当前打包的 Skills 直接调用 MCP 工具。单独导入 Skill 不会配置 MCP，也不会自动回退到 CLI。

## 安装

### Codex：本地 marketplace

先在 `plugins/pingpong-cli/.mcp.json` 配置真实 MCP 地址，再在本 marketplace 根目录执行：

```sh
codex plugin marketplace add "$PWD"
codex plugin add pingpong-cli@pingpong-marketplace
```

以上是本静态目录使用的名称。构建脚本生成的沙箱产物使用独立名称，请以所安装产物的清单为准；实际环境由配置的 MCP URL 决定。请将 `.agents/plugins/marketplace.json` 和完整插件目录一起保留，只复制 README 不足以安装。

按提示完成 OAuth，开启新 Codex 会话，在 `/mcp` 检查连接与工具列表。安装成功不等于授权完成或业务查询成功。

### Codex：发布后使用 Git 安装

发布者提供实际仓库地址后，将 `PINGPONG_MARKETPLACE_GIT_URL` 设置为该地址：

```sh
codex plugin marketplace add "${PINGPONG_MARKETPLACE_GIT_URL:?Set the published marketplace Git URL}"
codex plugin add pingpong-cli@pingpong-marketplace
```

CLI 指南只说明独立安装与使用，不包含 marketplace 清单；本仓库仍作为插件安装源。

### 其他 Agent

| 客户端 | 当前接入方式 | 状态 |
| --- | --- | --- |
| Claude Code | 配置远程 HTTP MCP，或通过终端使用 CLI | 已提供 marketplace 和 Claude 插件清单；安装及 OAuth 仍需客户端联调 |
| Cursor | 配置远程 MCP，或在允许终端执行时使用 CLI | 已提供 marketplace 和 Cursor 插件清单，尚未上架市场；默认动态注册可能因 HTTPS 回调限制失败 |
| Claude Desktop / Cowork / Claude.ai | 自定义远程连接器 | 平台 HTTPS 回调需服务端 OAuth 改造 |
| ChatGPT | 自定义 MCP，并在当前对话启用 | 平台 HTTPS 回调需服务端 OAuth 改造 |
| 其他 Agent | 兼容的远程 MCP 客户端或终端 CLI | 分别验证传输、OAuth 和工具发现 |

当前内嵌授权服务器仅接受 HTTP localhost/127.0.0.1 回调。MCP 入口是公网 HTTPS，并不会解除 OAuth 回调限制。各客户端分别使用自己的 marketplace 入口，共享 MCP 配置和 Skills；清单存在不代表运行时已兼容。

尚未验证一键安装纯 Skills 的工具。如果 Agent 支持导入本地 Skill 目录，仍需单独连接 MCP，并确认该客户端能找到对应工具。

Claude Code、Cursor、Grok 的步骤见 [客户端安装与验证](CLIENTS.md)。清单按已核对的参考目录格式整理，尚未完成这些客户端的端到端验收。

## 使用插件

连接启用后发送：

```text
使用 PingPong MCP 工具 list_account_balances，page_no=1，page_size=1。
报告实际查询结果或错误，不推测缺失余额。
```

Skill 名称与 MCP 工具名不同。需要明确调用 Skill 时，在 Codex 的 Skill 选择入口选择已安装项；客户端可能显示插件命名空间。Skill 可见不代表 MCP 工具已连接。

工具缺失时，检查当前客户端的连接与对话状态。CLI 或其他客户端登录不会为此插件完成认证。如果已经执行工具，但获取下游 token 失败，应查看服务端诊断，而不是反复安装。

CLI 的安装、登录、环境选择与排障见 [CLI 使用指南](./docs/cli.zh-CN.md)。

## 更新与卸载

发布者修改打包内容时应同步提升插件版本。通过 Git 安装的用户可刷新已登记的 marketplace：

```sh
codex plugin marketplace upgrade pingpong-marketplace
```

刷新后检查已安装插件版本，按客户端提示更新插件。插件变更后开启新会话；仅刷新 marketplace 不代表旧会话已加载最新工具。

卸载正式插件及 marketplace 注册：

```sh
codex plugin remove pingpong-cli@pingpong-marketplace
codex plugin marketplace remove pingpong-marketplace
```

卸载不等于撤销服务端 OAuth token。

## 仓库结构

本 marketplace 目录包含：

```text
.agents/plugins/marketplace.json
.claude-plugin/marketplace.json
.cursor-plugin/marketplace.json
.grok-plugin/marketplace.json
plugins/pingpong-cli/
  .codex-plugin/plugin.json
  .claude-plugin/plugin.json
  .cursor-plugin/plugin.json
  .mcp.json
  skills/
    list-account-balances/SKILL.md
    list-balance-movements/SKILL.md
    list-transaction-history/SKILL.md
README.md
README.zh-CN.md
SETUP.md
CLIENTS.md
docs/cli.md
docs/cli.zh-CN.md
```

参见 [本地准备与发布检查](SETUP.md)。将本目录复制到独立仓库时，保留隐藏清单与配置。公开产物必须使用已确认的沙箱或正式地址，不能包含内部 test 配置、账号凭据或 token。

## 验证与数据使用

本地 Codex marketplace 注册与插件安装已验证。真实账号授权、各客户端工具发现及业务成功需单独验收；尚未执行公网托管或市场发布。

只授权任务需要的账号与工具。公开反馈不得包含凭据、用户身份、余额或流水。向维护者提供脱敏客户端/版本、工具名、错误和 diagnosticId。结论必须来自真实工具响应，不能将一页查询称为完整对账。

本文不套用第三方许可证或条款；发布者应在公开分发前明确适用的许可证和支持政策。
