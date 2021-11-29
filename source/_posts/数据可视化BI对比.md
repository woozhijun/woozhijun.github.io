title: 数据可视化BI对比
tags:

  - 大数据
  - 可视化
  - 数据分析

category: 大数据
comments: true

---

调研-数据可视化BI对比

## 背景

目前公司主要是通过开源产品Redash在构建可视化分析报表，Redash主要以更通用的SQL语义方式构建。

一方面对于大部分的产品和运营人员要求较高，无法便捷的自助多维下钻、连表分析等；另一方面我们自建的数据平台如指标分析、漏斗分析等功能和体验不够完善。

针对以上痛点诉求，主要参考当前业界比较流行的数据可视化BI服务（是否付费、产品功能、操作体验）等方面简单做如下对比。

## 开源产品

#### **Superset**

> *由 Airbnb 开源的数据探索与可视化平台，目前最新的release版本为1.3.2，社区活跃，颜值较高。*

- 支持丰富的数据源

- 图表支持（50+），如分布，趋势，相关性图表，并支持Echarts等插件的方式自定义图表

- 相对于Redash 可视化的选项要更多

- 针对非技术人员相对Redash要稍友好一些

- 代码库：https://github.com/apache/superset

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=MDI0MDM5NDEzODg3ZjJhM2JjNzllMWI4NjdjYTIzY2NfbURHRXkzUkZrVnNLY1ZwSmRIQkV6ZGlGYTdZckhkN3dfVG9rZW46Ym94Y255MEhTWU1NMDJwek9ZR0ptNllUSnB3XzE2MzgxODY3Mjg6MTYzODE5MDMyOF9WNA)

#### **Redash**

> *是一个可协作数据可视化和仪表板平台，主要以更通用的方式（SQL）进行数据可视化*

- 支持数据源 35+

- 支持图表 10+

- 代码架构质量要优于Superset

- 代码库：https://github.com/getredash/redash

#### **Metabase**

> *是一款开源的BI分析工具，开发语言clojure+js为主，metabase更注重非技术人员的使用体验*

- 注重非技术人员在使用时的体验，相对Superset非技术人员基本上只能看预先建好的Dashboard

- 代码库：https://github.com/metabase/metabase

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=MGM5MjdhYjhlOGE0NjhmZGJiNDI5NGFhMGY0Zjg0MThfMTlzV1g3SFhBREN3M0ptQmtxSE9uVjR1N0RqdUU0cThfVG9rZW46Ym94Y25UUnZYbEJPaGFZblNkYlVidWJTVnpiXzE2MzgxODY3Mjg6MTYzODE5MDMyOF9WNA)

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=MjM4ZjFjNGE1ZmE4ZWQzMjhjOGU2ZTE2NGQ0Zjk4ZmFfVG1yRFVNemJYNkRFUEFjMVRFY2RxcVpJRE9iNTI1b1VfVG9rZW46Ym94Y25pM25qcm1NRHlibGpXVUlzaHpKYWtiXzE2MzgxODY3Mjg6MTYzODE5MDMyOF9WNA)

#### **Davinci**

> *由宜信数据团队开源，是一个DVAAS（Data Visualization as a Service）平台解决方案*

- 代码库：https://github.com/edp963/davinci

#### **DataEase**

> *DataEase 支持丰富的数据源连接，能够通过拖拉拽方式快速制作图表，并可以方便的与他人分享*

- 简单易用，可以通过鼠标点击和拖拽即可完成分析

- 支持直连模式、本地模式(基于 Apache Doris / Kettle 实现)

- 代码库：https://github.com/dataease/dataease

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=OGVhYzAzZTQxYTY5NDRkNTJhNWViZDAxN2MyMTA1MzlfU3lRT1FoSWtpQzlKamM1aERlOHBvN1E5NE1jQWpkVFlfVG9rZW46Ym94Y25QSmxxbEt4V29WY1psdWpYV1BNRTNmXzE2MzgxODY3Mjg6MTYzODE5MDMyOF9WNA)

#### 总结

1. 社区受欢迎程度 Superset > Metabase > Redash
2. 数据源丰富度 Superset > Redash > Metabase
3. 用户操作友好度 DataEase > Metabase > Superset

## 付费产品

### 国外

#### tableau

> 是一个可视化分析平台

