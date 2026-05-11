---
layout: default
title: 附錄：命名規則規範
parent: Azure Landing Zone 醫療雲端
nav_order: 7
---

# Azure 資源命名規則規範（醫療機構適用版）

{: .highlight }
**版本**：v1.0 ｜ **日期**：2026-03-28 ｜ **依據**：Azure Cloud Adoption Framework (CAF)

---

## 1. 命名原則

### 1.1 基本格式

```
{resource-type}-{org}-{workload}-{environment}-{region}-{instance}
```

**範例**（醫療資訊系統正式環境 VNet）：
```
vnet-hosp-his-prod-twn-001
```

### 1.2 設計原則

| 原則 | 說明 |
|------|------|
| 全小寫英數字 | 所有字元使用小寫英文字母 (a-z) 與數字 (0-9)。Storage Account 等不支援連字號時採緊接方式 |
| 連字號分隔 | 各欄位之間以連字號 (-) 分隔。不得使用底線 (_) 或空格 |
| 禁止縮寫醫療資料 | 名稱中不得含有病患 ID、員工編號、病房名稱等可識別個人的資訊（HIPAA / GDPR 合規） |
| 最大長度限制 | 整體命名原則不超過 63 個字元（部分資源更短，見各資源說明） |
| 環境強制標示 | `prod` / `stg` / `dev` / `test` 必須明確出現在名稱中，禁止省略，避免誤操作正式環境 |
| 實例編號保留 | 即使只有一個實例，仍加上 `001` 以保留擴充彈性 |

---

## 2. 元件代碼對照表

### 2.1 醫院代碼（org）

建議使用醫院英文縮寫，長度 2～5 碼：

| 醫院 | org 代碼 | 備註 |
|------|---------|------|
| （示範用）醫院名稱 | `hosp` | 本文件以 hosp 作為示範前綴；實際部署請替換為院所縮寫 |
| （自訂）台大醫院 | `ntuh` | National Taiwan University Hospital |
| （自訂）長庚醫院 | `cgh` | Chang Gung Memorial Hospital |
| （自訂）台北榮總 | `vghtpe` | Taipei Veterans General Hospital |

### 2.2 環境代碼（environment）

| 環境 | 代碼 | 說明 |
|------|------|------|
| 正式 | `prod` | Production：對外服務的正式環境，需最高權限控管 |
| 預備 | `stg` | Staging：上線前的最終驗證環境，配置與 prod 相近 |
| 開發 | `dev` | Development：開發人員日常使用的沙箱環境 |
| 測試 | `test` | Testing：系統測試與 QA 驗證環境 |

### 2.3 區域代碼（region）

| Azure Region | 代碼 | 說明 |
|---|---|---|
| Taiwan North（台灣北） | `twn` | 台灣的主要選項 |
| Southeast Asia（東南亞） | `sea` | 新加坡 — 適合 DR 備援 |
| Japan East（日本東部） | `jpe` | 東京 |
| Japan West（日本西部） | `jpw` | 大阪 — 適合 DR 備援 |
| Korea Central（韓國中部） | `krc` | 首爾 |

### 2.4 工作負載代碼（workload）

| 代碼 | 全稱 | 說明 |
|------|------|------|
| `his` | Hospital Information System | 醫院資訊系統主系統 |
| `ris` | Radiology Information System | 放射資訊系統 |
| `pacs` | Picture Archiving & Comm. System | 醫療影像儲存傳輸系統 |
| `emr` | Electronic Medical Records | 電子病歷系統 |
| `lis` | Laboratory Information System | 檢驗資訊系統 |
| `ois` | Operation Room Info System | 手術室資訊系統 |
| `portal` | Patient / Staff Portal | 病患或員工入口網站 |
| `shared` | Shared Services | 共用服務（AD、監控、DNS 等） |
| `admin` | Administration | 行政管理系統 |

---

## 3. 各資源類型命名規則

### 3.1 基礎網路資源

