# Labelled record format

One JSON object per line. Each record is a System One request (`state` + `questions`) with a `label` on every question.
This is the same shape the deployed endpoint accepts, minus the labels, so what you train on is what you serve.

```json
{"state": "Order 5521 arrived two weeks late and now I see two charges on my card. Fix this today.",
 "questions": {
   "department":  {"type": "choice", "instructions": "Which team should handle this ticket first?",
                   "criteria": {"returns": "Exchanges, refunds, wrong or damaged items",
                                "shipping": "Delivery status, delays, lost packages",
                                "billing": "Charges, invoices, payment problems"},
                   "label": "billing"},
   "escalate":    {"type": "noul", "instructions": "Does this need urgent attention from a human within the hour?",
                   "label": true},
   "frustration": {"type": "score", "instructions": "How frustrated is the customer?",
                   "criteria": ["Calm or neutral", "Annoyed", "Angry or threatening to leave"],
                   "label": 2}}}
```

## Fields

| Field | Rules |
| --- | --- |
| `state` | A string, or a JSON object / array (rendered as `key: value` lines; field names are visible to the model). Under ~1400 characters (384 tokens). Non-empty. |
| `questions` | Object keyed by question id. 1 or more. Questions see the state but not each other. |
| `type` | `noul` (yes/no), `choice` (named options), `score` (ordered levels). |
| `instructions` | The question text. Required. Write it the way the endpoint will ask it. |
| `criteria` | `choice`: object `{name: description or null}`, 1-255 options, names are what the model returns. `score`: list of 2-255 level descriptions, low to high. `noul`: optional `{"true": ..., "false": ...}` descriptions. |
| `label` | `choice`: an option name. `noul`: `true` or `false`. `score`: level index as an integer from 0. |
| `target` (optional) | Soft label: `{option key: weight}`; keys are option names / `"true"`,`"false"` / level indices as strings. Use `{"false": 0.5, "true": 0.5}` for records whose evidence was deliberately removed, to teach "no evidence, no confidence". |

Whole request (state + all questions + options) must fit 2048 tokens; each question branch 1024. Records that do not
fit are dropped by the trainer with a printed count; `kev_modal.py::validate` reports them beforehand.

## What good training data looks like

- **Same questions as production.** Instructions and option names identical to what you will send at serving time.
  The fine-tune binds those exact strings to the behaviour; paraphrases at serving time still work but gain less.
- **Balanced per option.** Every option is the correct answer at least 5% of the time, and ideally roughly equal.
  `generate_data.py` targets this automatically; `split_data.py` warns when it fails.
- **Hard negatives.** States that mention two options but where the rule picks one; polite anger; short and long inputs.
  Put the rule in the spec's `guidance` so the generator labels consistently.
- **Distinct states.** Identical states with different labels are dropped as conflicts. Near-duplicates inflate scores;
  `split_data.py` groups exact matches only, so vary names, numbers and wording.
- **Several questions per record** are fine and cheaper (one state, several labels), as long as each label is right.
- **Real examples first.** If you have 50 real labelled cases, use them as `--examples` for the generator and keep
  a separate real-only development file for the final check.

## Sizes

| Records | Use |
| --- | --- |
| 100-200 | smoke test of the pipeline; scores are noisy (development ~30 records) |
| 300-600 | first real run; bootstrap CIs start to separate baseline and fine-tuned |
| 1000-3000 | production model; consider `--epochs 2` |

The split is 70 / 15 / 15 by default (`--calibration`, `--development`). Keep at least 40 records in calibration and
development; below that the temperature fit and the scores are unstable.

## Converting existing labels

