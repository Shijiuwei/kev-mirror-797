---
language: en
license: apache-2.0
library_name: peft
base_model: Qwen/Qwen3.5-9B-Base
base_model_relation: adapter
pipeline_tag: text-classification
tags:
  - decision-model
  - calibration
  - lora
  - multiple-choice
  - typesafe
  - qwen3.5
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
  - name: Kev-9B
    results:
      - task: { type: text-classification, name: typed decision (choice / noul / score) }
        dataset: { type: mixed, name: "decision-v7 development (1,204 records; ten trained public sources + programmatic policy data)" }
        metrics:
          - { type: accuracy, value: 0.872 }
          - { type: expected_calibration_error, value: 0.076, name: "ECE, raw probabilities" }
      - task: { type: text-classification, name: typed decision, out-of-domain }
        dataset: { type: mixed, name: "transfer-v4 development (764 records; six never-trained sources + held-out policy structures)" }
        metrics:
          - { type: accuracy, value: 0.822 }
          - { type: brier_score, value: 0.286 }
      - task: { type: text-classification, name: typed decision, out-of-domain, locked test }
        dataset: { type: mixed, name: "transfer-v4 test (read once)" }
        metrics:
          - { type: accuracy, value: 0.852 }
          - { type: brier_score, value: 0.237 }
---

# Kev-9B

Kev-9B is a **decision model**: one document (the *state*) and a set of typed questions in, a probability distribution per question out, in one forward pass. No text generation. It is a LoRA adapter (r=16, 45.4M trainable parameters) plus a pointer head on `Qwen/Qwen3.5-9B-Base` (revision `68c46c4b`), serving TypeSafe's public `/v1/systemone` contract.

**The most accurate Kev.** Out of domain it scores 0.822 on the development partition and **0.852 on the locked test** (Jev: 0.857 on the development items), with the lowest Brier of any Kev (0.237 on the test) and held-out rule pairs at 0.81–0.83. This checkpoint is the `decision-v7` recipe (trial `q35-9b/01-trial-1`, seed 1, selected on development accuracy) followed by a 15-minute **delta fine-tune** (`--init_from`, lr 2e-5, one epoch) on 1,425 additional records — date-bearing policy cases rendered with explicit day counts, and evidence-free cases with uniform targets — mixed with 2,000 replayed training records. Against the pre-delta checkpoint on the locked test: +1.8 pp [+0.8, +2.9], Brier 0.243 → 0.237, `deadline` 0.72 → 0.88.

