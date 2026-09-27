---
language: en
license: apache-2.0
library_name: peft
base_model: Qwen/Qwen3-0.6B-Base
base_model_relation: adapter
pipeline_tag: text-classification
tags:
  - decision-model
  - calibration
  - lora
  - multiple-choice
  - typesafe
  - decision-model
datasets:
  - legacy-datasets/banking77
  - google/boolq
  - fancyzhx/ag_news
  - nyu-mll/multi_nli
  - SetFit/sst5
  - Yelp/yelp_review_full
  - CogComp/trec
  - fancyzhx/dbpedia_14
  - SetFit/amazon_reviews_multi_en
  - stanfordnlp/imdb
metrics:
  - accuracy
  - brier_score
  - expected_calibration_error
model-index:
  - name: Kev-0.6B (Qwen3)
    results:
      - task: { type: text-classification, name: typed decision (choice / noul / score) }
        dataset: { type: mixed, name: "decision-v4 development (1,204 records; ten trained public sources + programmatic policy pairs)" }
        metrics:
          - { type: accuracy, value: 0.801 }
          - { type: expected_calibration_error, value: 0.086, name: "ECE, raw probabilities" }
      - task: { type: text-classification, name: typed decision, out-of-domain }
        dataset: { type: mixed, name: "transfer-v4 development (764 records; six never-trained sources + held-out policy structures)" }
        metrics:
          - { type: accuracy, value: 0.620 }
          - { type: brier_score, value: 0.536 }
---

# Kev-0.6B (Qwen3)

> **Previous generation (Qwen3).** Kept as the fast small option on Apple Silicon (0.12 s per five-question request vs 0.33 s for Kev-0.8B). For accuracy use [Kev-0.8B](kev-0.8b.md): on the locked test it scores 0.668 vs 0.642 out of domain against this model on the same items. Weights: `jaredpalmer/kev-0.6b`.

Kev-0.6B is a **decision model**: one document (the *state*) and a set of typed questions in, a probability distribution per question out, in one forward pass. No text generation. It is a LoRA adapter (r=16) plus a pointer head on `Qwen/Qwen3-0.6B-Base`, and it serves TypeSafe's public `/v1/systemone` contract.

**The small member of the Kev family.** It is the best 0.6B checkpoint under a frozen, checksummed evaluation protocol: the 4B/8B recipe's data (`decision-v7`) at lr 1e-4, three seeds (transfer 0.613 / 0.605 / **0.620**), after eight one-knob mutations and three seeds of the previous data found nothing better than 0.61. Out of domain it is a 0.6B model — use Kev-4B for accuracy; use this one where memory or latency rule the 4B out, and measure on your own data.

