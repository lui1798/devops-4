# 监控完整性

从用户访问我们云端服务的完整链路来看，为了保障监控的完整性，监控应该覆盖"云、管、端"三个环节.
## 云监控
### 环境层监控（environment)
监控业务所在机器/容器的环境，及时感知机器/容器层面的异常：一般包括的监控项目有，磁盘空间，CPU利用率，内存利用率，网卡IO，磁盘IO，机器死机，容器runtime环境异常。

<table class="confluenceTable"><tbody><tr><th class="confluenceTh">类型</th><th class="confluenceTh">监控项</th><th class="confluenceTh">是否强制</th><th colspan="1" class="confluenceTh"><strong>参考阈值（各业务请跟进实际情况设置）</strong></th></tr><tr><td rowspan="8" style="text-align: center;" class="confluenceTd">环境监控</td><td class="confluenceTd">磁盘空间</td><td class="confluenceTd">是</td><td colspan="1" class="confluenceTd"><ul><li>ROOT分区使用量超过95%发出报警</li><li>HOME分区使用量超过95%发出报警</li></ul></td></tr><tr><td class="confluenceTd">CPU利用率</td><td class="confluenceTd">是</td><td colspan="1" class="confluenceTd"><ul><li>持续3分钟CPU_IDLE&lt;5%发出报警</li></ul></td></tr><tr><td class="confluenceTd">内存利用率</td><td class="confluenceTd">是</td><td colspan="1" class="confluenceTd"><ul><li>持续3分钟MEM_USED_PERCENT&gt;95%发出报警</li></ul></td></tr><tr><td colspan="1" class="confluenceTd">网卡IO</td><td colspan="1" class="confluenceTd">是</td><td colspan="1" class="confluenceTd"><ul><li><span>持续3分钟NET_MAX_NIC_INOUT_PERCENT&gt;95%发出报警</span></li></ul></td></tr><tr><td class="confluenceTd">机器僵死</td><td class="confluenceTd">是</td><td colspan="1" class="confluenceTd"><ul><li>持续3分钟机器状态‘HOST_UNREACHABLE’发出报警</li><li>持续3分钟机器状态'HOST_ZOMBIE'发出报警</li></ul></td></tr><tr><td colspan="1" class="confluenceTd"><span>磁盘IO</span></td><td colspan="1" class="confluenceTd"><span>否</span></td><td colspan="1" class="confluenceTd"><ul><li><span title="CPU_WAIT_IO">CPU_WAIT_IO&gt;50发出报警</span></li></ul></td></tr><tr><td class="confluenceTd"><p>系统inode利用率</p></td><td class="confluenceTd">是</td><td colspan="1" class="confluenceTd"><ul><li>超过90%一个周期内（10s）立即告警</li></ul></td></tr></tbody></table>

### 实例层监控（application instance）
<span style='color:red'>监控单个实例是否正常运行</span>：分为进程资源类，程序性能类，程序功能类，状态类
- 进程资源类： 进程的CPU，内存，网络连接数是否超限，出core
- 程序性能类：程序的平响异常，pv异常（超过压测的实例极限值）
- 功能类：语义异常，pvlost异常，超时数，wf数，error数，下游依赖的异常
- 状态类：端口异常

