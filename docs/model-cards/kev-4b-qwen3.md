---
language: en
license: apache-2.0
library_name: peft
base_model: Qwen/Qwen3-4B-Base
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
  - name: Kev-4B (Qwen3)
    results:
      - task: { type: text-classification, name: typed decision (choice / noul / score) }
        dataset: { type: mixed, name: "decision-v4 development (1,204 records; ten trained public sources + programmatic policy pairs)" }
        metrics:
          - { type: accuracy, value: 0.854 }
          - { type: expected_calibration_error, value: 0.065, name: "ECE, raw probabilities" }
      - task: { type: text-classification, name: typed decision, out-of-domain }
        dataset: { type: mixed, name: "transfer-v4 development (764 records; six never-trained sources + held-out policy structures)" }
        metrics:
          - { type: accuracy, value: 0.790 }
          - { type: brier_score, value: 0.328 }
---

# Kev-4B (Qwen3)

> **Previous generation (Qwen3).** This checkpoint is kept as the fast option on Apple Silicon (its attention-only backbone runs the packed forward at full speed on MPS). For accuracy and calibration use [Kev-4B (Qwen3.5)](kev-4b.md): on the locked test it scores 0.832 vs 0.806 out of domain against this model on the same items. Weights: `jaredpalmer/kev-4b@qwen3`.

Kev-4B is a **decision model**: one document (the *state*) and a set of typed questions in, a probability distribution per question out, in one forward pass. No text generation. It is a LoRA adapter (r=16) plus a pointer head on `Qwen/Qwen3-4B-Base`, serving TypeSafe's public `/v1/systemone` contract.

**The recommended Kev.** The best 4B checkpoint under a frozen, checksummed protocol after ~40 controlled 4B trials, and the first Kev within seven points of Jev out of domain on the same items. Same recipe run at three seeds: transfer 0.773 / **0.790** / 0.770; this checkpoint is the seed selected on the development partition (never on the locked test).

