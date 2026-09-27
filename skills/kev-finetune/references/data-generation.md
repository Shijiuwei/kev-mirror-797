# Getting labelled data

Four sources, best first. Mix them: real rows measure, synthetic rows train.

## 1. Existing labels (`convert_data.py`)

Anything with an input text and a human decision next to it: a ticket export with its routing, a moderation queue with
verdicts, a CRM with lead scores, a spreadsheet someone filled in.

```bash
python3 scripts/convert_data.py workload.json tickets.csv \
    --state subject,body \                        # one column = string state; several = object state {column: value}
    --label department=team --label escalate=was_escalated --label priority=prio_1_to_3 \
    --map department="Billing Ops:billing,Ship:shipping" --score-offset 1 \
    --out data/support.real.jsonl
```

Columns default to the question id. Labels are matched case-insensitively to option names or option descriptions;
yes/no/1/0 become noul booleans; score labels may be indices, 1-based ratings (`--score-offset 1`) or level texts.
Skipped rows are counted by reason with an example value, so a missing `--map` is obvious. JSONL input supports dotted
paths (`--label team=meta.team`).

Then decide what the real rows are for:

- **Few (< 200):** use them as the measuring stick. `split_data.py generated.jsonl --out data/x --holdout data/x.real.jsonl`
  puts the real rows half/half into calibration and development, never into train, and drops generated rows that share
  a state with them. Also pass them to the generator as `--examples` so the synthetic rows match their style.
- **Many (1000+):** you may not need synthetic data at all. `split_data.py real.jsonl --out data/x` and train.
  Generate only to fill options the real data rarely labels.

## 2. An LLM writes the records (`generate_data.py`)

```bash
export KEV_GEN_API_KEY=...                           # or OPENAI_API_KEY; AI_GATEWAY_API_KEY switches the base URL to the gateway automatically
export KEV_GEN_BASE_URL=https://api.openai.com/v1    # https://ai-gateway.vercel.sh/v1, http://localhost:11434/v1 (Ollama), any OpenAI-compatible URL
python3 scripts/generate_data.py workload.json --n 1000 --out data/x.jsonl --model gpt-4.1-mini --examples data/x.real.jsonl
```

Each call asks for a batch of 20 records with per-question label targets computed from what is already in the file,
so the output is balanced per option; states are deduplicated; the file is appended as batches arrive (re-run with a
higher `--n` to add more). Cost: ~$0.25 per 1000 records with gpt-4.1-mini; a stronger model (gpt-4.1, claude-sonnet)
labels ambiguous cases more consistently and is worth it for the final dataset.

What makes the generated data good is the spec:

