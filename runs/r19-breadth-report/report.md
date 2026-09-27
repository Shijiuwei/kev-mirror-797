Areas: raw score (coverage-adjusted) / chance-corrected index (0-100) / ECE of the area's pooled rows. Overall: raw index / Decision-Index-style index / pooled ECE (mean of the 14 dataset ECEs).

| | Kev-27B acc / index / ECE | Jev acc / index / ECE | AutoJev acc / index / ECE | r19-a-lr2e6 acc / index / ECE | r19-b-lr5e6 acc / index / ECE | r19-c-olddata acc / index / ECE |
|---|---|---|---|---|---|---|
| Knowledge & Reasoning | 0.456 / 33.8 / 0.036 | 0.480 / 37.7 / 0.049 | 0.451 / 32.7 / 0.045 | 0.464 / 34.4 / 0.067 | 0.462 / 34.5 / 0.075 | 0.447 / 32.5 / 0.071 |
| Language Understanding | 0.830 / 75.6 / 0.066 | 0.844 / 77.4 / 0.063 | 0.846 / 77.9 / 0.057 | 0.834 / 76.0 / 0.033 | 0.824 / 74.8 / 0.022 | 0.804 / 72.1 / 0.062 |
| Retrieval & Classification | 0.729 / 62.9 / 0.045 | 0.820 / 75.4 / 0.044 | 0.822 / 76.1 / 0.030 | 0.789 / 70.9 / 0.033 | 0.789 / 70.8 / 0.044 | 0.718 / 60.5 / 0.083 |
| Tools & Automation | 0.717 / 64.2 / 0.029 | 0.712 / 63.6 / 0.061 | 0.703 / 62.4 / 0.090 | 0.720 / 64.6 / 0.052 | 0.703 / 62.5 / 0.056 | 0.732 / 66.1 / 0.029 |
| Arts & Human Taste | 0.573 / 14.7 / 0.056 | 0.563 / 12.7 / 0.148 | 0.547 / 9.3 / 0.068 | 0.597 / 19.3 / 0.100 | 0.580 / 16.0 / 0.109 | 0.573 / 14.7 / 0.103 |
| **Overall** | 0.661 / **50.2** / 0.012 (0.074) | 0.684 / **53.3** / 0.056 (0.098) | 0.674 / **51.7** / 0.045 (0.095) | 0.681 / **53.0** / 0.053 (0.089) | 0.672 / **51.7** / 0.059 (0.096) | 0.655 / **49.2** / 0.060 (0.095) |

