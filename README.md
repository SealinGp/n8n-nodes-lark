[English](https://github.com/zhgqthomas/n8n-nodes-feishu-lark/blob/main/README-EN.md)

# n8n-nodes-feishu-lark

本项目是基于 [n8n-nodes-feishu-lite](https://github.com/other-blowsnow/n8n-nodes-feishu-lite) 的二次开发，感谢原作者的开源贡献。在原有功能基础上增加了 `LarkTrigger`、`LarkMCP` 节点以及多项操作优化。

> 原始仓库：[https://github.com/zhgqthomas/n8n-nodes-feishu-lark](https://github.com/zhgqthomas/n8n-nodes-feishu-lark)

---

## 节点说明

| 节点 | 说明 |
|------|------|
| `Lark` | 主操作节点，支持消息、文档、云空间、知识空间、多维表格、表格、日历、通讯录等资源 |
| `LarkTrigger` | 事件订阅触发节点，支持 WebSocket（飞书国内版）和 Webhook 两种方式 |

---

## 快速开始

### 1. 配置 Credentials

调用飞书 OpenAPI 支持两种认证方式：

**Tenant Access Token**（应用身份）
- 在 n8n 中选择 `Lark Tenant Token API`
- 填入飞书应用后台的 `App ID` 和 `App Secret`

**User Access Token**（用户身份，OAuth2）
- 在 n8n 中选择 `Lark OAuth2 API`
- 填入 `App ID`（对应 Client ID）和 `App Secret`（对应 Client Secret）
- 将 n8n 生成的 `OAuth Redirect URL` 配置到飞书后台的「安全设置 → 重定向 URL」
- 在飞书后台「权限管理」中开通所需权限，并将权限名称填入 `Scope` 输入框（如 `base:app:create,offline_access`）

> 建议开通 `offline_access` 权限，以便 n8n 通过 refresh token 自动续签 access token。

---

### 2. 事件触发（Trigger）

#### WebSocket（推荐，仅飞书国内版）

在 n8n canvas 上搜索并添加 `Lark Trigger` 节点，配置好 `Lark Tenant Token API` Credential 即可。

在飞书后台「事件与回调」中添加需要订阅的事件，节点即可接收对应事件推送。

#### Webhook（国际版 Lark 使用）

使用 `Parse Webhook Message` operation，配合 n8n 官方的 `Webhook` 和 `Respond to Webhook` 节点实现。

---

## 特色 Operations

### Parse Message
解析飞书事件回调的数据结构，按事件类型进行分支处理。

### Send and Wait
发送消息后暂停工作流，等待飞书用户确认后继续执行，适用于 Human-in-the-loop 场景。

### Send Streaming Message
将 AI Agent 的输出以流式方式推送给飞书机器人（需 n8n >= 1.3.0）。

---

## 开发

```bash
# 安装依赖
npm install

# 编译（输出到 dist/）
npm run build

# 监听模式
npm run dev

# 运行测试
npm test

# 代码检查
npm run lint
```

> 修改 TypeScript 源码后需重新执行 `npm run build`，n8n 加载的是 `dist/` 目录下的编译产物。

---

## 许可证

MIT License
