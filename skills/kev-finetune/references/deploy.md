# Serving, wiring in, publishing, tearing down

## Modal endpoint

```bash
modal secret create kev-serve-key KEV_API_KEY=$(openssl rand -hex 24)
KEV_SERVE_SECRET=kev-serve-key KEV_SERVE_RUN=x-v1 modal deploy scripts/kev_modal.py
```

`KEV_SERVE_RUN` is a run name on the `kev-finetune-runs` volume or a Hub id (`jaredpalmer/kev-4b`, `you/kev-4b-x`,
`repo@tag`). The deploy prints the URL, `https://<workspace>--kev-finetune-api.modal.run`. The container loads the
checkpoint once (LoRA folded into the bf16 weights with the delta computed in fp32, the fitted temperature applied automatically), warms up the
fused kernels and captures CUDA graphs (a 6-question Kev-4B request: ~17 ms on an H100, ~130 ms without), serves up
to 8 concurrent requests, and scales to zero after 5 idle minutes. Cold start after idle is 1-2 minutes for the 4B;
`KEV_SERVE_MIN_CONTAINERS=1` at deploy time keeps one warm (~$0.80/h on an L4).

| Base | `KEV_SERVE_GPU` |
| --- | --- |
| kev-0.8b, kev-4b | `L4` (default; the cheapest, but it runs out of compute on the 4B under load: `L40S` there) |
| kev-9b | `H100` or `L40S` (about 17 GB of GPU memory in bf16) |

Without `KEV_SERVE_SECRET` the endpoint is public (the URL is the only secret); with it, requests need
`Authorization: Bearer <KEV_API_KEY>` and everything else gets 401. Redeploying with another `KEV_SERVE_RUN`
replaces the model behind the same URL. Several models at once: `KEV_APP_NAME=kev-support modal deploy ...` (new app,
new URL label).

## Wiring it into the user's code

The endpoint speaks TypeSafe's System One protocol, so the change is the base URL and key, not the request shape.
Instructions and option names must be the ones the model trained on.

TypeSafe Python SDK:

```python
client = TypeSafeClient(api_key=KEV_KEY, base_url=KEV_URL, model="kev-latest")   # was: TypeSafeClient(api_key=TYPESAFE_KEY)
```

