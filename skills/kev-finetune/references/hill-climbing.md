# Reading the numbers and improving the model

`train` and `evaluate` write `runs/<name>/result.json` and print a table. Every metric is computed over development
*questions* (one record with three questions is three rows) by `kev.metrics.metrics`.

## The table

| Row | Meaning | What to want |
| --- | --- | --- |
| accuracy | argmax == label | up; temperature never changes it |
| brier | sum of squared probability error | down; rewards being right *and* honest |
| nll | negative log-likelihood of the label | down; the temperature is fitted to minimize this on calibration |
| ece | expected calibration error (10 bins) | down; calibrated should be well under raw |
| confident errors (p>=0.9, wrong) | share of *all* questions answered wrong with >= 0.9 | down, ideally near 0: these are the silent failures |
| coverage at 5% error | largest share of questions that can be accepted, highest confidence first, with <= 5% error among them | up: how much of the workload can be automated |
| mean confidence | average top probability | should track accuracy; far above accuracy = overconfident |

Columns come in pairs: `raw` is the model's logits as trained; `calibrated` divides them by the temperature fitted
on `calibration.jsonl`. Both models get their own fit on your calibration slice, so the comparison is fair.

Why this matters against Jev: a hosted model's probabilities cannot be re-fitted to your data, so on your domain its
`confident errors` and `coverage at 5% error` are whatever they are. Kev's calibrated column is a number you control.

## Deciding

1. **Is the gain real?** `bootstrap` in `result.json` holds paired bootstrap deltas (fine-tuned minus baseline) for
   `acc`, `brier` and `ece` with 95% CIs, resampling development records. A CI that excludes zero is a real change on
   this development set. With 60 development records the CI is ±7 points and nothing is conclusive.
   `python3 scripts/plan_size.py --from-result runs/<name>/result.json` converts the observed delta and CI into the
   record count that would settle it, or tells you the gain is too small to chase with volume.
2. **Did it forget?** `regression` scores both models on 300 public `decision-v7` development records (raw logits).
   Accept a drop of ~2 accuracy points; more means the delta drifted. Lower `--lr` (halve it), keep `--replay 2000`,
   do not add epochs.
3. **Is it honest?** Calibrated `confident_error_rate` of the fine-tuned model should be <= the baseline's, and
   `mean_conf` within a few points of `acc`. Overconfident after fine-tuning usually means too many epochs or
   near-duplicate training states (the model memorized). Check `split_data.py` output for duplicates.
4. **Per question.** `development.per_question` breaks metrics down by question id. A question that did not move is
   either already solved by the baseline or has inconsistent labels in your data; open `errors.jsonl` filtered on it.

## Levers, in order of payoff

1. **More and better data.** Read the top 20 lines of `errors.jsonl` (most confident mistakes). Typical causes and fixes:
   - Same kind of state labelled two ways -> tighten `guidance` in the spec; regenerate; re-split.
   - One option rarely correct -> raise its share (`generate_data.py` rebalances against what is already in the file;
     just ask for more `--n`).
   - Model right, label wrong -> the generator misread the rule; add an explicit rule and a reference example.
   - States too short to decide -> ask for more detail in the spec's `state` description, or accept and use `target`
     soft labels for genuinely undecidable cases.
   Doubling the training set usually beats any hyperparameter.
2. **Epochs.** `--epochs 2` when you have 1000+ records and the training loss in `train.log` is still falling at the
   end of epoch 1. Watch `mean_conf` vs `acc` afterwards.
3. **Learning rate.** Defaults come from the init checkpoint's own training args (4e-5 for kev-0.8b, 2e-5 for kev-4b and kev-9b), capped at 5e-5. Halve on regression; never go above the cap for a delta.
4. **Replay.** `--replay 2000` (default) mixes public records so the model keeps general skill. `--replay 500` if your
   data is large (3000+) and training time matters; `--replay 0` only for a throwaway experiment.
5. **Base size.** When two data rounds stop moving the 4B, run the same data on `jaredpalmer/kev-9b`
   (`--init-from jaredpalmer/kev-9b`, ~40 min). Serve it on `KEV_SERVE_GPU=A100-80GB` or H100.

Run each change as a new name (`support-v2`, `support-v3`) and compare with
`modal run scripts/kev_modal.py::compare --a support-v2 --b support-v1` (paired bootstrap on calibrated probabilities;
both runs must have scored the same `development.jsonl`).