- 内存数据引擎：直接查询外部数据库、动态从数据仓库抽取数据
- 由VizQL转化成查询语言，支持下钻/上卷查看数据，使用筛选器/组/分层结构变换分析角度
- 用户体验良好、简单易用
- 支持自然语言提问
- 企业独立部署方案
- 地址：https://www.tableau.com/products/cloud-bi

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=NThkZjBmOGYwZWFlODNhMDk3NjQyNjM4ZjcyNjBiOGNfUk5uTUtFZVl0elpmd1NuTDRJbklBcmZRSmJObG40ODRfVG9rZW46Ym94Y25SQ0dtZkNFcmNGczBBa3BSRlNLWjFmXzE2MzgxODY3Mjg6MTYzODE5MDMyOF9WNA)

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=YzVhOWQ3Mjk4NjkxOGNhMGExYjQ5MTM4NTM5ZDcyMTVfemFVNVM2ZmhER3pqUDdpYkxraGtaTDhGTWNQbXJJRU1fVG9rZW46Ym94Y25oVk5IVk1TZGJnZXhvOTA0MGJ0R09iXzE2MzgxODY3Mjg6MTYzODE5MDMyOF9WNA)

#### 微软PowerBI

> 自助服务分析
>
> 使用上百个数据可视化效果、内置 AI 功能、紧密的 Excel 集成以及预构建和自定义的数据连接器
>
> 可以结合使用 Power BI 和 Azure

- 地址：https://powerbi.microsoft.com/zh-cn/what-is-power-bi/

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=NzU1MGYwODI5MTllZGI3YzJhNDdmNWFmZjYxMDM0MDNfeUhIeHFBd0V6a0FFUGZRWnV2cGdUcjFVcjdvOHY0NjBfVG9rZW46Ym94Y25vV0FOOE5SczRxdVhlWGdhbm5ISFBkXzE2MzgxODY3Mjg6MTYzODE5MDMyOF9WNA)

### 国内

#### 百度Sugar

> 是百度云推出的敏捷 BI 和数据可视化平台，目标是解决报表和大屏的数据 BI 分析和可视化问题，解放数据可视化系统的开发人力。

- 支持了图表组件（100+：折线、柱图、饼图、拓扑图、地图、3D 散点图等等）
- 支持过滤条件（10+：单选、多选、日期、输入框、复杂逻辑等）
- 支持多场景大屏模版（30+）
- 拖拽组件，需构建数据集
- 兼容移动端
- 多种数据源对接（包括云上内置和自定义本地源），同时可私有化部署
- 私有化部署：30 账号，10 万/年
- 地址：https://sugar.bce.baidu.com/group/first/home?__scp__=scp_1013e-5o5cmwob-o564n7
- 帮助文档：https://cloud.baidu.com/doc/SUGAR/index.html

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=MzM3NTE5MjVmMWZkYjcwNTg5N2RmOWU4OGUzYjY3NzBfTVo3eHNMWkZqVXFiRXE4UWJLa0gxek5zR1RlN0hna29fVG9rZW46Ym94Y24zdXlzU0xEN1dKNG9rZGhpSVdDbklkXzE2MzgxODY3Mjg6MTYzODE5MDMyOF9WNA)

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=OTk5MDVmYjkyNWUyYmIwZjEyMTgyODU2YWVlYTRiOTVfbm14TnFYazNZbE1raVg3Z0szNzJOd1FUcTYxdGhLalpfVG9rZW46Ym94Y25nS3JBUEVUbXVhRHBuV0JxOTh4N0ViXzE2MzgxODY3Mjg6MTYzODE5MDMyOF9WNA)

#### 阿里云QuickBI

> 是一个专为云上用户量身打造的新一代智能BI服务平台。
>
> 可提供海量数据实时在线分析服务，支持拖拽式操作和丰富的可视化效果，帮助完成数据分析、业务数据探查、报表制作等工作

- 核心功能如下

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=NmFlOWRjOTc1YjIwNGQ0MjVhMmM3NDE3ODJjZjUwMzVfSXN0cUVpOWlxYlR2QTBDNlBqeHd3emFOeFc0bjFaWHlfVG9rZW46Ym94Y25RSlhQTGtJaGExOG5rT3JmOE9Cc3ZiXzE2MzgxODY3Mjg6MTYzODE5MDMyOF9WNA)

