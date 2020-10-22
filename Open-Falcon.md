# open-falcon的目标是做最开放、最好用的互联网企业级监控产品
已知对比：相较于zabbix来说，
- 强大灵活的数据采集：自动发现，支持falcon-agent、snmp、支持用户主动push、用户自定义插件支持、opentsdb data model like（timestamp、endpoint、metric、key-value tags）
> 简单点就是说，扩展多
- 水平扩展能力：支持每个周期上亿次的数据采集、告警判定、历史数据存储和查询
> 就是采集能力、数据储存、告警方式等能力优秀
- 高效率的告警策略管理：高效的portal、支持策略模板、模板继承和覆盖、多种告警方式、支持callback调用
> 前端页面好看
- 人性化的告警设置：最大告警次数、告警级别、告警恢复通知、告警暂停、不同时段不同阈值、支持维护周期
> 告警设置人性化
- 高效率的graph组件：单机支撑200万metric的上报、归档、存储（周期为1分钟）
> 处理信息能力强
- 高效的历史数据query组件：采用rrdtool的数据归档策略，秒级返回上百个metric一年的历史数据
> 查询历史数据能力强
- dashboard：多维度的数据展示，用户自定义Screen
> 前端展示页面设置多样
- 高可用：整个系统无核心单点，易运维，易部署，可水平扩展
> 简洁好用
- 开发语言： 整个系统的后端，全部golang编写，portal和dashboard使用python编写。
> python天下第一


# 这里着重说一下open-falcon采用的Storage方式
对于监控系统来讲，历史数据的存储和高效率查询，永远是个很难的问题！
- 因为平常的业务系统永远是读操作远远大于写操作，而对于监控系统来说写操作是远远大于读操作的，对用常用的MySQL、postgresql、mongodb来说都是无法完成的
- 对于监控系统来说是通过采集周期来收集数据的，而1分钟和5分钟的周期占了大多数，这意味着一天24小时都不会有数据量低的情况
- 往往在使用的时候一次查询都是上百个metrics，在1秒内返回给用户并绘图，这是一个不小的挑战

open-falcon在这块，投入了较大的精力。我们把数据按照用途分成两类，一类是用来绘图的，一类是用户做数据挖掘的。
> 参考rrdtool的理念，在数据每次存入的时候，会自动进行采样、归档，同时为了不丢失信息量，数据归档的时候，会按照平均值采样、最大值采样、最小值采样存三份


# 结构图
![alt text](http://www.yassor.xyz:81/photo/10.png)
<br/>


## Agent:
- agent用于采集机器负载监控指标，比如cpu.idle、load.1min、disk.io.util等等，每隔60秒push给Transfer。agent与Transfer建立了长连接，数据发送速度比较快，agent提供了一个http接口/v1/push用于接收用户手工push的一些数据，然后通过长连接迅速转发给Transfer。
- agent需要部署到所有要被监控的机器上，其资源消耗较少
- [配置文件说明及相关参数](https://book.open-falcon.org/zh_0_2/distributed_install/agent.html)
<br/>

## Transfer
- transfer是数据转发服务。它接收agent上报的数据，然后按照哈希规则进行数据分片、并将分片后的数据分别push给graph&judge等组件。同时 transfer 也支持将数据转发给 opentsdb 和 influxdb，也可以转发给另外一个 transfer。
- 部署完成transfer组件后，请修改agent的配置，使其指向正确的transfer地址。在安装完graph和judge后，请修改transfer的相应配置、使其能够正确寻址到这两个组件。
- [配置文件说明及相关参数](https://book.open-falcon.org/zh_0_2/distributed_install/transfer.html)
<br/>