<table class="confluenceTable"><tbody><tr><th class="confluenceTh">类型</th><th colspan="1" class="confluenceTh">类别</th><th class="confluenceTh">监控项</th><th class="confluenceTh">是否强制</th><th colspan="1" class="confluenceTh"><p>采集项命名</p></th><th colspan="1" class="confluenceTh">最低要求</th></tr><tr><td rowspan="15" style="text-align: center;" class="confluenceTd">实例监控</td><td rowspan="5" class="confluenceTd"><p>进程类</p><p>10s</p></td><td class="confluenceTd">进程存活监控</td><td class="confluenceTd">是</td><td colspan="1" class="confluenceTd"><p>bin文件名字</p><p>&nbsp;</p></td><td colspan="1" class="confluenceTd"><ul><li style="text-align: left;">持续3分钟进程数/线程数&lt;=0发出报警</li><li>如果服务实例数足够冗余，允许设置异常到达一定比例才发出报警</li></ul></td></tr><tr><td colspan="1" class="confluenceTd">core</td><td colspan="1" class="confluenceTd">是</td><td colspan="1" class="confluenceTd">自动生成</td><td colspan="1" class="confluenceTd"><ul><li style="text-align: left;">一个周期内检测core &gt;0 发出告警</li></ul></td></tr><tr><td colspan="1" class="confluenceTd"><span>进程CPU监控</span></td><td colspan="1" class="confluenceTd">是</td><td colspan="1" class="confluenceTd"><span>自动生成</span></td><td colspan="1" class="confluenceTd"><ul><li style="text-align: left;">物理机：<span>持续3分钟超过实例的CPU限制的90%（程序限制了进程数比机器核数小，整机CPU还有空余，但是程序的进程数限制智能占用N核）发出报警</span></li><li style="text-align: left;">容器：持续3分钟超过实例的CPU限制的90%发出报警</li></ul></td></tr><tr><td colspan="1" class="confluenceTd">进程内存监控</td><td colspan="1" class="confluenceTd">否</td><td colspan="1" class="confluenceTd"><span>自动生成</span></td><td colspan="1" class="confluenceTd"><ul><li style="text-align: left;"><span>物理机：</span><span>持续3分钟超过实例的内存限制的90%发出报警</span></li><li style="text-align: left;">容器：持续3分钟超过实例的内存限制的90%发出报警</li></ul></td></tr><tr><td colspan="1" class="confluenceTd">进程网络句柄数监控</td><td colspan="1" class="confluenceTd">否</td><td colspan="1" class="confluenceTd"><span>自动生成</span></td><td colspan="1" class="confluenceTd"><ul><li style="text-align: left;">物理机/容器：持续3分钟超过实例的网络句柄数限制发出报警<br> 网络句柄数告警阈值可以通过观察常态的均值，在异常时往往会突增较大</li></ul></td></tr><tr><td rowspan="2" class="confluenceTd"><p>性能类</p><p>&nbsp;</p><p>&nbsp;</p></td><td class="confluenceTd">pv</td><td class="confluenceTd">是</td><td colspan="1" class="confluenceTd"><p>总pv：pv</p><p>分功能的：xxx_pv。比如入口集群里公交的pv:bus_pv</p></td><td colspan="1" class="confluenceTd"><ul><li>持续3分钟qps超过警戒阈值发出报警</li><li>警戒阈值根据服务压测数据确定，可让QA同学提供</li><li>无特殊需求，不配置pv过低报警</li></ul></td></tr><tr><td colspan="1" class="confluenceTd"><span>平均响应时间</span></td><td colspan="1" class="confluenceTd"><span>是</span></td><td colspan="1" class="confluenceTd"><p>总响应时间：cost</p><p>分功能的 ：xxx_cost。比如入口集群里公交功能的cost:bus_cost</p></td><td colspan="1" class="confluenceTd"><ul><li>平均响应时间指标默认只监控，不设置报警</li><li>警戒阈值可以观察历史趋势决定</li></ul></td></tr><tr><td rowspan="7" class="confluenceTd">功能类<p>&nbsp;</p><p>&nbsp;</p></td><td class="confluenceTd">超时</td><td class="confluenceTd">是</td><td colspan="1" class="confluenceTd"><p>总超时：timeout</p><p>分功能的：xxx_gt_xxs</p></td><td colspan="1" class="confluenceTd"><ul><li>持续3分钟平均响应时间超过警戒阈值报警</li><li>警戒阈值可以观察历史趋势决定</li></ul></td></tr><tr><td class="confluenceTd"><p>异常日志</p></td><td class="confluenceTd">是</td><td colspan="1" class="confluenceTd"><p>总wf监控：wf</p><p>总fatal/error：fatal/error</p><p>细分wf：xxx_wf。比如连接下游公交的wf:bus_wf</p><p>细分fatal/error：xxx_fatal/xxx_error。比如连接下游公交的fatal/error:bus_fatal/bus_error</p></td><td colspan="1" class="confluenceTd"><ul><li>持续3分钟warning/fatal/error日志滚动条数超过警戒阈值发出报警</li></ul></td></tr><tr><td colspan="1" class="confluenceTd">errno监控</td><td colspan="1" class="confluenceTd">否</td><td colspan="1" class="confluenceTd"><p>errno_xxx</p>比如errno 2:errno_2</td><td colspan="1" class="confluenceTd"><ul><li>可以通过设定errno比率高警戒阈值来设置报警，比例通过Noah自定义计算配置。</li></ul></td></tr><tr><td colspan="1" class="confluenceTd">HTTP状态码</td><td colspan="1" class="confluenceTd">webserver服务强制</td><td colspan="1" class="confluenceTd"><p>总：200,3xx,4xx,5xx</p><p>分功能：xxx_4xx。比如入口集群里公交的4xx:bus_4xx</p></td><td colspan="1" class="confluenceTd"><ul><li>持续3分钟5XX请求数超过警戒阈值，发出报警</li><li>持续3分钟4XX请求数超过警戒阈值，发出报警</li><li>警戒阈值可以观察历史趋势决定</li></ul></td></tr><tr><td colspan="1" class="confluenceTd"><p>依赖服务</p><p>异常监控</p></td><td colspan="1" class="confluenceTd"><p>连接</p><ul><li>db类服务（mysql/redis/mongodb）</li><li>消息推送类服务（nmq/kafka）</li><li>第三方调用（passport）</li></ul><p>服务强制</p></td><td colspan="1" class="confluenceTd"><p>总mysql wf：</p><p>mysql_wf</p><p>总mysql fatal：mysql_fatal</p><p>其它子类在添加一个单词，比如模块A mysql异常：A_mysql_wf</p></td><td colspan="1" class="confluenceTd"><ul><li>对于依赖服务连接异常/读写超时/返回数据异常等情况，必须在warning日志中打出</li><li>通过设定异常出现次数来设置报警</li><li>警戒阈值可以观察历史趋势决定</li></ul></td></tr><tr><td colspan="1" class="confluenceTd">pvlost</td><td colspan="1" class="confluenceTd"><p>是</p></td><td colspan="1" class="confluenceTd"><p>总pvlost：pvlost</p><p>分功能的：xxx_pvlost。比如入口集群里公交的pvlost:bus_pvlost</p></td><td colspan="1" class="confluenceTd"><ul><li>每个模块都必须定义自己的pvlost<span>（可以从访问日志里把超时和异常匹配出来）</span></li><li>通过自定义计算配置pvlost/pv的比例值，1. 比例值超过警戒值并且，2.pv大于阈值时（避免无流量时的抖动），发出告警</li><li>警戒阈值可以观察历史趋势决定<span><br></span></li></ul></td></tr><tr><td colspan="1" class="confluenceTd">端口语义</td><td colspan="1" class="confluenceTd">有http接口的强制</td><td colspan="1" class="confluenceTd"><p>简短描述</p><p>比如导航：multinav</p></td><td colspan="1" class="confluenceTd"><ul><li>建议端口服务添加实例语义，反映实例是否工作正常。</li><li>noah不支持的协议，可以采用自行的语义监控脚本实现</li></ul></td></tr><tr><td class="confluenceTd">状态类</td><td colspan="1" class="confluenceTd">端口监控</td><td colspan="1" class="confluenceTd">有端口的服务强制</td><td colspan="1" class="confluenceTd"><p>静态端口：端口号_port。比如8888端口：8888_port</p><p>动态端口：main_port</p></td><td colspan="1" class="confluenceTd"><ul><li>持续3分钟端口连接拒绝/连接超时/端口不存在，发出报警</li></ul></td></tr></tbody></table>