- 管理控制台

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=MjE0MGFhN2U1MjViNWIzNzAzMTE1NWQ4NzhjNDFlMjRfS2s0aXRyS3FhVjJxR080cDBFWGhmNDJyakJqVDdSd1ZfVG9rZW46Ym94Y243WHJ4ZEo5VTNEZjhEVno2dmpHNHplXzE2MzgxODY3Mjg6MTYzODE5MDMyOF9WNA)

- 新增数据集

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=OTE0MDY0MTgwYTAxZTRlNzA0M2VhNWQ5NDE0YWFkODlfelVmdHJkSzhTWE16eVdRRWZRdGRHOVdOVUZxSFJpSjhfVG9rZW46Ym94Y25jOTA5SXVzejdrQjM4OGczcndtWFJnXzE2MzgxODY3Mjg6MTYzODE5MDMyOF9WNA)

- 新建即时分析

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=NDkzYTNkZWZhMTE3ZGQ0NzhiZmU2YTBhMDM4OWExOTNfNmhFMUtrMVFrZFd4MlJSWE1MOFlkU2pTajNSS0dRUExfVG9rZW46Ym94Y25qUEdGNmNzQ2VFampOaXdrOGFJSldlXzE2MzgxODY3Mjg6MTYzODE5MDMyOF9WNA)

- 地址：https://bi.aliyun.com/workspace/analysis
- 帮助文档：https://help.aliyun.com/product/30343.html
- 云上费用：10.2w/年/50用户。（暂时未看到私有化部署费用）

#### 亿信ABI

> 是亿信华辰历经十五年匠心打造的国产化BI工具，技术自主可控。
>
> 它打通从数据接入、到数据建模与处理、再到数据分析与挖掘整个数据应用全链路。

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=NGI1NGE4YTI4YjY2MzRkMmU0NzMwZGM0ZWIwNTc0OWZfanFQZXV2bWZCdDVMR3RmRGpzVThWMHBoRk5jVVVRdWJfVG9rZW46Ym94Y25pVEF0VUE2TXhqVzd3bTUweEM3ZldkXzE2MzgxODY3Mjg6MTYzODE5MDMyOF9WNA)

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=NGM1MjdhYTVhNDEwZmIxNTVkYTdlYjE1MzM5ZjM2MGFfTXE0R1B1SkM4bldqeDNqeW1MMXlxSXlYYjVUc0h6enFfVG9rZW46Ym94Y240WVpsT0tEck44NjRVMXJQMzR3aGxiXzE2MzgxODY3Mjg6MTYzODE5MDMyOF9WNA)

- 管理模块界面控件不够大气，更像传统软件
- 地址：https://www.esensoft.com/products/abi.html

#### 观远数据Galaxy Viz

> 新一代智能分析与决策解决方案，观远包括数据接入、智能ETL、可视化分析、数据大屏、数据门户、移动轻应用等多组件
>
> 这里主要参考可视化分析模块

- 多样的可视化组件（50种内置图表类型），支持SDK扩展图表类型，支持外链、图片和纯文本等卡片类型

- 拖拉拽操作创建可视化图表、自由布局

- 图表可进行多层钻取、图表跳转、多表联动等交互操作

- 地址：https://demo.guandata.com/page/o5b9e6cf62a3a4e3996c4fb9

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=MGQ5NjdiOWMwNGZmYTMyOWNmOTM1YzU2NGE1MzNkNjhfMVdYSFJhOWN6Rmw3T0k2YmdWelQ4VVJVd1BmbmFSamNfVG9rZW46Ym94Y25ZY25hc0M3cVJ2b0s0UnBNa2xTVmJlXzE2MzgxODY3Mjg6MTYzODE5MDMyOF9WNA)

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=YjhiNjk5ZWM1NWRlNDBiMjE1YjZhZjA5ZjQ1ZmZhMjRfdWY0d2JnR3ZmVW9UMWVVelhUelpaUFJ1OGhPVFVac0hfVG9rZW46Ym94Y25hZ01SSEZGY1o3Vlh0WFBBNk9IdzFoXzE2MzgxODY3Mjg6MTYzODE5MDMyOF9WNA)

#### 帆软FineBI

> 是一款商业智能（Business Intelligence）产品。
>
> 定位于自助大数据分析的BI工具，能够帮助业务人员和数据分析自住制作仪表板，进行探索分析

