# Client installation / 客户端安装

The directory includes marketplace entries for Codex, Claude Code, Cursor, and Grok. All share `plugins/pingpong-cli/.mcp.json` and the same three Skills. These are configuration artifacts, not evidence of completed client integration testing.

For standalone CLI installation and usage instead of the plugin path, see the [CLI guide](docs/cli.md).

目录已包含四个客户端的 marketplace 入口，共享 MCP 和三个 Skills。清单齐全不代表各端已经完成安装、授权或真实查询验收。

**Before use / 使用前：** The bundled `.mcp.json` points to production. For sandbox use, replace its URL with the sandbox endpoint provided by your administrator; see [Environment configuration](SETUP.md#environments). Sandbox/test generated artifacts use different names; substitute the names from their manifests in the examples below. Do not publish internal test configuration.

内置 `.mcp.json` 默认连接生产环境；如需沙箱，请替换为管理员提供的沙箱地址，详见 [环境配置](SETUP.md#environments)。以下名称对应本公开目录；沙箱/test生成产物应使用其清单中的名称。内部测试配置不能公开分发。

## Codex

From the marketplace root / 在 marketplace 根目录：

```sh
codex plugin marketplace add "$PWD"
codex plugin add pingpong-cli@pingpong-marketplace
```

Complete OAuth if prompted, start a new session, and check `/mcp`.
按提示完成授权，新建会话，检查 `/mcp`。

## Claude Code

From the marketplace root / 在 marketplace 根目录：

```sh
claude plugin marketplace add "$PWD"
claude plugin install pingpong-cli@pingpong-marketplace
```

Enable the plugin, complete authentication in `/mcp`, then start a new session if needed. For explicit Skill invocation, Claude uses the plugin namespace, for example `/pingpong-cli:list-account-balances`.

启用插件，在 `/mcp` 中完成认证，必要时新建会话。Skill 显式调用使用插件命名空间，例如 `/pingpong-cli:list-account-balances`。

This path uses `.claude-plugin/marketplace.json` and the plugin's `.claude-plugin/plugin.json`; the plugin-root `skills/` and `.mcp.json` are shared. Actual PingPong installation and OAuth remain untested in Claude Code.

该方式使用 Claude 专用清单和插件根目录默认的 Skills/MCP 位置；尚未完成 PingPong 在 Claude Code 中的实际安装与授权验收。

## Cursor

After the repository is published, open Cursor Settings → Plugins and use Import under Team Marketplaces with the actual repository URL, if this entry is available to your account. Install `pingpong-cli`, enable its MCP connection, and follow the authentication prompt. Team/admin policies may restrict this flow.

仓库发布后，在 Cursor Settings → Plugins 中，通过 Team Marketplaces 的 Import 填写真实仓库地址，安装并启用插件。入口和组织权限以实际客户端为准。目前没有上架官方市场，也不承诺该 UI 支持导入本地目录。

Cursor reads `.cursor-plugin/marketplace.json` and the plugin's `.cursor-plugin/plugin.json`. Default desktop dynamic registration may include HTTPS callbacks, which the current PingPong authorization server rejects. Adding the manifest does not fix this restriction.

Cursor 使用自己的根 marketplace 和插件清单。桌面默认动态注册也可能提交 HTTPS 回调，当前服务器会拒绝；新增清单不改变此限制。

## Grok

After publication, set the variable to the actual marketplace Git URL:
发布后设置真实 marketplace 仓库地址：

```sh
grok plugin marketplace add "${PINGPONG_MARKETPLACE_GIT_URL:?Set the published marketplace Git URL}"
grok plugin install pingpong-cli --trust
```

Review the source before granting trust. This command shape comes from the inspected marketplace installation example; it has not been executed against a Grok client here. Check `grok plugin --help` in your installed version before use.

授权信任前检查来源。命令形式基于已检查的 marketplace 安装示例，本项目未实际运行 Grok 客户端；执行前核对你版本的 `grok plugin --help`。

Grok's root `.grok-plugin/marketplace.json` points to the shared plugin directory. The inspected layout does not contain a plugin-level `.grok-plugin/plugin.json`; we do not invent one. OAuth callback behavior and discovery of shared Skills/MCP require client testing.

Grok 根清单指向共享插件目录。已核对的布局不含插件级 `.grok-plugin/plugin.json`，因此未虚构该文件。共享 Skills/MCP 的发现及 OAuth 回调仍需客户端验收。

## Acceptance / 验收

For each client, verify manifest loading, plugin enablement, MCP initialization, OAuth callback compatibility, three discovered tools, and an authorized read-only query. Missing tools and downstream API failures are distinct from plugin installation errors.

逐端验证：清单识别、插件启用、MCP 初始化、OAuth 回调兼容、三个工具发现、授权范围内只读查询。工具缺失与下游业务失败应分别排查。

The current server accepts only HTTP localhost/127.0.0.1 OAuth callbacks. HTTPS callback clients require server-side changes. Browser CORS configuration is separate and does not remove this limitation.

当前服务只接受 HTTP 本机 OAuth 回调，HTTPS 回调需服务端改造；浏览器 CORS 配置与此独立。尚未执行 Claude/Cursor/Grok 端到端联调，也未发布任何市场条目。