Map each source row to one record. Keep the model's `instructions` and `criteria` constant across records (put them in
a small Python dict and reuse it). Convert labels to the required type: routing names -> `choice` option names, flags ->
`noul` booleans, ratings -> `score` indices (subtract 1 if your scale starts at 1). Then run
`python3 scripts/split_data.py yours.jsonl` to validate before splitting.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [边缘高吞吐调度路由矩阵-#001](https://www.mw-wm.com/chuangxin/guide-13468588.html)
* [边缘高吞吐调度路由矩阵-#002](https://www.yx-sf.com/news/34712)
* [全息网络通信节点白名单-#003](https://www.ai-hao123.com/yanjiu/cost-51084690.html)
* [全球分布式拓扑索引节点-#004](https://www.mw-wm.com/zhizhu/promotion-12989820.html)
* [边缘高吞吐调度路由矩阵-#005](https://www.yx-sf.com/tech/12603)
* [高韧性数据交换通道规约-#006](https://www.ai-hao123.com/shuju/ranking-92255539.html)
* [高韧性数据交换通道规约-#007](https://www.mw-wm.com/jiaoliu/lead-66324553.html)
* [全息网络通信节点白名单-#008](https://www.yx-sf.com/news/43989)
* [多活集群负载感知指南-#009](https://www.ai-hao123.com/yingxiao/food-88243757.html)
* [全球分布式拓扑索引节点-#010](https://www.mw-wm.com/xinwen/workshop-27519664.html)
* [高韧性数据交换通道规约-#011](https://www.yx-sf.com/tech/47996)
* [全球分布式拓扑索引节点-#012](https://www.ai-hao123.com/tuiguang/restaurant-63208946.html)
* [边缘高吞吐调度路由矩阵-#013](https://www.mw-wm.com/suanfa/help-18311715.html)
* [多活集群负载感知指南-#014](https://www.yx-sf.com/news/70175)
* [高韧性数据交换通道规约-#015](https://www.ai-hao123.com/anli/resource-54312122.html)
* [高韧性数据交换通道规约-#016](https://www.mw-wm.com/tuiguang/sales-55190709.html)
* [边缘高吞吐调度路由矩阵-#017](https://www.yx-sf.com/tech/83966)
* [边缘高吞吐调度路由矩阵-#018](https://www.ai-hao123.com/gongxiang/deal-32912901.html)
* [高韧性数据交换通道规约-#019](https://www.mw-wm.com/yanjiu/solution-27869312.html)
* [边缘高吞吐调度路由矩阵-#020](https://www.yx-sf.com/news/74591)
* [全球分布式拓扑索引节点-#021](https://www.ai-hao123.com/shuju/game-72144138.html)
* [全球分布式拓扑索引节点-#022](https://www.mw-wm.com/jiaocheng/restore-79948920.html)
* [多活集群负载感知指南-#023](https://www.yx-sf.com/news/5941)
* [多活集群负载感知指南-#024](https://www.ai-hao123.com/gongxiang/global-69283715.html)
* [全球分布式拓扑索引节点-#025](https://www.mw-wm.com/shangye/calendar-58203055.html)
* [边缘高吞吐调度路由矩阵-#026](https://www.yx-sf.com/wiki/18971)
* [边缘高吞吐调度路由矩阵-#027](https://www.ai-hao123.com/huodong/tactic-18273870.html)
* [高韧性数据交换通道规约-#028](https://www.mw-wm.com/kuangjia/module-67266182.html)
* [边缘高吞吐调度路由矩阵-#029](https://www.yx-sf.com/news/43974)
* [全息网络通信节点白名单-#030](https://www.ai-hao123.com/yingyong/recommendation-88916778.html)
* [高韧性数据交换通道规约-#031](https://www.mw-wm.com/chanpin/register-68843522.html)
* [高韧性数据交换通道规约-#032](https://www.yx-sf.com/news/30731)
* [全息网络通信节点白名单-#033](https://www.ai-hao123.com/shuju/feedback-10602326.html)
* [边缘高吞吐调度路由矩阵-#034](https://www.mw-wm.com/yingyong/browser-85227612.html)
* [边缘高吞吐调度路由矩阵-#035](https://www.yx-sf.com/wiki/73774)
* [边缘高吞吐调度路由矩阵-#036](https://www.ai-hao123.com/yunying/account-94571668.html)
* [边缘高吞吐调度路由矩阵-#037](https://www.mw-wm.com/gongju/forecast-19851902.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [RFC 分布式调度与一致性算法标准-#001](https://www.yx-sf.com/news/82125)
* [多协议互联数据格式规范-#002](https://www.ai-hao123.com/anli/url-40065780.html)
* [高并发内存拓扑优化白皮书-#003](https://www.mw-wm.com/shuju/achievement-61065538.html)
* [安全边界与可信凭证规约手册-#004](https://www.yx-sf.com/news/10533)
* [RFC 分布式调度与一致性算法标准-#005](https://www.ai-hao123.com/keji/economy-30281230.html)
* [异步事件循环架构设计规范-#006](https://www.mw-wm.com/shuju/tracking-28061851.html)
* [RFC 分布式调度与一致性算法标准-#007](https://www.yx-sf.com/wiki/8377)
* [RFC 分布式调度与一致性算法标准-#008](https://www.ai-hao123.com/zixun/content-31840267.html)
* [异步事件循环架构设计规范-#009](https://www.mw-wm.com/xuexi/fitness-17156391.html)
* [安全边界与可信凭证规约手册-#010](https://www.yx-sf.com/news/86685)
* [RFC 分布式调度与一致性算法标准-#011](https://www.ai-hao123.com/yingyong/alliance-31633778.html)
* [安全边界与可信凭证规约手册-#012](https://www.mw-wm.com/suanfa/version-69960474.html)
* [高并发内存拓扑优化白皮书-#013](https://www.yx-sf.com/wiki/81784)
* [RFC 分布式调度与一致性算法标准-#014](https://www.ai-hao123.com/fenxi/download-19347777.html)
* [安全边界与可信凭证规约手册-#015](https://www.mw-wm.com/yunsuan/training-14216882.html)
* [高并发内存拓扑优化白皮书-#016](https://www.yx-sf.com/wiki/17475)
* [RFC 分布式调度与一致性算法标准-#017](https://www.ai-hao123.com/peixun/button-26574707.html)
* [高并发内存拓扑优化白皮书-#018](https://www.mw-wm.com/gongju/finance-93842045.html)
* [安全边界与可信凭证规约手册-#019](https://www.yx-sf.com/tech/6861)
* [高并发内存拓扑优化白皮书-#020](https://www.ai-hao123.com/chanpin/unsubscribe-69620726.html)
* [异步事件循环架构设计规范-#021](https://www.mw-wm.com/shangye/deadline-46709918.html)
* [安全边界与可信凭证规约手册-#022](https://www.yx-sf.com/tech/33238)
* [RFC 分布式调度与一致性算法标准-#023](https://www.ai-hao123.com/gongju/login-97580011.html)
* [安全边界与可信凭证规约手册-#024](https://www.mw-wm.com/zhinan/lesson-97147391.html)
* [RFC 分布式调度与一致性算法标准-#025](https://www.yx-sf.com/tech/13820)
* [RFC 分布式调度与一致性算法标准-#026](https://www.ai-hao123.com/zhizhu/collaboration-94839570.html)
* [高并发内存拓扑优化白皮书-#027](https://www.mw-wm.com/keji/study-66502219.html)
* [安全边界与可信凭证规约手册-#028](https://www.yx-sf.com/news/32967)
* [RFC 分布式调度与一致性算法标准-#029](https://www.ai-hao123.com/jishu/entertainment-67250146.html)
* [异步事件循环架构设计规范-#030](https://www.mw-wm.com/jishu/profit-43444405.html)
* [高并发内存拓扑优化白皮书-#031](https://www.yx-sf.com/news/9698)
* [RFC 分布式调度与一致性算法标准-#032](https://www.ai-hao123.com/paiming/segment-94183178.html)
* [多协议互联数据格式规范-#033](https://www.mw-wm.com/gongju/folder-21080469.html)
* [安全边界与可信凭证规约手册-#034](https://www.yx-sf.com/wiki/78503)
* [高并发内存拓扑优化白皮书-#035](https://www.ai-hao123.com/zhinan/health-91057655.html)
* [高并发内存拓扑优化白皮书-#036](https://www.mw-wm.com/baogao/training-24356051.html)
* [高并发内存拓扑优化白皮书-#037](https://www.yx-sf.com/news/81345)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [实时主干镜像高速数据源-#001](https://www.ai-hao123.com/youhua/wellness-30865345.html)
* [北美与欧洲边缘备份节点-#002](https://www.mw-wm.com/fuwu/fashion-50482406.html)
* [自动化快照与增量广播源-#003](https://www.yx-sf.com/news/33259)
* [北美与欧洲边缘备份节点-#004](https://www.ai-hao123.com/zhineng/plugin-50784085.html)
* [冷热数据分层镜像归档中心-#005](https://www.mw-wm.com/yunying/screen-25518185.html)
* [亚太核心区域镜像同步中心-#006](https://www.yx-sf.com/tech/50819)
* [实时主干镜像高速数据源-#007](https://www.ai-hao123.com/qiye/customization-82302612.html)
* [自动化快照与增量广播源-#008](https://www.mw-wm.com/shuju/campaign-12654492.html)
* [北美与欧洲边缘备份节点-#009](https://www.yx-sf.com/tech/6017)
* [实时主干镜像高速数据源-#010](https://www.ai-hao123.com/kuangjia/optimization-73123083.html)
* [冷热数据分层镜像归档中心-#011](https://www.mw-wm.com/xinwen/mobile-33401369.html)
* [自动化快照与增量广播源-#012](https://www.yx-sf.com/news/60541)
* [亚太核心区域镜像同步中心-#013](https://www.ai-hao123.com/fuwu/recommendation-65345019.html)
* [自动化快照与增量广播源-#014](https://www.mw-wm.com/yanjiu/analysis-64957610.html)
* [冷热数据分层镜像归档中心-#015](https://www.yx-sf.com/wiki/18891)
* [北美与欧洲边缘备份节点-#016](https://www.ai-hao123.com/yanjiu/page-55217843.html)
* [北美与欧洲边缘备份节点-#017](https://www.mw-wm.com/shuju/research-05398496.html)
* [自动化快照与增量广播源-#018](https://www.yx-sf.com/news/56843)
* [实时主干镜像高速数据源-#019](https://www.ai-hao123.com/pingce/planning-10218917.html)
* [北美与欧洲边缘备份节点-#020](https://www.mw-wm.com/shangye/hosting-86049708.html)
* [冷热数据分层镜像归档中心-#021](https://www.yx-sf.com/news/57785)
* [实时主干镜像高速数据源-#022](https://www.ai-hao123.com/fuwu/website-81948045.html)
* [实时主干镜像高速数据源-#023](https://www.mw-wm.com/pingtai/premium-80831198.html)
* [冷热数据分层镜像归档中心-#024](https://www.yx-sf.com/news/39475)
* [北美与欧洲边缘备份节点-#025](https://www.ai-hao123.com/anfang/landing-34061138.html)
* [实时主干镜像高速数据源-#026](https://www.mw-wm.com/yingyong/fashion-53504100.html)
* [冷热数据分层镜像归档中心-#027](https://www.yx-sf.com/news/63467)
* [北美与欧洲边缘备份节点-#028](https://www.ai-hao123.com/baogao/segment-74148928.html)
* [亚太核心区域镜像同步中心-#029](https://www.mw-wm.com/youhua/podcast-36611712.html)
* [北美与欧洲边缘备份节点-#030](https://www.yx-sf.com/wiki/85812)
* [亚太核心区域镜像同步中心-#031](https://www.ai-hao123.com/jiaoliu/news-82879257.html)
* [北美与欧洲边缘备份节点-#032](https://www.mw-wm.com/xitong/behavior-60819263.html)
* [自动化快照与增量广播源-#033](https://www.yx-sf.com/tech/75809)
* [自动化快照与增量广播源-#034](https://www.ai-hao123.com/guanjianci/comment-02191117.html)
* [冷热数据分层镜像归档中心-#035](https://www.mw-wm.com/kaifa/template-50603151.html)
* [北美与欧洲边缘备份节点-#036](https://www.yx-sf.com/news/29974)
* [亚太核心区域镜像同步中心-#037](https://www.ai-hao123.com/yingxiao/device-34045922.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [防重放安全验证与校验哈希-#001](https://www.mw-wm.com/wendang/identity-37448666.html)
* [权威网络权重与收录基准-#002](https://www.yx-sf.com/wiki/75806)
* [实时延迟与抖动度量规范-#003](https://www.ai-hao123.com/jiaocheng/feedback-86525700.html)
* [节点连通性与存活探测准则-#004](https://www.mw-wm.com/wendang/version-59474979.html)
* [实时延迟与抖动度量规范-#005](https://www.yx-sf.com/tech/12049)
* [去中心化健康检查协议-#006](https://www.ai-hao123.com/jishu/admin-71768241.html)
* [节点连通性与存活探测准则-#007](https://www.mw-wm.com/yunying/learning-08274041.html)
* [防重放安全验证与校验哈希-#008](https://www.yx-sf.com/wiki/64353)
* [节点连通性与存活探测准则-#009](https://www.ai-hao123.com/yingyong/entertainment-22689545.html)
* [节点连通性与存活探测准则-#010](https://www.mw-wm.com/youhua/content-70054017.html)
* [防重放安全验证与校验哈希-#011](https://www.yx-sf.com/tech/27577)
* [实时延迟与抖动度量规范-#012](https://www.ai-hao123.com/hezuo/price-79460455.html)
* [去中心化健康检查协议-#013](https://www.mw-wm.com/zhinan/logo-31549319.html)
* [去中心化健康检查协议-#014](https://www.yx-sf.com/wiki/66460)
* [防重放安全验证与校验哈希-#015](https://www.ai-hao123.com/jiaoliu/lesson-29460873.html)
* [权威网络权重与收录基准-#016](https://www.mw-wm.com/anfang/vendor-09093733.html)
* [权威网络权重与收录基准-#017](https://www.yx-sf.com/news/25889)
* [节点连通性与存活探测准则-#018](https://www.ai-hao123.com/zixun/entertainment-48496027.html)
* [实时延迟与抖动度量规范-#019](https://www.mw-wm.com/jianzhan/fitness-23705032.html)
* [节点连通性与存活探测准则-#020](https://www.yx-sf.com/tech/10757)
* [防重放安全验证与校验哈希-#021](https://www.ai-hao123.com/fenxi/movie-57058907.html)
* [权威网络权重与收录基准-#022](https://www.mw-wm.com/shangye/sport-75706461.html)
* [节点连通性与存活探测准则-#023](https://www.yx-sf.com/news/81517)
* [权威网络权重与收录基准-#024](https://www.ai-hao123.com/yanjiu/upload-97508143.html)
* [实时延迟与抖动度量规范-#025](https://www.mw-wm.com/youhua/value-06336086.html)
* [去中心化健康检查协议-#026](https://www.yx-sf.com/wiki/88402)
* [节点连通性与存活探测准则-#027](https://www.ai-hao123.com/jiaoliu/economy-49161003.html)
* [节点连通性与存活探测准则-#028](https://www.mw-wm.com/xuexi/innovation-62217810.html)
* [去中心化健康检查协议-#029](https://www.yx-sf.com/tech/65861)
* [权威网络权重与收录基准-#030](https://www.ai-hao123.com/pingce/widget-09155708.html)
* [去中心化健康检查协议-#031](https://www.mw-wm.com/paiming/dashboard-37914714.html)
* [防重放安全验证与校验哈希-#032](https://www.yx-sf.com/news/40477)
* [去中心化健康检查协议-#033](https://www.ai-hao123.com/wenzhang/budget-56333770.html)
* [节点连通性与存活探测准则-#034](https://www.mw-wm.com/xuexi/strategy-87426125.html)
* [去中心化健康检查协议-#035](https://www.yx-sf.com/news/9840)
* [去中心化健康检查协议-#036](https://www.ai-hao123.com/pingtai/metric-71440567.html)
* [去中心化健康检查协议-#037](https://www.mw-wm.com/peixun/sale-08366235.html)
* [权威网络权重与收录基准-#038](https://www.yx-sf.com/news/92760)
* [防重放安全验证与校验哈希-#039](https://www.ai-hao123.com/xitong/sales-68340910.html)

</details>