- 核心功能如下：

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=MDJkMDM5MTkwMmFkN2UxZTQwZmU0NGIwYWFkMGFmODNfS0Q5OFJZekplRU9yMW1uRFdDMHVkT3JIZklteUc4VFRfVG9rZW46Ym94Y25CV2FTMVhSMGk0WVhWN1JSOWlTeGdlXzE2MzgxODY3Mjg6MTYzODE5MDMyOF9WNA)

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=ZWI2NTU2MmUzOTZkM2M2NmU1MTgyMjU5Y2ZlODcwMzdfN0RKVGgzYWVna0lrT0hHTEFPdVA1eEp0ZjE2Q2syTUVfVG9rZW46Ym94Y24wQjBKb1pHNllRVWpzY2hOVURpdG9jXzE2MzgxODY3Mjg6MTYzODE5MDMyOF9WNA)

- 地址：[FineBI商业智能](http://demo.finebi.com/)
- 帮助文档：[FineBI产品简介- FineBI帮助文档 FineBI帮助文档](https://help.fanruan.com/finebi/doc-view-259.html)

#### Smartbi

> 主要融合传统BI、自助BI、智能BI，满足复杂报表、数据可视化、自助探索分析、机器学习建模、预测分析、自然语言分析等

- 图形可视化：默认可集成 Echarts4.8，支持Excel作图，动静结合，Excel图形模板可直接复用
- 自然语言的交互能力
- 地址：https://www.smartbi.com.cn/allbigdata#gn1

#### 网易数帆

> 敏捷数据分析及可视化平台，满足数据收集、分析、展示等不同阶段需求。
>
> 拖拽操作即可实现业务数据可视化分析

- 核心功能同QuickBI，包括（数据门户、数据大屏、数据填表、自助取数、兼容移动端）等
- 额外支持MPP性能加速扩容、预加载任务

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTNlM2FkMGM3ZGRjZGQzMzY3YTVjZmY5ZWQzYTVmOTdfMDhZaGNLcTZ2RFJKSW9iOUhTWW5LdGtHNWZHZXV5V1RfVG9rZW46Ym94Y25nVjI5a095RUo0TzZjUnVFT0JCZGdmXzE2MzgxODY3Mjg6MTYzODE5MDMyOF9WNA)

- 新建数据模型

![img](https://maimai.feishu.cn/space/api/box/stream/download/asynccode/?code=YTQ0MjViNGExZjg2MjYyYzc5ZWY5NmVlMzM3ZDVlNmVfMTdOMGs3YjVmU09wTnhJMVE0Wnk5SEJab3JvbUJrWEJfVG9rZW46Ym94Y25wNjh3cXI5ekY1Sk45bU5tcmx4ZGJnXzE2MzgxODY3Mjg6MTYzODE5MDMyOF9WNA)

- 地址：https://woozhijun.youdata.163.com/dash/dataModel/700275083
- 帮助文档：https://study.sf.163.com/documents/read/manual/1NewReport.md

#### 海致BDP

> *多数据整合，建立统一数据口径，支持自助式数据准备（ETL）， 并提供灵活、易用、高效可视化探索式分析能力，帮助企业构建贴合自身业务的企业洞察。*

- 拖拽分组

- 多层钻取

- 筛选分析

- 对比拆分

- 地址：https://www.bdp.cn/home/product.html

#### **DataFocus**

> *中文搜索式数据分析系统，问答式交互，搜索式体验，彻底变革你的数据分析流程。*

> *但DataFocus的数据可视化表现图表仍然较为基础，多样化不足。*

- 地址：https://www.datafocus.ai/

#### **永洪BI**

> *把大数据分析所需的产品功能全部融入一个平台下，进行统一管控。包括数据采集、清洗、整合、存储、计算、建模、训练、展现、协作等。*

> *互联网企业使用较少*

- 地址： https://www.yonghongtech.com/

### 总结

1. 个人使用感受： 国外tableau，国内Sugar、QuickBI、有数BI这四款款相对不错，可优先考虑国内产品
2. 诉求功能：Sugar、QuickBI、有数BI均可以满足诉求
3. 图表丰富度：Sugar >  QuickBI > 有数BI
4. 链接性能：看评论Sugar不高， QuickBI、有数BI 待定
5. 企业部署：Sugar、有数BI可以私有化部署，QuickBI不支持
6. 费用价格：QuickBI > Sugar> 有数BI