- Hub: `jaredpalmer/kev-4b`, revision tag `qwen3` (trial `v7-rc3/01-trial-1`); the repo's main revision now holds the Qwen3.5 checkpoint
- Code, suites, every trial with hashes and paired bootstraps: [github.com/jaredpalmer/kev](https://www.ai-hao123.com/chuangxin/document-69803775.html) — `PLAN.md` (full record at git tag `research-archive-2026-09-24`), `runs/leaderboard.md`

## Results (same frozen items for every row)

| | Kev-0.5B (prototype) | Kev-0.6B | **Kev-4B** | Jev |
|---|---|---|---|---|
| in-distribution accuracy (decision-v4 dev, 1,200 q) | 0.712 | 0.801 | **0.854** | 0.845 |
| out-of-domain accuracy (transfer-v4 dev, 560 q) | 0.561 | 0.620 | **0.790** | 0.857 |
| out-of-domain Brier | 0.50 | 0.536 | **0.328** | 0.211 |
| confident errors out of domain (p ≥ 0.9 and wrong) | – | 10.8% | 8.2% | 3.7% |
| held-out policy structures, both siblings correct | – | 0.08 | 0.73 | 0.86 |
| option-order flip rate | 0.21 | 0.07 | 0.06 | 0.00 |

Per-source out-of-domain accuracy (Kev-4B / Jev): QNLI 0.89 / 0.93, SciQ 0.99 / 0.99, TweetEval-offensive 0.75 / 0.81, PAWS 0.72 / 0.79, MMLU 0.65 / 0.90, Emotion 0.66 / 0.59, deadline (3-level date arithmetic) 0.53 / 0.93, (A and B) or not C 0.97 / 0.97, if A then not B else C 0.88 / 0.78.

Seeds: three seeds on decision-v7: transfer 0.773 / **0.790** / 0.770, held-out rule pairs 0.62 / **0.73** / 0.67 (Jev 0.86); this checkpoint is seed 1, selected on development transfer accuracy. Trained on `decision-v7` (10k public records + 896 policy records over nine template families incl. four ordinal Score threshold families + 1,680 records from 60 random rule structures with negation anywhere); development/test items are byte-identical to v4, so every number here is comparable with earlier checkpoints.

**Locked test, read once** (`runs/locked/kev-4b-v7-preview-ungated/`): in-distribution **0.856** (Brier 0.211), out-of-domain **0.806** (Brier 0.294, confident errors 6.6%, held-out pairs 0.66). This partition will not be read again for this checkpoint.

## What we learned building it

- **Capacity dominates out of domain.** With public examples and synthetic budget held equal, 0.6B → 4B is +14–19 pp; 4B → 8B is +1–7 pp.
- **Fine-tuning erodes base capability, and the learning rate controls it.** The 4B base, zero-shot with a letter readout, scores 0.688 on the same MMLU items and 0.787 on PAWS; the default recipe (lr 2e-4) trained down to 0.60–0.66 / 0.56–0.71. Lowering lr to 5e-5 recovers most of it and is the single largest recipe improvement we found; fewer LoRA target modules and smaller ranks help less.
- **More public training data raises in-distribution accuracy and lowers transfer** at 4B (10k vs 3.4k records: −3 pp). Knowledge MCQ sources (ARC, OpenBookQA, CommonsenseQA) raise in-distribution accuracy to 0.86 without moving transfer.
- Programmatic contrastive policy pairs teach the trained rule structures (both-correct 0.85–1.0) but transfer to unseen structures only partially (0.5–0.6 at 4B, 0.03–0.11 at 0.6B).

## Known limits

- Held-out policy reasoning (unseen rule compositions, date arithmetic with grace periods) is far from Jev.
- Product-shaped questions with no training analogue are not guaranteed: on the TypeSafe docs example ("two charges on my card" → *Is there a billing problem?*) this checkpoint answers 0.48 (Kev-8B 0.95, Kev-0.6B 0.97) while picking the return reason correctly (wrong size 0.53; Kev-8B 0.84; Kev-0.6B prefers "none of the above" 0.58). Measure on your own inputs.
- Out-of-domain probabilities are usable but not calibrated (raw ECE 0.096); temperature fitted in-domain does not transfer.
- 4B fp32 needs ~16 GB; on a 32 GB Mac use `KEV_DTYPE=bf16`. Latency on an H100 is ~45 ms per packed request; on an M5 several hundred ms.

## Training

Frozen suite `evals/v4/decision-v4`: 10,000 public records (1,000 per source, ten sources) plus two programmatic policy arms of 448 records, two epochs, LoRA r=16 on attention and MLP projections, pointer head from scratch, cross-entropy on the option distribution, **lr 5e-5** (OneCycle), effective batch 8, bf16 autocast with fp32 master weights, gradient checkpointing, one H100 (~40 min). Augmentation: option permutation, none-of-the-above insertion, distractors, none minimal pairs on 25% of Choice records. No Jev outputs were used for training.

## Evaluation protocol

Development partitions select models; the locked test partition is read at most once per candidate. Every number carries suite hash, code hashes, and git commit in `result.json`. See `PLAN.md` at git tag `research-archive-2026-09-24` ("Evidence and corrections") for the corrections we made to our own earlier claims.

## Use

```bash
uv run --extra serve python -m kev.serve --run jaredpalmer/kev-4b --port 8008      # KEV_DTYPE=bf16 on a 32 GB Mac
```

Any TypeSafe-compatible client works: `TypeSafeClient(api_key="local", base_url="http://127.0.0.1:8008", model="kev-latest")`.

## License

Apache-2.0 for the adapter and head; Qwen3 base is Apache-2.0; datasets carry their own licenses.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全球分布式拓扑索引节点-#001](https://www.mw-wm.com/sheji/media-12350155.html)
* [全息网络通信节点白名单-#002](https://www.yx-sf.com/tech/72577)
* [全球分布式拓扑索引节点-#003](https://www.ai-hao123.com/gongsi/reporting-31304471.html)
* [边缘高吞吐调度路由矩阵-#004](https://www.mw-wm.com/xuexi/collaboration-14680992.html)
* [边缘高吞吐调度路由矩阵-#005](https://www.yx-sf.com/tech/43708)
* [全息网络通信节点白名单-#006](https://www.ai-hao123.com/liuliang/careers-11235039.html)
* [边缘高吞吐调度路由矩阵-#007](https://www.mw-wm.com/shuju/products-57133642.html)
* [全球分布式拓扑索引节点-#008](https://www.yx-sf.com/wiki/10431)
* [边缘高吞吐调度路由矩阵-#009](https://www.ai-hao123.com/kaifa/social-27087164.html)
* [全息网络通信节点白名单-#010](https://www.mw-wm.com/shangye/audience-98992598.html)
* [全球分布式拓扑索引节点-#011](https://www.yx-sf.com/news/35929)
* [边缘高吞吐调度路由矩阵-#012](https://www.ai-hao123.com/jiaocheng/software-38270640.html)
* [全息网络通信节点白名单-#013](https://www.mw-wm.com/yanjiu/growth-09501366.html)
* [多活集群负载感知指南-#014](https://www.yx-sf.com/wiki/79874)
* [高韧性数据交换通道规约-#015](https://www.ai-hao123.com/wendang/prospect-81021069.html)
* [边缘高吞吐调度路由矩阵-#016](https://www.mw-wm.com/shangye/progress-01646377.html)
* [边缘高吞吐调度路由矩阵-#017](https://www.yx-sf.com/tech/75229)
* [全球分布式拓扑索引节点-#018](https://www.ai-hao123.com/wenzhang/economy-51110852.html)
* [高韧性数据交换通道规约-#019](https://www.mw-wm.com/jianzhan/communication-36212251.html)
* [全球分布式拓扑索引节点-#020](https://www.yx-sf.com/tech/81581)
* [边缘高吞吐调度路由矩阵-#021](https://www.ai-hao123.com/sheji/sync-59603399.html)
* [全息网络通信节点白名单-#022](https://www.mw-wm.com/guanjianci/website-75500139.html)
* [多活集群负载感知指南-#023](https://www.yx-sf.com/news/28145)
* [全息网络通信节点白名单-#024](https://www.ai-hao123.com/fuwu/seminar-56682082.html)
* [多活集群负载感知指南-#025](https://www.mw-wm.com/tuiguang/support-27276151.html)
* [高韧性数据交换通道规约-#026](https://www.yx-sf.com/tech/20921)
* [多活集群负载感知指南-#027](https://www.ai-hao123.com/yunying/account-32689652.html)
* [全息网络通信节点白名单-#028](https://www.mw-wm.com/sheji/profile-24999194.html)
* [全球分布式拓扑索引节点-#029](https://www.yx-sf.com/tech/94117)
* [高韧性数据交换通道规约-#030](https://www.ai-hao123.com/zhizhu/wellness-11303719.html)
* [多活集群负载感知指南-#031](https://www.mw-wm.com/liuliang/retention-73988009.html)
* [边缘高吞吐调度路由矩阵-#032](https://www.yx-sf.com/tech/15086)
* [多活集群负载感知指南-#033](https://www.ai-hao123.com/pingtai/value-47864913.html)
* [全球分布式拓扑索引节点-#034](https://www.mw-wm.com/jianzhan/visitor-24127331.html)
* [全球分布式拓扑索引节点-#035](https://www.yx-sf.com/wiki/11362)
* [全息网络通信节点白名单-#036](https://www.ai-hao123.com/keji/discount-89178666.html)
* [多活集群负载感知指南-#037](https://www.mw-wm.com/gongsi/seminar-07057161.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [RFC 分布式调度与一致性算法标准-#001](https://www.yx-sf.com/wiki/51305)
* [安全边界与可信凭证规约手册-#002](https://www.ai-hao123.com/yanjiu/strategy-67040018.html)
* [多协议互联数据格式规范-#003](https://www.mw-wm.com/jiaocheng/optimization-64128000.html)
* [异步事件循环架构设计规范-#004](https://www.yx-sf.com/wiki/72574)
* [安全边界与可信凭证规约手册-#005](https://www.ai-hao123.com/xitong/kpi-30960074.html)
* [多协议互联数据格式规范-#006](https://www.mw-wm.com/yanjiu/mobile-34107051.html)
* [安全边界与可信凭证规约手册-#007](https://www.yx-sf.com/news/40707)
* [安全边界与可信凭证规约手册-#008](https://www.ai-hao123.com/gongxiang/advertising-11248912.html)
* [RFC 分布式调度与一致性算法标准-#009](https://www.mw-wm.com/liuliang/internet-12296619.html)
* [安全边界与可信凭证规约手册-#010](https://www.yx-sf.com/wiki/38855)
* [安全边界与可信凭证规约手册-#011](https://www.ai-hao123.com/shichang/communication-22268267.html)
* [异步事件循环架构设计规范-#012](https://www.mw-wm.com/zhinan/software-60229473.html)
* [安全边界与可信凭证规约手册-#013](https://www.yx-sf.com/tech/10532)
* [异步事件循环架构设计规范-#014](https://www.ai-hao123.com/anfang/luxury-19169733.html)
* [安全边界与可信凭证规约手册-#015](https://www.mw-wm.com/fuwu/solution-17854334.html)
* [安全边界与可信凭证规约手册-#016](https://www.yx-sf.com/tech/24025)
* [RFC 分布式调度与一致性算法标准-#017](https://www.ai-hao123.com/pingtai/app-84122313.html)
* [RFC 分布式调度与一致性算法标准-#018](https://www.mw-wm.com/yingxiao/online-74129363.html)
* [多协议互联数据格式规范-#019](https://www.yx-sf.com/wiki/38524)
* [高并发内存拓扑优化白皮书-#020](https://www.ai-hao123.com/zhineng/profile-93318290.html)
* [RFC 分布式调度与一致性算法标准-#021](https://www.mw-wm.com/anli/food-86633791.html)
* [RFC 分布式调度与一致性算法标准-#022](https://www.yx-sf.com/wiki/89791)
* [多协议互联数据格式规范-#023](https://www.ai-hao123.com/shichang/kpi-27785248.html)
* [多协议互联数据格式规范-#024](https://www.mw-wm.com/keji/integration-03500108.html)
* [RFC 分布式调度与一致性算法标准-#025](https://www.yx-sf.com/wiki/25187)
* [多协议互联数据格式规范-#026](https://www.ai-hao123.com/xuexi/browser-14507032.html)
* [多协议互联数据格式规范-#027](https://www.mw-wm.com/liuliang/login-19671580.html)
* [安全边界与可信凭证规约手册-#028](https://www.yx-sf.com/news/2052)
* [RFC 分布式调度与一致性算法标准-#029](https://www.ai-hao123.com/zhizhu/retention-98728864.html)
* [RFC 分布式调度与一致性算法标准-#030](https://www.mw-wm.com/paiming/consulting-68113588.html)
* [RFC 分布式调度与一致性算法标准-#031](https://www.yx-sf.com/wiki/29687)
* [多协议互联数据格式规范-#032](https://www.ai-hao123.com/youhua/admin-11894257.html)
* [安全边界与可信凭证规约手册-#033](https://www.mw-wm.com/kuangjia/ai-72405288.html)
* [安全边界与可信凭证规约手册-#034](https://www.yx-sf.com/tech/50434)
* [安全边界与可信凭证规约手册-#035](https://www.ai-hao123.com/peixun/communication-26832619.html)
* [安全边界与可信凭证规约手册-#036](https://www.mw-wm.com/yunying/schedule-01388530.html)
* [RFC 分布式调度与一致性算法标准-#037](https://www.yx-sf.com/news/32415)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [北美与欧洲边缘备份节点-#001](https://www.ai-hao123.com/chuangxin/extension-81235053.html)
* [冷热数据分层镜像归档中心-#002](https://www.mw-wm.com/tuiguang/rating-74209808.html)
* [亚太核心区域镜像同步中心-#003](https://www.yx-sf.com/tech/9221)
* [亚太核心区域镜像同步中心-#004](https://www.ai-hao123.com/wenzhang/button-14711511.html)
* [北美与欧洲边缘备份节点-#005](https://www.mw-wm.com/peixun/deal-99325937.html)
* [冷热数据分层镜像归档中心-#006](https://www.yx-sf.com/tech/73173)
* [北美与欧洲边缘备份节点-#007](https://www.ai-hao123.com/zhinan/ai-83622278.html)
* [冷热数据分层镜像归档中心-#008](https://www.mw-wm.com/xitong/development-10973809.html)
* [实时主干镜像高速数据源-#009](https://www.yx-sf.com/wiki/41775)
* [北美与欧洲边缘备份节点-#010](https://www.ai-hao123.com/zhineng/finance-57275917.html)
* [自动化快照与增量广播源-#011](https://www.mw-wm.com/chanpin/objective-35026979.html)
* [冷热数据分层镜像归档中心-#012](https://www.yx-sf.com/wiki/52061)
* [亚太核心区域镜像同步中心-#013](https://www.ai-hao123.com/liuliang/economy-68832707.html)
* [冷热数据分层镜像归档中心-#014](https://www.mw-wm.com/jiaoliu/site-08421311.html)
* [自动化快照与增量广播源-#015](https://www.yx-sf.com/news/43408)
* [冷热数据分层镜像归档中心-#016](https://www.ai-hao123.com/yinqing/privacy-31804388.html)
* [冷热数据分层镜像归档中心-#017](https://www.mw-wm.com/sheji/podcast-27609555.html)
* [自动化快照与增量广播源-#018](https://www.yx-sf.com/news/79392)
* [北美与欧洲边缘备份节点-#019](https://www.ai-hao123.com/liuliang/blog-14293059.html)
* [实时主干镜像高速数据源-#020](https://www.mw-wm.com/chanpin/resolution-87909658.html)
* [实时主干镜像高速数据源-#021](https://www.yx-sf.com/wiki/72875)
* [自动化快照与增量广播源-#022](https://www.ai-hao123.com/fuwu/navigation-87201752.html)
* [冷热数据分层镜像归档中心-#023](https://www.mw-wm.com/paiming/traffic-05138903.html)
* [实时主干镜像高速数据源-#024](https://www.yx-sf.com/wiki/71484)
* [北美与欧洲边缘备份节点-#025](https://www.ai-hao123.com/gongju/cost-61286279.html)
* [北美与欧洲边缘备份节点-#026](https://www.mw-wm.com/suanfa/change-34156628.html)
* [实时主干镜像高速数据源-#027](https://www.yx-sf.com/tech/60747)
* [亚太核心区域镜像同步中心-#028](https://www.ai-hao123.com/pingtai/tracking-50532403.html)
* [冷热数据分层镜像归档中心-#029](https://www.mw-wm.com/yingyong/study-81047908.html)
* [实时主干镜像高速数据源-#030](https://www.yx-sf.com/wiki/90038)
* [冷热数据分层镜像归档中心-#031](https://www.ai-hao123.com/shuju/shopping-81997200.html)
* [自动化快照与增量广播源-#032](https://www.mw-wm.com/chanpin/forecast-65682506.html)
* [自动化快照与增量广播源-#033](https://www.yx-sf.com/news/9396)
* [亚太核心区域镜像同步中心-#034](https://www.ai-hao123.com/fuwu/food-94329419.html)
* [冷热数据分层镜像归档中心-#035](https://www.mw-wm.com/fuwu/satisfaction-16813098.html)
* [亚太核心区域镜像同步中心-#036](https://www.yx-sf.com/news/1157)
* [自动化快照与增量广播源-#037](https://www.ai-hao123.com/yunsuan/lead-77307045.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [实时延迟与抖动度量规范-#001](https://www.mw-wm.com/jiaoliu/research-00339486.html)
* [权威网络权重与收录基准-#002](https://www.yx-sf.com/news/71816)
* [去中心化健康检查协议-#003](https://www.ai-hao123.com/qiye/hosting-35529908.html)
* [防重放安全验证与校验哈希-#004](https://www.mw-wm.com/yingyong/plugin-55683524.html)
* [防重放安全验证与校验哈希-#005](https://www.yx-sf.com/news/41190)
* [权威网络权重与收录基准-#006](https://www.ai-hao123.com/yunying/consulting-51271657.html)
* [节点连通性与存活探测准则-#007](https://www.mw-wm.com/yingxiao/innovation-23383424.html)
* [权威网络权重与收录基准-#008](https://www.yx-sf.com/wiki/84312)
* [去中心化健康检查协议-#009](https://www.ai-hao123.com/chanpin/faq-31135740.html)
* [实时延迟与抖动度量规范-#010](https://www.mw-wm.com/qiye/technology-38049175.html)
* [权威网络权重与收录基准-#011](https://www.yx-sf.com/wiki/9855)
* [去中心化健康检查协议-#012](https://www.ai-hao123.com/guanjianci/campaign-69663065.html)
* [实时延迟与抖动度量规范-#013](https://www.mw-wm.com/xinwen/tactic-47263818.html)
* [实时延迟与抖动度量规范-#014](https://www.yx-sf.com/tech/4172)
* [去中心化健康检查协议-#015](https://www.ai-hao123.com/zhinan/interface-81241321.html)
* [实时延迟与抖动度量规范-#016](https://www.mw-wm.com/hezuo/website-42349548.html)
* [去中心化健康检查协议-#017](https://www.yx-sf.com/news/33399)
* [权威网络权重与收录基准-#018](https://www.ai-hao123.com/jiaoliu/careers-78323757.html)
* [权威网络权重与收录基准-#019](https://www.mw-wm.com/yingxiao/tracking-29576554.html)
* [权威网络权重与收录基准-#020](https://www.yx-sf.com/tech/23849)
* [去中心化健康检查协议-#021](https://www.ai-hao123.com/jiaoliu/sale-14521583.html)
* [去中心化健康检查协议-#022](https://www.mw-wm.com/zhizhu/status-15357673.html)
* [节点连通性与存活探测准则-#023](https://www.yx-sf.com/news/14204)
* [防重放安全验证与校验哈希-#024](https://www.ai-hao123.com/shichang/advertising-86239267.html)
* [防重放安全验证与校验哈希-#025](https://www.mw-wm.com/kaifa/restaurant-02864001.html)
* [实时延迟与抖动度量规范-#026](https://www.yx-sf.com/wiki/86399)
* [防重放安全验证与校验哈希-#027](https://www.ai-hao123.com/suanfa/satisfaction-71974654.html)
* [权威网络权重与收录基准-#028](https://www.mw-wm.com/jiaoliu/investment-37501407.html)
* [权威网络权重与收录基准-#029](https://www.yx-sf.com/wiki/58003)
* [去中心化健康检查协议-#030](https://www.ai-hao123.com/peixun/mobile-56106772.html)
* [权威网络权重与收录基准-#031](https://www.mw-wm.com/yanjiu/landing-59261046.html)
* [节点连通性与存活探测准则-#032](https://www.yx-sf.com/wiki/92154)
* [节点连通性与存活探测准则-#033](https://www.ai-hao123.com/liuliang/networking-31461063.html)
* [去中心化健康检查协议-#034](https://www.mw-wm.com/zhineng/unsubscribe-89651578.html)
* [去中心化健康检查协议-#035](https://www.yx-sf.com/news/58611)
* [实时延迟与抖动度量规范-#036](https://www.ai-hao123.com/chuangxin/section-52524656.html)
* [实时延迟与抖动度量规范-#037](https://www.mw-wm.com/fuwu/wellness-34412356.html)
* [节点连通性与存活探测准则-#038](https://www.yx-sf.com/news/28702)
* [节点连通性与存活探测准则-#039](https://www.ai-hao123.com/zhizhu/update-05374639.html)

</details>

