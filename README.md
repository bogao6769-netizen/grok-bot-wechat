# Grok Bot 接入微信

> 放在 Cursor 里照着做的技能配方：微信发来的文字会触发 Grok Bot 的 webhook Routine，盒子上的本地转发器再把回复送回。本仓库不运行桥接服务。

[Issues](https://github.com/bogao6769-netizen/grok-bot-wechat/issues/new/choose) · [MIT](LICENSE) · [CHANGELOG](CHANGELOG.md)

[![MIT License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

**English:** A Cursor skill recipe, not a running service. WeChat ClawBot (iLink) text triggers Grok Bot's webhook Routine, and a local forwarder on the box sends the reply back.

入站走 ClawBot（iLink），出站仍回到同一会话。账号、密钥和机器标识不进这个仓库。

## 架构

1. **微信 ClawBot（iLink）** — 手机扫码绑定的机器人会话。长轮询 `getupdates` 收消息，用 `sendmessage` 发回复。
2. **盒子转发器** — 跑在盒子上的 Node 进程，默认只监听 `http://127.0.0.1:17890`。它轮询 iLink，把签名后的唤醒 POST 到 Grok Bot 的 webhook Routine，并提供 `POST /reply` 把纯文本送回微信。
3. **Webhook Routine** — 触发器为 `{ "type": "webhook" }`。提示词说明如何把入站微信文字当成用户消息处理，以及如何经转发器回复。

额度走用户自己的 Grok Bot 方案。转发器在你的盒子上，不在本仓库里启动。

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

公开讨论里不要贴密钥或私人标识。怎么报告见 [安全说明](SECURITY.md)。

## 改这份配方

步骤和首页的改法写在 [贡献说明](CONTRIBUTING.md)。读者能感知的变化记在 [变更记录](CHANGELOG.md)。

## 许可

MIT © 2026 bogao6769-netizen。许可证全文是仓库根目录的 [LICENSE](LICENSE)。

English: MIT License, copyright 2026 bogao6769-netizen. Recipe and docs only.
