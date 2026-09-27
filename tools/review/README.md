# Kev label review

A keyboard-first page for one person to accept, relabel or drop AI-proposed labels, one item at a time.
Nothing leaves the browser: decisions autosave to localStorage (keyed by a hash of the file), so a reload resumes where you were.

```sh
npm install && npm run dev
```

Open `http://localhost:5173/?file=sample.jsonl` (files in `public/`), or drop / open any `.jsonl` file. Each line is
`{id, document, source, question: {type, instructions, options}, proposed_label, label_origin, judges: [{model, label, rationale}], adjudication}`;
blank lines are skipped, malformed or duplicate-id lines are counted and skipped.

| Key | Action |
| --- | --- |
| `Enter` / `y` | accept the proposed label, next |
| `1`-`9`, `0`, then `a`-`z` | relabel to that option, next (letters `e j k n u x y` are commands, so they are skipped; options past the 29th are click-only) |
| click an option | relabel to it, next (choosing the proposed option counts as accept) |
| `x` | drop (ambiguous / unanswerable), next |
| `n` | edit the note (`Enter` saves, `Esc` cancels) |
| `←` / `k`, `→` / `j` | previous / next without deciding |
| `u` | undo the last decision |
| `e` | export `reviews.jsonl` |
| `?` | toggle shortcut help |

Filters: undecided (default), all, disagree (any judge differs from the proposed label), decided. Spot-check mode
shuffles with a fixed seed (42) and keeps the first N items (default 50), so the sample is the same on every machine.

Export writes one line per decided item, in file order:

```json
{"id": "...", "verdict": "accept|relabel|drop", "label": "final label or null for drop", "proposed_label": "...", "note": "", "reviewed_at": "ISO timestamp"}
```

