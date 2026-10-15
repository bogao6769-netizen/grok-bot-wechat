# 安全说明

这个仓库是技能和文档。它自己不登录微信，也不替任何人保管会话。收发消息的进程在你的盒子上，按技能约定只听本机 `127.0.0.1`。

## 请私下说的事

公开的 Issue、PR 和讨论只适合写“哪一步对不上”。下面这些不要出现在帖子、截图或附件里：

- webhook 的密钥，以及面板里的 Authorization 内容
- `bot_token`、`context_token`
- 允许名单里的微信用户 id、机器 id
- `state/`、`.env` 和转发器日志

需要维护者看细节时，用 GitHub 的私密通道：[Security Advisories](https://github.com/bogao6769-netizen/grok-bot-wechat/security/advisories)。若该页面还不能提交，先开一条不含细节的 Issue，请维护者指定联系方式。

## 配方里已经约定的边界

监听地址保持回环，不要改成 `0.0.0.0` 或任何公网端口。会话过期（返回码 -14）时停止空转轮询，让用户重新扫码，不要把 token 打进日志。

本仓库没有删文件、清缓存或发布二进制的步骤。盒子上的 state 和环境变量留在那台机器上。
