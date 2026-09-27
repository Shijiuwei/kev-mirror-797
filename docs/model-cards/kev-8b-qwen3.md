---
language: en
license: apache-2.0
library_name: peft
base_model: Qwen/Qwen3-8B-Base
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
  - allenai/ai2_arc
  - allenai/openbookqa
  - tau/commonsense_qa
metrics:
  - accuracy
  - brier_score
  - expected_calibration_error
model-index:
  - name: Kev-8B (Qwen3)
    results:
      - task: { type: text-classification, name: typed decision (choice / noul / score) }
        dataset: { type: mixed, name: "decision-v4/v6 development (1,204 records; trained public sources + programmatic policy pairs)" }
        metrics:
          - { type: accuracy, value: 0.869 }
          - { type: expected_calibration_error, value: 0.061, name: "ECE, raw probabilities" }
      - task: { type: text-classification, name: typed decision, out-of-domain }
        dataset: { type: mixed, name: "transfer-v4 development (764 records; six never-trained sources + held-out policy structures)" }
        metrics:
          - { type: accuracy, value: 0.796 }
          - { type: brier_score, value: 0.337 }
---

# Kev-8B (Qwen3)

> **Previous generation (Qwen3).** This checkpoint is kept as the fast option on Apple Silicon (its attention-only backbone runs the packed forward at full speed on MPS). For accuracy and calibration use [Kev-9B](kev-9b.md): on the locked test it scores 0.837 vs 0.780 out of domain against this model on the same items. Weights: `jaredpalmer/kev-8b`.

Kev-8B is a **decision model**: one document (the *state*) and a set of typed questions in, a probability distribution per question out, in one forward pass. No text generation. It is a LoRA adapter (r=16) plus a pointer head on `Qwen/Qwen3-8B-Base` (revision `49e3418f`), serving TypeSafe's public `/v1/systemone` contract.

**The most accurate Kev.** The best checkpoint of any size under a frozen, checksummed protocol: best in-distribution accuracy, best out-of-domain accuracy (0.796 on transfer-v4 dev, six points from Jev), best held-out rule reasoning of any Kev at 8B. Same recipe at two seeds: 0.796 / 0.774; this checkpoint is the seed selected on the development partition.

