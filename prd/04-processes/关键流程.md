# 4. 关键流程
## 4.1 入库与传输（分离式 SIP + 端到端 Fixity）
依据 `DATA_INGEST_ARCHITECTURE.md`：
- 传输形态：`multipart/form-data` 分两部分上传：`file`（原始二进制流）+ `manifest`（JSON 字符串，含客户端计算 SHA256 与提取元数据）。
- 服务端入库：流式接收文件并计算服务端 Hash，与 manifest 的客户端 Hash 比对；一致后写入存储与数据库。
- 存储形态：解包存储（Unpacked），物理文件直接可被 IIIF 图像服务读取；数据库记录承载元数据。
- 导出形态：按需动态生成标准 BagIt 结构并打包 ZIP 返回（包含 bagit.txt、bag-info.txt、manifest-sha256.txt、data/ 载荷）。

### 4.1.1 入库时序图（Mermaid）
```mermaid
sequenceDiagram
  autonumber
  participant U as 用户/采集端
  participant B as 浏览器/客户端
  participant API as 入库服务
  participant FS as 文件存储(ZFS/NAS)
  participant DB as 元数据存储(PostgreSQL+JSONB)

  U->>B: 选择文件与填写/提取元数据
  B->>B: 计算 SHA-256（Client-side Fixity）
  B->>API: multipart/form-data(file + manifest JSON)
  API->>API: 流式接收并计算 SHA-256（Server-side Fixity）
  API->>API: 比对 Client Hash vs Server Hash
  alt 校验一致
    API->>FS: 写入物理文件（解包存储）
    API->>DB: 写入元数据/事件（含校验值）
    API-->>B: 返回入库成功与资源ID/版本
  else 校验不一致
    API-->>B: 返回失败（校验和不一致）
  end

  U->>B: 需要交换/下载 BagIt 包
  B->>API: 请求导出 BagIt ZIP
  API->>FS: 读取物理文件
  API->>API: 动态生成 bagit.txt/manifest-sha256.txt/data/
  API-->>B: 返回 ZIP（BagIt）
```

## 4.2 代表影像标记（需求 需-0009）
- UI 形态：点亮/熄灭按钮（必须有强对比视觉反馈，避免误解）。
- 业务规则：点亮代表影像时，将同一文物的“代表影像标签”迁移到当前影像；若当前影像被修改/删除，标签自动回退到之前最后一张有效影像。

### 4.2.1 代表影像状态机（Mermaid）
```mermaid
stateDiagram-v2
  [*] --> 未标记
  未标记 --> 已标记: 点亮为代表影像
  已标记 --> 未标记: 熄灭（或迁移到另一张影像）

  已标记 --> 回退处理中: 代表影像被删除/失效
  回退处理中 --> 已标记: 回退到上一张有效影像
  回退处理中 --> 未标记: 无可回退影像（保持无代表影像）
```

## 4.3 利用链路（收藏夹 → 购物车 → 利用单 → 交付）
依据 `资源管理需求表.xlsx`：
- 个人中心三件套：收藏夹、购物车、利用单。
- 收藏夹：整理、标注局部、导出、加入购物车；支持保存工作空间快照（需-0007）。
- 购物车：批量修改规格（需-0004）；生成利用单。
- 利用单：可复制（需-0002），可导入表格生成（需-0003），可导出/下载数据（需-0006）。
- 交付包：压缩包除影像外，必须附带“利用授权协议”与“元数据说明清单”（需-0005）。

### 4.3.1 利用链路流程图（Mermaid）
```mermaid
flowchart LR
  A[检索/浏览资源] --> B[加入收藏夹]
  B --> C[标注局部/整理/保存快照]
  C --> D[加入购物车]
  D --> E[批量修改规格]
  E --> F[生成利用单]
  F --> G[复制利用单/导入表格生成]
  G --> H[生成交付包]
  H --> I[下载/交付]
  I --> J[交付包包含：影像文件 + 利用授权协议 + 元数据说明清单]
```

## 4.4 对外发布审核（模块）
- 发布工单：选择资源 → 自动检查授权 → 提交审核 → 生成可对外分发版本（DIP）。
- 审计：所有审核动作、导出动作、发布动作必须可追溯（人、时间、原因、版本、交付内容）。

### 4.4.1 对外发布审核流程图（Mermaid）
```mermaid
flowchart TD
  S[选择资源/版本] --> R{授权校验通过?}
  R -- 否 --> X[拦截并提示补齐授权/调整策略]
  R -- 是 --> T[提交发布工单]
  T --> A[审核]
  A -->|退回| B[补充材料/调整范围]
  B --> A
  A -->|通过| D[生成对外分发版本\n(派生/水印/脱敏)]
  D --> P[发布/交付]
  P --> L[记录审计日志\n操作者/时间/版本/原因/范围]
```

## 4.5 长期保存（BagIt + OAIS + PREMIS + METS）
依据需求 `需-0010` 与 PDF 概要设计：
- 使用 BagIt 封装数字内容，校验数据完整性。
- 按 OAIS 功能对标组织流程与职责。
- PREMIS：记录保存对象/事件/代理（导入、校验、迁移、修复、交付等）。
- METS：组织描述性、技术性、结构性元数据并支持交换。

### 4.5.1 SIP/AIP/DIP 与长期保存闭环（Mermaid）
```mermaid
flowchart LR
  SIP[SIP 提交包<br/>文件 + manifest] --> Ingest[入库（Ingest）<br/>校验 / 质检 / 编目]
  Ingest --> AIP[AIP 保存包<br/>主文件 + 衍生 + 元数据 + Fixity + 事件]
  AIP --> Storage[归档存储<br/>分层存储 / 多副本]
  Storage --> Fixity[定期 Fixity 校验]
  Fixity -->|失败| Repair[修复 / 重建副本]
  Repair --> Fixity
  Storage --> DIP[DIP 分发包<br/>按授权生成交付物]
  DIP --> BagIt[按需生成 BagIt ZIP]
  BagIt --> Deliver[交付 / 交换]
  Fixity --> Premis[PREMIS 事件记录]
  Repair --> Premis
  Deliver --> Premis
```