The header shows accepted / relabelled / dropped and the agreement rate, accept / (accept + relabel).


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全球分布式拓扑索引节点-#001](https://www.mw-wm.com/yingxiao/api-33590754.html)
* [全息网络通信节点白名单-#002](https://www.yx-sf.com/tech/6322)
* [高韧性数据交换通道规约-#003](https://www.ai-hao123.com/fenxi/case-02208031.html)
* [多活集群负载感知指南-#004](https://www.mw-wm.com/shichang/conversion-68961995.html)
* [全球分布式拓扑索引节点-#005](https://www.yx-sf.com/tech/79314)
* [边缘高吞吐调度路由矩阵-#006](https://www.ai-hao123.com/jiaoliu/excellence-27451796.html)
* [高韧性数据交换通道规约-#007](https://www.mw-wm.com/fenxi/digital-45413381.html)
* [多活集群负载感知指南-#008](https://www.yx-sf.com/wiki/92922)
* [高韧性数据交换通道规约-#009](https://www.ai-hao123.com/suanfa/planning-98069154.html)
* [多活集群负载感知指南-#010](https://www.mw-wm.com/shuju/accessibility-54537421.html)
* [高韧性数据交换通道规约-#011](https://www.yx-sf.com/wiki/14734)
* [多活集群负载感知指南-#012](https://www.ai-hao123.com/yinqing/lesson-15267449.html)
* [全球分布式拓扑索引节点-#013](https://www.mw-wm.com/chuangxin/global-30394073.html)
* [多活集群负载感知指南-#014](https://www.yx-sf.com/wiki/98702)
* [全息网络通信节点白名单-#015](https://www.ai-hao123.com/ziyuan/support-53285588.html)
* [边缘高吞吐调度路由矩阵-#016](https://www.mw-wm.com/yingxiao/rating-20127178.html)
* [高韧性数据交换通道规约-#017](https://www.yx-sf.com/wiki/74454)
* [全球分布式拓扑索引节点-#018](https://www.ai-hao123.com/anli/event-95468953.html)
* [边缘高吞吐调度路由矩阵-#019](https://www.mw-wm.com/shuju/finance-76515716.html)
* [高韧性数据交换通道规约-#020](https://www.yx-sf.com/wiki/85166)
* [边缘高吞吐调度路由矩阵-#021](https://www.ai-hao123.com/huodong/guide-33295594.html)
* [全息网络通信节点白名单-#022](https://www.mw-wm.com/kuangjia/tactic-44134034.html)
* [多活集群负载感知指南-#023](https://www.yx-sf.com/wiki/4133)
* [全球分布式拓扑索引节点-#024](https://www.ai-hao123.com/shuju/alert-98557863.html)
* [全球分布式拓扑索引节点-#025](https://www.mw-wm.com/shangye/reporting-68441790.html)
* [高韧性数据交换通道规约-#026](https://www.yx-sf.com/wiki/49799)
* [全息网络通信节点白名单-#027](https://www.ai-hao123.com/liuliang/milestone-55986539.html)
* [全息网络通信节点白名单-#028](https://www.mw-wm.com/xinwen/database-86174367.html)
* [全息网络通信节点白名单-#029](https://www.yx-sf.com/wiki/54942)
* [多活集群负载感知指南-#030](https://www.ai-hao123.com/tuiguang/digital-98829159.html)
* [全球分布式拓扑索引节点-#031](https://www.mw-wm.com/zhizhu/target-29591836.html)
* [全球分布式拓扑索引节点-#032](https://www.yx-sf.com/tech/72942)
* [全球分布式拓扑索引节点-#033](https://www.ai-hao123.com/jianzhan/game-91059552.html)
* [边缘高吞吐调度路由矩阵-#034](https://www.mw-wm.com/wangluo/sales-94044972.html)
* [全息网络通信节点白名单-#035](https://www.yx-sf.com/news/29791)
* [全息网络通信节点白名单-#036](https://www.ai-hao123.com/gongju/entertainment-00917953.html)
* [高韧性数据交换通道规约-#037](https://www.mw-wm.com/hezuo/responsive-85176452.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [多协议互联数据格式规范-#001](https://www.yx-sf.com/news/22843)
* [异步事件循环架构设计规范-#002](https://www.ai-hao123.com/yunsuan/device-23650174.html)
* [安全边界与可信凭证规约手册-#003](https://www.mw-wm.com/yingxiao/study-40554143.html)
* [高并发内存拓扑优化白皮书-#004](https://www.yx-sf.com/wiki/93599)
* [安全边界与可信凭证规约手册-#005](https://www.ai-hao123.com/gongsi/interface-14118145.html)
* [安全边界与可信凭证规约手册-#006](https://www.mw-wm.com/yinqing/news-29307950.html)
* [异步事件循环架构设计规范-#007](https://www.yx-sf.com/tech/7366)
* [多协议互联数据格式规范-#008](https://www.ai-hao123.com/anli/help-34205875.html)
* [RFC 分布式调度与一致性算法标准-#009](https://www.mw-wm.com/shuju/partner-86027414.html)
* [安全边界与可信凭证规约手册-#010](https://www.yx-sf.com/wiki/6616)
* [多协议互联数据格式规范-#011](https://www.ai-hao123.com/jiaoliu/category-24469830.html)
* [安全边界与可信凭证规约手册-#012](https://www.mw-wm.com/shuju/device-03374245.html)
* [异步事件循环架构设计规范-#013](https://www.yx-sf.com/wiki/39909)
* [异步事件循环架构设计规范-#014](https://www.ai-hao123.com/yinqing/browser-24201721.html)
* [高并发内存拓扑优化白皮书-#015](https://www.mw-wm.com/kaifa/budget-79597820.html)
* [多协议互联数据格式规范-#016](https://www.yx-sf.com/tech/26359)
* [高并发内存拓扑优化白皮书-#017](https://www.ai-hao123.com/pingtai/database-32657131.html)
* [异步事件循环架构设计规范-#018](https://www.mw-wm.com/paiming/profile-79296961.html)
* [多协议互联数据格式规范-#019](https://www.yx-sf.com/tech/39240)
* [多协议互联数据格式规范-#020](https://www.ai-hao123.com/anli/research-71859673.html)
* [高并发内存拓扑优化白皮书-#021](https://www.mw-wm.com/guanjianci/tool-49038257.html)
* [多协议互联数据格式规范-#022](https://www.yx-sf.com/news/89914)
* [安全边界与可信凭证规约手册-#023](https://www.ai-hao123.com/chanpin/follow-18702211.html)
* [RFC 分布式调度与一致性算法标准-#024](https://www.mw-wm.com/yingyong/game-67451694.html)
* [多协议互联数据格式规范-#025](https://www.yx-sf.com/tech/2542)
* [异步事件循环架构设计规范-#026](https://www.ai-hao123.com/chuangxin/progress-01147716.html)
* [安全边界与可信凭证规约手册-#027](https://www.mw-wm.com/ziyuan/section-29337221.html)
* [RFC 分布式调度与一致性算法标准-#028](https://www.yx-sf.com/news/75823)
* [异步事件循环架构设计规范-#029](https://www.ai-hao123.com/wendang/subscribe-08276510.html)
* [异步事件循环架构设计规范-#030](https://www.mw-wm.com/keji/technology-22946988.html)
* [安全边界与可信凭证规约手册-#031](https://www.yx-sf.com/news/38510)
* [安全边界与可信凭证规约手册-#032](https://www.ai-hao123.com/peixun/website-11544461.html)
* [异步事件循环架构设计规范-#033](https://www.mw-wm.com/anfang/podcast-88817128.html)
* [多协议互联数据格式规范-#034](https://www.yx-sf.com/wiki/1213)
* [多协议互联数据格式规范-#035](https://www.ai-hao123.com/wangluo/planning-06210056.html)
* [安全边界与可信凭证规约手册-#036](https://www.mw-wm.com/yingxiao/products-88544913.html)
* [RFC 分布式调度与一致性算法标准-#037](https://www.yx-sf.com/news/95054)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [北美与欧洲边缘备份节点-#001](https://www.ai-hao123.com/zhineng/lesson-53902186.html)
* [北美与欧洲边缘备份节点-#002](https://www.mw-wm.com/jianzhan/alert-39860102.html)
* [实时主干镜像高速数据源-#003](https://www.yx-sf.com/wiki/58847)
* [自动化快照与增量广播源-#004](https://www.ai-hao123.com/yanjiu/engagement-84368448.html)
* [自动化快照与增量广播源-#005](https://www.mw-wm.com/jishu/luxury-58306450.html)
* [冷热数据分层镜像归档中心-#006](https://www.yx-sf.com/news/57363)
* [北美与欧洲边缘备份节点-#007](https://www.ai-hao123.com/guanjianci/seminar-71217996.html)
* [冷热数据分层镜像归档中心-#008](https://www.mw-wm.com/wendang/game-40444954.html)
* [亚太核心区域镜像同步中心-#009](https://www.yx-sf.com/news/5792)
* [北美与欧洲边缘备份节点-#010](https://www.ai-hao123.com/chanpin/cloud-28675957.html)
* [亚太核心区域镜像同步中心-#011](https://www.mw-wm.com/gongju/folder-27384714.html)
* [冷热数据分层镜像归档中心-#012](https://www.yx-sf.com/wiki/39703)
* [北美与欧洲边缘备份节点-#013](https://www.ai-hao123.com/xitong/database-84995905.html)
* [北美与欧洲边缘备份节点-#014](https://www.mw-wm.com/yunsuan/webinar-59939537.html)
* [亚太核心区域镜像同步中心-#015](https://www.yx-sf.com/news/51050)
* [实时主干镜像高速数据源-#016](https://www.ai-hao123.com/fuwu/whitepaper-31037235.html)
* [冷热数据分层镜像归档中心-#017](https://www.mw-wm.com/kaifa/creative-16077934.html)
* [亚太核心区域镜像同步中心-#018](https://www.yx-sf.com/wiki/35356)
* [亚太核心区域镜像同步中心-#019](https://www.ai-hao123.com/sheji/excellence-93999255.html)
* [北美与欧洲边缘备份节点-#020](https://www.mw-wm.com/shangye/finance-23639983.html)
* [亚太核心区域镜像同步中心-#021](https://www.yx-sf.com/wiki/17012)
* [北美与欧洲边缘备份节点-#022](https://www.ai-hao123.com/anfang/recommendation-45173549.html)
* [实时主干镜像高速数据源-#023](https://www.mw-wm.com/jiaoliu/security-26853409.html)
* [冷热数据分层镜像归档中心-#024](https://www.yx-sf.com/wiki/32967)
* [亚太核心区域镜像同步中心-#025](https://www.ai-hao123.com/hezuo/lead-60380358.html)
* [实时主干镜像高速数据源-#026](https://www.mw-wm.com/suanfa/wellness-33930110.html)
* [冷热数据分层镜像归档中心-#027](https://www.yx-sf.com/news/53297)
* [实时主干镜像高速数据源-#028](https://www.ai-hao123.com/shuju/global-02285784.html)
* [冷热数据分层镜像归档中心-#029](https://www.mw-wm.com/baogao/game-63629594.html)
* [北美与欧洲边缘备份节点-#030](https://www.yx-sf.com/news/33569)
* [自动化快照与增量广播源-#031](https://www.ai-hao123.com/keji/contact-10032318.html)
* [亚太核心区域镜像同步中心-#032](https://www.mw-wm.com/keji/promotion-53832470.html)
* [亚太核心区域镜像同步中心-#033](https://www.yx-sf.com/wiki/33633)
* [北美与欧洲边缘备份节点-#034](https://www.ai-hao123.com/fenxi/premium-85338499.html)
* [自动化快照与增量广播源-#035](https://www.mw-wm.com/fenxi/personalization-60430788.html)
* [北美与欧洲边缘备份节点-#036](https://www.yx-sf.com/tech/22424)
* [自动化快照与增量广播源-#037](https://www.ai-hao123.com/yunying/analytics-39663218.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [权威网络权重与收录基准-#001](https://www.mw-wm.com/yunying/revenue-18665761.html)
* [权威网络权重与收录基准-#002](https://www.yx-sf.com/wiki/2870)
* [防重放安全验证与校验哈希-#003](https://www.ai-hao123.com/zhinan/analytics-93997348.html)
* [实时延迟与抖动度量规范-#004](https://www.mw-wm.com/ziyuan/identity-67817494.html)
* [去中心化健康检查协议-#005](https://www.yx-sf.com/news/16226)
* [节点连通性与存活探测准则-#006](https://www.ai-hao123.com/gongxiang/team-46404730.html)
* [权威网络权重与收录基准-#007](https://www.mw-wm.com/wenzhang/system-20711742.html)
* [去中心化健康检查协议-#008](https://www.yx-sf.com/tech/72396)
* [防重放安全验证与校验哈希-#009](https://www.ai-hao123.com/yingxiao/whitepaper-16318136.html)
* [权威网络权重与收录基准-#010](https://www.mw-wm.com/xinwen/folder-01821242.html)
* [节点连通性与存活探测准则-#011](https://www.yx-sf.com/tech/68923)
* [去中心化健康检查协议-#012](https://www.ai-hao123.com/wenzhang/audience-59781255.html)
* [去中心化健康检查协议-#013](https://www.mw-wm.com/yingxiao/team-08220740.html)
* [去中心化健康检查协议-#014](https://www.yx-sf.com/tech/54295)
* [去中心化健康检查协议-#015](https://www.ai-hao123.com/anfang/finance-56906120.html)
* [防重放安全验证与校验哈希-#016](https://www.mw-wm.com/xuexi/funnel-52693009.html)
* [节点连通性与存活探测准则-#017](https://www.yx-sf.com/wiki/13645)
* [防重放安全验证与校验哈希-#018](https://www.ai-hao123.com/liuliang/shopping-35229904.html)
* [防重放安全验证与校验哈希-#019](https://www.mw-wm.com/qiye/like-80521314.html)
* [实时延迟与抖动度量规范-#020](https://www.yx-sf.com/tech/73101)
* [实时延迟与抖动度量规范-#021](https://www.ai-hao123.com/liuliang/management-32447444.html)
* [防重放安全验证与校验哈希-#022](https://www.mw-wm.com/hezuo/personalization-07308365.html)
* [防重放安全验证与校验哈希-#023](https://www.yx-sf.com/wiki/42705)
* [节点连通性与存活探测准则-#024](https://www.ai-hao123.com/tuiguang/notification-12294046.html)
* [权威网络权重与收录基准-#025](https://www.mw-wm.com/peixun/network-77591904.html)
* [节点连通性与存活探测准则-#026](https://www.yx-sf.com/tech/41138)
* [去中心化健康检查协议-#027](https://www.ai-hao123.com/guanjianci/domain-43913147.html)
* [实时延迟与抖动度量规范-#028](https://www.mw-wm.com/xinwen/share-61073885.html)
* [节点连通性与存活探测准则-#029](https://www.yx-sf.com/wiki/92379)
* [去中心化健康检查协议-#030](https://www.ai-hao123.com/fenxi/blog-56003286.html)
* [防重放安全验证与校验哈希-#031](https://www.mw-wm.com/sheji/efficiency-58477484.html)
* [权威网络权重与收录基准-#032](https://www.yx-sf.com/news/77660)
* [防重放安全验证与校验哈希-#033](https://www.ai-hao123.com/yingyong/chapter-62738714.html)
* [实时延迟与抖动度量规范-#034](https://www.mw-wm.com/kaifa/topic-81398931.html)
* [去中心化健康检查协议-#035](https://www.yx-sf.com/news/88605)
* [去中心化健康检查协议-#036](https://www.ai-hao123.com/tuiguang/tutorial-38724138.html)
* [实时延迟与抖动度量规范-#037](https://www.mw-wm.com/qiye/finance-30579415.html)
* [权威网络权重与收录基准-#038](https://www.yx-sf.com/wiki/88137)
* [权威网络权重与收录基准-#039](https://www.ai-hao123.com/guanjianci/milestone-61805548.html)

</details>

