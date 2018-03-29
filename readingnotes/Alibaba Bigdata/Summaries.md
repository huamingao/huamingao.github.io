### 大数据体系架构层次
- 数据采集
- 数据计算
- 数据服务
- 数据应用

### 数据采集层
##### 日志采集体系
> - 针对 Web 端的日志采集技术：Aplus.JS
> - 针对 APP 端的日志采集技术：UserTrack

##### 数据传输工具 
> TimeTunnel：既支持实时流式计算，也支持各时间窗口的批量计算

##### 数据同步工具
> DataX 和 同步中心：直连异构数据库，抽取各时间窗口的数据

### 数据计算层
##### 数据存储及计算云平台
- 离线计算平台 MaxCompute
- 实时计算平台 StreamCompute

##### 数据整合及管理系统 OneData

- 数据分层
> - 操作数据层（Operational Data Store, OSD）
> - 明细数据层（Data Warehouse Detail, DWD）
> - 汇总数据层（Data Warehouse Summary, DWS）
> - 应用数据层（Application Data Store, ADS）
> - 实时维表层（Dimension, DIM）

- 元数据模型整合
> - 数据源元数据
> - 数据仓库元数据
> - 数据链路元数据
> - 工具类元数据
> - 数据质量类元数据

### 数据服务层
数据源架构在多种数据库上，如MySQL, HBase等，后续将逐渐迁移到阿里云云数据库 ApsaraDB for RDS 和表格存储 Table Store

数据服务平台 OneService 提供三大特色数据服务
- 简单数据查询
- 复杂数据查询：用户识别，用户画像等
- 实时数据推送

### 数据应用层
提供给应用方的平台和产品，如流量分析平台，行业数据分析门户，自助式数据网站，实时数据监控等



##