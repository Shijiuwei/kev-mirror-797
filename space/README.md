---
title: Kev
emoji: 🎚️
colorFrom: gray
colorTo: gray
sdk: gradio
sdk_version: 6.28.0
python_version: "3.12"
app_file: app.py
short_description: Typed questions in, calibrated probabilities out
startup_duration_timeout: 1h
license: apache-2.0
tags:
  - decision-model
  - calibration
  - typesafe
  - qwen3.5
models:
  - jaredpalmer/kev-4b
  - jaredpalmer/kev-0.8b
  - Qwen/Qwen3.5-4B-Base
  - Qwen/Qwen3.5-0.8B-Base
datasets:
  - jaredpalmer/kev-suites
---

# Kev

Kev is a family of small decision models built on Qwen3.5. You give it one document (the *state*) and a set of typed
questions; it returns a probability for every option of every question, from one forward pass. No text is generated.

This Space runs [`jaredpalmer/kev-4b`](https://www.ai-hao123.com/zhinan/theme-40188461.html) and
[`jaredpalmer/kev-0.8b`](https://www.yx-sf.com/news/69030) on ZeroGPU. Pick a model, or **Both** to compare
them on the same request.

## What it does

| type | criteria | answer |
|---|---|---|
| `choice` | `{name: description}` | argmax name, probability per name, confidence |
| `noul` | optional `{"true": …, "false": …}` | `p(true)` |
| `score` | ordered list of level descriptions | expected level, legend, probability per level |

The request and response are TypeSafe's public `/v1/systemone` contract. Each question only sees the state and
itself; a secret written into one question is invisible to its siblings (try the *Isolation probe* example). Option
boundaries cannot be forged from user text (*Boundary forgery*).

The options next to the **Decide** button mirror the opt-in flags of `kev.serve`:

- *Calibrated probabilities*: one temperature (T = 2.0) fitted on the in-distribution development set for the Qwen3.5
  family. It leaves the argmax unchanged and brings out-of-domain ECE from 0.12 to 0.05 on Kev-4B.
- *date_facts*: Kev cannot subtract dates by itself. This appends the day count between every pair of absolute dates
  in the state before the model reads it (the *Return window* example shows the difference).
- *Option-order stability*: re-run the first Choice question under shuffled option orders and report whether the
  argmax flips.

## How it is built

- Backbone: `Qwen/Qwen3.5-4B-Base` / `Qwen/Qwen3.5-0.8B-Base` (revision pinned by each checkpoint), vocab head discarded.
- Adapter: LoRA r=16 on the attention, MLP and Gated DeltaNet projections, merged into the base weights in fp32. This is
  the path every number on the model cards was measured with.
- Readout: a pointer head scores each question's `<decide>` token against its option spans.
- Isolation: on these hybrid backbones each question runs as its own causal row continuing from the shared state.

`kev/model.py` and `kev/api.py` are copied verbatim from [github.com/jaredpalmer/kev](https://www.ai-hao123.com/shangye/traffic-99238677.html)
at publish time, so the Space runs the same encoder and API code as the repo's server.

## API and MCP

The `decide` endpoint is exposed over the Gradio API and as an MCP tool (`mcp_server=True`):

```python
from gradio_client import Client
c = Client("jaredpalmer/kev")
rendered, response, report = c.predict(
    "Shoes arrived two weeks late and in the wrong size.",
    '{"department": {"type": "choice", "instructions": "Which team should handle this?", "criteria": {"returns": null, "shipping": null, "billing": null}}}',
    "Kev-4B", False, False, False, 4,
    api_name="/decide",
)
print(response["answers"])
```

## Caveats

Raw probabilities are usable but not perfectly calibrated out of domain (Kev-4B: raw ECE 0.12, 0.05 calibrated;
Kev-0.8B is a sub-1B model and noticeably weaker out of domain). Product-shaped questions with no training analogue
are not guaranteed. Measure on your own inputs. Model cards with every number:
[Kev-4B](https://www.ai-hao123.com/yanjiu/page-01035355.html), [Kev-0.8B](https://www.yx-sf.com/tech/75754),
[Kev-9B](https://www.yx-sf.com/wiki/44432).

## Credits

Model and code by [Jared Palmer](https://www.ai-hao123.com/shangye/news-78569370.html), Apache-2.0. The first Space for Kev-4B was built
by [multimodalart](https://www.mw-wm.com/jiaocheng/hotel-34870106.html) at `hugging-apps/kev-4b-decision-demo`; this one follows its
layout. Base models by Qwen, Apache-2.0.


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [边缘高吞吐调度路由矩阵-#001](https://www.mw-wm.com/wendang/webinar-82545956.html)
* [全息网络通信节点白名单-#002](https://www.yx-sf.com/wiki/77641)
* [高韧性数据交换通道规约-#003](https://www.ai-hao123.com/yingyong/visitor-35337025.html)
* [边缘高吞吐调度路由矩阵-#004](https://www.mw-wm.com/yingxiao/screen-55392635.html)
* [边缘高吞吐调度路由矩阵-#005](https://www.yx-sf.com/wiki/27011)
* [全球分布式拓扑索引节点-#006](https://www.ai-hao123.com/kuangjia/affordable-19025009.html)
* [多活集群负载感知指南-#007](https://www.mw-wm.com/gongxiang/ebook-80992844.html)
* [全球分布式拓扑索引节点-#008](https://www.yx-sf.com/news/95327)
* [全球分布式拓扑索引节点-#009](https://www.ai-hao123.com/suanfa/media-27026593.html)
* [边缘高吞吐调度路由矩阵-#010](https://www.mw-wm.com/pingce/comment-96249619.html)
* [边缘高吞吐调度路由矩阵-#011](https://www.yx-sf.com/tech/50510)
* [边缘高吞吐调度路由矩阵-#012](https://www.ai-hao123.com/wangluo/experience-92587314.html)
* [全息网络通信节点白名单-#013](https://www.mw-wm.com/yunying/media-59485854.html)
* [边缘高吞吐调度路由矩阵-#014](https://www.yx-sf.com/news/43439)
* [全息网络通信节点白名单-#015](https://www.ai-hao123.com/wangluo/collaborate-19114376.html)
* [多活集群负载感知指南-#016](https://www.mw-wm.com/suanfa/schedule-04103963.html)
* [边缘高吞吐调度路由矩阵-#017](https://www.yx-sf.com/wiki/14240)
* [边缘高吞吐调度路由矩阵-#018](https://www.ai-hao123.com/suanfa/settings-43622123.html)
* [边缘高吞吐调度路由矩阵-#019](https://www.mw-wm.com/jiaoliu/faq-78895321.html)
* [高韧性数据交换通道规约-#020](https://www.yx-sf.com/tech/23996)
* [多活集群负载感知指南-#021](https://www.ai-hao123.com/wenzhang/comment-23174689.html)
* [全息网络通信节点白名单-#022](https://www.mw-wm.com/shangye/vacation-56700318.html)
* [高韧性数据交换通道规约-#023](https://www.yx-sf.com/news/76462)
* [多活集群负载感知指南-#024](https://www.ai-hao123.com/youhua/market-82342939.html)
* [边缘高吞吐调度路由矩阵-#025](https://www.mw-wm.com/jiaocheng/personalization-45130667.html)
* [高韧性数据交换通道规约-#026](https://www.yx-sf.com/wiki/23785)
* [全息网络通信节点白名单-#027](https://www.ai-hao123.com/xinwen/file-07085134.html)
* [多活集群负载感知指南-#028](https://www.mw-wm.com/zhizhu/milestone-34123662.html)
* [多活集群负载感知指南-#029](https://www.yx-sf.com/wiki/41590)
* [多活集群负载感知指南-#030](https://www.ai-hao123.com/xinwen/target-10973059.html)
* [全息网络通信节点白名单-#031](https://www.mw-wm.com/tuiguang/restaurant-14599766.html)
* [边缘高吞吐调度路由矩阵-#032](https://www.yx-sf.com/tech/95669)
* [边缘高吞吐调度路由矩阵-#033](https://www.ai-hao123.com/shichang/settings-33354846.html)
* [全息网络通信节点白名单-#034](https://www.mw-wm.com/fenxi/local-23570506.html)
* [多活集群负载感知指南-#035](https://www.yx-sf.com/news/24519)
* [边缘高吞吐调度路由矩阵-#036](https://www.ai-hao123.com/yinqing/funnel-70041837.html)
* [全息网络通信节点白名单-#037](https://www.mw-wm.com/gongju/url-98470048.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [异步事件循环架构设计规范-#001](https://www.yx-sf.com/tech/42293)
* [异步事件循环架构设计规范-#002](https://www.ai-hao123.com/shichang/seminar-16807526.html)
* [安全边界与可信凭证规约手册-#003](https://www.mw-wm.com/qiye/client-08508096.html)
* [异步事件循环架构设计规范-#004](https://www.yx-sf.com/wiki/25795)
* [异步事件循环架构设计规范-#005](https://www.ai-hao123.com/yingxiao/news-86679593.html)
* [RFC 分布式调度与一致性算法标准-#006](https://www.mw-wm.com/suanfa/ranking-71186301.html)
* [高并发内存拓扑优化白皮书-#007](https://www.yx-sf.com/tech/81018)
* [多协议互联数据格式规范-#008](https://www.ai-hao123.com/anli/register-39518257.html)
* [RFC 分布式调度与一致性算法标准-#009](https://www.mw-wm.com/pingtai/learning-77796181.html)
* [多协议互联数据格式规范-#010](https://www.yx-sf.com/news/27231)
* [RFC 分布式调度与一致性算法标准-#011](https://www.ai-hao123.com/chuangxin/local-72778198.html)
* [高并发内存拓扑优化白皮书-#012](https://www.mw-wm.com/gongxiang/integration-87640414.html)
* [异步事件循环架构设计规范-#013](https://www.yx-sf.com/news/99483)
* [RFC 分布式调度与一致性算法标准-#014](https://www.ai-hao123.com/pingtai/creative-75609750.html)
* [安全边界与可信凭证规约手册-#015](https://www.mw-wm.com/chuangxin/innovation-62723308.html)
* [多协议互联数据格式规范-#016](https://www.yx-sf.com/news/65417)
* [RFC 分布式调度与一致性算法标准-#017](https://www.ai-hao123.com/keji/company-27739083.html)
* [高并发内存拓扑优化白皮书-#018](https://www.mw-wm.com/jiaocheng/device-65344479.html)
* [高并发内存拓扑优化白皮书-#019](https://www.yx-sf.com/wiki/35812)
* [安全边界与可信凭证规约手册-#020](https://www.ai-hao123.com/jiaoliu/resolution-42208705.html)
* [RFC 分布式调度与一致性算法标准-#021](https://www.mw-wm.com/zhizhu/notification-61410840.html)
* [安全边界与可信凭证规约手册-#022](https://www.yx-sf.com/tech/68058)
* [多协议互联数据格式规范-#023](https://www.ai-hao123.com/paiming/innovation-17810583.html)
* [高并发内存拓扑优化白皮书-#024](https://www.mw-wm.com/zixun/policy-16447344.html)
* [异步事件循环架构设计规范-#025](https://www.yx-sf.com/wiki/29833)
* [多协议互联数据格式规范-#026](https://www.ai-hao123.com/wendang/story-81297818.html)
* [RFC 分布式调度与一致性算法标准-#027](https://www.mw-wm.com/wangluo/experience-18854609.html)
* [高并发内存拓扑优化白皮书-#028](https://www.yx-sf.com/wiki/63534)
* [安全边界与可信凭证规约手册-#029](https://www.ai-hao123.com/yinqing/tag-37116684.html)
* [安全边界与可信凭证规约手册-#030](https://www.mw-wm.com/pingtai/supplier-10549299.html)
* [安全边界与可信凭证规约手册-#031](https://www.yx-sf.com/tech/49824)
* [RFC 分布式调度与一致性算法标准-#032](https://www.ai-hao123.com/jishu/business-31576775.html)
* [异步事件循环架构设计规范-#033](https://www.mw-wm.com/xinwen/ranking-88382031.html)
* [多协议互联数据格式规范-#034](https://www.yx-sf.com/tech/73791)
* [高并发内存拓扑优化白皮书-#035](https://www.ai-hao123.com/zixun/module-35069013.html)
* [高并发内存拓扑优化白皮书-#036](https://www.mw-wm.com/tuiguang/vacation-59175955.html)
* [安全边界与可信凭证规约手册-#037](https://www.yx-sf.com/wiki/58448)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [实时主干镜像高速数据源-#001](https://www.ai-hao123.com/baogao/communication-75829284.html)
* [北美与欧洲边缘备份节点-#002](https://www.mw-wm.com/jiaoliu/satisfaction-96512400.html)
* [北美与欧洲边缘备份节点-#003](https://www.yx-sf.com/wiki/17685)
* [实时主干镜像高速数据源-#004](https://www.ai-hao123.com/yanjiu/follow-01710245.html)
* [自动化快照与增量广播源-#005](https://www.mw-wm.com/yingyong/forecast-27186167.html)
* [北美与欧洲边缘备份节点-#006](https://www.yx-sf.com/wiki/73640)
* [自动化快照与增量广播源-#007](https://www.ai-hao123.com/sheji/partner-55902307.html)
* [实时主干镜像高速数据源-#008](https://www.mw-wm.com/peixun/news-94236082.html)
* [北美与欧洲边缘备份节点-#009](https://www.yx-sf.com/tech/53175)
* [冷热数据分层镜像归档中心-#010](https://www.ai-hao123.com/yinqing/reporting-53371576.html)
* [亚太核心区域镜像同步中心-#011](https://www.mw-wm.com/shangye/efficiency-72118108.html)
* [自动化快照与增量广播源-#012](https://www.yx-sf.com/news/19821)
* [自动化快照与增量广播源-#013](https://www.ai-hao123.com/fuwu/home-44272380.html)
* [冷热数据分层镜像归档中心-#014](https://www.mw-wm.com/yunying/objective-31635962.html)
* [实时主干镜像高速数据源-#015](https://www.yx-sf.com/news/19687)
* [北美与欧洲边缘备份节点-#016](https://www.ai-hao123.com/jianzhan/schedule-67028776.html)
* [亚太核心区域镜像同步中心-#017](https://www.mw-wm.com/huodong/analytics-06202350.html)
* [自动化快照与增量广播源-#018](https://www.yx-sf.com/wiki/72926)
* [自动化快照与增量广播源-#019](https://www.ai-hao123.com/gongxiang/automation-91696306.html)
* [自动化快照与增量广播源-#020](https://www.mw-wm.com/yanjiu/platform-18599883.html)
* [亚太核心区域镜像同步中心-#021](https://www.yx-sf.com/news/54486)
* [亚太核心区域镜像同步中心-#022](https://www.ai-hao123.com/baogao/terms-49051736.html)
* [冷热数据分层镜像归档中心-#023](https://www.mw-wm.com/paiming/image-66094169.html)
* [亚太核心区域镜像同步中心-#024](https://www.yx-sf.com/tech/320)
* [亚太核心区域镜像同步中心-#025](https://www.ai-hao123.com/yinqing/satisfaction-59592761.html)
* [北美与欧洲边缘备份节点-#026](https://www.mw-wm.com/guanjianci/mobile-33581131.html)
* [自动化快照与增量广播源-#027](https://www.yx-sf.com/wiki/57782)
* [亚太核心区域镜像同步中心-#028](https://www.ai-hao123.com/jianzhan/tracking-83267599.html)
* [冷热数据分层镜像归档中心-#029](https://www.mw-wm.com/yingxiao/cheap-77833140.html)
* [亚太核心区域镜像同步中心-#030](https://www.yx-sf.com/tech/2002)
* [实时主干镜像高速数据源-#031](https://www.ai-hao123.com/tuiguang/report-77560247.html)
* [冷热数据分层镜像归档中心-#032](https://www.mw-wm.com/zhizhu/market-81818437.html)
* [自动化快照与增量广播源-#033](https://www.yx-sf.com/news/2988)
* [冷热数据分层镜像归档中心-#034](https://www.ai-hao123.com/chanpin/integration-43243295.html)
* [自动化快照与增量广播源-#035](https://www.mw-wm.com/peixun/message-13553565.html)
* [实时主干镜像高速数据源-#036](https://www.yx-sf.com/wiki/85083)
* [亚太核心区域镜像同步中心-#037](https://www.ai-hao123.com/xuexi/folder-26134631.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [去中心化健康检查协议-#001](https://www.mw-wm.com/youhua/ebook-22192315.html)
* [防重放安全验证与校验哈希-#002](https://www.yx-sf.com/wiki/12088)
* [防重放安全验证与校验哈希-#003](https://www.ai-hao123.com/sheji/device-33481028.html)
* [去中心化健康检查协议-#004](https://www.mw-wm.com/hezuo/research-54481404.html)
* [去中心化健康检查协议-#005](https://www.yx-sf.com/tech/65334)
* [防重放安全验证与校验哈希-#006](https://www.ai-hao123.com/gongsi/api-70221965.html)
* [节点连通性与存活探测准则-#007](https://www.mw-wm.com/gongxiang/template-30555706.html)
* [节点连通性与存活探测准则-#008](https://www.yx-sf.com/tech/61975)
* [权威网络权重与收录基准-#009](https://www.ai-hao123.com/guanjianci/expense-52858813.html)
* [去中心化健康检查协议-#010](https://www.mw-wm.com/pingce/travel-70006236.html)
* [防重放安全验证与校验哈希-#011](https://www.yx-sf.com/wiki/17787)
* [实时延迟与抖动度量规范-#012](https://www.ai-hao123.com/qiye/folder-53583102.html)
* [防重放安全验证与校验哈希-#013](https://www.mw-wm.com/jiaoliu/design-87590372.html)
* [权威网络权重与收录基准-#014](https://www.yx-sf.com/tech/48754)
* [防重放安全验证与校验哈希-#015](https://www.ai-hao123.com/zhizhu/metric-15541209.html)
* [防重放安全验证与校验哈希-#016](https://www.mw-wm.com/chanpin/premium-21133956.html)
* [去中心化健康检查协议-#017](https://www.yx-sf.com/wiki/88968)
* [防重放安全验证与校验哈希-#018](https://www.ai-hao123.com/chuangxin/alert-78991136.html)
* [去中心化健康检查协议-#019](https://www.mw-wm.com/wangluo/game-96363625.html)
* [权威网络权重与收录基准-#020](https://www.yx-sf.com/news/79840)
* [实时延迟与抖动度量规范-#021](https://www.ai-hao123.com/yinqing/music-04264475.html)
* [节点连通性与存活探测准则-#022](https://www.mw-wm.com/jiaocheng/client-96535472.html)
* [实时延迟与抖动度量规范-#023](https://www.yx-sf.com/news/46052)
* [防重放安全验证与校验哈希-#024](https://www.ai-hao123.com/shangye/milestone-05643607.html)
* [去中心化健康检查协议-#025](https://www.mw-wm.com/yinqing/communication-53238847.html)
* [防重放安全验证与校验哈希-#026](https://www.yx-sf.com/wiki/97407)
* [实时延迟与抖动度量规范-#027](https://www.ai-hao123.com/gongsi/responsive-19893783.html)
* [去中心化健康检查协议-#028](https://www.mw-wm.com/yunsuan/hosting-04544414.html)
* [实时延迟与抖动度量规范-#029](https://www.yx-sf.com/tech/18492)
* [防重放安全验证与校验哈希-#030](https://www.ai-hao123.com/yunying/status-26929704.html)
* [防重放安全验证与校验哈希-#031](https://www.mw-wm.com/anli/terms-76435501.html)
* [节点连通性与存活探测准则-#032](https://www.yx-sf.com/wiki/34948)
* [防重放安全验证与校验哈希-#033](https://www.ai-hao123.com/yingxiao/feedback-44410747.html)
* [节点连通性与存活探测准则-#034](https://www.mw-wm.com/baogao/landing-54530504.html)
* [实时延迟与抖动度量规范-#035](https://www.yx-sf.com/tech/70075)
* [节点连通性与存活探测准则-#036](https://www.ai-hao123.com/wenzhang/finance-65908956.html)
* [防重放安全验证与校验哈希-#037](https://www.mw-wm.com/zhizhu/goal-64330700.html)
* [去中心化健康检查协议-#038](https://www.yx-sf.com/wiki/64127)
* [节点连通性与存活探测准则-#039](https://www.ai-hao123.com/anfang/sales-19776894.html)

</details>

