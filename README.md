# OpenAI 免费模型速查

OpenAI **数据共享计划**的每日免费 token 名单速查 + 使用指南。一键部署到你自己的 Cloudflare 账号：不需要本地环境，不需要任何 API Key。

[![Deploy to Cloudflare](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/piscon/openai-free-models)

> 第三方整理工具，与 OpenAI 无隶属关系；数据来自 OpenAI 官方页面（见文末来源），以官方为准。

## 页面功能

- **速查表**：先选组织层级（Free / Build / Launch / Grow），34 个模型按 5 个任务分组（通用主力 / 深度推理 / 编程专用 / 视觉与长文 / 轻量高速），每个一键复制、整组复制、支持搜索
- **额度随层级切换**：旗舰池 / 小型池每日免费量（Build 25 万 · 250 万；Launch / Grow 100 万 · 1,000 万；Free 不在计划内为 0），含 UTC 00:00 重置倒计时和「去哪查自己层级」的路径
- **状态标记**：新锐 / 落幕（关停日期、剩余天数、官方接班人）/ 已关停
- **使用指南**：资格自查、四层级对照表、开启步骤、规则与坑（同池共享、超额整单计费、工具调用不覆盖、免费与 Fast 加速二选一）、落幕迁移表、FAQ
- 深浅双主题（浅色 OpenAI 黑白灰 / 深色 Tokyo Night），手机端适配

## 快速开始

需要：一个 **Cloudflare 账号**（免费）和一个 **GitHub / GitLab 账号**，全程在浏览器里完成。

### 路线 A：一键部署（推荐，约 2 分钟）

1. 点上方 **Deploy to Cloudflare** 按钮，登录 Cloudflare 并授权它访问你的 GitHub / GitLab
2. 确认项目名后点部署：Cloudflare 会在你账号下**复制一份仓库**（独立副本，不是 fork），自动开通 KV 并部署 Worker
3. **必做：开启访问地址** —— 进入 Worker → *Settings → Domains & Routes* → **workers.dev** 一行点 **Enable**（第一次用这个账号可能先让你定一个子域名，任填即可）。地址形如 `https://openai-free-models.<你的子域>.workers.dev`，打开看到速查页即完成
4. 顺手确认定时任务：Worker → *Settings → Triggers* 里应有一条 `17 3 * * *` 的 Cron（每天 UTC 03:17 自动同步关停数据）

注：部署生成的副本**不包含 `.github/workflows/` 里的文件**（平台限制，实测确认），所以「自动同步上游」的 Action 需要按下方「如何更新」手动放一次；这不影响 Worker 自带的每日数据同步。

### 路线 B：先 Fork（想用 GitHub 的 Sync fork 更新）

1. **Fork** 本仓库到你的账号
2. 在 fork 的 `wrangler.jsonc` 里补 KV id：先去 Cloudflare 控制台 *Storage & Databases → KV* 建一个命名空间，把 id 填进 `kv_namespaces`
3. 控制台 → *Workers → Create → Import a repository*，选你的 fork 导入部署；同样别忘了按路线 A 第 3 步开启 workers.dev 地址
4. 以后更新：fork 仓库页点 **Sync fork → Update branch** 即可

## 部署排错

- **地址打不开**：多半是 workers.dev 没开启，见路线 A 第 3 步
- **构建在最后一步报 `code: 10072`（Cron 触发器达到上限）**：免费版整个账号最多 5 个 Cron 触发器，本项目占 1 个。报错只卡在挂定时任务一步，**页面通常已经部署成功**。处理：删掉不用的 Worker（或删它 *Settings → Triggers* 里的 Cron）腾出名额，再回到本 Worker → *Settings → Triggers* 手动添加 `17 3 * * *`，不用重新部署。测试实例测完记得删

## 如何更新到新版

你的实例不会自动跟随上游。先记住：**不要再点一次部署按钮**（会新建一整套，不是更新）；更新只换代码，不动 KV 里的数据。

- **方式一（推荐，不用命令行）**：把你副本里 `docs/sync-upstream.yml` 的内容，手动新建为副本的 `.github/workflows/sync-upstream.yml`（副本仓库 → *Add file → Create new file* → 粘贴 → 提交，约 1 分钟，只做这一次）。之后在副本 **Actions** 页运行 **Sync from upstream** 即可同步，它也会每周一自动跑一次；同步时会保留你 `wrangler.jsonc` 里的 KV id，若你改过文件与上游撞车会停下不乱推
- **方式二（命令行）**：`git remote add upstream https://github.com/piscon/openai-free-models.git` → `git pull upstream main`（注意保留 `wrangler.jsonc` 里的 KV id）→ `git push`
- 推送后 Workers Builds 自动重新部署，1–2 分钟生效

## 数据与设计要点

- **关停信息自动同步**：Worker 每天 UTC 03:17 抓取 [OpenAI 官方下架页](https://developers.openai.com/api/docs/deprecations)，解析关停日期与接班人写入 KV；没变化不写库，解析异常就保留旧数据
- **前台没有任何触发抓取的入口**（无按钮、无接口）：站点是公开的，这样设计是防访客把免费额度刷光，抓取只由定时任务执行
- **名单与额度用种子数据**：官方帮助文章对程序抓取返回 403，无法自动读取，所以名单放在 `data/models.json` 里人工核实更新，页面标注「核实于」日期
- 全部运行在 Cloudflare 免费额度内：静态页面不限量，每天只有 1 次定时抓取

## 自定义

- **抓取时间**：改 `wrangler.jsonc` 的 `triggers.crons`；**关掉自动同步**：删掉 `triggers` 整段
- **改名单数据**：编辑 `data/models.json` 后运行 `npm run seed:sync`
- **本地开发**：`npm install` → `npm test`（解析器测试）→ `npm run dev`（本地预览）

## 数据来源

- 免费名单与各层级额度：[OpenAI 帮助文章 · 数据共享计划](https://help.openai.com/en/articles/10306912-sharing-feedback-evaluation-and-fine-tuning-data-and-api-inputs-and-outputs-with-openai)（种子核实于 2026-11-10）
- 层级定义：[OpenAI 开发者文档 · Rate limits（Usage tiers）](https://developers.openai.com/api/docs/guides/rate-limits)
- 关停日期与接班人：[OpenAI 官方 Deprecations 页](https://developers.openai.com/api/docs/deprecations)

## 致谢与交流

数据全部来自 OpenAI 官方公开文档，感谢官方把规则写明白；也感谢 [LINUX DO](https://linux.do) 社区的佬们——有问题、建议或想交流，欢迎到 [LINUX DO](https://linux.do) 找我，或者直接在本仓库提 Issue。

## License

代码 MIT（见 [LICENSE](./LICENSE)）。页面中的模型数据为对 OpenAI 官方公开信息的整理。
