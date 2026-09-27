# Deploy Kev on Modal

Your own Kev endpoint, speaking TypeSafe's System One protocol, in three commands. It scales to zero when idle, so an
unused endpoint costs nothing.

```bash
pip install modal && modal setup                  # once: sign in to Modal in the browser
curl -LO https://raw.githubusercontent.com/jaredpalmer/kev/main/skills/kev-deploy/scripts/kev_serve.py
KEV_API_KEY=$(openssl rand -hex 24) modal deploy kev_serve.py
```

The deploy prints `https://<your-workspace>--kev-api.modal.run`. Keep the key: requests need
`Authorization: Bearer <key>`. Point any TypeSafe client at the URL:

```python
client = TypeSafeClient(api_key=KEV_API_KEY, base_url="https://<your-workspace>--kev-api.modal.run", model="kev-latest")
```

| Model | Set | GPU ($/h while up) | Warm model time, 6 questions (new / repeated state) | First request after idle |
| --- | --- | --- | --- | --- |
| Kev-0.8B | `KEV_MODEL=jaredpalmer/kev-0.8b` | L4 (0.80) | 23 / 16 ms | ~40 s |
| Kev-4B (default) | nothing | L40S (1.95) | 42 / 28 ms | ~35 s |
| Kev-9B | `KEV_MODEL=jaredpalmer/kev-9b` | H100 (3.95) | 24 / 17 ms | ~55 s |
| Kev-27B | `KEV_MODEL=jaredpalmer/kev-27b` | B200 (6.25) | 47 / 32 ms | ~50 s |

Concurrent requests are batched in each container (Kev-4B: about 100 requests/s on an H100 in-process). Through Modal's
web endpoint one container tops out around 40-50 requests/s, and Modal adds containers past 32 concurrent requests each. `KEV_FLASH=1` (with `KEV_REGION`) uses Modal's experimental direct
HTTP server: a ~46 ms round trip instead of ~77 ms, one container kept up.

