# MCP 服务配置指南

技能文件（`.skill`）不包含 MCP 认证 URL——URL 通过本地配置文件在运行时注入。这样 `.skill` 文件可以安全分享，认证信息留在本地。

## 快速配置

```bash
# 1. 复制配置模板
cp mcp-config.example.json mcp-config.json

# 2. 编辑 mcp-config.json，填入从钉钉 MCP 市场获取的真实 URL
#    {"MCP_BOCHA_URL": "https://mcp-gw.dingtalk.com/server/xxx?key=yyy"}

# 3. 测试连接
python3 scripts/call_mcp.py list "$MCP_BOCHA_URL"
```

## 获取 MCP URL

1. 访问 MCP 市场对应服务：`https://mcp.dingtalk.com/#/detail?mcpId={mcpId}&detailType=marketMcpDetail`
2. 登录钉钉账号
3. 复制页面右侧的 **StreamableHTTP URL**

URL 格式：`https://mcp-gw.dingtalk.com/server/{hash}?key={key}`

## mcp-config.json 格式

```json
{
  "MCP_BOCHA_URL": {
    "url": "https://mcp-gw.dingtalk.com/server/xxx?key=yyy",
    "mcpId": "2028"
  },
  "MCP_DINGTALK_DOC_URL": {
    "url": "https://mcp-gw.dingtalk.com/server/zzz?key=www",
    "mcpId": "1047"
  }
}
```

- 顶层 key 即环境变量名（与 SKILL.md 中的 `$VAR_NAME` 对应）
- `url`：真实的 StreamableHTTP URL（必填，`call_mcp.py` 读取此字段注入环境变量）
- `mcpId`：MCP 市场服务 ID（选填，便于追溯来源和后续扩展）
- 对象结构可按需扩展同级字段（如 `name`、`charged` 等）
- `call_mcp.py` 启动时自动查找并加载（优先级：技能根目录 > scripts 目录 > 当前工作目录）
- 系统已有同名环境变量时，配置文件中的值**不会覆盖**（系统变量优先）

## mcp-config.example.json 格式

结构与 `mcp-config.json` 相同，`url` 留空，可安全打包分发。`mcpId` 建议填写真实值——不含敏感信息，填了方便接收者直接跳转市场页面获取 URL：

```json
{
  "MCP_BOCHA_URL": {
    "url": "",
    "mcpId": "2028"
  },
  "MCP_DINGTALK_DOC_URL": {
    "url": "",
    "mcpId": "1047"
  }
}
```

## 在 SKILL.md 中引用环境变量

技能文件里用 `$VAR_NAME` 格式作占位符（语法不变）：

```bash
python3 scripts/call_mcp.py call "$MCP_BOCHA_URL" web_search --params '{"query": "..."}'
```

`call_mcp.py` 识别 `$` 前缀，自动从环境中解析实际 URL。也可以直接传 URL 字符串（向后兼容）。

## 安全说明

| 文件 | 是否打包进 .skill | 是否含敏感信息 |
|------|------------------|----------------|
| `mcp-config.json` | ❌ 不打包（.skillignore 排除） | ✅ 含真实 URL |
| `mcp-config.example.json` | ✅ 打包 | ❌ 只有键名，无值 |
| `SKILL.md` | ✅ 打包 | ❌ 只有 `$VAR_NAME` 占位符 |

## 为什么用 JSON 而不是 .env？

`.env` 是隐藏文件（以 `.` 开头），部分钉钉运行容器和 CI 环境启用了安全管控，**禁止读取隐藏文件**，导致 URL 无法注入。`mcp-config.json` 是普通文件，不受此限制。

兼容性：如果技能目录中没有 `mcp-config.json`，`call_mcp.py` 会自动降级尝试读取 `.env`（兼容旧版技能）。

## 后续升级路径

在钉钉容器/CI 中，直接通过**系统环境变量**注入 URL，无需任何配置文件，`call_mcp.py` 的读取逻辑不变：

```bash
export MCP_BOCHA_URL=https://mcp-gw.dingtalk.com/server/xxx?key=yyy
python3 scripts/call_mcp.py list "$MCP_BOCHA_URL"
```
