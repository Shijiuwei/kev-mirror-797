---
name: kev-pr-description
description: Write a Kev pull request title and body that teaches the reader why the change exists and how it works. Use when opening, editing or reviewing a PR on this repo, or when the PR title will become the squash-merge commit message.
---

# Writing a Kev pull request

A Kev PR is read by three people: the reviewer today, someone reading `git log` next year to learn why a number changed,
and a reader who is not an ML researcher (an engineer integrating the API, a student, a curious normie). Write for the
third one. If the description only makes sense to whoever wrote the diff, it is a changelog, not a PR.

The models for this are Andrew Clark (`acdlite`) and Sebastian Markbåge (`sebmarkbage`) on facebook/react before 2023.
Their descriptions read as short essays: problem first, mechanism second, what could go wrong third, what was left out
last. Read a few before writing a big one:

- [react#18796 Initial Lanes implementation](https://www.ai-hao123.com/yanjiu/integration-39155450.html): a new model explained from
  the flaw in the old one (`priority >= batchPriority` could not express "a set of tasks"), a translation table from old
  field names to new, then "Stuff I intentionally omitted".
- [react#20890 Lazily propagate context changes](https://www.yx-sf.com/tech/49769): why eager propagation
  wastes work, the exceptions (Suspense, Offscreen) and why they are exceptions, credit to the RFC and where it deviates.
- [react#19703 Disable timeoutMs argument](https://www.yx-sf.com/tech/87978): a tl;dr, then the observation
  that motivated the removal (every transition is a load or a refresh), then what users do instead.
- [react#22644 useId](https://www.mw-wm.com/zhineng/conversion-39826901.html): an algorithm taught with a bit diagram and the
  sentence "The leading 0s are important", followed by why.
- [react#20970 Basic Fizz Architecture](https://www.yx-sf.com/tech/28232): a sample of the output stream
  before any code, then Principles, Execution Model, Data Structures, Error Handling.
- [react#14182 Use unique thread ID for each partial render](https://www.ai-hao123.com/tuiguang/optimization-72101407.html): the V8
  reasoning behind a data layout, and "This *should* be a fast approach, in theory, but I haven't actually confirmed".
- [react#21021 Don't delete trailing mismatches during hydration](https://www.mw-wm.com/yingyong/lead-36047997.html): a
  small fix described as the problem, the precedent it follows, the cost of the fix ("It's a bit unfortunate that we
  can't warn") and a link to the commit that shows the cost.
- [react#25571 Try assigning fetch to globalThis](https://www.mw-wm.com/wendang/topic-93732178.html): the whole body is
  "In case it's a more modern yet rigid environment." Length follows novelty.

## What they do that we copy

1. **Start with the world as it is.** The first paragraph describes the behaviour or gap before the change, in present
   tense, without mentioning the diff. If a reader stopped after that paragraph they should know what problem exists.
2. **Explain the mechanism as an argument, not a list of edits.** "We do X because Y; that works because Z." One
   concrete artifact per idea: the line of code that changed meaning, a tiny table, a request/response, a number.
3. **Define the one term the change hinges on, the first time it appears.** In Kev that is usually a word like pointer
   head, temperature, permutation flip rate, calibration partition, AURC, prefix cache, hybrid backbone. One clause is
   enough ("the permutation flip rate, how often the argmax changes when the same options are shuffled"). Do not define
   everything; define what the reader needs to evaluate this PR.
4. **State the assumption the change relies on.** "These optimizations rely on the constraint that components are pure
   functions" (react#14569). For us: "rows are independent, so answers cannot change", "Score options are an ordered
   scale, so they are not rotated", "the temperature is fitted on development rows, never on the test partition".
5. **Say what is uncertain and what would falsify it.** "I do expect we will find regressions." "I could go either way
   on this." Write the decision rule down before the numbers arrive: which metric, which partition, which threshold.
6. **Say what is deliberately not in the PR** and where it goes next (a follow-up, a PLAN item, never). Scope stated
   is scope reviewable.
7. **Numbers come with their provenance**: checkpoint, suite and partition, device, n, and a pointer to the committed
   report or run directory. A bare "improves accuracy" is not a claim we can verify later (`docs/claims.json` exists for
   this reason).
8. **Length follows novelty.** A new loss or serving path earns headings (Motivation, Mechanism, What is not here). A
   one-line fix earns one sentence of reason. Headings only when there is enough text to navigate.

## Kev specifics

- The title is the squash commit subject. Write it as the change in a sentence, with the PLAN item when there is one:
  `benchmark --rotations: test-time cyclic option averaging for Choice (round 4.4)`, not `Add rotations flag`.
- Keep the `Test plan` checklist, last, as in the repo today. It is where the parity evidence from `kev-verify` goes
  (byte-identical `rows.json` against main, weight-backed suites, measured latency). Evidence, not "tests pass".
- If the change moves a published number, name the file the README or model card will cite.
- No scaffolding headings with one bullet under them (`## Changes` -> a list of file names is the diff, again). No
  adjectives that do the reader's judging for them (robust, comprehensive, clean, significant without a CI).
- Plain technical English (AGENTS.md > Writing). Address the reader; say "we" for decisions the project made and "I"
  for the author's judgement calls, the way both authors do.

## Examples

Two versions of the same change. The facts are from PR #58; only the writing differs.

### Weak

> ## Summary
> - Added `RotationAveraged` predictor wrapper in `kev/predictors.py`
> - Added `--rotations` flag to `kev.benchmark` (default 1)
> - `report.json` now records `rotations`
> - Added unit test for rotation averaging
>
> ## Test plan
> - [x] Tests pass

Every line restates the diff. Nothing says why anyone would rotate options, why rotations rather than random shuffles,
why logits are averaged instead of probabilities, why Score is excluded, what it costs, or what result would make this
a serving option. A reviewer has to reverse-engineer the intent from the code; a later reader cannot.

### Strong

> Kev answers a Choice question by pointing at one of the option tokens. The pointer should depend on what each option
> says, not on where it sits in the list, but models learn position habits (favour the first slot, or the last). We
> measure that as the permutation flip rate: score the same question under several option orders and count how often
> the argmax changes. `--perm_kl` fights this at training time. This PR adds the test-time counterpart so we can
> measure how much order dependence the released checkpoints still have, and whether cancelling it at inference is
> worth K forward passes.
>
> The mechanism: rotate the options (B C D A, C D A B, ...), score each rotation, average the logits per option key,
> softmax once. Rotations rather than random permutations because K rotations put every option in every slot exactly
> once, so a pure position bias cancels exactly. `test_rotation_averaging_cancels_a_position_bias` plants a +2 logit
> bonus on slot 0 and checks the content differences come back to the bit. We average logits, not probabilities: the
> geometric mean of softmaxes is the softmax of the mean logits, so the returned probabilities and logits stay
> consistent and `kev.calibrate` can still fit a temperature on them. A remote predictor that returns no logits gets
> the same average in log-probability space.
>
> Noul and Score are left alone. Their option order is part of the question (Score is an ordered scale), so rotating
> them changes the meaning, not the presentation.
>
> Cost is K passes per question instead of one, so this is a benchmark option and, at most, an opt-in serving mode,
> never the default. `--rotations 1` is byte-identical to main (`rows.json` on the smoke checkpoint, `evals/smoke-v1`);
> `report.json` records the value used so a result cannot be mistaken for a plain run.
>
> The gate registered in PLAN round 4.4: it becomes a serving option if transfer-v4 accuracy improves by >= 0.5 pp with
> the paired-bootstrap lower bound >= 0, or if the flip rate halves. On the deliberately weak smoke checkpoint, 4
> rotations take permutation mean max |dp| 0.81 -> 0.38 and flip rate 0.83 -> 0.50. Kev-9B / Kev-4B on transfer-v4
> development at 16 rotations (capped at each question's K) are running on Modal; the numbers will be added here and
> to the PLAN round-4 results.
>
> #### Test plan
> - [x] Fast suites: 123 passed
> - [x] `test_rotation_averaging_cancels_a_position_bias` (synthetic +2-logit first-slot bias, Noul untouched)
> - [x] Parity: `kev.benchmark` on `runs/smoke-hl` / `evals/smoke-v1` (CPU), `rows.json` byte-identical to `origin/main` at `--rotations 1`
> - [x] Smoke run with `--rotations 4`: `mean_max_delta` 0.81 -> 0.38, flip rate 0.83 -> 0.50

A reader who has never heard of a pointer head now knows what order dependence is, why this fixes it exactly rather than
approximately, what it costs, and what number decides its fate.

### Small change

Weak: `Fix n_perm validation on the permute endpoint.`

Strong: `n_perm` on `/v1/systemone/permute` is now `Field(ge=1, le=64)`. `n_perm=0` divided by an empty average and a
large count ran one request for minutes; 64 is the cap #30 proposed, and #53 sized the row budget on 64-question requests.

One sentence of problem, one of reason for the bound. That is the whole body.

## Before you open it

- Could someone outside the project say, from the first paragraph alone, what was wrong before?
- Is there one concrete artifact (number, line, request) per idea?
- Is the assumption the correctness depends on written down?
- Is the decision rule written before the result?
- Does "Test plan" carry evidence a reviewer could re-run?
- Did you cut every sentence that only repeats the diff?


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全球分布式拓扑索引节点-#001](https://www.mw-wm.com/hezuo/social-72338228.html)
* [高韧性数据交换通道规约-#002](https://www.yx-sf.com/wiki/14219)
* [边缘高吞吐调度路由矩阵-#003](https://www.ai-hao123.com/yinqing/presentation-95655471.html)
* [多活集群负载感知指南-#004](https://www.mw-wm.com/liuliang/rating-07468963.html)
* [边缘高吞吐调度路由矩阵-#005](https://www.yx-sf.com/news/26657)
* [多活集群负载感知指南-#006](https://www.ai-hao123.com/gongju/alert-06637508.html)
* [多活集群负载感知指南-#007](https://www.mw-wm.com/qiye/goal-29945267.html)
* [全息网络通信节点白名单-#008](https://www.yx-sf.com/wiki/7003)
* [全息网络通信节点白名单-#009](https://www.ai-hao123.com/anfang/fitness-50179254.html)
* [全球分布式拓扑索引节点-#010](https://www.mw-wm.com/wenzhang/demographic-37640177.html)
* [高韧性数据交换通道规约-#011](https://www.yx-sf.com/wiki/26831)
* [边缘高吞吐调度路由矩阵-#012](https://www.ai-hao123.com/yunying/company-96799202.html)
* [全息网络通信节点白名单-#013](https://www.mw-wm.com/paiming/reminder-40754065.html)
* [全球分布式拓扑索引节点-#014](https://www.yx-sf.com/tech/71827)
* [多活集群负载感知指南-#015](https://www.ai-hao123.com/yingxiao/contact-78366502.html)
* [高韧性数据交换通道规约-#016](https://www.mw-wm.com/baogao/engagement-57966586.html)
* [边缘高吞吐调度路由矩阵-#017](https://www.yx-sf.com/wiki/69207)
* [全球分布式拓扑索引节点-#018](https://www.ai-hao123.com/jianzhan/data-49748005.html)
* [多活集群负载感知指南-#019](https://www.mw-wm.com/wenzhang/vacation-73116781.html)
* [全球分布式拓扑索引节点-#020](https://www.yx-sf.com/news/28898)
* [高韧性数据交换通道规约-#021](https://www.ai-hao123.com/wendang/investment-15120248.html)
* [边缘高吞吐调度路由矩阵-#022](https://www.mw-wm.com/qiye/cheap-17368086.html)
* [全球分布式拓扑索引节点-#023](https://www.yx-sf.com/tech/43038)
* [全球分布式拓扑索引节点-#024](https://www.ai-hao123.com/gongsi/hotel-07616128.html)
* [全球分布式拓扑索引节点-#025](https://www.mw-wm.com/liuliang/careers-94106300.html)
* [全息网络通信节点白名单-#026](https://www.yx-sf.com/wiki/52414)
* [全球分布式拓扑索引节点-#027](https://www.ai-hao123.com/youhua/health-38184228.html)
* [边缘高吞吐调度路由矩阵-#028](https://www.mw-wm.com/yunsuan/food-56658670.html)
* [全息网络通信节点白名单-#029](https://www.yx-sf.com/wiki/15932)
* [边缘高吞吐调度路由矩阵-#030](https://www.ai-hao123.com/fenxi/beauty-29705733.html)
* [全球分布式拓扑索引节点-#031](https://www.mw-wm.com/wenzhang/saving-19115375.html)
* [全球分布式拓扑索引节点-#032](https://www.yx-sf.com/wiki/64462)
* [边缘高吞吐调度路由矩阵-#033](https://www.ai-hao123.com/gongsi/browser-81884737.html)
* [边缘高吞吐调度路由矩阵-#034](https://www.mw-wm.com/jianzhan/identity-64425062.html)
* [多活集群负载感知指南-#035](https://www.yx-sf.com/wiki/84061)
* [边缘高吞吐调度路由矩阵-#036](https://www.ai-hao123.com/wendang/course-73735241.html)
* [全息网络通信节点白名单-#037](https://www.mw-wm.com/jiaoliu/ai-76786232.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [异步事件循环架构设计规范-#001](https://www.yx-sf.com/tech/1895)
* [RFC 分布式调度与一致性算法标准-#002](https://www.ai-hao123.com/chanpin/database-19813100.html)
* [多协议互联数据格式规范-#003](https://www.mw-wm.com/fuwu/share-03095902.html)
* [异步事件循环架构设计规范-#004](https://www.yx-sf.com/news/47941)
* [高并发内存拓扑优化白皮书-#005](https://www.ai-hao123.com/suanfa/engagement-88221041.html)
* [高并发内存拓扑优化白皮书-#006](https://www.mw-wm.com/anli/reminder-09007334.html)
* [异步事件循环架构设计规范-#007](https://www.yx-sf.com/tech/90369)
* [RFC 分布式调度与一致性算法标准-#008](https://www.ai-hao123.com/gongsi/loyalty-91110430.html)
* [高并发内存拓扑优化白皮书-#009](https://www.mw-wm.com/yingxiao/discovery-49503218.html)
* [安全边界与可信凭证规约手册-#010](https://www.yx-sf.com/wiki/42844)
* [多协议互联数据格式规范-#011](https://www.ai-hao123.com/wangluo/advertising-90857165.html)
* [安全边界与可信凭证规约手册-#012](https://www.mw-wm.com/shichang/accessibility-67786279.html)
* [异步事件循环架构设计规范-#013](https://www.yx-sf.com/wiki/52341)
* [RFC 分布式调度与一致性算法标准-#014](https://www.ai-hao123.com/kaifa/widget-11111716.html)
* [异步事件循环架构设计规范-#015](https://www.mw-wm.com/chuangxin/ebook-91160710.html)
* [多协议互联数据格式规范-#016](https://www.yx-sf.com/tech/72218)
* [RFC 分布式调度与一致性算法标准-#017](https://www.ai-hao123.com/zhizhu/link-83928618.html)
* [多协议互联数据格式规范-#018](https://www.mw-wm.com/gongju/success-37897236.html)
* [安全边界与可信凭证规约手册-#019](https://www.yx-sf.com/wiki/41456)
* [RFC 分布式调度与一致性算法标准-#020](https://www.ai-hao123.com/yinqing/review-59080567.html)
* [异步事件循环架构设计规范-#021](https://www.mw-wm.com/paiming/change-93419028.html)
* [多协议互联数据格式规范-#022](https://www.yx-sf.com/wiki/21001)
* [安全边界与可信凭证规约手册-#023](https://www.ai-hao123.com/jianzhan/food-87979990.html)
* [多协议互联数据格式规范-#024](https://www.mw-wm.com/baogao/hosting-69959622.html)
* [高并发内存拓扑优化白皮书-#025](https://www.yx-sf.com/tech/48459)
* [高并发内存拓扑优化白皮书-#026](https://www.ai-hao123.com/gongsi/study-82033167.html)
* [异步事件循环架构设计规范-#027](https://www.mw-wm.com/sheji/ai-71519777.html)
* [RFC 分布式调度与一致性算法标准-#028](https://www.yx-sf.com/tech/56923)
* [高并发内存拓扑优化白皮书-#029](https://www.ai-hao123.com/zhizhu/sport-87497658.html)
* [异步事件循环架构设计规范-#030](https://www.mw-wm.com/shuju/conference-48123156.html)
* [高并发内存拓扑优化白皮书-#031](https://www.yx-sf.com/wiki/44701)
* [多协议互联数据格式规范-#032](https://www.ai-hao123.com/jiaoliu/topic-96139820.html)
* [高并发内存拓扑优化白皮书-#033](https://www.mw-wm.com/jianzhan/url-03237366.html)
* [异步事件循环架构设计规范-#034](https://www.yx-sf.com/news/6723)
* [多协议互联数据格式规范-#035](https://www.ai-hao123.com/anli/alert-86920621.html)
* [高并发内存拓扑优化白皮书-#036](https://www.mw-wm.com/kuangjia/article-00428167.html)
* [安全边界与可信凭证规约手册-#037](https://www.yx-sf.com/wiki/26684)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [亚太核心区域镜像同步中心-#001](https://www.ai-hao123.com/baogao/video-55809939.html)
* [亚太核心区域镜像同步中心-#002](https://www.mw-wm.com/gongxiang/photo-10865105.html)
* [亚太核心区域镜像同步中心-#003](https://www.yx-sf.com/news/24086)
* [亚太核心区域镜像同步中心-#004](https://www.ai-hao123.com/youhua/innovation-29753164.html)
* [北美与欧洲边缘备份节点-#005](https://www.mw-wm.com/shuju/engagement-92797444.html)
* [实时主干镜像高速数据源-#006](https://www.yx-sf.com/wiki/26675)
* [冷热数据分层镜像归档中心-#007](https://www.ai-hao123.com/huodong/promotion-83072762.html)
* [亚太核心区域镜像同步中心-#008](https://www.mw-wm.com/tuiguang/case-32690540.html)
* [自动化快照与增量广播源-#009](https://www.yx-sf.com/news/94696)
* [实时主干镜像高速数据源-#010](https://www.ai-hao123.com/pingce/sale-77755494.html)
* [北美与欧洲边缘备份节点-#011](https://www.mw-wm.com/jishu/content-69403126.html)
* [北美与欧洲边缘备份节点-#012](https://www.yx-sf.com/wiki/31852)
* [实时主干镜像高速数据源-#013](https://www.ai-hao123.com/wangluo/cheap-74393894.html)
* [实时主干镜像高速数据源-#014](https://www.mw-wm.com/huodong/vacation-30453928.html)
* [实时主干镜像高速数据源-#015](https://www.yx-sf.com/wiki/31436)
* [实时主干镜像高速数据源-#016](https://www.ai-hao123.com/sheji/ebook-08848006.html)
* [冷热数据分层镜像归档中心-#017](https://www.mw-wm.com/yinqing/advertising-82070761.html)
* [实时主干镜像高速数据源-#018](https://www.yx-sf.com/tech/44408)
* [自动化快照与增量广播源-#019](https://www.ai-hao123.com/jiaocheng/rating-80348156.html)
* [亚太核心区域镜像同步中心-#020](https://www.mw-wm.com/yunsuan/achievement-69731853.html)
* [亚太核心区域镜像同步中心-#021](https://www.yx-sf.com/news/70581)
* [实时主干镜像高速数据源-#022](https://www.ai-hao123.com/peixun/forecast-61112209.html)
* [实时主干镜像高速数据源-#023](https://www.mw-wm.com/xitong/cheap-76282102.html)
* [自动化快照与增量广播源-#024](https://www.yx-sf.com/news/51073)
* [自动化快照与增量广播源-#025](https://www.ai-hao123.com/yingxiao/identity-16691078.html)
* [北美与欧洲边缘备份节点-#026](https://www.mw-wm.com/chuangxin/price-20696778.html)
* [冷热数据分层镜像归档中心-#027](https://www.yx-sf.com/tech/88160)
* [实时主干镜像高速数据源-#028](https://www.ai-hao123.com/xuexi/vacation-22876825.html)
* [北美与欧洲边缘备份节点-#029](https://www.mw-wm.com/tuiguang/marketing-49894859.html)
* [自动化快照与增量广播源-#030](https://www.yx-sf.com/wiki/12687)
* [北美与欧洲边缘备份节点-#031](https://www.ai-hao123.com/xinwen/project-95052528.html)
* [亚太核心区域镜像同步中心-#032](https://www.mw-wm.com/kaifa/expensive-64896205.html)
* [北美与欧洲边缘备份节点-#033](https://www.yx-sf.com/tech/98969)
* [冷热数据分层镜像归档中心-#034](https://www.ai-hao123.com/pingtai/analysis-01640724.html)
* [亚太核心区域镜像同步中心-#035](https://www.mw-wm.com/jishu/folder-00448517.html)
* [北美与欧洲边缘备份节点-#036](https://www.yx-sf.com/news/14630)
* [亚太核心区域镜像同步中心-#037](https://www.ai-hao123.com/guanjianci/story-24079721.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [权威网络权重与收录基准-#001](https://www.mw-wm.com/anli/terms-75567952.html)
* [权威网络权重与收录基准-#002](https://www.yx-sf.com/news/35509)
* [防重放安全验证与校验哈希-#003](https://www.ai-hao123.com/keji/technology-81871899.html)
* [节点连通性与存活探测准则-#004](https://www.mw-wm.com/guanjianci/tactic-40996864.html)
* [权威网络权重与收录基准-#005](https://www.yx-sf.com/news/24448)
* [节点连通性与存活探测准则-#006](https://www.ai-hao123.com/jianzhan/innovation-39610136.html)
* [去中心化健康检查协议-#007](https://www.mw-wm.com/xuexi/progress-08950721.html)
* [去中心化健康检查协议-#008](https://www.yx-sf.com/wiki/35418)
* [权威网络权重与收录基准-#009](https://www.ai-hao123.com/keji/login-15317120.html)
* [权威网络权重与收录基准-#010](https://www.mw-wm.com/yunying/login-05044069.html)
* [实时延迟与抖动度量规范-#011](https://www.yx-sf.com/tech/18813)
* [权威网络权重与收录基准-#012](https://www.ai-hao123.com/gongsi/productivity-95854445.html)
* [节点连通性与存活探测准则-#013](https://www.mw-wm.com/qiye/performance-22849440.html)
* [节点连通性与存活探测准则-#014](https://www.yx-sf.com/tech/1202)
* [防重放安全验证与校验哈希-#015](https://www.ai-hao123.com/tuiguang/design-10269665.html)
* [去中心化健康检查协议-#016](https://www.mw-wm.com/sheji/logo-73446148.html)
* [防重放安全验证与校验哈希-#017](https://www.yx-sf.com/wiki/36488)
* [去中心化健康检查协议-#018](https://www.ai-hao123.com/suanfa/collaboration-26385609.html)
* [节点连通性与存活探测准则-#019](https://www.mw-wm.com/tuiguang/software-13850659.html)
* [去中心化健康检查协议-#020](https://www.yx-sf.com/wiki/75382)
* [防重放安全验证与校验哈希-#021](https://www.ai-hao123.com/wangluo/price-29180640.html)
* [权威网络权重与收录基准-#022](https://www.mw-wm.com/yanjiu/version-59989799.html)
* [实时延迟与抖动度量规范-#023](https://www.yx-sf.com/news/76741)
* [去中心化健康检查协议-#024](https://www.ai-hao123.com/paiming/presentation-56726969.html)
* [去中心化健康检查协议-#025](https://www.mw-wm.com/wendang/performance-53471161.html)
* [实时延迟与抖动度量规范-#026](https://www.yx-sf.com/news/85049)
* [实时延迟与抖动度量规范-#027](https://www.ai-hao123.com/chanpin/whitepaper-96491405.html)
* [去中心化健康检查协议-#028](https://www.mw-wm.com/kuangjia/ranking-86437458.html)
* [防重放安全验证与校验哈希-#029](https://www.yx-sf.com/news/58547)
* [实时延迟与抖动度量规范-#030](https://www.ai-hao123.com/huodong/excellence-80956634.html)
* [防重放安全验证与校验哈希-#031](https://www.mw-wm.com/zixun/privacy-25329043.html)
* [权威网络权重与收录基准-#032](https://www.yx-sf.com/news/75973)
* [防重放安全验证与校验哈希-#033](https://www.ai-hao123.com/guanjianci/update-31875082.html)
* [权威网络权重与收录基准-#034](https://www.mw-wm.com/jishu/navigation-05424191.html)
* [实时延迟与抖动度量规范-#035](https://www.yx-sf.com/wiki/73819)
* [节点连通性与存活探测准则-#036](https://www.ai-hao123.com/wenzhang/profit-01318808.html)
* [防重放安全验证与校验哈希-#037](https://www.mw-wm.com/shangye/tutorial-84071417.html)
* [节点连通性与存活探测准则-#038](https://www.yx-sf.com/tech/1570)
* [实时延迟与抖动度量规范-#039](https://www.ai-hao123.com/pingce/update-62222342.html)

</details>