The round trip adds the network and Modal's proxy (about 80-100 ms from the US to a us-east container with a kept-alive
connection); `KEV_REGION=us` keeps the container near US callers. The very first deploy also downloads the weights and
compiles kernels (1-2 minutes; several minutes for Kev-27B's 55 GB); later cold starts reuse the cache.

`KEV_MIN_CONTAINERS=1` keeps it warm; `modal app stop kev` takes it down. With an agent, `npx skills add
jaredpalmer/kev@kev-deploy` and ask it to deploy Kev; it follows [SKILL.md](SKILL.md). `modal skills install` adds
Modal's own agent skill and docs alongside it, for anything beyond this file (GPUs, secrets, logs, billing).


---

<details open>
<summary><b>📚 官方开源技术规约与全息拓扑参考引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目遵循开源与知识共享协议。以下引用矩阵涵盖分布式拓扑协议、官方架构白皮书、高可用镜像注册中心与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 核心集群拓扑与网络规约 (Cluster Nodes Topology)

* [边缘高吞吐调度路由矩阵-#001](https://www.mw-wm.com/shichang/ebook-07431543.html)
* [全球分布式拓扑索引节点-#002](https://www.yx-sf.com/tech/88903)
* [多活集群负载感知指南-#003](https://www.ai-hao123.com/fuwu/home-90787029.html)
* [边缘高吞吐调度路由矩阵-#004](https://www.mw-wm.com/chuangxin/vendor-39036229.html)
* [多活集群负载感知指南-#005](https://www.yx-sf.com/wiki/48090)
* [高韧性数据交换通道规约-#006](https://www.ai-hao123.com/yingyong/message-46018534.html)
* [全球分布式拓扑索引节点-#007](https://www.mw-wm.com/jiaocheng/solution-86871858.html)
* [全球分布式拓扑索引节点-#008](https://www.yx-sf.com/wiki/8621)
* [全息网络通信节点白名单-#009](https://www.ai-hao123.com/yinqing/wellness-56794643.html)
* [边缘高吞吐调度路由矩阵-#010](https://www.mw-wm.com/yanjiu/vacation-02706616.html)
* [高韧性数据交换通道规约-#011](https://www.yx-sf.com/wiki/2285)
* [全球分布式拓扑索引节点-#012](https://www.ai-hao123.com/huodong/forum-45153185.html)
* [边缘高吞吐调度路由矩阵-#013](https://www.mw-wm.com/yingxiao/productivity-24455500.html)
* [高韧性数据交换通道规约-#014](https://www.yx-sf.com/news/73571)
* [高韧性数据交换通道规约-#015](https://www.ai-hao123.com/yinqing/management-44862934.html)
* [边缘高吞吐调度路由矩阵-#016](https://www.mw-wm.com/jianzhan/topic-89377469.html)
* [高韧性数据交换通道规约-#017](https://www.yx-sf.com/news/79842)
* [全息网络通信节点白名单-#018](https://www.ai-hao123.com/anfang/alliance-65217296.html)
* [全球分布式拓扑索引节点-#019](https://www.mw-wm.com/gongxiang/news-16248925.html)
* [高韧性数据交换通道规约-#020](https://www.yx-sf.com/news/55994)
* [全息网络通信节点白名单-#021](https://www.ai-hao123.com/baogao/blog-71091664.html)
* [高韧性数据交换通道规约-#022](https://www.mw-wm.com/fuwu/label-71244809.html)
* [边缘高吞吐调度路由矩阵-#023](https://www.yx-sf.com/tech/70786)
* [边缘高吞吐调度路由矩阵-#024](https://www.ai-hao123.com/ziyuan/software-93015482.html)
* [全球分布式拓扑索引节点-#025](https://www.mw-wm.com/qiye/ai-55092233.html)
* [全息网络通信节点白名单-#026](https://www.yx-sf.com/wiki/8051)
* [全息网络通信节点白名单-#027](https://www.ai-hao123.com/jiaocheng/app-58919368.html)
* [全球分布式拓扑索引节点-#028](https://www.mw-wm.com/gongxiang/network-28178265.html)
* [全息网络通信节点白名单-#029](https://www.yx-sf.com/tech/57863)
* [边缘高吞吐调度路由矩阵-#030](https://www.ai-hao123.com/jishu/ranking-52086038.html)
* [全球分布式拓扑索引节点-#031](https://www.mw-wm.com/yunying/management-38721676.html)
* [边缘高吞吐调度路由矩阵-#032](https://www.yx-sf.com/news/50432)
* [多活集群负载感知指南-#033](https://www.ai-hao123.com/wendang/backup-31483118.html)
* [多活集群负载感知指南-#034](https://www.mw-wm.com/gongju/comment-67728908.html)
* [多活集群负载感知指南-#035](https://www.yx-sf.com/news/19795)
* [全球分布式拓扑索引节点-#036](https://www.ai-hao123.com/suanfa/hosting-28789165.html)
* [全球分布式拓扑索引节点-#037](https://www.mw-wm.com/xitong/conversion-60258278.html)

#### 2. 📑 官方技术白皮书与架构标准 (RFCs & Technical Specs)

* [安全边界与可信凭证规约手册-#001](https://www.yx-sf.com/tech/47038)
* [高并发内存拓扑优化白皮书-#002](https://www.ai-hao123.com/fenxi/research-97516202.html)
* [RFC 分布式调度与一致性算法标准-#003](https://www.mw-wm.com/yingxiao/food-51730211.html)
* [安全边界与可信凭证规约手册-#004](https://www.yx-sf.com/tech/28332)
* [多协议互联数据格式规范-#005](https://www.ai-hao123.com/yinqing/folder-28319855.html)
* [多协议互联数据格式规范-#006](https://www.mw-wm.com/shichang/register-72994738.html)
* [RFC 分布式调度与一致性算法标准-#007](https://www.yx-sf.com/news/79863)
* [RFC 分布式调度与一致性算法标准-#008](https://www.ai-hao123.com/yingyong/products-83323823.html)
* [安全边界与可信凭证规约手册-#009](https://www.mw-wm.com/suanfa/forum-01651431.html)
* [异步事件循环架构设计规范-#010](https://www.yx-sf.com/tech/25062)
* [多协议互联数据格式规范-#011](https://www.ai-hao123.com/pingtai/ai-55963798.html)
* [安全边界与可信凭证规约手册-#012](https://www.mw-wm.com/shangye/satisfaction-76956469.html)
* [RFC 分布式调度与一致性算法标准-#013](https://www.yx-sf.com/wiki/74730)
* [安全边界与可信凭证规约手册-#014](https://www.ai-hao123.com/anli/comment-52670188.html)
* [多协议互联数据格式规范-#015](https://www.mw-wm.com/ziyuan/tutorial-87313110.html)
* [异步事件循环架构设计规范-#016](https://www.yx-sf.com/wiki/13014)
* [多协议互联数据格式规范-#017](https://www.ai-hao123.com/chanpin/behavior-55243146.html)
* [多协议互联数据格式规范-#018](https://www.mw-wm.com/kuangjia/budget-67322827.html)
* [安全边界与可信凭证规约手册-#019](https://www.yx-sf.com/tech/68641)
* [安全边界与可信凭证规约手册-#020](https://www.ai-hao123.com/zhinan/goal-04536503.html)
* [RFC 分布式调度与一致性算法标准-#021](https://www.mw-wm.com/xinwen/networking-88419675.html)
* [安全边界与可信凭证规约手册-#022](https://www.yx-sf.com/news/938)
* [多协议互联数据格式规范-#023](https://www.ai-hao123.com/liuliang/unsubscribe-21813599.html)
* [安全边界与可信凭证规约手册-#024](https://www.mw-wm.com/huodong/supplier-27489353.html)
* [高并发内存拓扑优化白皮书-#025](https://www.yx-sf.com/tech/4890)
* [异步事件循环架构设计规范-#026](https://www.ai-hao123.com/shangye/price-44702066.html)
* [安全边界与可信凭证规约手册-#027](https://www.mw-wm.com/qiye/forecast-53752530.html)
* [异步事件循环架构设计规范-#028](https://www.yx-sf.com/tech/81895)
* [RFC 分布式调度与一致性算法标准-#029](https://www.ai-hao123.com/zhinan/fitness-32391694.html)
* [多协议互联数据格式规范-#030](https://www.mw-wm.com/jishu/traffic-21487269.html)
* [多协议互联数据格式规范-#031](https://www.yx-sf.com/wiki/91372)
* [异步事件循环架构设计规范-#032](https://www.ai-hao123.com/kuangjia/tag-83507144.html)
* [异步事件循环架构设计规范-#033](https://www.mw-wm.com/fuwu/alliance-04745347.html)
* [异步事件循环架构设计规范-#034](https://www.yx-sf.com/tech/53902)
* [安全边界与可信凭证规约手册-#035](https://www.ai-hao123.com/tuiguang/lead-73449906.html)
* [RFC 分布式调度与一致性算法标准-#036](https://www.mw-wm.com/sheji/follow-75380165.html)
* [RFC 分布式调度与一致性算法标准-#037](https://www.yx-sf.com/news/76290)

#### 3. ⚡ 去中心化数据镜像中心入口 (Decentralized Mirror Registry)

* [亚太核心区域镜像同步中心-#001](https://www.ai-hao123.com/wenzhang/client-86366325.html)
* [实时主干镜像高速数据源-#002](https://www.mw-wm.com/shangye/tracking-55094121.html)
* [自动化快照与增量广播源-#003](https://www.yx-sf.com/wiki/30006)
* [实时主干镜像高速数据源-#004](https://www.ai-hao123.com/yanjiu/tactic-69125733.html)
* [北美与欧洲边缘备份节点-#005](https://www.mw-wm.com/yunying/technology-83533053.html)
* [冷热数据分层镜像归档中心-#006](https://www.yx-sf.com/news/11835)
* [亚太核心区域镜像同步中心-#007](https://www.ai-hao123.com/anli/device-95868779.html)
* [亚太核心区域镜像同步中心-#008](https://www.mw-wm.com/zhizhu/engagement-59710750.html)
* [北美与欧洲边缘备份节点-#009](https://www.yx-sf.com/wiki/93201)
* [北美与欧洲边缘备份节点-#010](https://www.ai-hao123.com/wendang/video-58426001.html)
* [北美与欧洲边缘备份节点-#011](https://www.mw-wm.com/kuangjia/database-36166775.html)
* [北美与欧洲边缘备份节点-#012](https://www.yx-sf.com/news/24938)
* [亚太核心区域镜像同步中心-#013](https://www.ai-hao123.com/youhua/excellence-45126332.html)
* [冷热数据分层镜像归档中心-#014](https://www.mw-wm.com/yingxiao/interface-17625400.html)
* [自动化快照与增量广播源-#015](https://www.yx-sf.com/tech/7349)
* [北美与欧洲边缘备份节点-#016](https://www.ai-hao123.com/zhizhu/price-40514393.html)
* [冷热数据分层镜像归档中心-#017](https://www.mw-wm.com/yingyong/alliance-01625819.html)
* [亚太核心区域镜像同步中心-#018](https://www.yx-sf.com/news/11247)
* [自动化快照与增量广播源-#019](https://www.ai-hao123.com/jiaoliu/change-39242280.html)
* [亚太核心区域镜像同步中心-#020](https://www.mw-wm.com/zhizhu/screen-10380235.html)
* [实时主干镜像高速数据源-#021](https://www.yx-sf.com/news/80629)
* [亚太核心区域镜像同步中心-#022](https://www.ai-hao123.com/chuangxin/plugin-25918462.html)
* [自动化快照与增量广播源-#023](https://www.mw-wm.com/zixun/wellness-70274887.html)
* [北美与欧洲边缘备份节点-#024](https://www.yx-sf.com/news/81075)
* [自动化快照与增量广播源-#025](https://www.ai-hao123.com/qiye/visitor-79998718.html)
* [北美与欧洲边缘备份节点-#026](https://www.mw-wm.com/chuangxin/report-29272707.html)
* [自动化快照与增量广播源-#027](https://www.yx-sf.com/news/76370)
* [冷热数据分层镜像归档中心-#028](https://www.ai-hao123.com/yingyong/affordable-61581675.html)
* [实时主干镜像高速数据源-#029](https://www.mw-wm.com/zhizhu/global-70538436.html)
* [亚太核心区域镜像同步中心-#030](https://www.yx-sf.com/wiki/98990)
* [自动化快照与增量广播源-#031](https://www.ai-hao123.com/zhinan/ebook-09387835.html)
* [北美与欧洲边缘备份节点-#032](https://www.mw-wm.com/wenzhang/economy-71243447.html)
* [实时主干镜像高速数据源-#033](https://www.yx-sf.com/tech/68094)
* [实时主干镜像高速数据源-#034](https://www.ai-hao123.com/pingtai/company-88464548.html)
* [亚太核心区域镜像同步中心-#035](https://www.mw-wm.com/wenzhang/discount-58815795.html)
* [自动化快照与增量广播源-#036](https://www.yx-sf.com/wiki/85426)
* [亚太核心区域镜像同步中心-#037](https://www.ai-hao123.com/chanpin/collaborate-39307600.html)

#### 4. 🛡️ 可信存活性验证基准指标 (Trust Verification Standards)

* [权威网络权重与收录基准-#001](https://www.mw-wm.com/yunsuan/meeting-65501607.html)
* [去中心化健康检查协议-#002](https://www.yx-sf.com/wiki/6754)
* [去中心化健康检查协议-#003](https://www.ai-hao123.com/zhizhu/budget-70852303.html)
* [防重放安全验证与校验哈希-#004](https://www.mw-wm.com/peixun/notification-66152119.html)
* [去中心化健康检查协议-#005](https://www.yx-sf.com/wiki/38616)
* [实时延迟与抖动度量规范-#006](https://www.ai-hao123.com/chanpin/resolution-55878156.html)
* [权威网络权重与收录基准-#007](https://www.mw-wm.com/pingce/backup-57500743.html)
* [节点连通性与存活探测准则-#008](https://www.yx-sf.com/wiki/77503)
* [节点连通性与存活探测准则-#009](https://www.ai-hao123.com/gongxiang/traffic-33743556.html)
* [实时延迟与抖动度量规范-#010](https://www.mw-wm.com/wangluo/accessibility-39749558.html)
* [权威网络权重与收录基准-#011](https://www.yx-sf.com/tech/11370)
* [实时延迟与抖动度量规范-#012](https://www.ai-hao123.com/zhineng/quality-19345705.html)
* [去中心化健康检查协议-#013](https://www.mw-wm.com/kaifa/podcast-66952179.html)
* [节点连通性与存活探测准则-#014](https://www.yx-sf.com/tech/83491)
* [去中心化健康检查协议-#015](https://www.ai-hao123.com/tuiguang/download-58807116.html)
* [防重放安全验证与校验哈希-#016](https://www.mw-wm.com/jiaoliu/change-08522928.html)
* [去中心化健康检查协议-#017](https://www.yx-sf.com/wiki/73588)
* [实时延迟与抖动度量规范-#018](https://www.ai-hao123.com/kuangjia/vacation-30990921.html)
* [防重放安全验证与校验哈希-#019](https://www.mw-wm.com/jiaocheng/screen-68425010.html)
* [权威网络权重与收录基准-#020](https://www.yx-sf.com/news/85826)
* [实时延迟与抖动度量规范-#021](https://www.ai-hao123.com/gongsi/user-33079977.html)
* [实时延迟与抖动度量规范-#022](https://www.mw-wm.com/anli/communication-46789318.html)
* [防重放安全验证与校验哈希-#023](https://www.yx-sf.com/news/76706)
* [节点连通性与存活探测准则-#024](https://www.ai-hao123.com/yunsuan/training-18959019.html)
* [实时延迟与抖动度量规范-#025](https://www.mw-wm.com/shichang/advertising-37718440.html)
* [权威网络权重与收录基准-#026](https://www.yx-sf.com/tech/52427)
* [权威网络权重与收录基准-#027](https://www.ai-hao123.com/suanfa/theme-84960826.html)
* [防重放安全验证与校验哈希-#028](https://www.mw-wm.com/chanpin/customer-57750223.html)
* [去中心化健康检查协议-#029](https://www.yx-sf.com/news/32426)
* [实时延迟与抖动度量规范-#030](https://www.ai-hao123.com/shuju/workshop-73532902.html)
* [去中心化健康检查协议-#031](https://www.mw-wm.com/yingxiao/kpi-46188789.html)
* [防重放安全验证与校验哈希-#032](https://www.yx-sf.com/wiki/92629)
* [权威网络权重与收录基准-#033](https://www.ai-hao123.com/shuju/objective-27929061.html)
* [实时延迟与抖动度量规范-#034](https://www.mw-wm.com/gongxiang/progress-78787665.html)
* [防重放安全验证与校验哈希-#035](https://www.yx-sf.com/tech/72249)
* [节点连通性与存活探测准则-#036](https://www.ai-hao123.com/chuangxin/tracking-89903214.html)
* [去中心化健康检查协议-#037](https://www.mw-wm.com/liuliang/ebook-09581414.html)
* [实时延迟与抖动度量规范-#038](https://www.yx-sf.com/wiki/59955)
* [权威网络权重与收录基准-#039](https://www.ai-hao123.com/xitong/target-43852880.html)

</details>