| 資源類型 | 縮寫 | 命名格式 | 範例 |
|---------|------|---------|------|
| Virtual Network | `vnet` | `vnet-{org}-{wl}-{env}-{region}-{inst}` | `vnet-hosp-his-prod-twn-001` |
| Subnet | `snet` | `snet-{org}-{wl}-{env}-{role}` | `snet-hosp-his-prod-app` |
| Network Security Group | `nsg` | `nsg-{org}-{wl}-{env}-{region}-{inst}` | `nsg-hosp-his-prod-twn-001` |
| Route Table | `rt` | `rt-{org}-{wl}-{env}-{region}-{inst}` | `rt-hosp-shared-prod-twn-001` |
| Public IP Address | `pip` | `pip-{org}-{wl}-{env}-{region}-{inst}` | `pip-hosp-portal-prod-twn-001` |
| Load Balancer | `lb` | `lb-{org}-{wl}-{env}-{region}-{inst}` | `lb-hosp-his-prod-twn-001` |
| Application Gateway | `agw` | `agw-{org}-{wl}-{env}-{region}-{inst}` | `agw-hosp-portal-prod-twn-001` |
| Azure Firewall | `afw` | `afw-{org}-{wl}-{env}-{region}-{inst}` | `afw-hosp-shared-prod-twn-001` |
| VPN Gateway | `vpng` | `vpng-{org}-{env}-{region}-{inst}` | `vpng-hosp-prod-twn-001` |
| Private Endpoint | `pep` | `pep-{org}-{wl}-{env}-{svc}` | `pep-hosp-his-prod-sql` |

{: .note }
Subnet 的 `{role}` 使用 `app` / `db` / `mgmt` / `dmz` / `gw` 等功能角色縮寫取代 region 與 instance。

### 3.2 運算資源

| 資源類型 | 縮寫 | 命名格式 | 範例 |
|---------|------|---------|------|
| Virtual Machine | `vm` | `vm-{org}-{wl}-{env}-{region}-{inst}` | `vm-hosp-his-prod-twn-001` |
| VM Scale Set | `vmss` | `vmss-{org}-{wl}-{env}-{region}-{inst}` | `vmss-hosp-his-prod-twn-001` |
| AKS Cluster | `aks` | `aks-{org}-{wl}-{env}-{region}-{inst}` | `aks-hosp-pacs-prod-twn-001` |
| AKS Node Pool | `np` | `np{wl}{env}{inst}`（無連字號） | `nppacspr001` |
| App Service Plan | `asp` | `asp-{org}-{wl}-{env}-{region}-{inst}` | `asp-hosp-portal-prod-twn-001` |
| App Service / Web App | `app` | `app-{org}-{wl}-{env}-{region}-{inst}` | `app-hosp-portal-prod-twn-001` |
| Azure Function App | `func` | `func-{org}-{wl}-{env}-{region}-{inst}` | `func-hosp-lis-prod-twn-001` |
| Container Registry | `cr` | `cr{org}{wl}{env}`（全小寫無分隔） | `crhosphi sprod` |

{: .note }
AKS Node Pool 與 Container Registry 有長度及字元限制（不支援連字號），採緊接方式。

### 3.3 儲存與資料庫資源

| 資源類型 | 縮寫 | 命名格式 | 範例 |
|---------|------|---------|------|
| Storage Account | `st` | `st{org}{wl}{env}{inst}`（無連字號） | `sthosphisprod001` |
| Blob Container | （依用途） | `{purpose}-{env}` | `dicom-images-prod` |
| Data Lake Gen2 | `dls` | `dls{org}{wl}{env}{inst}` | `dlshosppacspr001` |
| Azure SQL Server | `sql` | `sql-{org}-{wl}-{env}-{region}-{inst}` | `sql-hosp-his-prod-twn-001` |
| SQL Database | `sqldb` | `sqldb-{org}-{wl}-{env}-{inst}` | `sqldb-hosp-his-prod-001` |
| Cosmos DB Account | `cosmos` | `cosmos-{org}-{wl}-{env}-{region}-{inst}` | `cosmos-hosp-ris-prod-twn-001` |
| Redis Cache | `redis` | `redis-{org}-{wl}-{env}-{region}-{inst}` | `redis-hosp-his-prod-twn-001` |
| Azure Backup Vault | `bvault` | `bvault-{org}-{env}-{region}-{inst}` | `bvault-hosp-prod-twn-001` |

{: .important }
Storage Account 最多 **24 字元**，只允許英數字，不可含連字號。格式：`st + org + wl + env + inst`，所有元件盡量縮短（例如 `prod` → `pr`）。

### 3.4 身份與安全資源