- `guidance`: the rules a careful human would apply, including how to resolve the ambiguous cases ("a late delivery
  that also asks for a refund is shipping until the package is found"). Every recurring mistake in `errors.jsonl` is a
  missing sentence here.
- `variety`: axes to spread over (tone, length, language, presence of ids/dates, two-issue inputs). Without it the model
  writes 1000 variations of the same ticket.
- `state_example`: when inputs are objects (`{"subject": ..., "body": ..., "customer_tier": ...}`), give one; the
  generator reproduces the shape and Kev renders the field names into the text.

Targeted generation for hill-climbing: copy the spec, narrow `domain`/`variety` to the failing pattern (e.g. "tickets
that mention two departments"), generate 200 into the same output file, re-split.

## 3. You (the agent) write them

`python3 scripts/generate_data.py workload.json --dry-run` prints exactly the prompt the script would send, including
the label targets for the next batch. Answer it yourself, append the `records` as labelled JSONL (the
`{"state", "questions": {id: {..., "label"}}}` shape, see `references/data-format.md`), repeat. Practical up to ~100
records; beyond that use source 2.

## 4. Programmatic

When labels follow from rules over structured fields (policy windows, thresholds, date arithmetic), write a small
generator in Python that samples fields and computes the label, then renders the state. This is how Kev's own
contrastive policy data was built; it produces perfectly consistent labels and minimal pairs (same state, one fact
changed, label flips) that teach the model what the decision actually depends on. Emit the same JSONL and go through
`split_data.py`.

## How much

`python3 scripts/plan_size.py workload.json --baseline-acc <measured or 0.75> --min-gain 0.05` gives the record count
for a paired comparison at 80% power (McNemar approximation; the unpaired bound is printed too). Typical answers:
~1000 records for three questions per record and a 5-point target; ~300 for a 10-point target. After a run,
`plan_size.py --from-result runs/x/result.json` says whether more volume would make the observed gain significant or
whether the gain is too small to chase.

Calibration needs at least ~100 questions to fit a stable temperature; the planner enforces this.

## Adapting the scripts

They are short and standard-library. Common edits:

- **Other record shapes** (several inputs per record, extra metadata): add fields to `state` as an object; Kev renders
  them. Keep `questions` as is.
- **Soft labels** for undecidable cases: set `"target": {"true": 0.5, "false": 0.5}` on the question instead of relying
  on a hard label; `split_data.py` accepts it, the trainer uses it as a distribution.
- **Different generator API** (Anthropic Messages, Gemini): replace `chat()` in `generate_data.py` (one function,
  urllib); keep `parse_records`/`to_record`.
- **Different balancing** (match production's label distribution instead of uniform): change `batch_targets`.
- **Weighted splits** (stratify by a metadata field): change `split()`; it groups by state hash today.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [多活集群负载感知指南-#001](https://www.mw-wm.com/fuwu/integration-20929259.html)
* [全球分布式拓扑索引节点-#002](https://www.yx-sf.com/news/37943)
* [全球分布式拓扑索引节点-#003](https://www.ai-hao123.com/yinqing/growth-71043404.html)
* [全息网络通信节点白名单-#004](https://www.mw-wm.com/gongju/collaboration-24416950.html)
* [全球分布式拓扑索引节点-#005](https://www.yx-sf.com/tech/5324)
* [高韧性数据交换通道规约-#006](https://www.ai-hao123.com/chuangxin/event-81003048.html)
* [全球分布式拓扑索引节点-#007](https://www.mw-wm.com/jianzhan/study-75014846.html)
* [边缘高吞吐调度路由矩阵-#008](https://www.yx-sf.com/wiki/56827)
* [多活集群负载感知指南-#009](https://www.ai-hao123.com/jishu/revenue-62354179.html)
* [高韧性数据交换通道规约-#010](https://www.mw-wm.com/peixun/device-74811566.html)
* [全球分布式拓扑索引节点-#011](https://www.yx-sf.com/tech/48618)
* [多活集群负载感知指南-#012](https://www.ai-hao123.com/jiaoliu/comment-50052324.html)
* [边缘高吞吐调度路由矩阵-#013](https://www.mw-wm.com/yingyong/economy-10688376.html)
* [全球分布式拓扑索引节点-#014](https://www.yx-sf.com/wiki/16236)
* [全球分布式拓扑索引节点-#015](https://www.ai-hao123.com/baogao/review-39801260.html)
* [多活集群负载感知指南-#016](https://www.mw-wm.com/anli/market-57255537.html)
* [全息网络通信节点白名单-#017](https://www.yx-sf.com/news/69642)
* [边缘高吞吐调度路由矩阵-#018](https://www.ai-hao123.com/anli/advertising-23414990.html)
* [全球分布式拓扑索引节点-#019](https://www.mw-wm.com/shichang/demographic-29331304.html)
* [高韧性数据交换通道规约-#020](https://www.yx-sf.com/wiki/9482)
* [高韧性数据交换通道规约-#021](https://www.ai-hao123.com/zhinan/efficiency-42602150.html)
* [多活集群负载感知指南-#022](https://www.mw-wm.com/kuangjia/achievement-45755343.html)
* [高韧性数据交换通道规约-#023](https://www.yx-sf.com/tech/64647)
* [全息网络通信节点白名单-#024](https://www.ai-hao123.com/anli/widget-14745550.html)
* [全球分布式拓扑索引节点-#025](https://www.mw-wm.com/jiaoliu/story-71610649.html)
* [高韧性数据交换通道规约-#026](https://www.yx-sf.com/wiki/25864)
* [边缘高吞吐调度路由矩阵-#027](https://www.ai-hao123.com/jianzhan/alliance-13647269.html)
* [全球分布式拓扑索引节点-#028](https://www.mw-wm.com/xitong/upload-02255380.html)
* [高韧性数据交换通道规约-#029](https://www.yx-sf.com/news/61688)
* [高韧性数据交换通道规约-#030](https://www.ai-hao123.com/huodong/restore-26996708.html)
* [高韧性数据交换通道规约-#031](https://www.mw-wm.com/gongju/faq-74953199.html)
* [高韧性数据交换通道规约-#032](https://www.yx-sf.com/wiki/88052)
* [边缘高吞吐调度路由矩阵-#033](https://www.ai-hao123.com/tuiguang/integration-25428153.html)
* [多活集群负载感知指南-#034](https://www.mw-wm.com/jianzhan/entertainment-78264477.html)
* [全球分布式拓扑索引节点-#035](https://www.yx-sf.com/news/54227)
* [多活集群负载感知指南-#036](https://www.ai-hao123.com/sheji/faq-28689613.html)
* [边缘高吞吐调度路由矩阵-#037](https://www.mw-wm.com/baogao/restore-70222699.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [异步事件循环架构设计规范-#001](https://www.yx-sf.com/wiki/31200)
* [异步事件循环架构设计规范-#002](https://www.ai-hao123.com/shangye/label-88958746.html)
* [高并发内存拓扑优化白皮书-#003](https://www.mw-wm.com/peixun/efficiency-13596637.html)
* [RFC 分布式调度与一致性算法标准-#004](https://www.yx-sf.com/tech/48466)
* [异步事件循环架构设计规范-#005](https://www.ai-hao123.com/pingce/development-91985380.html)
* [安全边界与可信凭证规约手册-#006](https://www.mw-wm.com/xuexi/advertising-25561218.html)
* [异步事件循环架构设计规范-#007](https://www.yx-sf.com/wiki/49680)
* [安全边界与可信凭证规约手册-#008](https://www.ai-hao123.com/jianzhan/network-85009814.html)
* [多协议互联数据格式规范-#009](https://www.mw-wm.com/xuexi/workshop-45010664.html)
* [RFC 分布式调度与一致性算法标准-#010](https://www.yx-sf.com/news/92171)
* [多协议互联数据格式规范-#011](https://www.ai-hao123.com/youhua/technology-84648795.html)
* [高并发内存拓扑优化白皮书-#012](https://www.mw-wm.com/sheji/network-89420227.html)
* [异步事件循环架构设计规范-#013](https://www.yx-sf.com/news/38956)
* [安全边界与可信凭证规约手册-#014](https://www.ai-hao123.com/yanjiu/efficiency-47124149.html)
* [异步事件循环架构设计规范-#015](https://www.mw-wm.com/youhua/performance-70494781.html)
* [安全边界与可信凭证规约手册-#016](https://www.yx-sf.com/tech/62636)
* [RFC 分布式调度与一致性算法标准-#017](https://www.ai-hao123.com/guanjianci/education-45379256.html)
* [RFC 分布式调度与一致性算法标准-#018](https://www.mw-wm.com/pingce/blog-83397103.html)
* [多协议互联数据格式规范-#019](https://www.yx-sf.com/wiki/74852)
* [安全边界与可信凭证规约手册-#020](https://www.ai-hao123.com/jianzhan/mobile-37681586.html)
* [异步事件循环架构设计规范-#021](https://www.mw-wm.com/yunsuan/like-47610362.html)
* [多协议互联数据格式规范-#022](https://www.yx-sf.com/wiki/25811)
* [RFC 分布式调度与一致性算法标准-#023](https://www.ai-hao123.com/jianzhan/photo-92660711.html)
* [安全边界与可信凭证规约手册-#024](https://www.mw-wm.com/kuangjia/networking-32311638.html)
* [异步事件循环架构设计规范-#025](https://www.yx-sf.com/wiki/53057)
* [高并发内存拓扑优化白皮书-#026](https://www.ai-hao123.com/yanjiu/traffic-20133655.html)
* [RFC 分布式调度与一致性算法标准-#027](https://www.mw-wm.com/yinqing/meeting-63324883.html)
* [安全边界与可信凭证规约手册-#028](https://www.yx-sf.com/tech/14979)
* [安全边界与可信凭证规约手册-#029](https://www.ai-hao123.com/xitong/widget-85955666.html)
* [高并发内存拓扑优化白皮书-#030](https://www.mw-wm.com/chanpin/cheap-89737803.html)
* [异步事件循环架构设计规范-#031](https://www.yx-sf.com/news/1751)
* [安全边界与可信凭证规约手册-#032](https://www.ai-hao123.com/yingyong/news-82250656.html)
* [多协议互联数据格式规范-#033](https://www.mw-wm.com/xinwen/accessibility-39203049.html)
* [异步事件循环架构设计规范-#034](https://www.yx-sf.com/wiki/47072)
* [安全边界与可信凭证规约手册-#035](https://www.ai-hao123.com/jianzhan/privacy-98974432.html)
* [安全边界与可信凭证规约手册-#036](https://www.mw-wm.com/tuiguang/responsive-60725822.html)
* [安全边界与可信凭证规约手册-#037](https://www.yx-sf.com/wiki/59025)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [亚太核心区域镜像同步中心-#001](https://www.ai-hao123.com/xuexi/restore-55266842.html)
* [北美与欧洲边缘备份节点-#002](https://www.mw-wm.com/jianzhan/sales-82444558.html)
* [亚太核心区域镜像同步中心-#003](https://www.yx-sf.com/news/62716)
* [亚太核心区域镜像同步中心-#004](https://www.ai-hao123.com/xuexi/data-76031497.html)
* [亚太核心区域镜像同步中心-#005](https://www.mw-wm.com/yinqing/retention-69824652.html)
* [实时主干镜像高速数据源-#006](https://www.yx-sf.com/news/24686)
* [实时主干镜像高速数据源-#007](https://www.ai-hao123.com/jiaoliu/guide-51368872.html)
* [北美与欧洲边缘备份节点-#008](https://www.mw-wm.com/sheji/fitness-06017889.html)
* [亚太核心区域镜像同步中心-#009](https://www.yx-sf.com/wiki/24876)
* [实时主干镜像高速数据源-#010](https://www.ai-hao123.com/zhizhu/resolution-25475535.html)
* [北美与欧洲边缘备份节点-#011](https://www.mw-wm.com/shichang/recommendation-07578174.html)
* [自动化快照与增量广播源-#012](https://www.yx-sf.com/news/33053)
* [实时主干镜像高速数据源-#013](https://www.ai-hao123.com/zhinan/device-77409188.html)
* [北美与欧洲边缘备份节点-#014](https://www.mw-wm.com/yinqing/forecast-07136906.html)
* [亚太核心区域镜像同步中心-#015](https://www.yx-sf.com/news/16985)
* [北美与欧洲边缘备份节点-#016](https://www.ai-hao123.com/keji/sync-72066761.html)
* [北美与欧洲边缘备份节点-#017](https://www.mw-wm.com/zhizhu/account-32438777.html)
* [自动化快照与增量广播源-#018](https://www.yx-sf.com/wiki/81506)
* [北美与欧洲边缘备份节点-#019](https://www.ai-hao123.com/hezuo/module-53460469.html)
* [北美与欧洲边缘备份节点-#020](https://www.mw-wm.com/wenzhang/funnel-26196505.html)
* [实时主干镜像高速数据源-#021](https://www.yx-sf.com/news/72954)
* [自动化快照与增量广播源-#022](https://www.ai-hao123.com/qiye/consulting-76152749.html)
* [亚太核心区域镜像同步中心-#023](https://www.mw-wm.com/zixun/sales-52680646.html)
* [自动化快照与增量广播源-#024](https://www.yx-sf.com/tech/60319)
* [亚太核心区域镜像同步中心-#025](https://www.ai-hao123.com/kuangjia/policy-56678639.html)
* [亚太核心区域镜像同步中心-#026](https://www.mw-wm.com/yunsuan/funnel-27454828.html)
* [北美与欧洲边缘备份节点-#027](https://www.yx-sf.com/wiki/29215)
* [冷热数据分层镜像归档中心-#028](https://www.ai-hao123.com/gongxiang/button-81781341.html)
* [冷热数据分层镜像归档中心-#029](https://www.mw-wm.com/jishu/management-60167547.html)
* [北美与欧洲边缘备份节点-#030](https://www.yx-sf.com/tech/97307)
* [冷热数据分层镜像归档中心-#031](https://www.ai-hao123.com/huodong/movie-87289950.html)
* [实时主干镜像高速数据源-#032](https://www.mw-wm.com/zhinan/collaborate-29120651.html)
* [实时主干镜像高速数据源-#033](https://www.yx-sf.com/news/68591)
* [冷热数据分层镜像归档中心-#034](https://www.ai-hao123.com/huodong/contact-92800695.html)
* [亚太核心区域镜像同步中心-#035](https://www.mw-wm.com/shangye/performance-71877402.html)
* [亚太核心区域镜像同步中心-#036](https://www.yx-sf.com/wiki/56743)
* [冷热数据分层镜像归档中心-#037](https://www.ai-hao123.com/paiming/saving-24491528.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [实时延迟与抖动度量规范-#001](https://www.mw-wm.com/paiming/webinar-00082212.html)
* [节点连通性与存活探测准则-#002](https://www.yx-sf.com/tech/7537)
* [实时延迟与抖动度量规范-#003](https://www.ai-hao123.com/yinqing/alert-32484316.html)
* [节点连通性与存活探测准则-#004](https://www.mw-wm.com/chuangxin/extension-50238585.html)
* [实时延迟与抖动度量规范-#005](https://www.yx-sf.com/wiki/73618)
* [去中心化健康检查协议-#006](https://www.ai-hao123.com/shangye/api-54910297.html)
* [去中心化健康检查协议-#007](https://www.mw-wm.com/shuju/admin-56965761.html)
* [去中心化健康检查协议-#008](https://www.yx-sf.com/wiki/35762)
* [实时延迟与抖动度量规范-#009](https://www.ai-hao123.com/peixun/register-38472634.html)
* [防重放安全验证与校验哈希-#010](https://www.mw-wm.com/chuangxin/link-04194391.html)
* [节点连通性与存活探测准则-#011](https://www.yx-sf.com/news/49247)
* [节点连通性与存活探测准则-#012](https://www.ai-hao123.com/liuliang/trading-63665880.html)
* [防重放安全验证与校验哈希-#013](https://www.mw-wm.com/xitong/optimization-02468545.html)
* [去中心化健康检查协议-#014](https://www.yx-sf.com/tech/33449)
* [权威网络权重与收录基准-#015](https://www.ai-hao123.com/yanjiu/help-35423728.html)
* [节点连通性与存活探测准则-#016](https://www.mw-wm.com/chuangxin/follow-82827325.html)
* [节点连通性与存活探测准则-#017](https://www.yx-sf.com/tech/43415)
* [防重放安全验证与校验哈希-#018](https://www.ai-hao123.com/peixun/ebook-07458096.html)
* [权威网络权重与收录基准-#019](https://www.mw-wm.com/guanjianci/unsubscribe-50499788.html)
* [防重放安全验证与校验哈希-#020](https://www.yx-sf.com/wiki/8839)
* [权威网络权重与收录基准-#021](https://www.ai-hao123.com/jianzhan/success-99468359.html)
* [节点连通性与存活探测准则-#022](https://www.mw-wm.com/jianzhan/settings-71835964.html)
* [节点连通性与存活探测准则-#023](https://www.yx-sf.com/wiki/76419)
* [去中心化健康检查协议-#024](https://www.ai-hao123.com/pingtai/podcast-54518105.html)
* [防重放安全验证与校验哈希-#025](https://www.mw-wm.com/xitong/responsive-65113753.html)
* [权威网络权重与收录基准-#026](https://www.yx-sf.com/wiki/73828)
* [实时延迟与抖动度量规范-#027](https://www.ai-hao123.com/paiming/media-29017049.html)
* [权威网络权重与收录基准-#028](https://www.mw-wm.com/pingtai/cloud-44461288.html)
* [防重放安全验证与校验哈希-#029](https://www.yx-sf.com/news/58165)
* [去中心化健康检查协议-#030](https://www.ai-hao123.com/sheji/consulting-36335157.html)
* [权威网络权重与收录基准-#031](https://www.mw-wm.com/gongju/coupon-13529003.html)
* [节点连通性与存活探测准则-#032](https://www.yx-sf.com/tech/4181)
* [权威网络权重与收录基准-#033](https://www.ai-hao123.com/gongsi/solution-44574596.html)
* [实时延迟与抖动度量规范-#034](https://www.mw-wm.com/zixun/machine-73481875.html)
* [节点连通性与存活探测准则-#035](https://www.yx-sf.com/news/6580)
* [防重放安全验证与校验哈希-#036](https://www.ai-hao123.com/fenxi/guide-95574760.html)
* [实时延迟与抖动度量规范-#037](https://www.mw-wm.com/paiming/team-53013904.html)
* [节点连通性与存活探测准则-#038](https://www.yx-sf.com/wiki/49223)
* [实时延迟与抖动度量规范-#039](https://www.ai-hao123.com/jianzhan/behavior-90936933.html)

</details>