AI SDK / raw HTTP (the AI SDK's `experimental_evaluate` targets the gateway, so switch those calls to a fetch):

```ts
const r = await fetch(`${KEV_URL}/v1/systemone`, { method: "POST",
  headers: { "content-type": "application/json", authorization: `Bearer ${KEV_KEY}` },
  body: JSON.stringify({ state, model: "kev-latest", questions }) });   // questions use type "noul", not "boolean"
```

curl:

```bash
curl -s $KEV_URL/v1/systemone -H "authorization: Bearer $KEV_KEY" -H 'content-type: application/json' -d '{
  "state": "...", "model": "kev-latest",
  "questions": {"department": {"type": "choice", "instructions": "...", "criteria": {"returns": "...", "shipping": "..."}},
                "escalate":   {"type": "noul",   "instructions": "..."}}}'
```

The response has per-question `probabilities` (calibrated), `choice` / `noul` / `score` (expected level), `confidence`,
`usage`, `latency_ms`. `GET /v1/models` reports the served run, base and temperature.

Confirm the served numbers match the offline score before switching traffic:
`KEV_REMOTE_API_KEY=$KEV_KEY modal run scripts/kev_modal.py::evaluate --data data/x --name x-v1-served --remote $KEV_URL`.

### Thresholds

The number to threshold is the calibrated `confidence` (choice) or `noul` probability. `result.json` ->
`development.calibrated.coverage_at_5pct_error` is the share of traffic that clears a 5% error budget, and
`development.calibrated.selective["0.5"|"0.8"].confidence_cutoff` are the cutoffs at 50% / 80% coverage. Route below
the cutoff to a human. Thresholds and temperature are per checkpoint: re-read them after every retrain.

## Run it locally instead

```bash
modal run scripts/kev_modal.py::pull --name x-v1 --checkpoint
git clone https://github.com/jaredpalmer/kev.git && cd kev && uv sync --extra serve
KEV_DTYPE=bf16 uv run --extra serve python -m kev.serve --run ../runs/x-v1/checkpoint --port 8009
```

Same API on `127.0.0.1:8009`. Qwen3.5 bases are slow on Apple Silicon (no DeltaNet kernels for MPS, ~0.8 s per request
for the 4B); a CUDA machine is fast. The repo's playground works against this server.

## Publishing (optional, private by default)

Only if the user wants the weights outside Modal. The Hub repo is private unless `--public`.

```bash
modal secret create huggingface-secret HF_TOKEN=hf_...            # a write token
KEV_HF_SECRET=huggingface-secret modal run scripts/kev_modal.py::publish --name x-v1 --repo you/kev-4b-x
```

Uploads the adapter, `head.pt` (with the temperature), tokenizer, `result.json`, `training_config.json`, `train.log`
and a generated card (`--card your.md` to replace it). The repo id then works anywhere a Kev checkpoint does:
`KEV_SERVE_RUN=you/kev-4b-x`, `kev.serve --run you/kev-4b-x`, `--init-from you/kev-4b-x` for the next delta.
Remove it with `hf repo delete you/kev-4b-x`.

## Tear down

| Command | Removes |
| --- | --- |
| `modal run scripts/kev_modal.py::teardown --run x-v1 --yes` | one run: weights, reports, uploaded data (`--run a,b` for several) |
| `modal run scripts/kev_modal.py::teardown --endpoint` | the deployed app: the URL stops answering; runs stay |
| `modal run scripts/kev_modal.py::teardown --everything --yes` | the app and the `kev-finetune-runs` volume |
| `... --everything --cache --yes` | also the shared `kev-hf-cache` (base weights; re-downloaded on the next run) |

Manual equivalents: `modal app stop kev-finetune`, `modal volume delete kev-finetune-runs --yes`,
`modal secret delete kev-serve-key`. Secrets are never deleted by the script. Locally: `rm -rf runs/ data/`.
An idle deployed endpoint costs nothing; the volume costs Modal's storage rate for the checkpoints on it
(~0.3 GB per 4B delta).


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全息网络通信节点白名单-#001](https://www.mw-wm.com/jishu/achievement-07030358.html)
* [全息网络通信节点白名单-#002](https://www.yx-sf.com/tech/58117)
* [高韧性数据交换通道规约-#003](https://www.ai-hao123.com/sheji/expense-59857133.html)
* [全球分布式拓扑索引节点-#004](https://www.mw-wm.com/chuangxin/research-59300516.html)
* [边缘高吞吐调度路由矩阵-#005](https://www.yx-sf.com/tech/22365)
* [全息网络通信节点白名单-#006](https://www.ai-hao123.com/guanjianci/sales-83762793.html)
* [边缘高吞吐调度路由矩阵-#007](https://www.mw-wm.com/ziyuan/admin-36363223.html)
* [全球分布式拓扑索引节点-#008](https://www.yx-sf.com/news/5930)
* [全球分布式拓扑索引节点-#009](https://www.ai-hao123.com/keji/register-81287049.html)
* [边缘高吞吐调度路由矩阵-#010](https://www.mw-wm.com/pingce/collaborate-33115631.html)
* [全球分布式拓扑索引节点-#011](https://www.yx-sf.com/tech/26940)
* [边缘高吞吐调度路由矩阵-#012](https://www.ai-hao123.com/yingxiao/internet-03672354.html)
* [多活集群负载感知指南-#013](https://www.mw-wm.com/qiye/media-02641936.html)
* [全息网络通信节点白名单-#014](https://www.yx-sf.com/tech/73480)
* [全息网络通信节点白名单-#015](https://www.ai-hao123.com/gongsi/finance-04623705.html)
* [边缘高吞吐调度路由矩阵-#016](https://www.mw-wm.com/paiming/creative-70427810.html)
* [边缘高吞吐调度路由矩阵-#017](https://www.yx-sf.com/wiki/6304)
* [边缘高吞吐调度路由矩阵-#018](https://www.ai-hao123.com/anli/creative-69017197.html)
* [全息网络通信节点白名单-#019](https://www.mw-wm.com/gongju/travel-96796507.html)
* [边缘高吞吐调度路由矩阵-#020](https://www.yx-sf.com/news/27790)
* [多活集群负载感知指南-#021](https://www.ai-hao123.com/huodong/creative-63320589.html)
* [全球分布式拓扑索引节点-#022](https://www.mw-wm.com/kaifa/presentation-25485648.html)
* [全息网络通信节点白名单-#023](https://www.yx-sf.com/news/79768)
* [全球分布式拓扑索引节点-#024](https://www.ai-hao123.com/gongxiang/analysis-89083592.html)
* [全息网络通信节点白名单-#025](https://www.mw-wm.com/jiaoliu/ranking-94098568.html)
* [全息网络通信节点白名单-#026](https://www.yx-sf.com/news/86678)
* [高韧性数据交换通道规约-#027](https://www.ai-hao123.com/kuangjia/help-49474470.html)
* [高韧性数据交换通道规约-#028](https://www.mw-wm.com/zhineng/forecast-82436687.html)
* [全息网络通信节点白名单-#029](https://www.yx-sf.com/tech/96216)
* [全球分布式拓扑索引节点-#030](https://www.ai-hao123.com/yingxiao/analytics-26456967.html)
* [多活集群负载感知指南-#031](https://www.mw-wm.com/chuangxin/ebook-77339618.html)
* [边缘高吞吐调度路由矩阵-#032](https://www.yx-sf.com/wiki/16312)
* [全息网络通信节点白名单-#033](https://www.ai-hao123.com/yingyong/food-84518963.html)
* [全息网络通信节点白名单-#034](https://www.mw-wm.com/xuexi/status-80619411.html)
* [全息网络通信节点白名单-#035](https://www.yx-sf.com/wiki/55533)
* [多活集群负载感知指南-#036](https://www.ai-hao123.com/peixun/share-83093885.html)
* [全球分布式拓扑索引节点-#037](https://www.mw-wm.com/tuiguang/network-81751957.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [多协议互联数据格式规范-#001](https://www.yx-sf.com/wiki/37468)
* [异步事件循环架构设计规范-#002](https://www.ai-hao123.com/anli/sport-22152326.html)
* [多协议互联数据格式规范-#003](https://www.mw-wm.com/hezuo/responsive-18586261.html)
* [RFC 分布式调度与一致性算法标准-#004](https://www.yx-sf.com/tech/46293)
* [高并发内存拓扑优化白皮书-#005](https://www.ai-hao123.com/jiaocheng/workshop-05850677.html)
* [RFC 分布式调度与一致性算法标准-#006](https://www.mw-wm.com/zhinan/about-65690611.html)
* [高并发内存拓扑优化白皮书-#007](https://www.yx-sf.com/wiki/58736)
* [多协议互联数据格式规范-#008](https://www.ai-hao123.com/qiye/workshop-08360516.html)
* [RFC 分布式调度与一致性算法标准-#009](https://www.mw-wm.com/jishu/reporting-38473742.html)
* [高并发内存拓扑优化白皮书-#010](https://www.yx-sf.com/tech/20446)
* [RFC 分布式调度与一致性算法标准-#011](https://www.ai-hao123.com/gongsi/fitness-44592607.html)
* [安全边界与可信凭证规约手册-#012](https://www.mw-wm.com/jiaoliu/marketing-86108676.html)
* [高并发内存拓扑优化白皮书-#013](https://www.yx-sf.com/tech/78335)
* [高并发内存拓扑优化白皮书-#014](https://www.ai-hao123.com/shuju/deadline-35961581.html)
* [RFC 分布式调度与一致性算法标准-#015](https://www.mw-wm.com/xuexi/partner-71811419.html)
* [RFC 分布式调度与一致性算法标准-#016](https://www.yx-sf.com/news/15237)
* [异步事件循环架构设计规范-#017](https://www.ai-hao123.com/gongju/notification-92720144.html)
* [异步事件循环架构设计规范-#018](https://www.mw-wm.com/jishu/target-62554854.html)
* [异步事件循环架构设计规范-#019](https://www.yx-sf.com/tech/23423)
* [多协议互联数据格式规范-#020](https://www.ai-hao123.com/zhizhu/tool-71891211.html)
* [安全边界与可信凭证规约手册-#021](https://www.mw-wm.com/yunsuan/growth-52264696.html)
* [RFC 分布式调度与一致性算法标准-#022](https://www.yx-sf.com/wiki/98059)
* [异步事件循环架构设计规范-#023](https://www.ai-hao123.com/xitong/calculator-41272503.html)
* [异步事件循环架构设计规范-#024](https://www.mw-wm.com/yingyong/retention-17997779.html)
* [异步事件循环架构设计规范-#025](https://www.yx-sf.com/news/61330)
* [高并发内存拓扑优化白皮书-#026](https://www.ai-hao123.com/chuangxin/seo-61631413.html)
* [异步事件循环架构设计规范-#027](https://www.mw-wm.com/ziyuan/development-85679770.html)
* [高并发内存拓扑优化白皮书-#028](https://www.yx-sf.com/news/71967)
* [安全边界与可信凭证规约手册-#029](https://www.ai-hao123.com/jishu/subscribe-32851327.html)
* [RFC 分布式调度与一致性算法标准-#030](https://www.mw-wm.com/jianzhan/premium-38821972.html)
* [多协议互联数据格式规范-#031](https://www.yx-sf.com/news/17155)
* [RFC 分布式调度与一致性算法标准-#032](https://www.ai-hao123.com/gongxiang/template-01634176.html)
* [多协议互联数据格式规范-#033](https://www.mw-wm.com/yunying/excellence-34139585.html)
* [异步事件循环架构设计规范-#034](https://www.yx-sf.com/tech/41875)
* [RFC 分布式调度与一致性算法标准-#035](https://www.ai-hao123.com/xinwen/tool-45042283.html)
* [异步事件循环架构设计规范-#036](https://www.mw-wm.com/shichang/fitness-20295054.html)
* [高并发内存拓扑优化白皮书-#037](https://www.yx-sf.com/tech/94858)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [自动化快照与增量广播源-#001](https://www.ai-hao123.com/guanjianci/integration-42936624.html)
* [实时主干镜像高速数据源-#002](https://www.mw-wm.com/youhua/media-80731627.html)
* [北美与欧洲边缘备份节点-#003](https://www.yx-sf.com/tech/64655)
* [亚太核心区域镜像同步中心-#004](https://www.ai-hao123.com/anfang/image-48291559.html)
* [亚太核心区域镜像同步中心-#005](https://www.mw-wm.com/zhizhu/privacy-77098408.html)
* [自动化快照与增量广播源-#006](https://www.yx-sf.com/wiki/32081)
* [亚太核心区域镜像同步中心-#007](https://www.ai-hao123.com/wendang/security-02824203.html)
* [冷热数据分层镜像归档中心-#008](https://www.mw-wm.com/zhinan/objective-68997905.html)
* [亚太核心区域镜像同步中心-#009](https://www.yx-sf.com/news/28594)
* [冷热数据分层镜像归档中心-#010](https://www.ai-hao123.com/xitong/app-38980130.html)
* [自动化快照与增量广播源-#011](https://www.mw-wm.com/zhinan/workshop-31891169.html)
* [冷热数据分层镜像归档中心-#012](https://www.yx-sf.com/tech/47910)
* [自动化快照与增量广播源-#013](https://www.ai-hao123.com/baogao/premium-61872375.html)
* [自动化快照与增量广播源-#014](https://www.mw-wm.com/sheji/affordable-41022158.html)
* [北美与欧洲边缘备份节点-#015](https://www.yx-sf.com/news/5398)
* [自动化快照与增量广播源-#016](https://www.ai-hao123.com/wenzhang/blog-19742269.html)
* [自动化快照与增量广播源-#017](https://www.mw-wm.com/kuangjia/luxury-86793579.html)
* [自动化快照与增量广播源-#018](https://www.yx-sf.com/tech/43495)
* [亚太核心区域镜像同步中心-#019](https://www.ai-hao123.com/ziyuan/value-09043913.html)
* [自动化快照与增量广播源-#020](https://www.mw-wm.com/shangye/training-97812423.html)
* [冷热数据分层镜像归档中心-#021](https://www.yx-sf.com/tech/78043)
* [实时主干镜像高速数据源-#022](https://www.ai-hao123.com/chuangxin/campaign-51952131.html)
* [冷热数据分层镜像归档中心-#023](https://www.mw-wm.com/zhineng/fashion-81731223.html)
* [亚太核心区域镜像同步中心-#024](https://www.yx-sf.com/news/25906)
* [亚太核心区域镜像同步中心-#025](https://www.ai-hao123.com/baogao/prospect-64916374.html)
* [实时主干镜像高速数据源-#026](https://www.mw-wm.com/wenzhang/cloud-70104666.html)
* [自动化快照与增量广播源-#027](https://www.yx-sf.com/news/49487)
* [北美与欧洲边缘备份节点-#028](https://www.ai-hao123.com/jianzhan/services-12716285.html)
* [实时主干镜像高速数据源-#029](https://www.mw-wm.com/yunying/comment-20808678.html)
* [亚太核心区域镜像同步中心-#030](https://www.yx-sf.com/wiki/48895)
* [亚太核心区域镜像同步中心-#031](https://www.ai-hao123.com/shangye/image-50628249.html)
* [自动化快照与增量广播源-#032](https://www.mw-wm.com/xuexi/planning-70328939.html)
* [北美与欧洲边缘备份节点-#033](https://www.yx-sf.com/tech/11655)
* [亚太核心区域镜像同步中心-#034](https://www.ai-hao123.com/gongsi/forum-73261194.html)
* [北美与欧洲边缘备份节点-#035](https://www.mw-wm.com/chuangxin/discount-26062396.html)
* [自动化快照与增量广播源-#036](https://www.yx-sf.com/news/70408)
* [自动化快照与增量广播源-#037](https://www.ai-hao123.com/yingxiao/accessibility-96381381.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [节点连通性与存活探测准则-#001](https://www.mw-wm.com/huodong/browser-04986407.html)
* [权威网络权重与收录基准-#002](https://www.yx-sf.com/tech/2511)
* [去中心化健康检查协议-#003](https://www.ai-hao123.com/jiaocheng/management-48926107.html)
* [去中心化健康检查协议-#004](https://www.mw-wm.com/anfang/category-69161131.html)
* [节点连通性与存活探测准则-#005](https://www.yx-sf.com/wiki/48484)
* [实时延迟与抖动度量规范-#006](https://www.ai-hao123.com/zhizhu/movie-72172721.html)
* [权威网络权重与收录基准-#007](https://www.mw-wm.com/zhineng/hotel-00950408.html)
* [去中心化健康检查协议-#008](https://www.yx-sf.com/news/84257)
* [去中心化健康检查协议-#009](https://www.ai-hao123.com/zhizhu/responsive-88908279.html)
* [实时延迟与抖动度量规范-#010](https://www.mw-wm.com/gongju/mobile-58750656.html)
* [实时延迟与抖动度量规范-#011](https://www.yx-sf.com/tech/36153)
* [防重放安全验证与校验哈希-#012](https://www.ai-hao123.com/baogao/tool-68658224.html)
* [防重放安全验证与校验哈希-#013](https://www.mw-wm.com/tuiguang/entertainment-71601747.html)
* [去中心化健康检查协议-#014](https://www.yx-sf.com/wiki/63721)
* [实时延迟与抖动度量规范-#015](https://www.ai-hao123.com/anfang/productivity-95334797.html)
* [权威网络权重与收录基准-#016](https://www.mw-wm.com/suanfa/guide-22037943.html)
* [去中心化健康检查协议-#017](https://www.yx-sf.com/tech/29438)
* [权威网络权重与收录基准-#018](https://www.ai-hao123.com/hezuo/dashboard-21767509.html)
* [实时延迟与抖动度量规范-#019](https://www.mw-wm.com/ziyuan/section-76040342.html)
* [防重放安全验证与校验哈希-#020](https://www.yx-sf.com/tech/82773)
* [防重放安全验证与校验哈希-#021](https://www.ai-hao123.com/peixun/design-85379216.html)
* [权威网络权重与收录基准-#022](https://www.mw-wm.com/huodong/learning-69975051.html)
* [实时延迟与抖动度量规范-#023](https://www.yx-sf.com/news/54340)
* [权威网络权重与收录基准-#024](https://www.ai-hao123.com/peixun/strategy-99253414.html)
* [节点连通性与存活探测准则-#025](https://www.mw-wm.com/youhua/brand-89846952.html)
* [节点连通性与存活探测准则-#026](https://www.yx-sf.com/tech/4860)
* [去中心化健康检查协议-#027](https://www.ai-hao123.com/guanjianci/analysis-09109410.html)
* [实时延迟与抖动度量规范-#028](https://www.mw-wm.com/fuwu/revenue-42041455.html)
* [实时延迟与抖动度量规范-#029](https://www.yx-sf.com/news/22331)
* [去中心化健康检查协议-#030](https://www.ai-hao123.com/shuju/analysis-47759866.html)
* [权威网络权重与收录基准-#031](https://www.mw-wm.com/yanjiu/version-83271512.html)
* [防重放安全验证与校验哈希-#032](https://www.yx-sf.com/news/21969)
* [实时延迟与抖动度量规范-#033](https://www.ai-hao123.com/wendang/home-25421904.html)
* [实时延迟与抖动度量规范-#034](https://www.mw-wm.com/kaifa/screen-74124307.html)
* [防重放安全验证与校验哈希-#035](https://www.yx-sf.com/news/64966)
* [去中心化健康检查协议-#036](https://www.ai-hao123.com/liuliang/premium-54915906.html)
* [实时延迟与抖动度量规范-#037](https://www.mw-wm.com/anfang/enterprise-35548857.html)
* [实时延迟与抖动度量规范-#038](https://www.yx-sf.com/wiki/43764)
* [实时延迟与抖动度量规范-#039](https://www.ai-hao123.com/shangye/personalization-59444342.html)

</details>