| 資源類型 | 縮寫 | 命名格式 | 範例 |
|---------|------|---------|------|
| Key Vault | `kv` | `kv-{org}-{wl}-{env}-{region}-{inst}` | `kv-hosp-his-prod-twn-001` |
| Managed Identity | `id` | `id-{org}-{wl}-{env}-{role}` | `id-hosp-aks-prod-workload` |
| Log Analytics Workspace | `log` | `log-{org}-{wl}-{env}-{region}-{inst}` | `log-hosp-shared-prod-twn-001` |
| Application Insights | `appi` | `appi-{org}-{wl}-{env}-{region}-{inst}` | `appi-hosp-portal-prod-twn-001` |
| Resource Group | `rg` | `rg-{org}-{wl}-{env}-{region}` | `rg-hosp-his-prod-twn` |
| Management Group | `mg` | `mg-{org}-{level}` | `mg-hosp-corp` |
| Azure Policy | `policy` | `policy-{org}-{scope}-{rule}` | `policy-hosp-sub-tagging` |

---

## 4. 完整範例場景（HIS 正式環境）

以「醫療資訊系統（HIS）正式環境部署於台灣北區域」為範例：

| 資源類型 | 命名結果 | 說明 |
|---------|---------|------|
| Management Group | `mg-hosp-corp` | 最頂層管理群組 |
| Resource Group | `rg-hosp-his-prod-twn` | HIS 正式環境資源群組 |
| Virtual Network | `vnet-hosp-his-prod-twn-001` | HIS 主要虛擬網路 |
| Subnet（應用層） | `snet-hosp-his-prod-app` | 應用程式服務子網路 |
| Subnet（資料層） | `snet-hosp-his-prod-db` | 資料庫服務子網路 |
| NSG | `nsg-hosp-his-prod-twn-001` | 綁定應用層 Subnet |
| VM（應用伺服器） | `vm-hosp-his-prod-twn-001` | HIS AP Server #1 |
| VM Scale Set | `vmss-hosp-his-prod-twn-001` | HIS 自動擴展群組 |
| SQL Server | `sql-hosp-his-prod-twn-001` | HIS 邏輯 SQL Server |
| SQL Database | `sqldb-hosp-his-prod-001` | HIS 主要資料庫 |
| Storage Account | `sthosphisprod001` | 縮短：st + hosp + his + pr + 001 |
| Key Vault | `kv-hosp-his-prod-twn-001` | HIS 機密金鑰儲存 |
| Log Analytics | `log-hosp-shared-prod-twn-001` | 共用監控工作區 |
| Application Insights | `appi-hosp-his-prod-twn-001` | HIS 應用程式監控 |

---

## 5. 字元限制快速參考

| 資源類型 | 最大字元 | 允許字元 | 命名注意事項 |
|---------|---------|---------|------------|
| Storage Account | 24 | 英數字 | 不允許連字號，全小寫，建議 `st + org + wl + env + inst`（縮短各元件） |
| AKS Node Pool | 12 | 英數字 | 不允許連字號，必須以英文字母開頭 |
| Container Registry | 50 | 英數字 | 不允許連字號，格式：`cr{org}{wl}{env}` |
| Key Vault | 24 | 英數字、連字號 | 需全球唯一，建議加入 region 後綴 |
| Virtual Machine | 15 | 英數字、連字號 | Windows VM 限 15 字元（含 OS 前綴） |
| SQL Server (Logical) | 63 | 英數字、連字號 | 需全球唯一，建議加入 region |
| App Service | 60 | 英數字、連字號 | 需全球唯一，URL 為 `{name}.azurewebsites.net` |
| Resource Group | 90 | 英數字、連字號、底線、括號 | Subscription 內唯一即可 |
| Management Group | 90 | 英數字、連字號、底線 | 全租戶唯一 |

---

## 6. Tag（標籤）強制規範

命名無法承載所有資訊，搭配以下強制 Tag 進行資源治理與成本分配：

| Tag 鍵 | 範例值 | 說明 |
|--------|-------|------|
| `Environment` | `prod` / `dev` / `test` | 環境識別，對應命名中的 env 欄位 |
| `Workload` | `his` / `pacs` / `ris` | 工作負載系統，對應命名中的 wl 欄位 |
| `Owner` | `it-team@hospital.org` | 資源負責人或負責團隊 email |
| `CostCenter` | `CC-IT-001` | 成本中心代碼，供財務部門分攤費用 |
| `Department` | `radiology` / `icu` / `admin` | 使用部門（放射科、加護病房、行政） |
| `Compliance` | `hipaa` / `gdpr` / `twhca` | 適用的法規標準（台灣健保署、HIPAA 等） |
| `BackupPolicy` | `daily-30d` / `weekly-1y` | 備份週期與保留期限 |
| `CreatedBy` | `terraform` / `portal` | 建立方式（IaC 來源或手動） |

