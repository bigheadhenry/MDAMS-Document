# 3.8 OAIS 功能实体映射 (OAIS Mapping)

为确保系统具备符合 ISO 14721 (OAIS) 标准的长期保存能力，明确各业务模块与 OAIS 功能实体的映射关系。

## 1. OAIS 功能模型映射表

| OAIS 功能实体 | 对应系统模块 | 关键职责 (System Responsibilities) | 涉及数据流 |
| :--- | :--- | :--- | :--- |
| **Ingest**<br>(接收与摄入) | **资源采集模块**<br>(Resource Ingest Module) | - SIP 接收与验证<br>- 病毒扫描与格式校验<br>- 描述元数据绑定<br>- 提交审核 (QC) | User -> SIP -> Ingest -> QC -> AIP |
| **Archival Storage**<br>(档案存储) | **长期保存模块 (存储层)**<br>(Preservation Module / Storage) | - AIP 打包与分层存储<br>- Fixity 校验与完整性监控<br>- 灾备恢复 (Disaster Recovery) | AIP -> Storage -> Checksum -> Restore |
| **Data Management**<br>(数据管理) | **元数据管理 / 索引服务**<br>(Metadata Management / Indexing) | - 描述元数据存储与更新<br>- 索引构建 (全文/分面)<br>- 受控词表管理<br>- 关联数据 (Linked Data) | Metadata -> Database -> Search Index -> API |
| **Administration**<br>(管理) | **系统管理 / 权限与审计**<br>(System Admin / Auth & Audit) | - 提交协议配置<br>- 存储策略配置<br>- 用户权限与审计日志<br>- 系统监控与告警 | Policies -> System Config -> Logs -> Report |
| **Preservation Planning**<br>(保存规划) | **长期保存模块 (规划层)**<br>(Preservation Planning Module) | - 格式风险监测 (Format Watch)<br>- 迁移策略制定 (Migration Plan)<br>- 仿真环境维护 (Emulation) | Format Registry -> Risk Alert -> Plan -> Action |
| **Access**<br>(访问与利用) | **资源展示 / 利用模块**<br>(Show / Use Module) | - 检索与浏览 (Search & Browse)<br>- DIP 生成与分发<br>- 授权控制 (Rights Management)<br>- IIIF 图像服务 | User Query -> Index -> Auth Check -> DIP -> User |

## 2. 关键实体实现说明

### 2.1 Ingest (摄入)
- **实现重点**：必须确保 SIP 的完整性与真实性。
- **关键动作**：
    - **SIP Validation**：基于 BagIt 规范校验 SIP 包完整性。
    - **Format Identification**：使用 PRONOM/DROID 工具识别文件格式。
    - **Virus Scan**：ClamAV 病毒扫描。

### 2.2 Archival Storage (存储)
- **实现重点**：确保存储介质的可靠性与数据一致性。
- **关键动作**：
    - **AIP Packaging**：生成 METS 封装包，包含 PREMIS 保存元数据。
    - **Fixity Check**：定期巡检 SHA-256 校验和。
    - **Storage Hierarchy**：基于 ILM 策略自动迁移冷热数据。

### 2.3 Access (访问)
- **实现重点**：提供便捷、受控的资源访问服务。
- **关键动作**：
    - **Discovery**：基于 Elasticsearch 的全文检索与分面筛选。
    - **Presentation**：基于 IIIF Image API 提供切片浏览与深度缩放。
    - **Delivery**：按需生成 DIP 下载包，支持水印与脱敏。

## 3. 保存规划 (Preservation Planning) 策略示例

针对常见格式的迁移策略：

| 原始格式 (Source Format) | 风险等级 | 迁移策略 (Action) | 目标格式 (Target Format) | 触发条件 |
| :--- | :--- | :--- | :--- | :--- |
| **TIFF (Uncompressed)** | 低 | 保持原样 / 刷新介质 | - | 介质寿命到期 |
| **CR2 (Canon Raw)** | 中 | 格式归一化 (Normalization) | DNG (Digital Negative) | 软件支持度下降 |
| **Flash (SWF)** | 高 | 仿真 / 视频录制 | HTML5 / MP4 | 浏览器不再支持 |
| **WMV (Windows Media)** | 中 | 格式迁移 (Migration) | H.264 / MP4 | 播放器兼容性下降 |
