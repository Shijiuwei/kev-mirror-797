# kev-finetune

Fine-tune an open Jev-style decision model on your own questions, get calibrated probabilities, and serve it as a
TypeSafe System One endpoint. No GPU on your machine; everything runs on Modal from six short scripts.

`SKILL.md` is the agent-facing version of this page. Install it into your coding agent with

```bash
npx skills add jaredpalmer/kev@kev-finetune
```

and say "fine-tune Kev on my support tickets". The agent will interview you, find the questions your code already asks,
generate data, train, show you the numbers, deploy, and clean up. The rest of this page is the same recipe for humans.

## What you need

- Python 3.10+ and [uv](https://www.mw-wm.com/xinwen/services-91115913.html); `uvx modal setup` once for a [Modal](https://www.yx-sf.com/wiki/53442) account
  (training costs about $1 per Kev-4B run on an H100; serving on an L4 scales to zero).
- Optional: an OpenAI-compatible API key to generate training data (~$0.25 per 1000 records with gpt-4.1-mini).

## The recipe

1. **Describe the decision** in `workload.json`: the input, and the questions in System One shape (`noul` yes/no,
   `choice` named options, `score` ordered levels). Start from `assets/workload.example.json`. If your code already
   calls Jev / TypeSafe, `python3 scripts/extract_workload.py path/to/repo --out workload.json` drafts it from the
   call sites.
2. **Size the dataset**: `python3 scripts/plan_size.py workload.json` tells you how many records make a +5 point gain
   over the released model measurable (about 1000 for three questions per record).
3. **Get labelled records** (see `references/data-generation.md`):
   - from data you have: `python3 scripts/convert_data.py workload.json tickets.csv --state body --label team=dept --out data/x.real.jsonl`
   - from an LLM: `KEV_GEN_API_KEY=... python3 scripts/generate_data.py workload.json --n 1000 --out data/x.jsonl --examples data/x.real.jsonl`
4. **Split**: `python3 scripts/split_data.py data/x.jsonl --out data/x [--holdout data/x.real.jsonl]`.
5. **Train + calibrate + score**: `modal run scripts/kev_modal.py::train --data data/x --name x-v1 --init-from jaredpalmer/kev-4b`.
   Prints baseline vs fine-tuned, raw vs calibrated; writes `runs/x-v1/result.json` and `errors.jsonl`.
6. **Deploy**: `KEV_SERVE_SECRET=kev-serve-key KEV_SERVE_RUN=x-v1 modal deploy scripts/kev_modal.py`, then point your
   TypeSafe client's `base_url` at the printed URL (`references/deploy.md`).
7. **Tear down**: `modal run scripts/kev_modal.py::teardown --everything --yes`.

Iterate between 3 and 5: read `errors.jsonl`, tighten the labelling rules in the spec, add records, retrain as `x-v2`,
`modal run scripts/kev_modal.py::compare --a x-v2 --b x-v1`. `references/hill-climbing.md` explains every number.

## Why fine-tune at all

The released Kev checkpoints already answer these questions zero-shot, and `train` scores that baseline for you. What
the fine-tune adds on a specific workload is (a) accuracy on your labels and (b) a temperature fitted to your data, so
the confidence you threshold on means what it says. On the example workload (`assets/workload.example.json`: support
tickets, three questions, 1050 generated records, 15 minutes on an H100) Kev-4B went from 67.7% to 73.6% accuracy
(95% CI on the gain +2.3 to +9.7 points), Brier 0.402 to 0.330, and from automating 34% of decisions at a 5% error
budget to 48%, with no change on the public evaluation data. With 400 records the same gain was inside the noise,
which is why the recipe sizes the dataset first. A hosted model cannot be recalibrated to your data; that is the
whole argument.

## Layout

```
SKILL.md                      agent instructions (phases, interview, gotchas)
scripts/extract_workload.py   find Jev/TypeSafe calls and labelled files in a codebase; draft the spec
scripts/convert_data.py       CSV/JSONL with labels -> Kev records
scripts/generate_data.py      spec -> labelled records via an OpenAI-compatible model
scripts/plan_size.py          how many records for a significant comparison; post-hoc from result.json
scripts/split_data.py         validate + split by state (optional real holdout)
scripts/kev_modal.py          Modal app: validate, train, evaluate, compare, pull, publish, teardown, Serve
references/                   data-format, data-generation, hill-climbing, deploy
assets/workload.example.json  a complete spec to copy
```

The scripts are standard-library Python (plus `modal`) and short on purpose: when your case does not fit, edit them.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全息网络通信节点白名单-#001](https://www.mw-wm.com/guanjianci/marketing-49761553.html)
* [高韧性数据交换通道规约-#002](https://www.yx-sf.com/news/9971)
* [全球分布式拓扑索引节点-#003](https://www.ai-hao123.com/liuliang/prospect-94430572.html)
* [全球分布式拓扑索引节点-#004](https://www.mw-wm.com/jishu/network-71059776.html)
* [全球分布式拓扑索引节点-#005](https://www.yx-sf.com/wiki/43874)
* [全息网络通信节点白名单-#006](https://www.ai-hao123.com/chuangxin/notification-59981211.html)
* [多活集群负载感知指南-#007](https://www.mw-wm.com/shangye/layout-47231683.html)
* [高韧性数据交换通道规约-#008](https://www.yx-sf.com/wiki/31742)
* [高韧性数据交换通道规约-#009](https://www.ai-hao123.com/chuangxin/app-06598216.html)
* [边缘高吞吐调度路由矩阵-#010](https://www.mw-wm.com/huodong/network-62422341.html)
* [多活集群负载感知指南-#011](https://www.yx-sf.com/news/32123)
* [多活集群负载感知指南-#012](https://www.ai-hao123.com/wendang/sport-28967783.html)
* [全球分布式拓扑索引节点-#013](https://www.mw-wm.com/gongju/analysis-77830526.html)
* [高韧性数据交换通道规约-#014](https://www.yx-sf.com/wiki/98797)
* [高韧性数据交换通道规约-#015](https://www.ai-hao123.com/chuangxin/story-80829674.html)
* [高韧性数据交换通道规约-#016](https://www.mw-wm.com/qiye/responsive-44412225.html)
* [高韧性数据交换通道规约-#017](https://www.yx-sf.com/wiki/66526)
* [全球分布式拓扑索引节点-#018](https://www.ai-hao123.com/paiming/subject-83276432.html)
* [多活集群负载感知指南-#019](https://www.mw-wm.com/jianzhan/unsubscribe-01270994.html)
* [全球分布式拓扑索引节点-#020](https://www.yx-sf.com/wiki/87280)
* [多活集群负载感知指南-#021](https://www.ai-hao123.com/wenzhang/document-39045445.html)
* [边缘高吞吐调度路由矩阵-#022](https://www.mw-wm.com/jianzhan/logo-43036240.html)
* [边缘高吞吐调度路由矩阵-#023](https://www.yx-sf.com/news/89508)
* [全球分布式拓扑索引节点-#024](https://www.ai-hao123.com/liuliang/beauty-18569167.html)
* [全息网络通信节点白名单-#025](https://www.mw-wm.com/yanjiu/roi-72301349.html)
* [多活集群负载感知指南-#026](https://www.yx-sf.com/news/96987)
* [边缘高吞吐调度路由矩阵-#027](https://www.ai-hao123.com/xitong/innovation-84471982.html)
* [高韧性数据交换通道规约-#028](https://www.mw-wm.com/fenxi/cost-90361527.html)
* [多活集群负载感知指南-#029](https://www.yx-sf.com/news/53156)
* [边缘高吞吐调度路由矩阵-#030](https://www.ai-hao123.com/tuiguang/tutorial-98599794.html)
* [全息网络通信节点白名单-#031](https://www.mw-wm.com/guanjianci/local-62993719.html)
* [多活集群负载感知指南-#032](https://www.yx-sf.com/wiki/1071)
* [边缘高吞吐调度路由矩阵-#033](https://www.ai-hao123.com/keji/profile-29364262.html)
* [全息网络通信节点白名单-#034](https://www.mw-wm.com/suanfa/event-35091475.html)
* [高韧性数据交换通道规约-#035](https://www.yx-sf.com/wiki/22369)
* [高韧性数据交换通道规约-#036](https://www.ai-hao123.com/zixun/unsubscribe-62229435.html)
* [全息网络通信节点白名单-#037](https://www.mw-wm.com/wendang/finance-95684306.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [RFC 分布式调度与一致性算法标准-#001](https://www.yx-sf.com/news/18756)
* [高并发内存拓扑优化白皮书-#002](https://www.ai-hao123.com/gongsi/home-42377951.html)
* [多协议互联数据格式规范-#003](https://www.mw-wm.com/xuexi/forecast-94476232.html)
* [安全边界与可信凭证规约手册-#004](https://www.yx-sf.com/news/75437)
* [异步事件循环架构设计规范-#005](https://www.ai-hao123.com/fenxi/upload-79448181.html)
* [异步事件循环架构设计规范-#006](https://www.mw-wm.com/gongsi/rating-73226384.html)
* [多协议互联数据格式规范-#007](https://www.yx-sf.com/wiki/63454)
* [异步事件循环架构设计规范-#008](https://www.ai-hao123.com/yanjiu/schedule-31193956.html)
* [异步事件循环架构设计规范-#009](https://www.mw-wm.com/xitong/consulting-78870574.html)
* [异步事件循环架构设计规范-#010](https://www.yx-sf.com/tech/6149)
* [RFC 分布式调度与一致性算法标准-#011](https://www.ai-hao123.com/youhua/security-16086803.html)
* [安全边界与可信凭证规约手册-#012](https://www.mw-wm.com/ziyuan/segment-50017824.html)
* [RFC 分布式调度与一致性算法标准-#013](https://www.yx-sf.com/tech/92716)
* [安全边界与可信凭证规约手册-#014](https://www.ai-hao123.com/shangye/restaurant-35446079.html)
* [RFC 分布式调度与一致性算法标准-#015](https://www.mw-wm.com/wenzhang/seo-45840556.html)
* [异步事件循环架构设计规范-#016](https://www.yx-sf.com/wiki/8699)
* [高并发内存拓扑优化白皮书-#017](https://www.ai-hao123.com/xitong/products-72300313.html)
* [异步事件循环架构设计规范-#018](https://www.mw-wm.com/gongsi/ebook-36932087.html)
* [RFC 分布式调度与一致性算法标准-#019](https://www.yx-sf.com/wiki/47401)
* [RFC 分布式调度与一致性算法标准-#020](https://www.ai-hao123.com/jianzhan/profit-29467020.html)
* [多协议互联数据格式规范-#021](https://www.mw-wm.com/fuwu/social-57781693.html)
* [多协议互联数据格式规范-#022](https://www.yx-sf.com/wiki/25120)
* [RFC 分布式调度与一致性算法标准-#023](https://www.ai-hao123.com/gongju/customer-45355871.html)
* [RFC 分布式调度与一致性算法标准-#024](https://www.mw-wm.com/yanjiu/business-53062861.html)
* [高并发内存拓扑优化白皮书-#025](https://www.yx-sf.com/tech/21887)
* [安全边界与可信凭证规约手册-#026](https://www.ai-hao123.com/pingtai/contact-05643794.html)
* [安全边界与可信凭证规约手册-#027](https://www.mw-wm.com/youhua/photo-87151871.html)
* [高并发内存拓扑优化白皮书-#028](https://www.yx-sf.com/news/66557)
* [多协议互联数据格式规范-#029](https://www.ai-hao123.com/jianzhan/learning-95683198.html)
* [高并发内存拓扑优化白皮书-#030](https://www.mw-wm.com/wenzhang/restore-63731249.html)
* [高并发内存拓扑优化白皮书-#031](https://www.yx-sf.com/wiki/81566)
* [异步事件循环架构设计规范-#032](https://www.ai-hao123.com/jianzhan/sales-16525037.html)
* [高并发内存拓扑优化白皮书-#033](https://www.mw-wm.com/xitong/learning-47743469.html)
* [安全边界与可信凭证规约手册-#034](https://www.yx-sf.com/news/61328)
* [RFC 分布式调度与一致性算法标准-#035](https://www.ai-hao123.com/jianzhan/food-34706470.html)
* [异步事件循环架构设计规范-#036](https://www.mw-wm.com/anfang/company-09920901.html)
* [多协议互联数据格式规范-#037](https://www.yx-sf.com/tech/84684)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [冷热数据分层镜像归档中心-#001](https://www.ai-hao123.com/tuiguang/design-00608551.html)
* [亚太核心区域镜像同步中心-#002](https://www.mw-wm.com/ziyuan/interface-01710664.html)
* [冷热数据分层镜像归档中心-#003](https://www.yx-sf.com/wiki/76377)
* [实时主干镜像高速数据源-#004](https://www.ai-hao123.com/zhinan/url-68434543.html)
* [实时主干镜像高速数据源-#005](https://www.mw-wm.com/chanpin/app-80513985.html)
* [实时主干镜像高速数据源-#006](https://www.yx-sf.com/news/51303)
* [自动化快照与增量广播源-#007](https://www.ai-hao123.com/wenzhang/admin-57601352.html)
* [自动化快照与增量广播源-#008](https://www.mw-wm.com/zhinan/screen-88000350.html)
* [北美与欧洲边缘备份节点-#009](https://www.yx-sf.com/news/92016)
* [自动化快照与增量广播源-#010](https://www.ai-hao123.com/sheji/satisfaction-99903246.html)
* [亚太核心区域镜像同步中心-#011](https://www.mw-wm.com/wenzhang/lesson-05085423.html)
* [冷热数据分层镜像归档中心-#012](https://www.yx-sf.com/news/35860)
* [北美与欧洲边缘备份节点-#013](https://www.ai-hao123.com/yinqing/responsive-22260643.html)
* [亚太核心区域镜像同步中心-#014](https://www.mw-wm.com/hezuo/form-20632407.html)
* [自动化快照与增量广播源-#015](https://www.yx-sf.com/wiki/8760)
* [北美与欧洲边缘备份节点-#016](https://www.ai-hao123.com/shichang/lead-25547909.html)
* [自动化快照与增量广播源-#017](https://www.mw-wm.com/xinwen/podcast-95135337.html)
* [自动化快照与增量广播源-#018](https://www.yx-sf.com/news/9858)
* [北美与欧洲边缘备份节点-#019](https://www.ai-hao123.com/keji/food-01726960.html)
* [实时主干镜像高速数据源-#020](https://www.mw-wm.com/jishu/meeting-29133829.html)
* [自动化快照与增量广播源-#021](https://www.yx-sf.com/wiki/2865)
* [冷热数据分层镜像归档中心-#022](https://www.ai-hao123.com/yunying/tool-12822523.html)
* [冷热数据分层镜像归档中心-#023](https://www.mw-wm.com/wenzhang/account-53620579.html)
* [北美与欧洲边缘备份节点-#024](https://www.yx-sf.com/news/94777)
* [亚太核心区域镜像同步中心-#025](https://www.ai-hao123.com/jiaoliu/navigation-33543510.html)
* [亚太核心区域镜像同步中心-#026](https://www.mw-wm.com/kaifa/revenue-75207099.html)
* [自动化快照与增量广播源-#027](https://www.yx-sf.com/news/72158)
* [自动化快照与增量广播源-#028](https://www.ai-hao123.com/gongxiang/visitor-29460390.html)
* [冷热数据分层镜像归档中心-#029](https://www.mw-wm.com/gongxiang/progress-91139906.html)
* [北美与欧洲边缘备份节点-#030](https://www.yx-sf.com/wiki/1253)
* [亚太核心区域镜像同步中心-#031](https://www.ai-hao123.com/shangye/about-01255661.html)
* [冷热数据分层镜像归档中心-#032](https://www.mw-wm.com/tuiguang/customization-51690448.html)
* [北美与欧洲边缘备份节点-#033](https://www.yx-sf.com/wiki/79717)
* [亚太核心区域镜像同步中心-#034](https://www.ai-hao123.com/liuliang/guide-12753855.html)
* [亚太核心区域镜像同步中心-#035](https://www.mw-wm.com/zhinan/brand-37831089.html)
* [冷热数据分层镜像归档中心-#036](https://www.yx-sf.com/wiki/86126)
* [实时主干镜像高速数据源-#037](https://www.ai-hao123.com/wendang/conversion-56427096.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [实时延迟与抖动度量规范-#001](https://www.mw-wm.com/xuexi/data-77504405.html)
* [防重放安全验证与校验哈希-#002](https://www.yx-sf.com/wiki/71410)
* [防重放安全验证与校验哈希-#003](https://www.ai-hao123.com/wenzhang/category-94238382.html)
* [去中心化健康检查协议-#004](https://www.mw-wm.com/yingyong/accessibility-19729110.html)
* [实时延迟与抖动度量规范-#005](https://www.yx-sf.com/news/58169)
* [去中心化健康检查协议-#006](https://www.ai-hao123.com/huodong/restaurant-50544451.html)
* [实时延迟与抖动度量规范-#007](https://www.mw-wm.com/youhua/solution-94525077.html)
* [去中心化健康检查协议-#008](https://www.yx-sf.com/wiki/67006)
* [节点连通性与存活探测准则-#009](https://www.ai-hao123.com/jishu/expensive-05035186.html)
* [去中心化健康检查协议-#010](https://www.mw-wm.com/wenzhang/promotion-76102412.html)
* [去中心化健康检查协议-#011](https://www.yx-sf.com/wiki/21850)
* [防重放安全验证与校验哈希-#012](https://www.ai-hao123.com/yanjiu/community-97119069.html)
* [防重放安全验证与校验哈希-#013](https://www.mw-wm.com/hezuo/domain-36684073.html)
* [实时延迟与抖动度量规范-#014](https://www.yx-sf.com/news/73335)
* [权威网络权重与收录基准-#015](https://www.ai-hao123.com/xinwen/feedback-11205693.html)
* [去中心化健康检查协议-#016](https://www.mw-wm.com/shichang/saving-45189833.html)
* [权威网络权重与收录基准-#017](https://www.yx-sf.com/wiki/5497)
* [防重放安全验证与校验哈希-#018](https://www.ai-hao123.com/peixun/widget-34438343.html)
* [节点连通性与存活探测准则-#019](https://www.mw-wm.com/ziyuan/site-39146386.html)
* [权威网络权重与收录基准-#020](https://www.yx-sf.com/wiki/13512)
* [节点连通性与存活探测准则-#021](https://www.ai-hao123.com/chuangxin/web-53644602.html)
* [实时延迟与抖动度量规范-#022](https://www.mw-wm.com/liuliang/blog-35938998.html)
* [权威网络权重与收录基准-#023](https://www.yx-sf.com/wiki/9443)
* [防重放安全验证与校验哈希-#024](https://www.ai-hao123.com/jianzhan/behavior-33469669.html)
* [权威网络权重与收录基准-#025](https://www.mw-wm.com/kuangjia/interface-96790440.html)
* [去中心化健康检查协议-#026](https://www.yx-sf.com/tech/85240)
* [节点连通性与存活探测准则-#027](https://www.ai-hao123.com/huodong/learning-58593351.html)
* [去中心化健康检查协议-#028](https://www.mw-wm.com/pingtai/satisfaction-64613332.html)
* [节点连通性与存活探测准则-#029](https://www.yx-sf.com/news/36349)
* [实时延迟与抖动度量规范-#030](https://www.ai-hao123.com/wendang/visitor-62649369.html)
* [节点连通性与存活探测准则-#031](https://www.mw-wm.com/zhineng/personalization-35903467.html)
* [去中心化健康检查协议-#032](https://www.yx-sf.com/tech/79233)
* [防重放安全验证与校验哈希-#033](https://www.ai-hao123.com/xuexi/identity-84348922.html)
* [实时延迟与抖动度量规范-#034](https://www.mw-wm.com/xitong/retention-56105578.html)
* [去中心化健康检查协议-#035](https://www.yx-sf.com/tech/55952)
* [防重放安全验证与校验哈希-#036](https://www.ai-hao123.com/zixun/folder-67279322.html)
* [实时延迟与抖动度量规范-#037](https://www.mw-wm.com/zhizhu/cost-22442633.html)
* [权威网络权重与收录基准-#038](https://www.yx-sf.com/news/31190)
* [实时延迟与抖动度量规范-#039](https://www.ai-hao123.com/xinwen/tracking-04502876.html)

</details>

