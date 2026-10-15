# Grok Bot 接入微信

**English:** A Cursor skill recipe, not a running service: WeChat ClawBot (iLink) text wakes a Grok Bot webhook Routine, and a local forwarder on the box sends the reply back.

可复用的 **Cursor skill 操作配方**，不是正在运行的桥接服务：入站微信文字唤醒一条 webhook Routine，回复经盒子上的本地转发进程送回 ClawBot（iLink）。不含账号、密钥或机器标识。

## 架构

1. **微信 ClawBot（iLink）** — 手机扫码绑定的机器人会话。长轮询 `getupdates` 收消息，用 `sendmessage` 发回复。
2. **盒子转发器** — 跑在盒子上的 Node 进程，默认只监听 `http://127.0.0.1:17890`。它轮询 iLink，把签名后的唤醒 POST 到 Grok Bot 的 webhook Routine，并提供 `POST /reply` 把纯文本送回微信。
3. **Webhook Routine** — 触发器为 `{ "type": "webhook" }`。提示词说明如何把入站微信文字当成用户消息处理，以及如何经转发器回复。

额度走用户自己的 Grok Bot 方案。

## 怎么用

技能全文在 [`.cursor/skills/grok-bot-wechat/SKILL.md`](.cursor/skills/grok-bot-wechat/SKILL.md)。在 Cursor 里按该配方操作，步骤概要：

1. 创建 webhook Routine（无定时）。
2. 在盒子上放置转发器，只监听 `127.0.0.1`。
3. 启动转发器，用健康检查确认进程在听。
4. 手机扫码绑定，由转发器保存 `bot_token`。
5. 按需配置允许名单；出站去掉 Markdown，按纯文本发送。
6. 端到端：微信发一条短文本，Routine 被唤醒，回复出现在微信里。

账号相关的值（webhook URL、密钥、`bot_token`、允许的微信用户 id、机器 id）放在环境变量、盒子上的 state 文件或 Routine 面板里，不要写进本仓库。

## 安全

- 转发器只绑定 `127.0.0.1`，不要监听 `0.0.0.0` 或公网端口。
- 不要把 webhook 密钥、`bot_token`、`context_token` 写入技能文本、git、日志或本 README。
- 会话过期（返回码 **-14**）时停止空转轮询，请用户重新扫码。

## 许可

[MIT](LICENSE)