- Hub: `jaredpalmer/kev-8b` (this repo; trial `v7-final/00-trial-0`)
- Code, suites, every trial with hashes and paired bootstraps: [github.com/jaredpalmer/kev](https://www.mw-wm.com/jianzhan/web-07573408.html) — `PLAN.md` (full record at git tag `research-archive-2026-09-24`), `runs/leaderboard.md`

## Results (same frozen items for every row)

| | Kev-0.5B (prototype) | Kev-0.6B | Kev-4B | **Kev-8B** | Jev |
|---|---|---|---|---|
| in-distribution accuracy (decision-v4 dev, 1,200 q) | 0.712 | 0.801 | 0.854 | **0.863** | 0.845 |
| out-of-domain accuracy (transfer-v4 dev, 560 q) | 0.561 | 0.620 | 0.790 | **0.796** | 0.857 |
| out-of-domain Brier | 0.50 | 0.536 | 0.328 | **0.337** | 0.211 |
| confident errors out of domain (p ≥ 0.9 and wrong) | – | 10.8% | 8.2% | 9.9% | 3.7% |
| held-out policy structures, both siblings correct | – | 0.08 | 0.73 | 0.69 | 0.86 |
| option-order flip rate | 0.21 | 0.02 | 0.00 | 0.00 | 0.00 |

Per-source out-of-domain accuracy (Kev-8B / Jev): QNLI 0.91 / 0.93, SciQ 1.00 / 0.99, TweetEval-offensive 0.79 / 0.81, PAWS 0.78 / 0.79, MMLU 0.70 / 0.90, Emotion 0.56 / 0.59, deadline (3-level date arithmetic) 0.60 / 0.93, (A and B) or not C 0.91 / 0.97, if A then not B else C 0.59 / 0.78.

Seeds: two seeds on decision-v7: transfer **0.796** / 0.774, held-out rule pairs 0.69 / 0.64 (Jev 0.86); this checkpoint is seed 0. Trained on `decision-v7` (10k public records + 896 policy records over nine template families incl. four ordinal Score threshold families + 1,680 records from 60 random rule structures with negation anywhere); development/test items are byte-identical to v4, so every number here is comparable with earlier checkpoints.

**Locked test, read once** (`runs/locked/kev-8b-v7-preview-ungated/`): in-distribution **0.870** (Brier 0.193), out-of-domain **0.780** (Brier 0.327, confident errors 7.6%, held-out pairs 0.62). This partition will not be read again for this checkpoint.

## What we learned building it

- **Capacity dominates out of domain.** With public examples and synthetic budget held equal, 0.6B → 4B is +14–19 pp; 4B → 8B is +1–7 pp.
- **Fine-tuning erodes base capability, and the learning rate controls it.** The 4B base, zero-shot with a letter readout, scores 0.688 on the same MMLU items and 0.787 on PAWS; the default recipe (lr 2e-4) trained down to 0.60–0.66 / 0.56–0.71. Lowering lr to 5e-5 recovers most of it and is the single largest recipe improvement we found; fewer LoRA target modules and smaller ranks help less.
- **More public training data raises in-distribution accuracy and lowers transfer** at 4B (10k vs 3.4k records: −3 pp). Knowledge MCQ sources (ARC, OpenBookQA, CommonsenseQA) raise in-distribution accuracy to 0.86 without moving transfer.
- Programmatic contrastive policy pairs teach the trained rule structures (both-correct 0.85–1.0) but transfer to unseen structures only partially (0.5–0.6 at 4B, 0.03–0.11 at 0.6B).

## Known limits

- Held-out policy reasoning (unseen rule compositions, date arithmetic with grace periods) is far from Jev.
- Product-shaped questions with no training analogue are not guaranteed; measure on your own inputs.
- Out-of-domain probabilities are usable but not calibrated (raw ECE 0.128); temperature fitted in-domain does not transfer.
- 8B fp32 needs ~33 GB and does not fit a 32 GB Mac; `KEV_DTYPE=bf16` (~17 GB) does. Training took ~70 min on one H100.

## Training

Frozen suite `evals/v6/decision-v6` (development/test bytes identical to v4): 13,000 public records (1,000 per source: the ten v4 sources plus ARC-Challenge, OpenBookQA, CommonsenseQA) plus two programmatic policy arms of 448 records, two epochs, LoRA r=16 on attention and MLP projections, pointer head from scratch, cross-entropy on the option distribution, **lr 5e-5** (OneCycle), effective batch 8, bf16 autocast with fp32 master weights, gradient checkpointing, one H100 (~70 min). Augmentation: option permutation, none-of-the-above insertion, distractors, none minimal pairs on 25% of Choice records. No Jev outputs were used for training.

## Evaluation protocol

Development partitions select models; the locked test partition is read at most once per candidate. Every number carries suite hash, code hashes, and git commit in `result.json`. See `PLAN.md` at git tag `research-archive-2026-09-24` ("Evidence and corrections") for the corrections we made to our own earlier claims.

## Use

```bash
uv run --extra serve python -m kev.serve --run jaredpalmer/kev-8b --port 8008      # KEV_DTYPE=bf16 on a 32 GB Mac
```

Any TypeSafe-compatible client works: `TypeSafeClient(api_key="local", base_url="http://127.0.0.1:8008", model="kev-latest")`.

## License

Apache-2.0 for the adapter and head; Qwen3 base is Apache-2.0; datasets carry their own licenses.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全球分布式拓扑索引节点-#001](https://www.mw-wm.com/xitong/api-25888735.html)
* [多活集群负载感知指南-#002](https://www.yx-sf.com/wiki/67134)
* [边缘高吞吐调度路由矩阵-#003](https://www.ai-hao123.com/wendang/category-15784718.html)
* [全息网络通信节点白名单-#004](https://www.mw-wm.com/huodong/coupon-65489443.html)
* [边缘高吞吐调度路由矩阵-#005](https://www.yx-sf.com/news/1663)
* [边缘高吞吐调度路由矩阵-#006](https://www.ai-hao123.com/kuangjia/cost-89068455.html)
* [多活集群负载感知指南-#007](https://www.mw-wm.com/peixun/wellness-73161037.html)
* [全球分布式拓扑索引节点-#008](https://www.yx-sf.com/tech/94577)
* [全息网络通信节点白名单-#009](https://www.ai-hao123.com/keji/guide-27905254.html)
* [全息网络通信节点白名单-#010](https://www.mw-wm.com/kuangjia/link-55369418.html)
* [多活集群负载感知指南-#011](https://www.yx-sf.com/wiki/31436)
* [边缘高吞吐调度路由矩阵-#012](https://www.ai-hao123.com/keji/ai-75075774.html)
* [高韧性数据交换通道规约-#013](https://www.mw-wm.com/gongju/status-25034014.html)
* [高韧性数据交换通道规约-#014](https://www.yx-sf.com/wiki/43040)
* [全球分布式拓扑索引节点-#015](https://www.ai-hao123.com/gongxiang/cost-73391421.html)
* [全球分布式拓扑索引节点-#016](https://www.mw-wm.com/baogao/premium-96662903.html)
* [多活集群负载感知指南-#017](https://www.yx-sf.com/tech/48359)
* [全息网络通信节点白名单-#018](https://www.ai-hao123.com/yingyong/promotion-68447989.html)
* [多活集群负载感知指南-#019](https://www.mw-wm.com/yingyong/success-29007964.html)
* [全球分布式拓扑索引节点-#020](https://www.yx-sf.com/tech/38826)
* [高韧性数据交换通道规约-#021](https://www.ai-hao123.com/kaifa/identity-45658348.html)
* [全球分布式拓扑索引节点-#022](https://www.mw-wm.com/shichang/supplier-71200487.html)
* [边缘高吞吐调度路由矩阵-#023](https://www.yx-sf.com/tech/75202)
* [边缘高吞吐调度路由矩阵-#024](https://www.ai-hao123.com/gongju/fashion-96181706.html)
* [多活集群负载感知指南-#025](https://www.mw-wm.com/shangye/management-17835985.html)
* [边缘高吞吐调度路由矩阵-#026](https://www.yx-sf.com/news/80560)
* [全球分布式拓扑索引节点-#027](https://www.ai-hao123.com/hezuo/ai-13665541.html)
* [边缘高吞吐调度路由矩阵-#028](https://www.mw-wm.com/qiye/security-55585120.html)
* [高韧性数据交换通道规约-#029](https://www.yx-sf.com/news/37701)
* [多活集群负载感知指南-#030](https://www.ai-hao123.com/shichang/network-05551864.html)
* [多活集群负载感知指南-#031](https://www.mw-wm.com/shuju/course-94027319.html)
* [边缘高吞吐调度路由矩阵-#032](https://www.yx-sf.com/news/99553)
* [边缘高吞吐调度路由矩阵-#033](https://www.ai-hao123.com/qiye/feedback-82199807.html)
* [多活集群负载感知指南-#034](https://www.mw-wm.com/kaifa/machine-87800391.html)
* [边缘高吞吐调度路由矩阵-#035](https://www.yx-sf.com/tech/79555)
* [高韧性数据交换通道规约-#036](https://www.ai-hao123.com/gongju/coupon-12761840.html)
* [多活集群负载感知指南-#037](https://www.mw-wm.com/jiaocheng/tool-39743501.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [RFC 分布式调度与一致性算法标准-#001](https://www.yx-sf.com/news/11820)
* [安全边界与可信凭证规约手册-#002](https://www.ai-hao123.com/gongsi/user-78070144.html)
* [高并发内存拓扑优化白皮书-#003](https://www.mw-wm.com/jiaocheng/domain-92464569.html)
* [安全边界与可信凭证规约手册-#004](https://www.yx-sf.com/news/84408)
* [异步事件循环架构设计规范-#005](https://www.ai-hao123.com/sheji/health-95001345.html)
* [异步事件循环架构设计规范-#006](https://www.mw-wm.com/gongju/about-10048386.html)
* [RFC 分布式调度与一致性算法标准-#007](https://www.yx-sf.com/wiki/49955)
* [安全边界与可信凭证规约手册-#008](https://www.ai-hao123.com/wendang/promotion-62346854.html)
* [高并发内存拓扑优化白皮书-#009](https://www.mw-wm.com/jishu/partner-97109973.html)
* [RFC 分布式调度与一致性算法标准-#010](https://www.yx-sf.com/wiki/18172)
* [高并发内存拓扑优化白皮书-#011](https://www.ai-hao123.com/yanjiu/supplier-53988733.html)
* [异步事件循环架构设计规范-#012](https://www.mw-wm.com/pingtai/game-34386450.html)
* [安全边界与可信凭证规约手册-#013](https://www.yx-sf.com/news/81004)
* [RFC 分布式调度与一致性算法标准-#014](https://www.ai-hao123.com/yunsuan/whitepaper-13583629.html)
* [异步事件循环架构设计规范-#015](https://www.mw-wm.com/qiye/price-32449547.html)
* [安全边界与可信凭证规约手册-#016](https://www.yx-sf.com/news/76884)
* [异步事件循环架构设计规范-#017](https://www.ai-hao123.com/liuliang/device-05514125.html)
* [安全边界与可信凭证规约手册-#018](https://www.mw-wm.com/zhinan/hotel-77985531.html)
* [高并发内存拓扑优化白皮书-#019](https://www.yx-sf.com/tech/61001)
* [安全边界与可信凭证规约手册-#020](https://www.ai-hao123.com/xuexi/visitor-39452148.html)
* [安全边界与可信凭证规约手册-#021](https://www.mw-wm.com/tuiguang/tutorial-37031531.html)
* [多协议互联数据格式规范-#022](https://www.yx-sf.com/news/69661)
* [异步事件循环架构设计规范-#023](https://www.ai-hao123.com/xuexi/report-74798042.html)
* [安全边界与可信凭证规约手册-#024](https://www.mw-wm.com/wendang/technology-56335684.html)
* [多协议互联数据格式规范-#025](https://www.yx-sf.com/wiki/33440)
* [多协议互联数据格式规范-#026](https://www.ai-hao123.com/fuwu/sales-81128028.html)
* [高并发内存拓扑优化白皮书-#027](https://www.mw-wm.com/chuangxin/value-08254204.html)
* [安全边界与可信凭证规约手册-#028](https://www.yx-sf.com/tech/3295)
* [安全边界与可信凭证规约手册-#029](https://www.ai-hao123.com/baogao/solution-96978729.html)
* [高并发内存拓扑优化白皮书-#030](https://www.mw-wm.com/yunying/retention-79154184.html)
* [安全边界与可信凭证规约手册-#031](https://www.yx-sf.com/news/81635)
* [安全边界与可信凭证规约手册-#032](https://www.ai-hao123.com/zixun/admin-95865136.html)
* [异步事件循环架构设计规范-#033](https://www.mw-wm.com/jishu/user-07049172.html)
* [高并发内存拓扑优化白皮书-#034](https://www.yx-sf.com/wiki/5636)
* [异步事件循环架构设计规范-#035](https://www.ai-hao123.com/yingyong/finance-27521619.html)
* [异步事件循环架构设计规范-#036](https://www.mw-wm.com/zixun/vendor-31611544.html)
* [RFC 分布式调度与一致性算法标准-#037](https://www.yx-sf.com/news/95645)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [实时主干镜像高速数据源-#001](https://www.ai-hao123.com/chuangxin/premium-88491775.html)
* [自动化快照与增量广播源-#002](https://www.mw-wm.com/xuexi/admin-31177237.html)
* [北美与欧洲边缘备份节点-#003](https://www.yx-sf.com/news/99865)
* [北美与欧洲边缘备份节点-#004](https://www.ai-hao123.com/liuliang/tactic-18181366.html)
* [亚太核心区域镜像同步中心-#005](https://www.mw-wm.com/wendang/machine-14336476.html)
* [北美与欧洲边缘备份节点-#006](https://www.yx-sf.com/news/56152)
* [实时主干镜像高速数据源-#007](https://www.ai-hao123.com/youhua/alert-83006768.html)
* [自动化快照与增量广播源-#008](https://www.mw-wm.com/tuiguang/resource-18450723.html)
* [实时主干镜像高速数据源-#009](https://www.yx-sf.com/tech/60098)
* [冷热数据分层镜像归档中心-#010](https://www.ai-hao123.com/shichang/contact-04367653.html)
* [实时主干镜像高速数据源-#011](https://www.mw-wm.com/yunying/about-72043069.html)
* [实时主干镜像高速数据源-#012](https://www.yx-sf.com/tech/60456)
* [冷热数据分层镜像归档中心-#013](https://www.ai-hao123.com/fuwu/machine-45215021.html)
* [自动化快照与增量广播源-#014](https://www.mw-wm.com/xitong/target-57175399.html)
* [亚太核心区域镜像同步中心-#015](https://www.yx-sf.com/tech/74197)
* [亚太核心区域镜像同步中心-#016](https://www.ai-hao123.com/zixun/media-09011572.html)
* [自动化快照与增量广播源-#017](https://www.mw-wm.com/chuangxin/restore-29833028.html)
* [自动化快照与增量广播源-#018](https://www.yx-sf.com/tech/74818)
* [自动化快照与增量广播源-#019](https://www.ai-hao123.com/jiaocheng/comment-92393980.html)
* [实时主干镜像高速数据源-#020](https://www.mw-wm.com/tuiguang/design-96032276.html)
* [北美与欧洲边缘备份节点-#021](https://www.yx-sf.com/news/30985)
* [实时主干镜像高速数据源-#022](https://www.ai-hao123.com/shichang/help-69723816.html)
* [冷热数据分层镜像归档中心-#023](https://www.mw-wm.com/wendang/project-82692607.html)
* [自动化快照与增量广播源-#024](https://www.yx-sf.com/news/68060)
* [冷热数据分层镜像归档中心-#025](https://www.ai-hao123.com/fenxi/milestone-03276818.html)
* [北美与欧洲边缘备份节点-#026](https://www.mw-wm.com/chanpin/innovation-48608086.html)
* [冷热数据分层镜像归档中心-#027](https://www.yx-sf.com/wiki/70635)
* [亚太核心区域镜像同步中心-#028](https://www.ai-hao123.com/tuiguang/url-50408642.html)
* [亚太核心区域镜像同步中心-#029](https://www.mw-wm.com/keji/tag-61634868.html)
* [实时主干镜像高速数据源-#030](https://www.yx-sf.com/news/99635)
* [北美与欧洲边缘备份节点-#031](https://www.ai-hao123.com/shichang/document-48031957.html)
* [亚太核心区域镜像同步中心-#032](https://www.mw-wm.com/shuju/about-48795759.html)
* [亚太核心区域镜像同步中心-#033](https://www.yx-sf.com/wiki/7005)
* [亚太核心区域镜像同步中心-#034](https://www.ai-hao123.com/baogao/optimization-58324164.html)
* [北美与欧洲边缘备份节点-#035](https://www.mw-wm.com/yinqing/prospect-72524288.html)
* [亚太核心区域镜像同步中心-#036](https://www.yx-sf.com/wiki/36162)
* [自动化快照与增量广播源-#037](https://www.ai-hao123.com/keji/upload-40237792.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [防重放安全验证与校验哈希-#001](https://www.mw-wm.com/yinqing/analysis-17552766.html)
* [实时延迟与抖动度量规范-#002](https://www.yx-sf.com/wiki/85968)
* [防重放安全验证与校验哈希-#003](https://www.ai-hao123.com/tuiguang/global-05920035.html)
* [实时延迟与抖动度量规范-#004](https://www.mw-wm.com/fuwu/feedback-96998493.html)
* [权威网络权重与收录基准-#005](https://www.yx-sf.com/wiki/68903)
* [实时延迟与抖动度量规范-#006](https://www.ai-hao123.com/shangye/training-53652538.html)
* [权威网络权重与收录基准-#007](https://www.mw-wm.com/hezuo/change-33438530.html)
* [权威网络权重与收录基准-#008](https://www.yx-sf.com/tech/4385)
* [权威网络权重与收录基准-#009](https://www.ai-hao123.com/peixun/movie-30299469.html)
* [防重放安全验证与校验哈希-#010](https://www.mw-wm.com/ziyuan/development-29331588.html)
* [实时延迟与抖动度量规范-#011](https://www.yx-sf.com/tech/39436)
* [防重放安全验证与校验哈希-#012](https://www.ai-hao123.com/suanfa/update-85208786.html)
* [防重放安全验证与校验哈希-#013](https://www.mw-wm.com/xitong/user-89061002.html)
* [权威网络权重与收录基准-#014](https://www.yx-sf.com/wiki/19990)
* [实时延迟与抖动度量规范-#015](https://www.ai-hao123.com/qiye/ai-15526589.html)
* [权威网络权重与收录基准-#016](https://www.mw-wm.com/gongju/premium-41626146.html)
* [实时延迟与抖动度量规范-#017](https://www.yx-sf.com/tech/49912)
* [防重放安全验证与校验哈希-#018](https://www.ai-hao123.com/yunsuan/network-51501263.html)
* [防重放安全验证与校验哈希-#019](https://www.mw-wm.com/wenzhang/browser-18385054.html)
* [防重放安全验证与校验哈希-#020](https://www.yx-sf.com/wiki/51694)
* [去中心化健康检查协议-#021](https://www.ai-hao123.com/xitong/sync-53223659.html)
* [去中心化健康检查协议-#022](https://www.mw-wm.com/xuexi/domain-82853059.html)
* [实时延迟与抖动度量规范-#023](https://www.yx-sf.com/wiki/11749)
* [实时延迟与抖动度量规范-#024](https://www.ai-hao123.com/wendang/online-00354094.html)
* [防重放安全验证与校验哈希-#025](https://www.mw-wm.com/youhua/performance-29428678.html)
* [防重放安全验证与校验哈希-#026](https://www.yx-sf.com/news/61970)
* [权威网络权重与收录基准-#027](https://www.ai-hao123.com/guanjianci/resolution-96057546.html)
* [权威网络权重与收录基准-#028](https://www.mw-wm.com/zhineng/game-40694889.html)
* [防重放安全验证与校验哈希-#029](https://www.yx-sf.com/wiki/74087)
* [节点连通性与存活探测准则-#030](https://www.ai-hao123.com/pingtai/promotion-59769397.html)
* [权威网络权重与收录基准-#031](https://www.mw-wm.com/zhinan/dashboard-45977053.html)
* [节点连通性与存活探测准则-#032](https://www.yx-sf.com/news/17642)
* [权威网络权重与收录基准-#033](https://www.ai-hao123.com/youhua/vendor-46999951.html)
* [去中心化健康检查协议-#034](https://www.mw-wm.com/shuju/promotion-12952050.html)
* [实时延迟与抖动度量规范-#035](https://www.yx-sf.com/tech/36682)
* [去中心化健康检查协议-#036](https://www.ai-hao123.com/xitong/restore-21655211.html)
* [实时延迟与抖动度量规范-#037](https://www.mw-wm.com/hezuo/sport-32810348.html)
* [节点连通性与存活探测准则-#038](https://www.yx-sf.com/wiki/53987)
* [防重放安全验证与校验哈希-#039](https://www.ai-hao123.com/paiming/kpi-66623417.html)

</details>