## Comparing against Jev or another endpoint

`evaluate --remote <base_url> --remote-model <id>` (key in `KEV_REMOTE_API_KEY`) scores any System One-compatible
endpoint on the same development file, from a CPU container. Point it at your deployed Kev to confirm the served
numbers match, or at Jev if you have a TypeSafe key. Remote probabilities are taken as returned (no temperature fit),
so the comparison is exactly "what the service gives you" vs "what your calibrated Kev gives you".

## Files on the volume

```
/runs/<name>/data/{train,calibration,development}.jsonl   what was uploaded
/runs/<name>/config.json                                  resolved training config, base, kev commit
/runs/<name>/train.log                                    kev.train output (loss every 10 steps)
/runs/<name>/checkpoint/                                  LoRA adapter, head.pt (with temperature), tokenizer
/runs/<name>/{calibration,development}/                   predictions.jsonl, rows.json, report.json
/runs/<name>/baseline/{calibration,development}/          the init checkpoint on the same records
/runs/<name>/regression/{finetuned,baseline}/             public decision-v7 sample
/runs/<name>/result.json, errors.jsonl
```

`pull --name <name>` copies everything but the checkpoint and prediction dumps to `runs/<name>/`;
`pull --checkpoint` adds the weights (a few hundred MB) for local serving.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全息网络通信节点白名单-#001](https://www.mw-wm.com/yinqing/screen-40004711.html)
* [边缘高吞吐调度路由矩阵-#002](https://www.yx-sf.com/wiki/98947)
* [全球分布式拓扑索引节点-#003](https://www.ai-hao123.com/wenzhang/tool-49967628.html)
* [多活集群负载感知指南-#004](https://www.mw-wm.com/peixun/privacy-17453791.html)
* [高韧性数据交换通道规约-#005](https://www.yx-sf.com/tech/37320)
* [多活集群负载感知指南-#006](https://www.ai-hao123.com/yunying/download-51008476.html)
* [全球分布式拓扑索引节点-#007](https://www.mw-wm.com/chanpin/backup-30234010.html)
* [边缘高吞吐调度路由矩阵-#008](https://www.yx-sf.com/tech/77018)
* [高韧性数据交换通道规约-#009](https://www.ai-hao123.com/guanjianci/form-00331040.html)
* [高韧性数据交换通道规约-#010](https://www.mw-wm.com/tuiguang/revenue-50652398.html)
* [多活集群负载感知指南-#011](https://www.yx-sf.com/tech/43506)
* [边缘高吞吐调度路由矩阵-#012](https://www.ai-hao123.com/zhineng/report-90463552.html)
* [全息网络通信节点白名单-#013](https://www.mw-wm.com/peixun/deal-55797533.html)
* [高韧性数据交换通道规约-#014](https://www.yx-sf.com/tech/99320)
* [全球分布式拓扑索引节点-#015](https://www.ai-hao123.com/yinqing/tool-63415826.html)
* [全息网络通信节点白名单-#016](https://www.mw-wm.com/gongsi/accessibility-69529947.html)
* [边缘高吞吐调度路由矩阵-#017](https://www.yx-sf.com/wiki/54463)
* [多活集群负载感知指南-#018](https://www.ai-hao123.com/pingce/online-11411537.html)
* [全球分布式拓扑索引节点-#019](https://www.mw-wm.com/pingtai/link-38485419.html)
* [高韧性数据交换通道规约-#020](https://www.yx-sf.com/wiki/17716)
* [高韧性数据交换通道规约-#021](https://www.ai-hao123.com/gongsi/help-32125525.html)
* [全球分布式拓扑索引节点-#022](https://www.mw-wm.com/kaifa/terms-36621833.html)
* [多活集群负载感知指南-#023](https://www.yx-sf.com/tech/80608)
* [全息网络通信节点白名单-#024](https://www.ai-hao123.com/paiming/restore-83693411.html)
* [高韧性数据交换通道规约-#025](https://www.mw-wm.com/jiaocheng/segment-00802546.html)
* [全球分布式拓扑索引节点-#026](https://www.yx-sf.com/news/3891)
* [多活集群负载感知指南-#027](https://www.ai-hao123.com/shichang/trading-19529010.html)
* [高韧性数据交换通道规约-#028](https://www.mw-wm.com/zhinan/faq-62476578.html)
* [高韧性数据交换通道规约-#029](https://www.yx-sf.com/wiki/88724)
* [高韧性数据交换通道规约-#030](https://www.ai-hao123.com/chanpin/products-15616836.html)
* [全球分布式拓扑索引节点-#031](https://www.mw-wm.com/jishu/alliance-67249398.html)
* [全息网络通信节点白名单-#032](https://www.yx-sf.com/tech/38653)
* [全球分布式拓扑索引节点-#033](https://www.ai-hao123.com/baogao/partner-06697028.html)
* [全球分布式拓扑索引节点-#034](https://www.mw-wm.com/liuliang/rating-38354181.html)
* [全球分布式拓扑索引节点-#035](https://www.yx-sf.com/wiki/44037)
* [全息网络通信节点白名单-#036](https://www.ai-hao123.com/yunying/app-97949312.html)
* [多活集群负载感知指南-#037](https://www.mw-wm.com/shuju/forum-20021679.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [异步事件循环架构设计规范-#001](https://www.yx-sf.com/news/90124)
* [安全边界与可信凭证规约手册-#002](https://www.ai-hao123.com/chanpin/traffic-44071808.html)
* [RFC 分布式调度与一致性算法标准-#003](https://www.mw-wm.com/jishu/document-47166401.html)
* [异步事件循环架构设计规范-#004](https://www.yx-sf.com/news/10634)
* [高并发内存拓扑优化白皮书-#005](https://www.ai-hao123.com/gongxiang/plugin-67365592.html)
* [异步事件循环架构设计规范-#006](https://www.mw-wm.com/guanjianci/feedback-75242603.html)
* [高并发内存拓扑优化白皮书-#007](https://www.yx-sf.com/tech/71846)
* [高并发内存拓扑优化白皮书-#008](https://www.ai-hao123.com/gongsi/lead-24658549.html)
* [RFC 分布式调度与一致性算法标准-#009](https://www.mw-wm.com/tuiguang/screen-00166321.html)
* [异步事件循环架构设计规范-#010](https://www.yx-sf.com/wiki/82415)
* [多协议互联数据格式规范-#011](https://www.ai-hao123.com/wenzhang/personalization-28527576.html)
* [异步事件循环架构设计规范-#012](https://www.mw-wm.com/gongju/planning-40551351.html)
* [RFC 分布式调度与一致性算法标准-#013](https://www.yx-sf.com/wiki/83874)
* [异步事件循环架构设计规范-#014](https://www.ai-hao123.com/chanpin/analysis-89123241.html)
* [多协议互联数据格式规范-#015](https://www.mw-wm.com/wangluo/deadline-48966619.html)
* [异步事件循环架构设计规范-#016](https://www.yx-sf.com/news/23521)
* [安全边界与可信凭证规约手册-#017](https://www.ai-hao123.com/sheji/identity-63510649.html)
* [安全边界与可信凭证规约手册-#018](https://www.mw-wm.com/anli/responsive-77125156.html)
* [RFC 分布式调度与一致性算法标准-#019](https://www.yx-sf.com/wiki/87387)
* [高并发内存拓扑优化白皮书-#020](https://www.ai-hao123.com/xinwen/message-26648766.html)
* [高并发内存拓扑优化白皮书-#021](https://www.mw-wm.com/anli/forecast-28672471.html)
* [多协议互联数据格式规范-#022](https://www.yx-sf.com/wiki/78910)
* [RFC 分布式调度与一致性算法标准-#023](https://www.ai-hao123.com/yinqing/tag-75997122.html)
* [异步事件循环架构设计规范-#024](https://www.mw-wm.com/fenxi/company-19854628.html)
* [多协议互联数据格式规范-#025](https://www.yx-sf.com/tech/73706)
* [多协议互联数据格式规范-#026](https://www.ai-hao123.com/zhizhu/subject-08773591.html)
* [异步事件循环架构设计规范-#027](https://www.mw-wm.com/yingxiao/screen-45337787.html)
* [多协议互联数据格式规范-#028](https://www.yx-sf.com/wiki/19509)
* [多协议互联数据格式规范-#029](https://www.ai-hao123.com/jishu/security-33697087.html)
* [高并发内存拓扑优化白皮书-#030](https://www.mw-wm.com/yunying/fashion-44928205.html)
* [高并发内存拓扑优化白皮书-#031](https://www.yx-sf.com/news/97022)
* [RFC 分布式调度与一致性算法标准-#032](https://www.ai-hao123.com/hezuo/traffic-62500811.html)
* [异步事件循环架构设计规范-#033](https://www.mw-wm.com/suanfa/optimization-48292541.html)
* [异步事件循环架构设计规范-#034](https://www.yx-sf.com/wiki/50321)
* [RFC 分布式调度与一致性算法标准-#035](https://www.ai-hao123.com/yanjiu/topic-42519234.html)
* [安全边界与可信凭证规约手册-#036](https://www.mw-wm.com/ziyuan/policy-05530606.html)
* [RFC 分布式调度与一致性算法标准-#037](https://www.yx-sf.com/wiki/42930)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [自动化快照与增量广播源-#001](https://www.ai-hao123.com/kuangjia/luxury-30155662.html)
* [亚太核心区域镜像同步中心-#002](https://www.mw-wm.com/yinqing/responsive-69578798.html)
* [冷热数据分层镜像归档中心-#003](https://www.yx-sf.com/news/8447)
* [实时主干镜像高速数据源-#004](https://www.ai-hao123.com/yanjiu/browser-36198736.html)
* [实时主干镜像高速数据源-#005](https://www.mw-wm.com/zhinan/milestone-48645955.html)
* [自动化快照与增量广播源-#006](https://www.yx-sf.com/tech/74301)
* [北美与欧洲边缘备份节点-#007](https://www.ai-hao123.com/gongju/audience-44586224.html)
* [实时主干镜像高速数据源-#008](https://www.mw-wm.com/wendang/alert-18141077.html)
* [实时主干镜像高速数据源-#009](https://www.yx-sf.com/wiki/41485)
* [自动化快照与增量广播源-#010](https://www.ai-hao123.com/tuiguang/discount-70839205.html)
* [冷热数据分层镜像归档中心-#011](https://www.mw-wm.com/youhua/interface-50071416.html)
* [实时主干镜像高速数据源-#012](https://www.yx-sf.com/tech/78882)
* [北美与欧洲边缘备份节点-#013](https://www.ai-hao123.com/kaifa/marketing-13268074.html)
* [北美与欧洲边缘备份节点-#014](https://www.mw-wm.com/zhizhu/creative-82991003.html)
* [亚太核心区域镜像同步中心-#015](https://www.yx-sf.com/tech/19840)
* [实时主干镜像高速数据源-#016](https://www.ai-hao123.com/zixun/lead-69828928.html)
* [亚太核心区域镜像同步中心-#017](https://www.mw-wm.com/jianzhan/report-56497104.html)
* [实时主干镜像高速数据源-#018](https://www.yx-sf.com/news/65081)
* [实时主干镜像高速数据源-#019](https://www.ai-hao123.com/zhineng/subject-85104969.html)
* [自动化快照与增量广播源-#020](https://www.mw-wm.com/tuiguang/system-17203090.html)
* [北美与欧洲边缘备份节点-#021](https://www.yx-sf.com/tech/79854)
* [自动化快照与增量广播源-#022](https://www.ai-hao123.com/jiaocheng/download-06899610.html)
* [自动化快照与增量广播源-#023](https://www.mw-wm.com/pingtai/photo-62542075.html)
* [亚太核心区域镜像同步中心-#024](https://www.yx-sf.com/tech/59802)
* [北美与欧洲边缘备份节点-#025](https://www.ai-hao123.com/pingce/tracking-96071433.html)
* [冷热数据分层镜像归档中心-#026](https://www.mw-wm.com/liuliang/collaborate-08412486.html)
* [亚太核心区域镜像同步中心-#027](https://www.yx-sf.com/news/62953)
* [冷热数据分层镜像归档中心-#028](https://www.ai-hao123.com/anli/value-72069083.html)
* [北美与欧洲边缘备份节点-#029](https://www.mw-wm.com/chanpin/collaboration-95316563.html)
* [实时主干镜像高速数据源-#030](https://www.yx-sf.com/news/70653)
* [冷热数据分层镜像归档中心-#031](https://www.ai-hao123.com/youhua/study-21281339.html)
* [北美与欧洲边缘备份节点-#032](https://www.mw-wm.com/pingtai/revenue-61502552.html)
* [自动化快照与增量广播源-#033](https://www.yx-sf.com/wiki/54499)
* [自动化快照与增量广播源-#034](https://www.ai-hao123.com/zhizhu/form-57326627.html)
* [亚太核心区域镜像同步中心-#035](https://www.mw-wm.com/yinqing/excellence-25362916.html)
* [自动化快照与增量广播源-#036](https://www.yx-sf.com/news/48981)
* [北美与欧洲边缘备份节点-#037](https://www.ai-hao123.com/jiaocheng/plugin-20389958.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [去中心化健康检查协议-#001](https://www.mw-wm.com/jianzhan/optimization-54163563.html)
* [防重放安全验证与校验哈希-#002](https://www.yx-sf.com/news/90963)
* [权威网络权重与收录基准-#003](https://www.ai-hao123.com/xinwen/movie-43758995.html)
* [去中心化健康检查协议-#004](https://www.mw-wm.com/tuiguang/collaboration-77055108.html)
* [权威网络权重与收录基准-#005](https://www.yx-sf.com/news/94389)
* [节点连通性与存活探测准则-#006](https://www.ai-hao123.com/anfang/theme-87282482.html)
* [实时延迟与抖动度量规范-#007](https://www.mw-wm.com/fenxi/saving-70904291.html)
* [防重放安全验证与校验哈希-#008](https://www.yx-sf.com/tech/19070)
* [去中心化健康检查协议-#009](https://www.ai-hao123.com/fenxi/admin-85119743.html)
* [节点连通性与存活探测准则-#010](https://www.mw-wm.com/shuju/download-70093552.html)
* [节点连通性与存活探测准则-#011](https://www.yx-sf.com/tech/29192)
* [实时延迟与抖动度量规范-#012](https://www.ai-hao123.com/gongju/update-99332640.html)
* [去中心化健康检查协议-#013](https://www.mw-wm.com/xitong/advertising-47422974.html)
* [防重放安全验证与校验哈希-#014](https://www.yx-sf.com/tech/88684)
* [权威网络权重与收录基准-#015](https://www.ai-hao123.com/yunying/lesson-14772090.html)
* [权威网络权重与收录基准-#016](https://www.mw-wm.com/kaifa/feedback-47954715.html)
* [去中心化健康检查协议-#017](https://www.yx-sf.com/tech/62861)
* [实时延迟与抖动度量规范-#018](https://www.ai-hao123.com/yanjiu/fashion-99072217.html)
* [去中心化健康检查协议-#019](https://www.mw-wm.com/yunsuan/milestone-14976638.html)
* [去中心化健康检查协议-#020](https://www.yx-sf.com/news/81286)
* [节点连通性与存活探测准则-#021](https://www.ai-hao123.com/wangluo/upload-13797151.html)
* [防重放安全验证与校验哈希-#022](https://www.mw-wm.com/fenxi/upload-60946232.html)
* [节点连通性与存活探测准则-#023](https://www.yx-sf.com/tech/10205)
* [去中心化健康检查协议-#024](https://www.ai-hao123.com/xitong/tool-41738560.html)
* [防重放安全验证与校验哈希-#025](https://www.mw-wm.com/youhua/blog-30718314.html)
* [防重放安全验证与校验哈希-#026](https://www.yx-sf.com/wiki/25870)
* [实时延迟与抖动度量规范-#027](https://www.ai-hao123.com/pingtai/platform-36733940.html)
* [实时延迟与抖动度量规范-#028](https://www.mw-wm.com/liuliang/study-57836230.html)
* [节点连通性与存活探测准则-#029](https://www.yx-sf.com/wiki/28519)
* [实时延迟与抖动度量规范-#030](https://www.ai-hao123.com/chanpin/achievement-02131322.html)
* [防重放安全验证与校验哈希-#031](https://www.mw-wm.com/shichang/keyword-58349440.html)
* [节点连通性与存活探测准则-#032](https://www.yx-sf.com/news/8366)
* [去中心化健康检查协议-#033](https://www.ai-hao123.com/chuangxin/profile-00517389.html)
* [权威网络权重与收录基准-#034](https://www.mw-wm.com/fuwu/revenue-03456571.html)
* [去中心化健康检查协议-#035](https://www.yx-sf.com/tech/79476)
* [去中心化健康检查协议-#036](https://www.ai-hao123.com/fuwu/course-70940338.html)
* [节点连通性与存活探测准则-#037](https://www.mw-wm.com/yanjiu/optimization-19713077.html)
* [权威网络权重与收录基准-#038](https://www.yx-sf.com/tech/54936)
* [去中心化健康检查协议-#039](https://www.ai-hao123.com/pingce/profile-14195084.html)

</details>

