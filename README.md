# OpenAI 免费模型速查

查 OpenAI 数据共享计划的每日免费 token：哪些模型免费、你的层级每天能用多少、哪些快关停了。一键部署一份到你自己的 Cloudflare 账号，之后不用管，它每天自己更新数据。

[![Deploy to Cloudflare](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/piscon/openai-free-models)

## 部署只需三步

需要一个免费的 Cloudflare 账号和一个 GitHub 账号，不需要本地环境、不需要任何 API Key。

1. **点上面的按钮**，登录 Cloudflare，确认项目名，点部署（等它跑完，约 2 分钟）
2. **开启你的网址**：进入刚建好的 Worker → *Settings → Domains & Routes* → **workers.dev** 一行点 **Enable**
3. **打开网址**（形如 `https://openai-free-models.<你的子域>.workers.dev`），看到速查页就完成了

## 你只需要知道三件事

- **数据不用你维护**：页面每天自动同步官方的关停信息；免费名单和额度偶有变化时，跟随上游更新一次即可（见下方进阶）
- **别再点第二次部署按钮**：那会新建一套，不是更新
- **网址打不开**，99% 是第 2 步的 workers.dev 没开启，回去点 Enable

<details>
<summary><strong>进阶：更新、排错、Fork 路线、自定义</strong>（大多数人用不到，遇到问题再看）</summary>

### 如何更新到新版

部署生成的是你账号下的**独立副本**（不是 fork，也不会自动跟随上游）。上游更新后，任选一种：

- **同步 Action（推荐）**：部署副本不包含 `.github/workflows/` 里的文件（平台限制），所以先手动放一次模板：把你副本里 `docs/sync-upstream.yml` 的内容，新建为副本的 `.github/workflows/sync-upstream.yml`（仓库页 *Add file → Create new file*，粘贴提交）。之后在副本 **Actions** 页运行 **Sync from upstream** 就能同步（它也会每周一自动跑）；同步保留你 `wrangler.jsonc` 里的 KV id
- **命令行**：`git remote add upstream https://github.com/piscon/openai-free-models.git` → `git pull upstream main`（保留 `wrangler.jsonc` 的 KV id）→ `git push`
- 更新只换代码，不动 KV 里的数据；推送后自动重新部署

### Fork 路线

先 Fork 本仓库 → 在 fork 的 `wrangler.jsonc` 里给 `kv_namespaces` 补上你新建 KV 命名空间的 id（控制台 *Storage & Databases → KV*）→ 控制台 *Workers → Create → Import a repository* 选 fork 导入 → 同样要开启 workers.dev。以后更新在 fork 页点 **Sync fork → Update branch**。

### 排错：构建最后一步报 code: 10072

免费版整个账号最多 5 个 Cron 触发器，本项目占 1 个。报错只卡在挂定时任务一步，页面通常已部署成功：删掉不用的 Worker（或其 *Settings → Triggers* 里的 Cron）腾名额，再到本 Worker → *Settings → Triggers* 手动添加 `17 3 * * *` 即可，不用重新部署。正常情况下 Triggers 里应有这条 Cron（每天 UTC 03:17 同步数据）。

### 自定义与原理

- 改抓取时间：`wrangler.jsonc` 的 `triggers.crons`；关自动同步：删 `triggers` 段；改名单：编辑 `data/models.json` 后 `npm run seed:sync`；本地开发：`npm install`、`npm test`、`npm run dev`
- 数据保持新鲜的方式：Worker 每天抓一次 [OpenAI 官方下架页](https://developers.openai.com/api/docs/deprecations) 更新关停与接班人信息，没变化不写库；名单与额度来自官方帮助文章，该页无法被程序抓取（403），所以用种子数据人工核实（页面标注核实日期）。前台没有任何触发抓取的入口，是防公开访客刷光免费额度的有意设计
- 全部运行在 Cloudflare 免费额度内

### 数据来源

- [OpenAI 帮助文章 · 数据共享计划](https://help.openai.com/en/articles/10306912-sharing-feedback-evaluation-and-fine-tuning-data-and-api-inputs-and-outputs-with-openai)（名单与额度，核实于 2026-11-10）
- [OpenAI 开发者文档 · Rate limits](https://developers.openai.com/api/docs/guides/rate-limits)（Free / Build / Launch / Grow 层级定义）
- [OpenAI 官方 Deprecations 页](https://developers.openai.com/api/docs/deprecations)（关停日期与接班人）

</details>

## 致谢与交流

数据来自 OpenAI 官方公开文档；感谢 [LINUX DO](https://linux.do) 社区的佬们，有问题或建议欢迎到社区找我，或在本仓库提 Issue。

第三方整理工具，与 OpenAI 无隶属关系，最终以官方页面为准。代码 MIT（见 [LICENSE](./LICENSE)）。