{: .important }
建議透過 Azure Policy 強制要求所有資源必須附帶上述 Tag，避免資源孤立無主。

---

## 7. NSG Rule 命名規則

### 7.1 命名格式

NSG Rule 命名代表「這條規則做什麼」，讓人不用開啟詳細設定就能理解意圖：

```
{direction}-{priority}-{protocol}-{source}-{destination}-{action}
```

### 7.2 優先序規劃建議（醫院分層）

| 優先序 | 方向 | 用途 | 說明 |
|--------|------|------|------|
| 100–199 | Inbound | 允許特定管理來源 | 堡壘機（Azure Bastion）或 VPN 連入管理用 |
| 200–299 | Inbound | 允許 AGW / LB 到應用層 | 應用閘道流量進入 App Subnet |
| 300–399 | Inbound | 應用層到資料層 | App Subnet → DB Subnet（TCP 1433 / 3306） |
| 400–499 | Inbound | 醫療裝置或 HL7 介接 | 放射設備、DICOM 3.0 來源 IP 白名單 |
| 900 | Inbound | 拒絕所有其他 Inbound | 預設拒絕規則，**必須存在** |
| 100–199 | Outbound | 允許到 Azure PaaS | Storage、SQL、Key Vault 的 Private Endpoint |
| 200–299 | Outbound | 允許到監控端點 | Log Analytics、Application Insights |
| 300–399 | Outbound | 允許指定醫療系統介接 | HIS ↔ LIS / RIS 跨 VNet 或 On-Prem |
| 900 | Outbound | 拒絕其他 Internet Outbound | 防止資料外洩，需配合 Azure Firewall 做白名單 |

### 7.3 醫院場景完整範例

HIS 正式環境 NSG（`nsg-hosp-his-prod-twn-001`）的 Rule 命名範例：

| Rule 名稱 | 方向 | 優先序 | 目的 |
|----------|------|--------|------|
| `inbound-100-tcp-bastion-app-allow` | Inbound | 100 | 允許 Bastion 連入 App Subnet（管理用） |
| `inbound-200-tcp-agw-app-allow` | Inbound | 200 | 允許 Application Gateway 流量到 App Subnet |
| `inbound-300-tcp-app-db-allow` | Inbound | 300 | App Subnet 可連 DB Subnet（TCP 1433） |
| `inbound-400-tcp-dicom-pacs-allow` | Inbound | 400 | 允許 DICOM 裝置連 PACS Subnet |
| `inbound-900-any-any-any-deny` | Inbound | 900 | 拒絕所有未明確允許的 Inbound 流量 |
| `outbound-100-tcp-app-storage-allow` | Outbound | 100 | App → Storage Private Endpoint |
| `outbound-200-tcp-app-sql-allow` | Outbound | 200 | App → SQL Private Endpoint（TCP 1433） |
| `outbound-300-tcp-app-kv-allow` | Outbound | 300 | App → Key Vault Private Endpoint |
| `outbound-400-tcp-any-loganalytics-allow` | Outbound | 400 | 所有 Subnet → Log Analytics（監控） |
| `outbound-900-any-any-internet-deny` | Outbound | 900 | 拒絕所有 Outbound 到 Internet |

---

## 8. Azure Firewall Rule 命名規則

### 8.1 架構層級

| 層級 | 元件 | 命名格式 | 範例 |
|------|------|---------|------|
| 1 | Firewall Policy | `afwp-{org}-{env}-{region}-{inst}` | `afwp-hosp-prod-twn-001` |
| 2 | Rule Collection Group (RCG) | `rcg-{org}-{type}-{priority}` | `rcg-hosp-net-100` |
| 3 | Rule Collection (RC) | `rc-{org}-{wl}-{env}-{type}-{inst}` | `rc-hosp-his-prod-net-001` |
| 4 | Individual Rule | 依規則類型（見下） | `allow-tcp-app-sql-1433` |

### 8.2 Individual Rule 命名格式

**Network Rule（Layer 4）：**
```
{action}-{protocol}-{source}-{destination}-{port}
```

**Application Rule（Layer 7 FQDN）：**
```
{action}-{source}-{fqdn-category}
```

**DNAT Rule：**
```
dnat-{external-port}-to-{internal-service}-{internal-port}
```
