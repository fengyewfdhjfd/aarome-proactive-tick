# aarome-proactive-tick

「回响AI陪伴」（[www.aarome.xyz](https://www.aarome.xyz)）的**定时器仓库**：这里只有
一个 GitHub Actions 工作流，没有站点代码。

## 为什么单独有这么一个仓库

主动联系功能要求「用户不在线时，角色也会想起他」。站点跑在 Vercel 单个 Serverless
函数上，而 **Vercel Hobby 计划的 cron 每个项目一天只能触发一次**，写 `*/10 * * * *`
会让整次部署直接失败。GitHub Actions 的定时工作流免费，所以由它每 10 分钟敲一次
站点的 `/api/internal/proactive`；要不要真的判断、判断谁，仍由站点自己的排期
（`proactive.next_at`）决定，所以空跑很便宜。

## 一次性配置（三步）

1. 仓库 **Settings → Secrets and variables → Actions → Repository secrets**，新建两个：

   | 名字 | 值 |
   |---|---|
   | `PERSONA_SITE_URL` | `https://www.aarome.xyz` |
   | `PERSONA_PROACTIVE_TOKEN` | 与站点环境变量同值的一串随机字符（在 Vercel 里配的那个） |

2. Vercel 项目 **Settings → Environment Variables** 加同名变量
   `PERSONA_PROACTIVE_TOKEN`，然后重新部署一次让它生效。
3. 回到本仓库 **Actions → 主动联系心跳 → Run workflow** 手动跑一次验证。
   日志里期望看到 `HTTP 200 {...}`。

## 没配好之前不会天天报警

- 站点还没部署带这个功能的版本 → `/api/internal/proactive` 返回 404，工作流直接跳过。
- 部署了但 Vercel 没配 token → `/health` 里 `proactive_timer: false`，接口返回 503，
  工作流标一个 warning 后正常结束（不算失败）。
- 只有真正的网络/服务异常才会红。

## 为什么这个仓库是公开的

GitHub 私有仓库的 Actions 有每月 2000 分钟额度，公开仓库不限时。这里跑的都是对外的
HTTPS 调用，仓库本身不含任何秘密——token 只存在 Secrets 里，绝不写进代码或日志。