### 业务层监控（business logic）
监控业务的整体情况，反应业务当前是否正常，单个机器或者实例的异常并不一定会影响业务整体的可用性（上游的负载均衡或者重试策略），所以业务层的整体监控在判断业务告警的影响范围时尤其重要，参考《Google SRE》有如下四大黄金指标：
- 流量监控：业务整体流量的突增突降
- 容量监控：业务整体流量是否超过集群的阈值
- 可用性监控：业务整体可用性是否有突降（可以区分核心与非核心）
- 平响监控：业务整体的平响是否上升

## 网络监控
依据用户访问我们服务中间所有经过的网络链路，分为三部分：外网-》接入层-》内网
- 外网：指运营商网络，用户客户端发出请求后先经过运营商的网络才能到百度的接入层
- 接入层：指域名的各个vip
- 内网：各个idc的连通性


### 外网监控
- 运营商联通性:联通 移动 铁通(pc 无线) 等运营商的用户访问服务端的连通性


### 接入层监控
- vip流量:服务入口ip的流量　
- 域名联通性:域名是否可以正确解析并访问
- dns域名解析正确性:各个地域核心域名的dns解析正确性
- cdn:CDN的图片缓存命中率是否符合预期


### 内网监控
- 网络连通性:机房间网络连通性 交换机的网络连通性
- 网络带宽容量：机房带宽容量是否拥塞
- 交换机上联带宽打满	：底层交换机的上联带宽拥塞情况(网络敏感服务关注)

## 客户端监控
原理 端监控指从客户端的角度对稳定性, 用户体验等指标进行监控. 目前监控手段主要有两种:
- 日志回传, 通过某种方式将客户端的信息回传到后端服务. 监控方式主要是: 端上行为> 行为编码/加密->后端接口->后端服务日志.
  - NA:NA端目前采用离线日志回传的方式监控, 具体方式为: 客户端通过js函数addLog(), 记录日志在本地存储, 日志组件择时(延迟3-30秒)将本地存储中的日志加密上传至后端lbslogger服务的接口, 通过lbslogger服务的日志记录端上信息.对页面错误率的统计和对页面打开性能的统计依赖业务方对框架接口的正常调用
    - 页面加载速度
    - 页面加载成功率
    - 页面加载内容正确性
    - app crash率
  - PC/WAP端监控采用js脚本调用后端服务的方式, 目前主要覆盖以下几类异常:
    - 页面JS报错
    - 页面发送请求报错
    - 页面操作触发报错.
- 请求模拟, 通过模拟用户的实际请求监控线上服务可用性.
