# 贡献说明

改动对象是技能步骤和说明文字，让下一次在 Cursor 里照着做的人少猜一步。本仓库不收转发器实现、安装包或自动发布配置。

## 开 Issue

写明卡在 [技能](.cursor/skills/grok-bot-wechat/SKILL.md) 的哪一节，以及你希望文档改成什么样。可以写现象，不要写能用来登录的材料：webhook 密钥、`bot_token`、`context_token`、微信用户 id、机器 id，以及盒子上的 `state/` 或 `.env`。

和密钥、公网监听有关的事，改走 [SECURITY.md](SECURITY.md)，不要先贴在公开讨论里。

## 改文件时

- 操作顺序以 `.cursor/skills/grok-bot-wechat/SKILL.md` 为准。
- 仓库首页 `README.md` 保持中文在前，英文只留短句，并写明这是配方而不是正在运行的服务。
- 转发器继续只听 `127.0.0.1`。
- 账号材料留在环境变量、盒子上的 state，或 Routine 面板。

读一遍 diff：步骤还能串起来，链接还指向仓库里的文件，没有秘密。

## 提 PR

用一小节说明你动了哪一段、读者会有什么不同。不要带上本机日志、token 或转发器目录的拷贝。维护者合并时，再把 [CHANGELOG.md](CHANGELOG.md) 里的 Unreleased 收进具体版本。

贡献和仓库一样，适用 [MIT](LICENSE)（© 2026 bogao6769-netizen）。