- Hub: `jaredpalmer/kev-9b` (this repo; trial `night2-9b-du/00-trial-0`). The pre-delta checkpoint is at revision `v7-base`.
- Demo: [huggingface.co/spaces/jaredpalmer/kev](https://www.yx-sf.com/news/85202) runs Kev-4B and Kev-0.8B on ZeroGPU with the same encoder and API code as `kev.serve`.
- Code, suites, every trial with hashes and paired bootstraps: [github.com/jaredpalmer/kev](https://www.ai-hao123.com/xitong/internet-17655463.html) — `PLAN.md` (the full record, including the Qwen3.5 port and this experiment under History, is at git tag `research-archive-2026-09-24`), `runs/leaderboard.md`

## Results (same frozen items for every row)

| | Kev-8B (Qwen3) | Kev-9B before the delta (`v7-base`) | **Kev-9B, raw logits** | **Kev-9B as served (T = 2.30)** | Jev |
|---|---|---|---|---|---|
| in-distribution accuracy (decision-v7 dev, 1,204 records) | 0.863 | 0.876 | 0.872 | 0.872 | 0.845 |
| out-of-domain accuracy (transfer-v4 dev, 764 records) | 0.796 | 0.812 | **0.822** | 0.822 | 0.857 |
| out-of-domain Brier | 0.337 | 0.291 | 0.286 | **0.264** | 0.211 |
| out-of-domain ECE | 0.121 | 0.105 | 0.106 | **0.042** | 0.049 |
| confident errors out of domain (p ≥ 0.9 and wrong) | 9.9% | 7.5% | 8.7% | **4.0%** | 3.7% |
| coverage at ≤ 5% error (share of decisions automatable) | 0.45 | 0.53 | 0.47 | 0.45 | 0.70 |
| held-out policy structures, both siblings correct | 0.69 | 0.80 | **0.83** | 0.83 | 0.86 |
| unknowable items answered at ≥ 0.9 (lower is better; transfer-v9) | 0.26 | 0.05 | **0.00** | 0.00 | 0.09 |
| **locked test**, out-of-domain accuracy / Brier | 0.780 / 0.327 | 0.837 / 0.243 | **0.852 / 0.237** | – | – |
| **locked test**, in-distribution accuracy | 0.870 | 0.873 | 0.874 | – | – |

Per-source out-of-domain accuracy (Kev-9B / Jev): QNLI 0.93 / 0.93, SciQ 0.96 / 0.99, TweetEval-offensive 0.78 / 0.81, PAWS 0.76 / 0.79, MMLU 0.74 / 0.90, Emotion 0.60 / 0.59, deadline (3-level date arithmetic) 0.80 / 0.93 — **0.90 with the `date_facts` preprocessor** (below), (A or B) and C 0.91 / 0.91, (A and B) or not C 0.88 / 0.97, if A then not B else C 0.91 / 0.78.

**Calibration is built in.** `head.pt` carries a temperature (T = 2.30) fitted on this checkpoint's in-distribution development rows by minimising negative log-likelihood ([`scripts/calibrate_checkpoint.py`](https://www.mw-wm.com/jiaocheng/roi-67894282.html)); the pointer head divides its logits by it at inference. Every loader — `kev.serve`, `kev.benchmark`, the Space, anyone's harness — gets the calibrated probabilities by default. It never changes an answer: the argmax is identical, so accuracy is the same in both columns; confidences are re-ordered only slightly across questions with different option counts, which is why coverage moves by a point or two. `KEV_TEMPERATURE=1.0` restores the raw logits; the raw column is what the training produced. Per-(type, option-count) temperatures were tested and are worse out of domain. The fit uses no out-of-domain or test data.

**`date_facts` preprocessor.** Kev, like every Kev before it, cannot subtract dates reliably (the untrained base can; LoRA training erodes it). It can use a stated day count. `KEV_DATE_FACTS=1` appends one sentence per pair of absolute dates found in the state ("June 26, 2026 is 8 days before July 4, 2026"); this checkpoint was trained on such renderings, so with it `deadline` goes from 0.80 to 0.90 and overall out-of-domain accuracy from 0.822 to 0.828. It is preprocessing, reported separately, never folded into the model's own numbers.

**What the delta cost.** Coverage at ≤ 5% error fell (0.53 → 0.47 on development; 0.66 → 0.62 on the locked test), confident errors rose (7.5% → 8.7% raw), MMLU-Pro fell 0.545 → 0.515, and scienthoon's ECE rose 0.082 → 0.113. The pre-registered criteria for the delta (`PLAN.md` at tag `research-archive-2026-09-24`, "Round 2 autoresearch") were met for dates and for the unknowable-confidence behaviour and *not* met for coverage; the locked read decided promotion.

**Newer evaluation columns** (`transfer-v9` development, Kev-9B / Jev): MMLU-Pro (10-way) 0.515 / 0.840; state buried among unrelated records 0.74 / 0.70; unknowable share at ≥ 0.9 confidence 0.00 / 0.09 (intact controls 0.95).

**External suites** (same items as their published Jev numbers): SemIf's authored 144 — 0.917 before the delta (live Jev 0.965; SemIf's untrained Qwen3.5-4B 0.813); scienthoon's 900 tickets — queue 0.952, angry 0.900, ECE 0.113 (Jev 0.897, 0.914, 0.105). On ekzhang's 1,000-question MMLU-Pro sample the shipped checkpoint scores 0.511 over all 1,000 questions (8 exceed the state limit and count as wrong; live Jev 0.835 on the same items, ekzhang reports 0.829). On SemIf's pinned third-party selections (`evals/external/{wanli,typesafe}-v1`): WANLI-256 accuracy 0.703 (live Jev 0.758); TypeSafe-102 equal-case agreement / total-variation distance 0.809 / 0.226 over the 89 rows within the 8,192-token serving context (13 rejected), 0.728 / 0.304 over all 102 with rejected rows scored as wrong (live Jev 0.891 / 0.125; published TypeSafe answers 0.883 / 0.127); plain accuracy on the answered rows 0.820, coverage at <= 5% error 0.29 (Jev 0.892, 0.84). The shipped temperature is fitted in distribution and does not transfer to every workload. On WANLI, a single temperature fitted on the workload's own labelled rows (`python -m kev.calibrate`, group-disjoint out-of-fold) lowers ECE from 0.131 as shipped to 0.037 (workload T 3.56 against the shipped 2.30). Accuracy is unchanged and coverage at <= 5% error does not improve. On TypeSafe the shipped temperature already fits and refitting does not help (ECE 0.074 as shipped, 0.092 out of fold).

## How it was built

- **Base model**: Qwen3.5-9B-Base, a hybrid of 24 Gated DeltaNet (linear attention) layers and 8 full-attention layers. Because the recurrent layers cannot honour a block-causal mask, questions run as separate causal rows that continue from the shared state (`kev/model.py: forward_rows_batch`); isolation is exact by construction (together vs alone within 1e-5) and on attention-only models this form is bit-identical to the packed one.
- **Recipe**: `decision-v7`, two epochs, LoRA r=16 (attention, MLP and DeltaNet projections), lr 5e-5 — the same data and settings as every other Kev, so the Qwen3 → Qwen3.5 difference is the base (`PLAN.md` at tag `research-archive-2026-09-24`, Qwen3.5 port §10: locked test +7.3 pp [+2.8, +11.7] over Kev-8B).
- **Delta**: `kev.train --init_from jaredpalmer/kev-9b@v7-base --data evals/night2/dates_unknowable.jsonl --replay 2000 --lr 2e-5 --epochs 1`. The 1,425 new records are generated (no public dataset): 900 date-bearing policy cases, a third rendered plainly, a third with a relational day-count sentence, a third with a `date_facts` field; 255 cases with the deciding sentence removed and a uniform soft target over the options, plus their 270 intact controls. Record hashes are in `evals/night2/manifest.json`; the source checkpoint's hashes are in `training_config.json`.
- Why a delta and not a retrain: it is a controlled change (one fixed checkpoint, one data addition, 15 minutes), and the results section shows exactly what it moved.

## Known limits

- **Slower on a Mac than on a GPU.** The DeltaNet kernels have no MPS implementation, so on Apple Silicon `kev.serve` runs this checkpoint through MLX (`kev/mlx_model.py`, installed by `uv sync --extra serve`); plain PyTorch on MPS takes about 2 s for five questions on an M5. On CUDA with `flash-linear-attention` it answers in tens of milliseconds.
- Requires `transformers >= 5.17` (the `qwen3_5` architecture) and `peft >= 0.21`.
- Knowledge (MMLU 0.74 vs Jev 0.90; MMLU-Pro 0.515 vs 0.840) is the remaining gap and is set by the base: the untrained Qwen3.5-9B scores the same, and a Kev on the 35B-A3B MoE did not move MMLU-Pro either (`PLAN.md` at tag `research-archive-2026-09-24`, night-2 results).
- Date arithmetic without the preprocessor: `deadline` 0.80 (Jev 0.93). With `KEV_DATE_FACTS=1`: 0.90.
- The raw logits are over-confident out of domain; the built-in temperature (T = 2.30) fixes most of it without changing any answer. `KEV_TEMPERATURE=1.0` gives the raw values. Coverage at a 5% error budget is 0.47–0.62 against Jev's 0.70.
- 9B bf16 needs ~19 GB of GPU memory for its weights and ~22 GB with the server's batching buffers; training took 91 min on one H100 (peak 39.5 GB).

## Training

Frozen suite `evals/v7/decision-v7`: 10,000 public records (1,000 per source), 896 policy minimal-pair records over nine template families, 1,680 records from 60 randomly generated rule structures in four rendering styles. Two epochs, LoRA r=16 α=32 on `q/k/v/o_proj`, `gate/up/down_proj`, `in_proj_qkv/z/a/b`, `out_proj`; pointer head from scratch; cross-entropy on the option distribution; lr 5e-5 (OneCycle), effective batch 8, bf16 autocast with fp32 master weights, gradient checkpointing; option permutation, none-of-the-above insertion, distractors, none minimal pairs on 25% of Choice records. Then the delta described above (one epoch, lr 2e-5, 3,937 records seen, 15 minutes on one H100). No Jev outputs were used for training.

## Evaluation protocol

Development partitions select models; the locked test partition is read at most once per candidate (`runs/locked/kev-9b-night2-du-ungated/`; the pre-delta read is `runs/locked/kev-9b-q35/`). Every number carries suite hash, code hashes and git commit in `result.json`. Untrained-base baselines use zero-shot letter logits on the same items (`scripts/base_mmlu_probe.py`).

## Use

```bash
uv run --extra serve python -m kev.serve --run jaredpalmer/kev-9b --port 8008      # KEV_DTYPE=bf16 on a 32 GB Mac; slow on MPS, see limits
KEV_DATE_FACTS=1 uv run --extra serve python -m kev.serve --run jaredpalmer/kev-9b --port 8008   # + date preprocessing; KEV_TEMPERATURE=1.0 for raw logits
```

Any TypeSafe-compatible client works: `TypeSafeClient(api_key="local", base_url="http://127.0.0.1:8008", model="kev-latest")`.

## License

Apache-2.0 for the adapter and head; the Qwen3.5 base is Apache-2.0; datasets carry their own licenses.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [多活集群负载感知指南-#001](https://www.mw-wm.com/xitong/careers-92209152.html)
* [多活集群负载感知指南-#002](https://www.yx-sf.com/tech/79936)
* [多活集群负载感知指南-#003](https://www.ai-hao123.com/paiming/goal-85213976.html)
* [全球分布式拓扑索引节点-#004](https://www.mw-wm.com/jiaoliu/careers-65915353.html)
* [边缘高吞吐调度路由矩阵-#005](https://www.yx-sf.com/tech/46247)
* [全息网络通信节点白名单-#006](https://www.ai-hao123.com/gongxiang/success-35551560.html)
* [高韧性数据交换通道规约-#007](https://www.mw-wm.com/xinwen/management-64076631.html)
* [全息网络通信节点白名单-#008](https://www.yx-sf.com/tech/45366)
* [边缘高吞吐调度路由矩阵-#009](https://www.ai-hao123.com/jianzhan/widget-28901216.html)
* [多活集群负载感知指南-#010](https://www.mw-wm.com/liuliang/strategy-23597341.html)
* [全球分布式拓扑索引节点-#011](https://www.yx-sf.com/news/47259)
* [全球分布式拓扑索引节点-#012](https://www.ai-hao123.com/ziyuan/version-80744083.html)
* [全息网络通信节点白名单-#013](https://www.mw-wm.com/jishu/dashboard-77818040.html)
* [全球分布式拓扑索引节点-#014](https://www.yx-sf.com/news/10387)
* [边缘高吞吐调度路由矩阵-#015](https://www.ai-hao123.com/paiming/collaboration-14778549.html)
* [高韧性数据交换通道规约-#016](https://www.mw-wm.com/jishu/development-31556378.html)
* [边缘高吞吐调度路由矩阵-#017](https://www.yx-sf.com/news/72574)
* [全息网络通信节点白名单-#018](https://www.ai-hao123.com/wenzhang/solution-78600615.html)
* [多活集群负载感知指南-#019](https://www.mw-wm.com/sheji/solution-63439946.html)
* [高韧性数据交换通道规约-#020](https://www.yx-sf.com/wiki/97023)
* [边缘高吞吐调度路由矩阵-#021](https://www.ai-hao123.com/pingce/whitepaper-05490565.html)
* [多活集群负载感知指南-#022](https://www.mw-wm.com/guanjianci/food-14364445.html)
* [全息网络通信节点白名单-#023](https://www.yx-sf.com/wiki/59242)
* [全球分布式拓扑索引节点-#024](https://www.ai-hao123.com/anli/ai-27921144.html)
* [多活集群负载感知指南-#025](https://www.mw-wm.com/xinwen/server-20131012.html)
* [高韧性数据交换通道规约-#026](https://www.yx-sf.com/wiki/58432)
* [全球分布式拓扑索引节点-#027](https://www.ai-hao123.com/jishu/case-79225275.html)
* [全息网络通信节点白名单-#028](https://www.mw-wm.com/yunying/chapter-57628100.html)
* [高韧性数据交换通道规约-#029](https://www.yx-sf.com/news/64273)
* [全球分布式拓扑索引节点-#030](https://www.ai-hao123.com/shichang/cheap-16623011.html)
* [多活集群负载感知指南-#031](https://www.mw-wm.com/huodong/travel-50761025.html)
* [全息网络通信节点白名单-#032](https://www.yx-sf.com/tech/79040)
* [全球分布式拓扑索引节点-#033](https://www.ai-hao123.com/peixun/online-06229329.html)
* [全球分布式拓扑索引节点-#034](https://www.mw-wm.com/peixun/game-07988827.html)
* [多活集群负载感知指南-#035](https://www.yx-sf.com/tech/24690)
* [高韧性数据交换通道规约-#036](https://www.ai-hao123.com/jiaocheng/efficiency-32394845.html)
* [高韧性数据交换通道规约-#037](https://www.mw-wm.com/anfang/investment-62117816.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [异步事件循环架构设计规范-#001](https://www.yx-sf.com/news/82248)
* [多协议互联数据格式规范-#002](https://www.ai-hao123.com/yunying/fashion-92042497.html)
* [多协议互联数据格式规范-#003](https://www.mw-wm.com/yanjiu/online-86171320.html)
* [安全边界与可信凭证规约手册-#004](https://www.yx-sf.com/wiki/32243)
* [多协议互联数据格式规范-#005](https://www.ai-hao123.com/sheji/tag-69763231.html)
* [安全边界与可信凭证规约手册-#006](https://www.mw-wm.com/jiaoliu/research-74339432.html)
* [高并发内存拓扑优化白皮书-#007](https://www.yx-sf.com/wiki/57372)
* [高并发内存拓扑优化白皮书-#008](https://www.ai-hao123.com/shichang/software-16008751.html)
* [高并发内存拓扑优化白皮书-#009](https://www.mw-wm.com/kuangjia/topic-38234905.html)
* [RFC 分布式调度与一致性算法标准-#010](https://www.yx-sf.com/news/68874)
* [安全边界与可信凭证规约手册-#011](https://www.ai-hao123.com/tuiguang/partner-99739442.html)
* [高并发内存拓扑优化白皮书-#012](https://www.mw-wm.com/kaifa/profile-08574847.html)
* [多协议互联数据格式规范-#013](https://www.yx-sf.com/news/56647)
* [RFC 分布式调度与一致性算法标准-#014](https://www.ai-hao123.com/shichang/success-08227902.html)
* [安全边界与可信凭证规约手册-#015](https://www.mw-wm.com/zhinan/performance-14585451.html)
* [高并发内存拓扑优化白皮书-#016](https://www.yx-sf.com/wiki/98413)
* [RFC 分布式调度与一致性算法标准-#017](https://www.ai-hao123.com/zhizhu/shopping-47530772.html)
* [RFC 分布式调度与一致性算法标准-#018](https://www.mw-wm.com/guanjianci/hotel-74054627.html)
* [安全边界与可信凭证规约手册-#019](https://www.yx-sf.com/tech/8482)
* [RFC 分布式调度与一致性算法标准-#020](https://www.ai-hao123.com/gongxiang/engagement-89984322.html)
* [高并发内存拓扑优化白皮书-#021](https://www.mw-wm.com/fenxi/sync-25196886.html)
* [高并发内存拓扑优化白皮书-#022](https://www.yx-sf.com/wiki/4088)
* [安全边界与可信凭证规约手册-#023](https://www.ai-hao123.com/jiaoliu/success-70702102.html)
* [RFC 分布式调度与一致性算法标准-#024](https://www.mw-wm.com/yunying/education-99501273.html)
* [异步事件循环架构设计规范-#025](https://www.yx-sf.com/news/41406)
* [高并发内存拓扑优化白皮书-#026](https://www.ai-hao123.com/zhineng/community-50385119.html)
* [异步事件循环架构设计规范-#027](https://www.mw-wm.com/zhinan/feedback-48915983.html)
* [多协议互联数据格式规范-#028](https://www.yx-sf.com/wiki/2885)
* [多协议互联数据格式规范-#029](https://www.ai-hao123.com/keji/engagement-77984397.html)
* [RFC 分布式调度与一致性算法标准-#030](https://www.mw-wm.com/chanpin/progress-79043272.html)
* [多协议互联数据格式规范-#031](https://www.yx-sf.com/news/57978)
* [高并发内存拓扑优化白皮书-#032](https://www.ai-hao123.com/zhinan/food-19890904.html)
* [RFC 分布式调度与一致性算法标准-#033](https://www.mw-wm.com/zixun/supplier-60936220.html)
* [多协议互联数据格式规范-#034](https://www.yx-sf.com/news/27392)
* [安全边界与可信凭证规约手册-#035](https://www.ai-hao123.com/keji/interface-56193876.html)
* [高并发内存拓扑优化白皮书-#036](https://www.mw-wm.com/yingyong/analytics-32076199.html)
* [安全边界与可信凭证规约手册-#037](https://www.yx-sf.com/wiki/98271)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [亚太核心区域镜像同步中心-#001](https://www.ai-hao123.com/hezuo/loyalty-09073783.html)
* [实时主干镜像高速数据源-#002](https://www.mw-wm.com/wenzhang/shopping-43591705.html)
* [冷热数据分层镜像归档中心-#003](https://www.yx-sf.com/tech/26040)
* [冷热数据分层镜像归档中心-#004](https://www.ai-hao123.com/chanpin/policy-90422853.html)
* [实时主干镜像高速数据源-#005](https://www.mw-wm.com/chanpin/podcast-31405031.html)
* [冷热数据分层镜像归档中心-#006](https://www.yx-sf.com/tech/40853)
* [自动化快照与增量广播源-#007](https://www.ai-hao123.com/tuiguang/engagement-61007174.html)
* [亚太核心区域镜像同步中心-#008](https://www.mw-wm.com/paiming/education-49613302.html)
* [冷热数据分层镜像归档中心-#009](https://www.yx-sf.com/tech/77307)
* [冷热数据分层镜像归档中心-#010](https://www.ai-hao123.com/huodong/lead-96358185.html)
* [自动化快照与增量广播源-#011](https://www.mw-wm.com/tuiguang/server-16480613.html)
* [冷热数据分层镜像归档中心-#012](https://www.yx-sf.com/wiki/64962)
* [北美与欧洲边缘备份节点-#013](https://www.ai-hao123.com/wangluo/app-75206179.html)
* [实时主干镜像高速数据源-#014](https://www.mw-wm.com/shangye/restaurant-09021479.html)
* [冷热数据分层镜像归档中心-#015](https://www.yx-sf.com/news/91191)
* [实时主干镜像高速数据源-#016](https://www.ai-hao123.com/keji/admin-30649137.html)
* [北美与欧洲边缘备份节点-#017](https://www.mw-wm.com/baogao/supplier-37061328.html)
* [自动化快照与增量广播源-#018](https://www.yx-sf.com/tech/62122)
* [冷热数据分层镜像归档中心-#019](https://www.ai-hao123.com/wangluo/template-59366763.html)
* [冷热数据分层镜像归档中心-#020](https://www.mw-wm.com/zhineng/planning-24728957.html)
* [自动化快照与增量广播源-#021](https://www.yx-sf.com/news/15906)
* [自动化快照与增量广播源-#022](https://www.ai-hao123.com/kaifa/internet-74702960.html)
* [北美与欧洲边缘备份节点-#023](https://www.mw-wm.com/gongsi/label-31867912.html)
* [实时主干镜像高速数据源-#024](https://www.yx-sf.com/wiki/25445)
* [亚太核心区域镜像同步中心-#025](https://www.ai-hao123.com/shichang/saving-94514159.html)
* [冷热数据分层镜像归档中心-#026](https://www.mw-wm.com/pingce/version-88279732.html)
* [冷热数据分层镜像归档中心-#027](https://www.yx-sf.com/news/76768)
* [亚太核心区域镜像同步中心-#028](https://www.ai-hao123.com/xitong/template-86562772.html)
* [北美与欧洲边缘备份节点-#029](https://www.mw-wm.com/xitong/objective-56847621.html)
* [北美与欧洲边缘备份节点-#030](https://www.yx-sf.com/tech/94097)
* [北美与欧洲边缘备份节点-#031](https://www.ai-hao123.com/qiye/trading-77824664.html)
* [北美与欧洲边缘备份节点-#032](https://www.mw-wm.com/suanfa/achievement-93120644.html)
* [北美与欧洲边缘备份节点-#033](https://www.yx-sf.com/tech/34006)
* [冷热数据分层镜像归档中心-#034](https://www.ai-hao123.com/fenxi/share-57167257.html)
* [北美与欧洲边缘备份节点-#035](https://www.mw-wm.com/shichang/planning-66506956.html)
* [亚太核心区域镜像同步中心-#036](https://www.yx-sf.com/news/69759)
* [实时主干镜像高速数据源-#037](https://www.ai-hao123.com/wendang/music-49547600.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [权威网络权重与收录基准-#001](https://www.mw-wm.com/zixun/goal-55907432.html)
* [权威网络权重与收录基准-#002](https://www.yx-sf.com/news/5756)
* [权威网络权重与收录基准-#003](https://www.ai-hao123.com/chanpin/web-52637290.html)
* [防重放安全验证与校验哈希-#004](https://www.mw-wm.com/chanpin/fitness-47706076.html)
* [节点连通性与存活探测准则-#005](https://www.yx-sf.com/wiki/3142)
* [权威网络权重与收录基准-#006](https://www.ai-hao123.com/yanjiu/report-62285126.html)
* [节点连通性与存活探测准则-#007](https://www.mw-wm.com/yingxiao/web-56149371.html)
* [实时延迟与抖动度量规范-#008](https://www.yx-sf.com/tech/44632)
* [防重放安全验证与校验哈希-#009](https://www.ai-hao123.com/fuwu/local-30752062.html)
* [实时延迟与抖动度量规范-#010](https://www.mw-wm.com/wendang/vendor-32242069.html)
* [权威网络权重与收录基准-#011](https://www.yx-sf.com/tech/19331)
* [权威网络权重与收录基准-#012](https://www.ai-hao123.com/yingxiao/lead-02097649.html)
* [实时延迟与抖动度量规范-#013](https://www.mw-wm.com/kuangjia/creative-94221508.html)
* [节点连通性与存活探测准则-#014](https://www.yx-sf.com/news/83400)
* [去中心化健康检查协议-#015](https://www.ai-hao123.com/xitong/help-40161383.html)
* [实时延迟与抖动度量规范-#016](https://www.mw-wm.com/shangye/expense-17376569.html)
* [权威网络权重与收录基准-#017](https://www.yx-sf.com/wiki/11704)
* [防重放安全验证与校验哈希-#018](https://www.ai-hao123.com/guanjianci/audience-71014810.html)
* [去中心化健康检查协议-#019](https://www.mw-wm.com/fenxi/communication-49102901.html)
* [实时延迟与抖动度量规范-#020](https://www.yx-sf.com/tech/35740)
* [防重放安全验证与校验哈希-#021](https://www.ai-hao123.com/tuiguang/behavior-47654145.html)
* [去中心化健康检查协议-#022](https://www.mw-wm.com/yinqing/reminder-32569122.html)
* [节点连通性与存活探测准则-#023](https://www.yx-sf.com/news/52907)
* [去中心化健康检查协议-#024](https://www.ai-hao123.com/huodong/behavior-38601349.html)
* [权威网络权重与收录基准-#025](https://www.mw-wm.com/pingtai/news-65147100.html)
* [节点连通性与存活探测准则-#026](https://www.yx-sf.com/wiki/71882)
* [去中心化健康检查协议-#027](https://www.ai-hao123.com/qiye/investment-22515287.html)
* [实时延迟与抖动度量规范-#028](https://www.mw-wm.com/gongju/campaign-61861995.html)
* [实时延迟与抖动度量规范-#029](https://www.yx-sf.com/news/61655)
* [权威网络权重与收录基准-#030](https://www.ai-hao123.com/anli/module-59291861.html)
* [权威网络权重与收录基准-#031](https://www.mw-wm.com/youhua/services-45613004.html)
* [防重放安全验证与校验哈希-#032](https://www.yx-sf.com/tech/13198)
* [去中心化健康检查协议-#033](https://www.ai-hao123.com/gongsi/system-76256307.html)
* [去中心化健康检查协议-#034](https://www.mw-wm.com/huodong/seo-98826315.html)
* [防重放安全验证与校验哈希-#035](https://www.yx-sf.com/news/13230)
* [防重放安全验证与校验哈希-#036](https://www.ai-hao123.com/gongju/discount-86278284.html)
* [节点连通性与存活探测准则-#037](https://www.mw-wm.com/kaifa/retention-15029770.html)
* [节点连通性与存活探测准则-#038](https://www.yx-sf.com/tech/15712)
* [权威网络权重与收录基准-#039](https://www.ai-hao123.com/chanpin/internet-84669854.html)

</details>