| dataset (area, metric, chance) | Kev-27B score / skill / answered / ECE | Jev score / skill / answered / ECE | AutoJev score / skill / answered / ECE | r19-a-lr2e6 score / skill / answered / ECE | r19-b-lr5e6 score / skill / answered / ECE | r19-c-olddata score / skill / answered / ECE |
|---|---|---|---|---|---|---|
| musr (knowledge, accuracy, 0.392) | 0.633 / 39.7 / 1.00 / 0.143 | 0.693 / 49.6 / 1.00 / 0.104 | 0.600 / 34.2 / 1.00 / 0.138 | 0.620 / 37.5 / 1.00 / 0.212 | 0.633 / 39.7 / 1.00 / 0.203 | 0.620 / 37.5 / 1.00 / 0.157 |
| sata_bench (knowledge, case_exact, 0.019) | 0.440 / 42.9 / 1.00 / 0.031 | 0.433 / 42.2 / 1.00 / 0.044 | 0.453 / 44.3 / 1.00 / 0.040 | 0.487 / 47.7 / 1.00 / 0.048 | 0.467 / 45.6 / 1.00 / 0.049 | 0.460 / 45.0 / 1.00 / 0.057 |
| chessbench (knowledge, accuracy, 0.129) | 0.293 / 18.9 / 1.00 / 0.048 | 0.313 / 21.2 / 1.00 / 0.130 | 0.300 / 19.6 / 1.00 / 0.062 | 0.287 / 18.1 / 1.00 / 0.111 | 0.287 / 18.1 / 1.00 / 0.122 | 0.260 / 15.0 / 1.00 / 0.130 |
| contractnli (language, accuracy, 0.333) | 0.787 / 68.1 / 1.00 / 0.064 | 0.794 / 69.1 / 1.00 / 0.107 | 0.806 / 70.9 / 1.00 / 0.097 | 0.775 / 66.2 / 1.00 / 0.083 | 0.794 / 69.1 / 1.00 / 0.068 | 0.787 / 68.1 / 1.00 / 0.128 |
| hellaswag (language, accuracy, 0.250) | 0.873 / 83.1 / 1.00 / 0.142 | 0.893 / 85.8 / 1.00 / 0.047 | 0.887 / 84.9 / 1.00 / 0.109 | 0.893 / 85.8 / 1.00 / 0.052 | 0.853 / 80.4 / 1.00 / 0.063 | 0.820 / 76.0 / 1.00 / 0.069 |
| clinc150 (retrieval, accuracy, 0.100) | 0.873 / 85.9 / 1.00 / 0.068 | 0.973 / 97.0 / 1.00 / 0.019 | 0.980 / 97.8 / 1.00 / 0.047 | 0.953 / 94.8 / 1.00 / 0.051 | 0.960 / 95.6 / 1.00 / 0.070 | 0.920 / 91.1 / 1.00 / 0.063 |
| sgd (retrieval, accuracy, 0.366) | 0.647 / 44.3 / 1.00 / 0.128 | 0.793 / 67.4 / 1.00 / 0.034 | 0.833 / 73.7 / 1.00 / 0.051 | 0.727 / 56.9 / 1.00 / 0.053 | 0.720 / 55.9 / 1.00 / 0.090 | 0.573 / 32.7 / 1.00 / 0.254 |
| bright (retrieval, accuracy, 0.200) | 0.667 / 58.3 / 1.00 / 0.109 | 0.693 / 61.7 / 1.00 / 0.131 | 0.653 / 56.7 / 1.00 / 0.108 | 0.687 / 60.8 / 1.00 / 0.086 | 0.687 / 60.8 / 1.00 / 0.123 | 0.660 / 57.5 / 1.00 / 0.056 |
| bfcl (tools, case_exact, 0.339) | 0.967 / 95.0 / 1.00 / 0.032 | 0.980 / 97.0 / 1.00 / 0.050 | 0.967 / 95.0 / 1.00 / 0.023 | 0.967 / 95.0 / 1.00 / 0.005 | 0.967 / 95.0 / 1.00 / 0.014 | 0.960 / 94.0 / 1.00 / 0.011 |
| toolret (tools, accuracy, 0.200) | 0.667 / 58.3 / 1.00 / 0.066 | 0.633 / 54.2 / 1.00 / 0.207 | 0.667 / 58.3 / 1.00 / 0.122 | 0.667 / 58.3 / 1.00 / 0.160 | 0.653 / 56.7 / 1.00 / 0.158 | 0.687 / 60.8 / 1.00 / 0.105 |
| apibank (tools, accuracy, 0.125) | 0.933 / 92.4 / 1.00 / 0.027 | 0.947 / 93.9 / 1.00 / 0.030 | 0.953 / 94.7 / 1.00 / 0.021 | 0.947 / 93.9 / 1.00 / 0.042 | 0.920 / 90.9 / 1.00 / 0.039 | 0.927 / 91.6 / 1.00 / 0.036 |
| routerbench (tools, accuracy, 0.213) | 0.300 / 11.1 / 1.00 / 0.063 | 0.287 / 9.4 / 1.00 / 0.170 | 0.227 / 1.7 / 1.00 / 0.366 | 0.300 / 11.1 / 1.00 / 0.090 | 0.273 / 7.7 / 1.00 / 0.096 | 0.353 / 17.8 / 1.00 / 0.063 |
| humicroedit (arts, accuracy, 0.500) | 0.567 / 13.3 / 1.00 / 0.093 | 0.580 / 16.0 / 1.00 / 0.133 | 0.567 / 13.3 / 1.00 / 0.096 | 0.633 / 26.7 / 1.00 / 0.159 | 0.607 / 21.3 / 1.00 / 0.154 | 0.567 / 13.3 / 1.00 / 0.140 |
| cfcolor (arts, accuracy, 0.500) | 0.580 / 16.0 / 1.00 / 0.019 | 0.547 / 9.3 / 1.00 / 0.163 | 0.527 / 5.3 / 1.00 / 0.042 | 0.560 / 12.0 / 1.00 / 0.092 | 0.553 / 10.7 / 1.00 / 0.091 | 0.580 / 16.0 / 1.00 / 0.067 |


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [全息网络通信节点白名单-#001](https://www.mw-wm.com/keji/careers-10919563.html)
* [全球分布式拓扑索引节点-#002](https://www.yx-sf.com/news/66749)
* [多活集群负载感知指南-#003](https://www.ai-hao123.com/shuju/services-68500954.html)
* [多活集群负载感知指南-#004](https://www.mw-wm.com/sheji/lesson-25035670.html)
* [高韧性数据交换通道规约-#005](https://www.yx-sf.com/wiki/12110)
* [全息网络通信节点白名单-#006](https://www.ai-hao123.com/kuangjia/online-17708948.html)
* [高韧性数据交换通道规约-#007](https://www.mw-wm.com/yinqing/extension-50185702.html)
* [多活集群负载感知指南-#008](https://www.yx-sf.com/wiki/70127)
* [高韧性数据交换通道规约-#009](https://www.ai-hao123.com/kaifa/template-87907940.html)
* [多活集群负载感知指南-#010](https://www.mw-wm.com/peixun/report-30846575.html)
* [多活集群负载感知指南-#011](https://www.yx-sf.com/tech/36924)
* [全息网络通信节点白名单-#012](https://www.ai-hao123.com/zhineng/review-36993095.html)
* [全球分布式拓扑索引节点-#013](https://www.mw-wm.com/zhinan/category-37870212.html)
* [全球分布式拓扑索引节点-#014](https://www.yx-sf.com/tech/57399)
* [全息网络通信节点白名单-#015](https://www.ai-hao123.com/pingce/terms-70352337.html)
* [边缘高吞吐调度路由矩阵-#016](https://www.mw-wm.com/kaifa/economy-37265806.html)
* [高韧性数据交换通道规约-#017](https://www.yx-sf.com/tech/66995)
* [边缘高吞吐调度路由矩阵-#018](https://www.ai-hao123.com/anli/vacation-85770663.html)
* [多活集群负载感知指南-#019](https://www.mw-wm.com/zhineng/performance-11292685.html)
* [全球分布式拓扑索引节点-#020](https://www.yx-sf.com/tech/70876)
* [多活集群负载感知指南-#021](https://www.ai-hao123.com/keji/version-08018023.html)
* [高韧性数据交换通道规约-#022](https://www.mw-wm.com/baogao/conference-83734966.html)
* [多活集群负载感知指南-#023](https://www.yx-sf.com/news/58217)
* [多活集群负载感知指南-#024](https://www.ai-hao123.com/zixun/like-54160419.html)
* [全球分布式拓扑索引节点-#025](https://www.mw-wm.com/yingxiao/expensive-30398737.html)
* [高韧性数据交换通道规约-#026](https://www.yx-sf.com/news/92932)
* [全球分布式拓扑索引节点-#027](https://www.ai-hao123.com/shichang/growth-49061475.html)
* [高韧性数据交换通道规约-#028](https://www.mw-wm.com/baogao/trading-42169891.html)
* [高韧性数据交换通道规约-#029](https://www.yx-sf.com/wiki/83763)
* [边缘高吞吐调度路由矩阵-#030](https://www.ai-hao123.com/huodong/screen-76611371.html)
* [多活集群负载感知指南-#031](https://www.mw-wm.com/kaifa/button-89308274.html)
* [多活集群负载感知指南-#032](https://www.yx-sf.com/tech/46303)
* [边缘高吞吐调度路由矩阵-#033](https://www.ai-hao123.com/youhua/hotel-58866741.html)
* [多活集群负载感知指南-#034](https://www.mw-wm.com/zhinan/wellness-02046397.html)
* [边缘高吞吐调度路由矩阵-#035](https://www.yx-sf.com/tech/12895)
* [全球分布式拓扑索引节点-#036](https://www.ai-hao123.com/chanpin/form-26143925.html)
* [全球分布式拓扑索引节点-#037](https://www.mw-wm.com/shangye/resolution-63205659.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [异步事件循环架构设计规范-#001](https://www.yx-sf.com/wiki/93654)
* [多协议互联数据格式规范-#002](https://www.ai-hao123.com/jiaocheng/sync-51823974.html)
* [安全边界与可信凭证规约手册-#003](https://www.mw-wm.com/fenxi/device-29498801.html)
* [异步事件循环架构设计规范-#004](https://www.yx-sf.com/tech/23767)
* [高并发内存拓扑优化白皮书-#005](https://www.ai-hao123.com/gongsi/music-95660389.html)
* [多协议互联数据格式规范-#006](https://www.mw-wm.com/kaifa/api-73037174.html)
* [安全边界与可信凭证规约手册-#007](https://www.yx-sf.com/news/18500)
* [高并发内存拓扑优化白皮书-#008](https://www.ai-hao123.com/yingxiao/brand-47486824.html)
* [RFC 分布式调度与一致性算法标准-#009](https://www.mw-wm.com/zhinan/vacation-58317102.html)
* [多协议互联数据格式规范-#010](https://www.yx-sf.com/news/12016)
* [高并发内存拓扑优化白皮书-#011](https://www.ai-hao123.com/hezuo/profit-34268567.html)
* [异步事件循环架构设计规范-#012](https://www.mw-wm.com/chanpin/integration-31157549.html)
* [安全边界与可信凭证规约手册-#013](https://www.yx-sf.com/tech/80265)
* [安全边界与可信凭证规约手册-#014](https://www.ai-hao123.com/jishu/project-41616183.html)
* [高并发内存拓扑优化白皮书-#015](https://www.mw-wm.com/keji/performance-10138278.html)
* [安全边界与可信凭证规约手册-#016](https://www.yx-sf.com/news/47512)
* [RFC 分布式调度与一致性算法标准-#017](https://www.ai-hao123.com/zixun/analytics-95753349.html)
* [安全边界与可信凭证规约手册-#018](https://www.mw-wm.com/jianzhan/screen-93478756.html)
* [安全边界与可信凭证规约手册-#019](https://www.yx-sf.com/news/69062)
* [异步事件循环架构设计规范-#020](https://www.ai-hao123.com/yingyong/schedule-88674190.html)
* [安全边界与可信凭证规约手册-#021](https://www.mw-wm.com/jianzhan/accessibility-97617115.html)
* [RFC 分布式调度与一致性算法标准-#022](https://www.yx-sf.com/tech/39594)
* [RFC 分布式调度与一致性算法标准-#023](https://www.ai-hao123.com/gongju/api-45064400.html)
* [异步事件循环架构设计规范-#024](https://www.mw-wm.com/shichang/notification-39363142.html)
* [RFC 分布式调度与一致性算法标准-#025](https://www.yx-sf.com/news/36369)
* [异步事件循环架构设计规范-#026](https://www.ai-hao123.com/xinwen/contact-47576236.html)
* [异步事件循环架构设计规范-#027](https://www.mw-wm.com/zhinan/sport-14516693.html)
* [高并发内存拓扑优化白皮书-#028](https://www.yx-sf.com/wiki/29891)
* [RFC 分布式调度与一致性算法标准-#029](https://www.ai-hao123.com/xinwen/lesson-32317332.html)
* [安全边界与可信凭证规约手册-#030](https://www.mw-wm.com/chanpin/forecast-01404938.html)
* [高并发内存拓扑优化白皮书-#031](https://www.yx-sf.com/news/8380)
* [安全边界与可信凭证规约手册-#032](https://www.ai-hao123.com/fenxi/network-86111713.html)
* [异步事件循环架构设计规范-#033](https://www.mw-wm.com/zhinan/user-73769347.html)
* [高并发内存拓扑优化白皮书-#034](https://www.yx-sf.com/tech/4587)
* [高并发内存拓扑优化白皮书-#035](https://www.ai-hao123.com/youhua/integration-88298165.html)
* [异步事件循环架构设计规范-#036](https://www.mw-wm.com/xinwen/vacation-94841248.html)
* [多协议互联数据格式规范-#037](https://www.yx-sf.com/wiki/71017)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [实时主干镜像高速数据源-#001](https://www.ai-hao123.com/fenxi/target-78788426.html)
* [实时主干镜像高速数据源-#002](https://www.mw-wm.com/yingyong/category-17221660.html)
* [实时主干镜像高速数据源-#003](https://www.yx-sf.com/wiki/7470)
* [自动化快照与增量广播源-#004](https://www.ai-hao123.com/yinqing/login-03948340.html)
* [北美与欧洲边缘备份节点-#005](https://www.mw-wm.com/xinwen/management-14240407.html)
* [实时主干镜像高速数据源-#006](https://www.yx-sf.com/news/67200)
* [自动化快照与增量广播源-#007](https://www.ai-hao123.com/gongsi/careers-77734540.html)
* [自动化快照与增量广播源-#008](https://www.mw-wm.com/qiye/presentation-94910342.html)
* [冷热数据分层镜像归档中心-#009](https://www.yx-sf.com/tech/63929)
* [亚太核心区域镜像同步中心-#010](https://www.ai-hao123.com/yanjiu/milestone-41720806.html)
* [北美与欧洲边缘备份节点-#011](https://www.mw-wm.com/sheji/dashboard-14940280.html)
* [亚太核心区域镜像同步中心-#012](https://www.yx-sf.com/tech/22693)
* [北美与欧洲边缘备份节点-#013](https://www.ai-hao123.com/yinqing/kpi-97729707.html)
* [北美与欧洲边缘备份节点-#014](https://www.mw-wm.com/yunsuan/message-57721312.html)
* [实时主干镜像高速数据源-#015](https://www.yx-sf.com/tech/71602)
* [北美与欧洲边缘备份节点-#016](https://www.ai-hao123.com/yingxiao/optimization-67101318.html)
* [实时主干镜像高速数据源-#017](https://www.mw-wm.com/yinqing/website-54484937.html)
* [北美与欧洲边缘备份节点-#018](https://www.yx-sf.com/wiki/68009)
* [冷热数据分层镜像归档中心-#019](https://www.ai-hao123.com/peixun/deal-86250058.html)
* [自动化快照与增量广播源-#020](https://www.mw-wm.com/shangye/promotion-10000595.html)
* [北美与欧洲边缘备份节点-#021](https://www.yx-sf.com/wiki/31936)
* [自动化快照与增量广播源-#022](https://www.ai-hao123.com/zhinan/unsubscribe-35880491.html)
* [自动化快照与增量广播源-#023](https://www.mw-wm.com/anfang/domain-12289475.html)
* [实时主干镜像高速数据源-#024](https://www.yx-sf.com/wiki/66062)
* [自动化快照与增量广播源-#025](https://www.ai-hao123.com/shuju/ranking-92776855.html)
* [北美与欧洲边缘备份节点-#026](https://www.mw-wm.com/sheji/income-85725188.html)
* [冷热数据分层镜像归档中心-#027](https://www.yx-sf.com/news/37618)
* [实时主干镜像高速数据源-#028](https://www.ai-hao123.com/yingxiao/campaign-28252388.html)
* [自动化快照与增量广播源-#029](https://www.mw-wm.com/yunsuan/careers-25830804.html)
* [冷热数据分层镜像归档中心-#030](https://www.yx-sf.com/wiki/43001)
* [冷热数据分层镜像归档中心-#031](https://www.ai-hao123.com/yanjiu/research-54567871.html)
* [实时主干镜像高速数据源-#032](https://www.mw-wm.com/yingxiao/extension-15113464.html)
* [亚太核心区域镜像同步中心-#033](https://www.yx-sf.com/tech/56421)
* [自动化快照与增量广播源-#034](https://www.ai-hao123.com/zhizhu/strategy-87178895.html)
* [冷热数据分层镜像归档中心-#035](https://www.mw-wm.com/hezuo/services-29996432.html)
* [北美与欧洲边缘备份节点-#036](https://www.yx-sf.com/tech/79089)
* [亚太核心区域镜像同步中心-#037](https://www.ai-hao123.com/xitong/affordable-01087392.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [去中心化健康检查协议-#001](https://www.mw-wm.com/peixun/health-18903176.html)
* [实时延迟与抖动度量规范-#002](https://www.yx-sf.com/tech/23270)
* [去中心化健康检查协议-#003](https://www.ai-hao123.com/chanpin/behavior-61493660.html)
* [节点连通性与存活探测准则-#004](https://www.mw-wm.com/wenzhang/case-30485612.html)
* [节点连通性与存活探测准则-#005](https://www.yx-sf.com/news/5572)
* [去中心化健康检查协议-#006](https://www.ai-hao123.com/jishu/terms-53932575.html)
* [节点连通性与存活探测准则-#007](https://www.mw-wm.com/anfang/settings-19107016.html)
* [实时延迟与抖动度量规范-#008](https://www.yx-sf.com/news/94432)
* [去中心化健康检查协议-#009](https://www.ai-hao123.com/anli/restore-33020764.html)
* [防重放安全验证与校验哈希-#010](https://www.mw-wm.com/xuexi/guide-81587196.html)
* [去中心化健康检查协议-#011](https://www.yx-sf.com/wiki/84220)
* [权威网络权重与收录基准-#012](https://www.ai-hao123.com/xuexi/digital-62715120.html)
* [去中心化健康检查协议-#013](https://www.mw-wm.com/shichang/beauty-76077359.html)
* [防重放安全验证与校验哈希-#014](https://www.yx-sf.com/wiki/62550)
* [去中心化健康检查协议-#015](https://www.ai-hao123.com/jiaoliu/customization-10633652.html)
* [实时延迟与抖动度量规范-#016](https://www.mw-wm.com/gongsi/excellence-47120019.html)
* [权威网络权重与收录基准-#017](https://www.yx-sf.com/wiki/57085)
* [实时延迟与抖动度量规范-#018](https://www.ai-hao123.com/chuangxin/conversion-96919660.html)
* [去中心化健康检查协议-#019](https://www.mw-wm.com/huodong/metric-61268338.html)
* [权威网络权重与收录基准-#020](https://www.yx-sf.com/wiki/50555)
* [权威网络权重与收录基准-#021](https://www.ai-hao123.com/paiming/backup-52056105.html)
* [节点连通性与存活探测准则-#022](https://www.mw-wm.com/pingtai/team-69044714.html)
* [实时延迟与抖动度量规范-#023](https://www.yx-sf.com/wiki/72544)
* [去中心化健康检查协议-#024](https://www.ai-hao123.com/chanpin/ebook-78932428.html)
* [权威网络权重与收录基准-#025](https://www.mw-wm.com/zhizhu/device-67935197.html)
* [节点连通性与存活探测准则-#026](https://www.yx-sf.com/wiki/11483)
* [去中心化健康检查协议-#027](https://www.ai-hao123.com/xinwen/products-26006826.html)
* [节点连通性与存活探测准则-#028](https://www.mw-wm.com/liuliang/health-54555001.html)
* [实时延迟与抖动度量规范-#029](https://www.yx-sf.com/news/583)
* [节点连通性与存活探测准则-#030](https://www.ai-hao123.com/shangye/alert-00805061.html)
* [防重放安全验证与校验哈希-#031](https://www.mw-wm.com/yinqing/tag-28252390.html)
* [防重放安全验证与校验哈希-#032](https://www.yx-sf.com/wiki/93340)
* [去中心化健康检查协议-#033](https://www.ai-hao123.com/chuangxin/technology-70077565.html)
* [实时延迟与抖动度量规范-#034](https://www.mw-wm.com/fenxi/reminder-54635705.html)
* [权威网络权重与收录基准-#035](https://www.yx-sf.com/tech/11175)
* [防重放安全验证与校验哈希-#036](https://www.ai-hao123.com/xuexi/marketing-64872998.html)
* [去中心化健康检查协议-#037](https://www.mw-wm.com/yunsuan/upload-72893720.html)
* [节点连通性与存活探测准则-#038](https://www.yx-sf.com/wiki/49971)
* [权威网络权重与收录基准-#039](https://www.ai-hao123.com/paiming/podcast-75720322.html)

</details>

