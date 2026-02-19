# MDAMS (Museum Digital Asset Management System) PRD

## 项目简介
本项目是博物馆数字资源管理系统（MDAMS）的产品需求文档（PRD）仓库。该系统旨在为博物馆提供全流程的数字资产管理解决方案，涵盖资源采集、编目、存储、长期保存（OAIS）、授权管理与对外发布等核心能力。

## 目录结构
- `prd/`：核心 PRD 文档集
  - `01-summary/`：一页摘要
  - `02-standards-and-terms/`：参考标准（OAIS/DDI等）与术语表
  - `03-architecture/`：总体架构与深化设计
    - `深化设计/`：包含数据流转、存储分层、OAIS映射、集成架构、安全架构等详细设计
  - `04-processes/`：关键业务流程
  - `05-resource-repo-design/`：资源库层设计（影像/三维/视频）
  - `06-requirements/`：功能需求清单
  - `07-auth-and-audit/`：权限与审计体系
  - `08-nfr/`：非功能需求（性能、安全、可靠性）
  - `09-risks-and-open-items/`：风险与待确认事项
- `reference/`：原始参考资料与需求输入

## 更新日志 (Changelog)

### 2026-02-19
- **架构深化设计**：在 `prd/03-architecture/深化设计/` 下新增 5 份核心架构文档：
  1. **数据流转全景图**：定义 SIP -> AIP -> DIP 的流转与处理节点。
  2. **分层存储架构**：明确在线（Hot）、近线（Warm）、离线（Cold）存储策略与 ILM 规则。
  3. **OAIS 功能实体映射**：将系统模块与 ISO 14721 标准对齐。
  4. **集成架构与接口模式**：定义与文物管理系统、数字文物库等外部系统的集成规范。
  5. **安全与权限架构**：构建网络、应用、数据三层安全防御体系。

## 协作指南
- 本文档采用 Markdown 格式编写，建议使用 VS Code + Markdown Preview Enhanced 插件阅读。
- 所有架构图建议使用 Mermaid 或 Draw.io 绘制并归档于 `assets/` 目录。
- 每次重大修改请同步更新本 `README.md` 的更新日志。