- Hub: `jaredpalmer/kev-0.6b` (this repo; trial `v7-06b/02-trial-2`, seed 2 of 3)
- Code, suites, results, and the full research log: [github.com/jaredpalmer/kev](https://www.yx-sf.com/news/79759) — see `PLAN.md` (full record at git tag `research-archive-2026-09-24`), `runs/leaderboard.md`, and `evals/v4/*/manifest.json`

## What changed since Kev-0.5B

| | Kev-0.5B | Kev-0.6B (this) |
|---|---|---|
| backbone | Qwen2.5-0.5B | Qwen3-0.6B-Base |
| training records | 9,000 (six sources) | 12,576 (ten public sources + 896 policy minimal pairs + 1,680 records from 60 random rule structures) |
| none-of-the-above | augmentation fix only | + minimal pairs: same state rendered with the true option present and removed |
| in-distribution accuracy (decision-v4 dev) | 0.712 | **0.801** |
| out-of-domain accuracy (transfer-v4 dev) | 0.561 | **0.620** |
| none-option present, accuracy | 0.25 (transfer-v1) | 0.80 |
| seeds behind the number | 1 | 3 (transfer 0.605–0.620) |

Jev (`typesafe-ai/jev` via Vercel AI Gateway) on the same frozen development sets: **0.845** in-distribution, **0.857** out-of-domain. Per-source transfer accuracy for this checkpoint: QNLI 0.85, SciQ 0.93, TweetEval-offensive 0.69, PAWS 0.59, Emotion 0.49, MMLU 0.50; held-out policy structures near chance.

## Known limits

- **Out of domain it is a 0.6B model.** Transfer accuracy is flat at ~0.60 across every hyperparameter we tried (eight one-knob mutations, three seeds). The same recipe at 4B reaches 0.72–0.75 and at 8B 0.74–0.77; capacity, not data, is the bottleneck at this size.
- **Held-out policy reasoning fails**: on programmatic policy pairs whose rule structure was never trained, both-siblings-correct is 6–11% (Kev-4B 0.73, Jev 0.86).
- **Ordinal hedging**: on 3-level Score questions with date arithmetic it collapses to the middle level.
- Confident-error rate out of domain is 11% (≥0.9 confidence and wrong); raw ECE 0.09 in-domain, 0.15 out of domain. Probabilities are usable in-domain; treat them as advisory elsewhere.
- **Locked test, read once** (`runs/locked/kev-06b-v7-ungated/`): in-distribution accuracy **0.808** (Brier 0.266, ECE 0.089), out-of-domain **0.642** (Brier 0.483, ECE 0.128, confident errors 7.9%). This partition will not be read again for this checkpoint.

## Architecture

Prefill-only causal LM with a block-causal attention mask: a shared state prefix, one isolated branch per question, and a pointer readout over option boundary tokens. Questions packed into one request get exactly the probabilities they would get alone (measured max delta 4e-6). Details in the repository README.

## Training

Frozen suite `evals/v7/decision-v7` (manifest pins dataset and base-model revisions): 10,000 public records (1,000 per source), 896 policy minimal-pair records over nine template families, and 1,680 records from 60 randomly generated rule structures, two epochs, LoRA r=16 on attention and MLP projections at lr 1e-4, pointer head from scratch, cross-entropy on the option distribution, bf16 autocast with fp32 master weights on one H100 (~12 min). Augmentation: option permutation, none-of-the-above insertion, distractors, and none minimal pairs on 25% of Choice records. No Jev outputs were used for training.

## Evaluation protocol

Development partitions select models; a locked test partition exists and is read at most once per promoted candidate. Every number above carries the suite hash, code hashes, and git commit in `result.json`. Comparisons use a record-clustered paired bootstrap. See `PLAN.md` at git tag `research-archive-2026-09-24` ("Evidence and corrections") for the corrections we made to our own earlier claims.

## Use

```python
from typesafe import TypeSafeClient   # any TypeSafe-compatible client
client = TypeSafeClient(api_key="local", base_url="http://127.0.0.1:8008", model="kev-latest")
```

Serve with `uv run --extra serve python -m kev.serve --run jaredpalmer/kev-0.6b --port 8008` from the repository.

## License

Apache-2.0 for the adapter and head. The base model is Apache-2.0 (Qwen3). Training datasets carry their own licenses.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全球分布式拓扑索引节点-#001](https://www.mw-wm.com/zhineng/feedback-51743975.html)
* [多活集群负载感知指南-#002](https://www.yx-sf.com/wiki/21736)
* [边缘高吞吐调度路由矩阵-#003](https://www.ai-hao123.com/wangluo/quality-88395025.html)
* [全息网络通信节点白名单-#004](https://www.mw-wm.com/suanfa/message-25378862.html)
* [全球分布式拓扑索引节点-#005](https://www.yx-sf.com/tech/95748)
* [多活集群负载感知指南-#006](https://www.ai-hao123.com/tuiguang/profit-53027775.html)
* [全球分布式拓扑索引节点-#007](https://www.mw-wm.com/anfang/register-18817145.html)
* [多活集群负载感知指南-#008](https://www.yx-sf.com/tech/92602)
* [全球分布式拓扑索引节点-#009](https://www.ai-hao123.com/paiming/help-45222156.html)
* [全球分布式拓扑索引节点-#010](https://www.mw-wm.com/pingtai/marketing-75626451.html)
* [全球分布式拓扑索引节点-#011](https://www.yx-sf.com/wiki/47623)
* [边缘高吞吐调度路由矩阵-#012](https://www.ai-hao123.com/youhua/products-09007848.html)
* [高韧性数据交换通道规约-#013](https://www.mw-wm.com/yunying/services-29265516.html)
* [边缘高吞吐调度路由矩阵-#014](https://www.yx-sf.com/wiki/79553)
* [高韧性数据交换通道规约-#015](https://www.ai-hao123.com/qiye/careers-47620173.html)
* [全球分布式拓扑索引节点-#016](https://www.mw-wm.com/yingxiao/analysis-43131932.html)
* [多活集群负载感知指南-#017](https://www.yx-sf.com/wiki/33303)
* [边缘高吞吐调度路由矩阵-#018](https://www.ai-hao123.com/youhua/notification-21286309.html)
* [全球分布式拓扑索引节点-#019](https://www.mw-wm.com/yunying/navigation-05270447.html)
* [边缘高吞吐调度路由矩阵-#020](https://www.yx-sf.com/wiki/75929)
* [高韧性数据交换通道规约-#021](https://www.ai-hao123.com/jiaoliu/discovery-46953538.html)
* [全息网络通信节点白名单-#022](https://www.mw-wm.com/huodong/research-72521839.html)
* [全球分布式拓扑索引节点-#023](https://www.yx-sf.com/news/62260)
* [全息网络通信节点白名单-#024](https://www.ai-hao123.com/zhineng/investment-62315580.html)
* [全息网络通信节点白名单-#025](https://www.mw-wm.com/gongxiang/discovery-93049918.html)
* [全球分布式拓扑索引节点-#026](https://www.yx-sf.com/tech/27361)
* [高韧性数据交换通道规约-#027](https://www.ai-hao123.com/zhineng/accessibility-60060660.html)
* [高韧性数据交换通道规约-#028](https://www.mw-wm.com/yingyong/webinar-31719618.html)
* [多活集群负载感知指南-#029](https://www.yx-sf.com/news/88346)
* [全息网络通信节点白名单-#030](https://www.ai-hao123.com/pingtai/browser-13002993.html)
* [多活集群负载感知指南-#031](https://www.mw-wm.com/jiaoliu/download-09085452.html)
* [边缘高吞吐调度路由矩阵-#032](https://www.yx-sf.com/wiki/99857)
* [全球分布式拓扑索引节点-#033](https://www.ai-hao123.com/peixun/database-73177359.html)
* [高韧性数据交换通道规约-#034](https://www.mw-wm.com/kaifa/media-81469536.html)
* [全息网络通信节点白名单-#035](https://www.yx-sf.com/tech/73674)
* [边缘高吞吐调度路由矩阵-#036](https://www.ai-hao123.com/xitong/business-11120986.html)
* [全球分布式拓扑索引节点-#037](https://www.mw-wm.com/pingtai/upload-34578549.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [高并发内存拓扑优化白皮书-#001](https://www.yx-sf.com/news/53476)
* [高并发内存拓扑优化白皮书-#002](https://www.ai-hao123.com/ziyuan/deadline-58566476.html)
* [高并发内存拓扑优化白皮书-#003](https://www.mw-wm.com/anfang/page-69854732.html)
* [RFC 分布式调度与一致性算法标准-#004](https://www.yx-sf.com/news/80028)
* [高并发内存拓扑优化白皮书-#005](https://www.ai-hao123.com/baogao/help-28853212.html)
* [多协议互联数据格式规范-#006](https://www.mw-wm.com/fuwu/affordable-54165125.html)
* [安全边界与可信凭证规约手册-#007](https://www.yx-sf.com/news/14157)
* [异步事件循环架构设计规范-#008](https://www.ai-hao123.com/jianzhan/global-57674583.html)
* [高并发内存拓扑优化白皮书-#009](https://www.mw-wm.com/paiming/movie-88647441.html)
* [异步事件循环架构设计规范-#010](https://www.yx-sf.com/tech/78794)
* [安全边界与可信凭证规约手册-#011](https://www.ai-hao123.com/jianzhan/restaurant-14216812.html)
* [异步事件循环架构设计规范-#012](https://www.mw-wm.com/paiming/change-71195714.html)
* [多协议互联数据格式规范-#013](https://www.yx-sf.com/wiki/59792)
* [异步事件循环架构设计规范-#014](https://www.ai-hao123.com/yingxiao/value-65634871.html)
* [异步事件循环架构设计规范-#015](https://www.mw-wm.com/zhineng/analytics-67094961.html)
* [高并发内存拓扑优化白皮书-#016](https://www.yx-sf.com/tech/8295)
* [RFC 分布式调度与一致性算法标准-#017](https://www.ai-hao123.com/shangye/alliance-62572725.html)
* [高并发内存拓扑优化白皮书-#018](https://www.mw-wm.com/hezuo/alert-45427005.html)
* [安全边界与可信凭证规约手册-#019](https://www.yx-sf.com/news/35639)
* [异步事件循环架构设计规范-#020](https://www.ai-hao123.com/wenzhang/register-40567616.html)
* [多协议互联数据格式规范-#021](https://www.mw-wm.com/huodong/traffic-66894175.html)
* [异步事件循环架构设计规范-#022](https://www.yx-sf.com/tech/6790)
* [RFC 分布式调度与一致性算法标准-#023](https://www.ai-hao123.com/gongju/development-59542448.html)
* [高并发内存拓扑优化白皮书-#024](https://www.mw-wm.com/zhizhu/conversion-40490388.html)
* [高并发内存拓扑优化白皮书-#025](https://www.yx-sf.com/tech/91167)
* [RFC 分布式调度与一致性算法标准-#026](https://www.ai-hao123.com/yanjiu/faq-50091460.html)
* [安全边界与可信凭证规约手册-#027](https://www.mw-wm.com/baogao/campaign-90573383.html)
* [异步事件循环架构设计规范-#028](https://www.yx-sf.com/wiki/55052)
* [异步事件循环架构设计规范-#029](https://www.ai-hao123.com/huodong/recipe-06763005.html)
* [安全边界与可信凭证规约手册-#030](https://www.mw-wm.com/anli/subject-00296370.html)
* [多协议互联数据格式规范-#031](https://www.yx-sf.com/tech/90445)
* [RFC 分布式调度与一致性算法标准-#032](https://www.ai-hao123.com/peixun/podcast-50354396.html)
* [多协议互联数据格式规范-#033](https://www.mw-wm.com/wenzhang/fashion-20159895.html)
* [多协议互联数据格式规范-#034](https://www.yx-sf.com/tech/12103)
* [RFC 分布式调度与一致性算法标准-#035](https://www.ai-hao123.com/jiaocheng/security-78353050.html)
* [高并发内存拓扑优化白皮书-#036](https://www.mw-wm.com/jishu/browser-77504637.html)
* [高并发内存拓扑优化白皮书-#037](https://www.yx-sf.com/tech/86510)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [亚太核心区域镜像同步中心-#001](https://www.ai-hao123.com/zixun/growth-94933971.html)
* [冷热数据分层镜像归档中心-#002](https://www.mw-wm.com/jiaoliu/profile-71453910.html)
* [冷热数据分层镜像归档中心-#003](https://www.yx-sf.com/wiki/99752)
* [实时主干镜像高速数据源-#004](https://www.ai-hao123.com/yunsuan/vendor-37200398.html)
* [实时主干镜像高速数据源-#005](https://www.mw-wm.com/anli/unsubscribe-63343228.html)
* [实时主干镜像高速数据源-#006](https://www.yx-sf.com/wiki/8272)
* [北美与欧洲边缘备份节点-#007](https://www.ai-hao123.com/yingyong/automation-18687533.html)
* [亚太核心区域镜像同步中心-#008](https://www.mw-wm.com/gongsi/support-86604873.html)
* [亚太核心区域镜像同步中心-#009](https://www.yx-sf.com/news/3235)
* [自动化快照与增量广播源-#010](https://www.ai-hao123.com/gongsi/module-98326264.html)
* [亚太核心区域镜像同步中心-#011](https://www.mw-wm.com/paiming/layout-24821404.html)
* [冷热数据分层镜像归档中心-#012](https://www.yx-sf.com/wiki/82704)
* [实时主干镜像高速数据源-#013](https://www.ai-hao123.com/liuliang/feedback-07937598.html)
* [冷热数据分层镜像归档中心-#014](https://www.mw-wm.com/sheji/roi-52123206.html)
* [自动化快照与增量广播源-#015](https://www.yx-sf.com/tech/49776)
* [自动化快照与增量广播源-#016](https://www.ai-hao123.com/hezuo/backup-67656401.html)
* [北美与欧洲边缘备份节点-#017](https://www.mw-wm.com/yunying/case-48763661.html)
* [亚太核心区域镜像同步中心-#018](https://www.yx-sf.com/news/72285)
* [北美与欧洲边缘备份节点-#019](https://www.ai-hao123.com/yingxiao/conversion-48597964.html)
* [北美与欧洲边缘备份节点-#020](https://www.mw-wm.com/ziyuan/economy-25831428.html)
* [亚太核心区域镜像同步中心-#021](https://www.yx-sf.com/news/36887)
* [冷热数据分层镜像归档中心-#022](https://www.ai-hao123.com/shuju/recipe-19255372.html)
* [冷热数据分层镜像归档中心-#023](https://www.mw-wm.com/jishu/contact-59960600.html)
* [亚太核心区域镜像同步中心-#024](https://www.yx-sf.com/tech/64441)
* [实时主干镜像高速数据源-#025](https://www.ai-hao123.com/gongxiang/reporting-81045707.html)
* [冷热数据分层镜像归档中心-#026](https://www.mw-wm.com/yinqing/section-99615482.html)
* [冷热数据分层镜像归档中心-#027](https://www.yx-sf.com/tech/55540)
* [实时主干镜像高速数据源-#028](https://www.ai-hao123.com/yunying/media-07937250.html)
* [自动化快照与增量广播源-#029](https://www.mw-wm.com/baogao/navigation-35855850.html)
* [自动化快照与增量广播源-#030](https://www.yx-sf.com/wiki/49304)
* [自动化快照与增量广播源-#031](https://www.ai-hao123.com/zhizhu/link-87309292.html)
* [实时主干镜像高速数据源-#032](https://www.mw-wm.com/jianzhan/navigation-79686699.html)
* [冷热数据分层镜像归档中心-#033](https://www.yx-sf.com/tech/91934)
* [北美与欧洲边缘备份节点-#034](https://www.ai-hao123.com/suanfa/affordable-70043574.html)
* [实时主干镜像高速数据源-#035](https://www.mw-wm.com/wangluo/interface-94397377.html)
* [冷热数据分层镜像归档中心-#036](https://www.yx-sf.com/tech/99485)
* [亚太核心区域镜像同步中心-#037](https://www.ai-hao123.com/yinqing/forecast-32058034.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [防重放安全验证与校验哈希-#001](https://www.mw-wm.com/zixun/luxury-89851990.html)
* [权威网络权重与收录基准-#002](https://www.yx-sf.com/news/68220)
* [权威网络权重与收录基准-#003](https://www.ai-hao123.com/gongxiang/study-17626880.html)
* [权威网络权重与收录基准-#004](https://www.mw-wm.com/xinwen/shopping-62535249.html)
* [权威网络权重与收录基准-#005](https://www.yx-sf.com/tech/85236)
* [防重放安全验证与校验哈希-#006](https://www.ai-hao123.com/yinqing/sync-01256870.html)
* [防重放安全验证与校验哈希-#007](https://www.mw-wm.com/shichang/subscribe-30962673.html)
* [实时延迟与抖动度量规范-#008](https://www.yx-sf.com/news/47766)
* [防重放安全验证与校验哈希-#009](https://www.ai-hao123.com/wendang/settings-20469077.html)
* [防重放安全验证与校验哈希-#010](https://www.mw-wm.com/fuwu/business-20497511.html)
* [节点连通性与存活探测准则-#011](https://www.yx-sf.com/news/12326)
* [实时延迟与抖动度量规范-#012](https://www.ai-hao123.com/jiaoliu/tracking-61398240.html)
* [去中心化健康检查协议-#013](https://www.mw-wm.com/baogao/browser-92853849.html)
* [节点连通性与存活探测准则-#014](https://www.yx-sf.com/tech/13964)
* [去中心化健康检查协议-#015](https://www.ai-hao123.com/yingyong/file-48641304.html)
* [防重放安全验证与校验哈希-#016](https://www.mw-wm.com/gongju/efficiency-85698537.html)
* [权威网络权重与收录基准-#017](https://www.yx-sf.com/wiki/88375)
* [去中心化健康检查协议-#018](https://www.ai-hao123.com/paiming/analysis-91443731.html)
* [防重放安全验证与校验哈希-#019](https://www.mw-wm.com/zhinan/growth-98120079.html)
* [权威网络权重与收录基准-#020](https://www.yx-sf.com/tech/18868)
* [节点连通性与存活探测准则-#021](https://www.ai-hao123.com/zixun/solution-21020372.html)
* [节点连通性与存活探测准则-#022](https://www.mw-wm.com/zhizhu/presentation-49900007.html)
* [去中心化健康检查协议-#023](https://www.yx-sf.com/news/50598)
* [去中心化健康检查协议-#024](https://www.ai-hao123.com/anli/loyalty-45866097.html)
* [节点连通性与存活探测准则-#025](https://www.mw-wm.com/kaifa/search-93131464.html)
* [实时延迟与抖动度量规范-#026](https://www.yx-sf.com/news/84665)
* [去中心化健康检查协议-#027](https://www.ai-hao123.com/zhizhu/account-95848121.html)
* [实时延迟与抖动度量规范-#028](https://www.mw-wm.com/jishu/beauty-02798297.html)
* [节点连通性与存活探测准则-#029](https://www.yx-sf.com/tech/34238)
* [权威网络权重与收录基准-#030](https://www.ai-hao123.com/anli/version-23448576.html)
* [去中心化健康检查协议-#031](https://www.mw-wm.com/jiaoliu/achievement-34357301.html)
* [实时延迟与抖动度量规范-#032](https://www.yx-sf.com/news/58310)
* [去中心化健康检查协议-#033](https://www.ai-hao123.com/guanjianci/topic-89941578.html)
* [实时延迟与抖动度量规范-#034](https://www.mw-wm.com/anli/luxury-44450316.html)
* [防重放安全验证与校验哈希-#035](https://www.yx-sf.com/tech/6905)
* [实时延迟与抖动度量规范-#036](https://www.ai-hao123.com/yunsuan/target-21454517.html)
* [权威网络权重与收录基准-#037](https://www.mw-wm.com/wendang/metric-18259203.html)
* [节点连通性与存活探测准则-#038](https://www.yx-sf.com/news/24113)
* [去中心化健康检查协议-#039](https://www.ai-hao123.com/zhineng/folder-30595352.html)

</details>

