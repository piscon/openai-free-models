# OpenAI 免费模型速查

OpenAI **数据共享计划**的每日免费 token 名单速查 + 使用指南。点下面的按钮，别人就能在**自己的 Cloudflare 账号**里一键生成同款页面，不需要本地环境、不需要任何 API Key。

[![Deploy to Cloudflare](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/piscon/openai-free-models)

> 本项目是第三方整理工具，与 OpenAI 无隶属关系。数据来自 OpenAI 官方页面（见文末来源），最终以官方为准。

## 页面长什么样

- **速查表**：两个共享额度池的大数字（旗舰池 100 万/天、小型池 1,000 万/天，含 Build 层级数字）、距 UTC 00:00 重置的实时倒计时
- **5 个任务分组**：通用主力 / 深度推理 / 编程专用 / 视觉与长文 / 轻量高速，共 34 个模型，每个模型一键复制、每组整组复制、支持搜索
- **状态标记**：最新代模型挂「新锐」；已宣布关停的挂「落幕」并显示关停日期、剩余天数和官方接班人；已关停的单独标记
- **使用指南**：资格自查、开启步骤、额度规则与坑（同池共享、超额整单计费、不计入的部分）、按任务选模型、落幕迁移表、FAQ
- 深浅色双主题，手机端适配

## 一键部署（约 2 分钟）

1. 点上方 **Deploy to Cloudflare** 按钮
2. 登录 Cloudflare，并授权访问你的 GitHub / GitLab
3. Cloudflare 会自动：把本仓库克隆到你的账号 → 按 `wrangler.jsonc` 开通一个 KV 命名空间 → 构建并部署 Worker
4. 完成后你会得到一个 `https://openai-free-models.<你的子域>.workers.dev` 地址，打开即用

仓库必须公开（部署按钮的要求）。整个运行在 Cloudflare 免费额度内：页面是静态资源（免费不限量），每天只有 1 次定时抓取。

## 数据怎么保持新鲜（设计要点）

- **关停信息自动同步**：Worker 带一个 Cron 定时任务（默认每天 UTC 03:17），抓取 [OpenAI 官方下架页](https://developers.openai.com/api/docs/deprecations) 并解析关停日期与接班人，写入 KV。下架通知通常提前 3–6 个月发布，每日一次绰绰有余。
- **前台没有任何触发抓取的入口** —— 没有刷新按钮、没有刷新接口、没有参数。这是有意为之：站点是公开的，访客可触发的抓取会把免费额度（KV 免费版仅 1,000 次写/天）打光。抓取代码只存在于定时任务入口，并带三道闸：距上次尝试不足 20 小时直接跳过；条件请求（ETag）+ 内容哈希，没变化不写库；解析结果异常（匹配数过低）时放弃本次更新、保留旧数据。
- **模型名单与额度用种子数据**：名单出自官方帮助文章，但该页对普通程序抓取返回 403，无法可靠机读，所以名单放在 `data/models.json` 里人工核实后更新（页面会显示「核实于」日期）。名单历史上变化很慢；真有变化时改种子文件并重新部署即可，改完记得跑 `npm run seed:sync` 同步前端副本。

## 自定义

- **改抓取频率/时间**：改 `wrangler.jsonc` 里的 `triggers.crons`（cron 表达式，UTC）
- **改 Worker 名字**：改 `wrangler.jsonc` 的 `name`（部署前改；已部署的改名等于新建）
- **改名单数据**：编辑 `data/models.json`（`taskGroups` 五组、`pools` 两池、`verifiedAt` 核实日期），然后 `npm run seed:sync`
- **关掉自动同步**：删掉 `wrangler.jsonc` 的 `triggers` 段即可，页面退化为纯静态种子数据

## 本地开发

```bash
npm install
npm test          # 解析器自带夹具测试
node test/parse.test.mjs /path/to/deprecations.html   # 用真实下架页全文回归
npm run dev       # wrangler dev 本地预览
npm run deploy    # 手动部署（需要你自己的 Cloudflare 账号与 KV id）
```

手动部署时先建 KV：`npx wrangler kv namespace create MODELS_KV`，把得到的 id 填回 `wrangler.jsonc`。

## 数据来源

- 免费名单与额度：[OpenAI 帮助文章 · Sharing feedback, evaluation and fine-tuning data…](https://help.openai.com/en/articles/10306912-sharing-feedback-evaluation-and-fine-tuning-data-and-api-inputs-and-outputs-with-openai)（种子核实于 2026-11-10）
- 关停日期与接班人：[OpenAI 官方 Deprecations 页](https://developers.openai.com/api/docs/deprecations)

## License

代码 MIT（见 [LICENSE](./LICENSE)）。页面中的模型数据为对 OpenAI 官方公开信息的整理。
