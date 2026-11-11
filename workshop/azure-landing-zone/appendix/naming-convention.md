---
layout: default
title: 附錄：命名規則規範
parent: Azure Landing Zone 醫療雲端
nav_order: 7
---

# Azure 資源命名規則規範（醫療機構適用版）

{: .highlight }
**版本**：v1.1 ｜ **日期**：2026-11-11 ｜ **依據**：[Azure 官方命名限制](https://learn.microsoft.com/azure/azure-resource-manager/management/resource-name-rules) 與 Azure Cloud Adoption Framework (CAF)

本文件的前綴、元件順序、小寫、環境標示及標籤要求是**本專案自訂慣例**，不是所有 Azure 資源的強制規則。Azure 的字元、長度、唯一性與固定名稱限制優先適用；第 5 節列出服務限制，不是資源數量或容量 quota。

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
| 全小寫英數字 | 自訂元件使用小寫英文字母 (a-z) 與數字 (0-9)，必要時以連字號分隔；Azure 指定的固定名稱除外 |
| 連字號分隔 | 一般欄位以連字號 (-) 分隔；Storage Account、AKS Node Pool、Container Registry 等採緊接方式。自訂名稱不使用底線或空格，不以連字號開頭、結尾，也不連續使用連字號 |
| 禁止縮寫醫療資料 | 名稱中不得含有病患 ID、員工編號、病房名稱等可識別個人的資訊（HIPAA / GDPR 合規） |
| 最大長度限制 | 自訂名稱原則不超過 63 字元，並取本專案上限與該資源 Azure 上限的較小值；連字號也計入長度。不可只檢查單一元件 |
| 環境標示 | 一般使用 `prod` / `stg` / `dev` / `test`；受限短名稱使用第 2.2 節代碼。管理群組、Policy、規則與固定子網路等依專屬格式，由父資源或治理範圍識別環境 |
| 實例編號保留 | 格式含 `{inst}` 時，使用 `001`～`999`；無實例欄位的格式不另加編號 |

---

## 2. 元件代碼對照表

### 2.1 醫院代碼（org）

本專案使用 2～6 碼小寫英數字、以英文字母開頭的醫院英文縮寫（含下表的 `vghtpe`）。較短上限的資源另受整體長度預算限制，不代表所有 org / workload 都能任意組合。

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

僅 AKS Node Pool 與 VM 主機名稱等短格式使用 `{env-short}`：`prod` → `p`、`stg` → `s`、`dev` → `d`、`test` → `t`。這是本專案固定映射，不可臨時以 `pr` 等未定義縮寫取代；`Environment` Tag 仍使用完整代碼。

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

`{wl}` 是 `{workload}` 的簡寫；本專案使用 3～6 碼小寫英數字、以英文字母開頭。`{region}` 固定使用上表的 3 碼代碼（不是 Azure API 的 location 值）。其他自訂元件如 `{role}`、`{svc}`、`{scope}`、`{rule}`、`{level}`、`{purpose}` 必須非空、以英數字開頭與結尾；內含連字號時不可連續使用，並計入最終名稱長度。

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

{: .important }
上表只適用一般工作負載子網路。VPN / ExpressRoute Gateway 使用 `GatewaySubnet`；Azure Firewall 使用 `AzureFirewallSubnet`，需要管理子網路時使用 `AzureFirewallManagementSubnet`；Bastion 需要專用子網路的部署使用 `AzureBastionSubnet`。必須保留這些固定名稱與大小寫，不加前綴、環境或編號，也不可將保留名稱用於一般工作負載。Application Gateway 的子網路則沒有強制固定名稱。來源：[Gateway 設定](https://learn.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings#gateway-subnet)、[Hub-Spoke 子網路規劃](https://learn.microsoft.com/azure/networking/cross-service-scenarios/design-secure-hub-spoke-network#network-foundation-vnet-and-subnet-planning)。

### 3.2 運算資源

| 資源類型 | 縮寫 | 命名格式 | 範例 |
|---------|------|---------|------|
| Virtual Machine（ARM 資源名稱） | `vm` | `vm-{org}-{wl}-{env}-{region}-{inst}` | `vm-hosp-his-prod-twn-001` |
| VM Scale Set（ARM 資源名稱） | `vmss` | `vmss-{org}-{wl}-{env}-{region}-{inst}` | `vmss-hosp-his-prod-twn-001` |
| AKS Cluster | `aks` | `aks-{org}-{wl}-{env}-{region}-{inst}` | `aks-hosp-pacs-prod-twn-001` |
| AKS Node Pool（Linux） | `np` | `np{wl}{env-short}{inst}`（無連字號） | `nppacsp001` |
| AKS Node Pool（Windows） | `np` | `np{env-short}{inst}`（無連字號，工作負載由叢集識別） | `npp001` |
| App Service Plan | `asp` | `asp-{org}-{wl}-{env}-{region}-{inst}` | `asp-hosp-portal-prod-twn-001` |
| App Service / Web App | `app` | `app-{org}-{wl}-{env}-{region}-{inst}` | `app-hosp-portal-prod-twn-001` |
| Azure Function App | `func` | `func-{org}-{wl}-{env}-{region}-{inst}` | `func-hosp-lis-prod-twn-001` |
| Container Registry | `cr` | `cr{org}{wl}{env}`（全小寫無分隔） | `crhosphisprod` |

{: .note }
AKS Node Pool 僅允許小寫英數字且以字母開頭；Linux 為 1～12 字元，Windows 為 1～6 字元。上列格式最長分別為 12 與 6 字元，不可直接套用完整 `prod`。Container Registry 為 5～50 字元英數字，本專案統一使用小寫。

{: .important }
VM 的 ARM 資源名稱最多 64 字元，**不等於** OS 的 `computerName`：Windows 為 1～15 字元，Linux 為 1～64 字元；Portal 建立時通常共用同一名稱。保留上列長資源名時，請透過支援分別設定的部署方式指定 `computerName`，本專案短格式為 `vm{wl}{env-short}{inst}`（例：`vmhisp001`，最長 12 字元）；若介面不能分開設定，Windows VM 應改用短名稱。主機名稱須避免在所屬 DNS / 網域中重複，不能只靠不同資源群組區隔。

VMSS 的 `computerNamePrefix` 也必須獨立設定，不可直接複製長資源名；例如 `hisp`，並為平台附加的實例字尾預留空間，確認最終 OS 主機名稱符合 Windows / Linux 上限。API 將此前綴定義為 1～15 字元，不能據此將完整 Windows 主機名稱放寬到超過 15 字元。來源：[Compute 命名限制](https://learn.microsoft.com/azure/azure-resource-manager/management/resource-name-rules#microsoftcompute)、[VMSS OSProfile](https://learn.microsoft.com/azure/templates/microsoft.compute/virtualmachinescalesets#virtualmachinescalesetosprofile)。

### 3.3 儲存與資料庫資源

| 資源類型 | 縮寫 | 命名格式 | 範例 |
|---------|------|---------|------|
| Storage Account | `st` | `st{org}{wl}{env}{inst}`（無連字號） | `sthosphisprod001` |
| Blob Container | （依用途） | `{purpose}-{env}` | `dicom-images-prod` |
| Data Lake Gen2（啟用 HNS 的 Storage Account） | `dls` | `dls{org}{wl}{env}{inst}` | `dlshosppacsprod001` |
| Azure SQL Server | `sql` | `sql-{org}-{wl}-{env}-{region}-{inst}` | `sql-hosp-his-prod-twn-001` |
| SQL Database | `sqldb` | `sqldb-{org}-{wl}-{env}-{inst}` | `sqldb-hosp-his-prod-001` |
| Cosmos DB Account | `cosmos` | `cosmos-{org}-{wl}-{env}-{region}-{inst}` | `cosmos-hosp-ris-prod-twn-001` |
| Redis Cache | `redis` | `redis-{org}-{wl}-{env}-{region}-{inst}` | `redis-hosp-his-prod-twn-001` |
| Azure Backup Vault（Microsoft.DataProtection/backupVaults） | `bvault` | `bvault-{org}-{env}-{region}-{inst}` | `bvault-hosp-prod-twn-001` |

{: .important }
Storage Account 與 Data Lake Gen2 帳戶均為 **3～24 字元、僅小寫英數字、全球唯一**。依本文件 org / wl 上限與完整環境代碼，`st + org + wl + env + inst` 最長 21 字元，`dls + org + wl + env + inst` 最長 22 字元，無須縮寫 `prod`。帳戶名稱之外，Data Lake 檔案系統名稱遵守 Blob Container 規則：3～63 字元、小寫英數字及單一連字號，首尾為英數字；一般 `{purpose}-{env}` 格式不適用服務特殊容器，例如 `$root`。來源：[Container 命名規則](https://learn.microsoft.com/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata#container-names)。

### 3.4 身份與安全資源

| 資源類型 | 縮寫 | 命名格式 | 範例 |
|---------|------|---------|------|
| Key Vault | `kv` | `kv-{org}-{wl}-{env}-{region}-{inst}` | `kv-hosp-his-prod-twn-001` |
| Managed Identity | `id` | `id-{org}-{wl}-{env}-{role}` | `id-hosp-aks-prod-workload` |
| Log Analytics Workspace | `log` | `log-{org}-{wl}-{env}-{region}-{inst}` | `log-hosp-shared-prod-twn-001` |
| Application Insights | `appi` | `appi-{org}-{wl}-{env}-{region}-{inst}` | `appi-hosp-portal-prod-twn-001` |
| Resource Group | `rg` | `rg-{org}-{wl}-{env}-{region}` | `rg-hosp-his-prod-twn` |
| Management Group | `mg` | `mg-{org}-{level}` | `mg-hosp-corp` |
| Azure Policy Definition | `policy` | `policy-{org}-{scope}-{rule}` | `policy-hosp-sub-tagging` |

{: .important }
Key Vault 的完整格式長度為 `13 + len(org) + len(wl) + len(env)`，最多 24 字元。因此 `prod` / `test` 時 org 與 wl 合計最多 **7** 碼，`stg` / `dev` 時合計最多 **8** 碼；例如 `kv-hosp-his-prod-twn-001` 為 24 字元，合法，但 `kv-vghtpe-portal-prod-twn-001` 為 29 字元，不可直接部署。超長時須先登記較短且不衝突的 org / wl 別名（例如 `kv-vgh-ptl-prod-twn-001`），將完整資訊放在 Tag；不可任意截斷或直接加字尾而超過上限。

Azure Policy 的 resource name 與 display name 不同：Definition 的 resource name 上限 64；Assignment 一般上限 64，**管理群組範圍為 24**。若以 `policy-{org}-{scope}-{rule}` 作 Assignment，也要套用相應上限（本專案的 63 字元慣例不能取代此限制）。

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
| VM（應用伺服器） | `vm-hosp-his-prod-twn-001` | ARM 資源名稱；Windows `computerName` 另設 `vmhisp001` |
| VM Scale Set | `vmss-hosp-his-prod-twn-001` | ARM 資源名稱；`computerNamePrefix` 另設 `hisp` |
| SQL Server | `sql-hosp-his-prod-twn-001` | HIS 邏輯 SQL Server |
| SQL Database | `sqldb-hosp-his-prod-001` | HIS 主要資料庫 |
| Storage Account | `sthosphisprod001` | st + hosp + his + prod + 001（15 字元） |
| Key Vault | `kv-hosp-his-prod-twn-001` | HIS 機密金鑰儲存 |
| Log Analytics | `log-hosp-shared-prod-twn-001` | 共用監控工作區 |
| Application Insights | `appi-hosp-his-prod-twn-001` | HIS 應用程式監控 |

---

## 5. Azure 強制命名限制快速參考

以下依 [官方命名限制](https://learn.microsoft.com/azure/azure-resource-manager/management/resource-name-rules) 整理；「英數字」包含大小寫，「小寫英數字」僅 a-z、0-9。本專案仍採較嚴格的小寫、英數字首尾與單一連字號慣例。`RG` 指資源群組；唯一性不是只看名稱格式，加入 region 或 instance **不保證**全球可用。

| 資源類型 | Azure 長度（最小～最大） | Azure 允許字元及首尾限制 | 唯一性範圍 |
|---------|---------|---------|------------|
| VNet、Private Endpoint | 2～64 | 英數字、`_`、`.`、`-`；英數字開頭，英數字或 `_` 結尾 | RG |
| Subnet | 1～80 | 同上；固定基礎設施名稱見第 3.1 節 | VNet |
| NSG、Route Table、Public IP、Load Balancer、Application Gateway、Azure Firewall、VPN Gateway（virtualNetworkGateways）、Firewall Policy | 1～80 | 英數字、`_`、`.`、`-`；英數字開頭，英數字或 `_` 結尾 | RG |
| NSG Rule | 1～80 | 英數字、`_`、`.`、`-`；英數字開頭，英數字或 `_` 結尾 | NSG |
| VM / VMSS ARM 資源名稱 | 最多 64 | 本專案採英數字及 `-`，英數字首尾；OS 名稱另受下列限制 | RG |
| VM OS 主機名稱 | Windows 1～15；Linux 1～64 | 不可空格、控制字元或 Compute 表列特殊字元；本專案採小寫英數字及 `-`，英數字首尾 | 所屬 DNS / 網域需避免重複 |
| AKS Cluster | 1～63 | 英數字、`_`、`-`；英數字首尾 | RG |
| AKS Node Pool | Linux 1～12；Windows 1～6 | 小寫英數字，字母開頭 | AKS Cluster |
| App Service Plan | 1～60 | 英數字、`-`、可轉為 Punycode 的 Unicode | RG |
| App Service / Function App | 2～60 | 英數字、`-`、可轉為 Punycode 的 Unicode；不可 `-` 開頭或結尾 | 一般為全球；ASE 為其網域內 |
| Container Registry | 5～50 | 英數字（本專案採小寫），不可連字號 | 全球 |
| Storage Account / Data Lake Gen2 帳戶 | 3～24 | 僅小寫英數字 | 全球 |
| Blob Container / Data Lake 檔案系統 | 3～63 | 小寫英數字及 `-`；英數字首尾，不可連續 `-`（特殊容器例外） | Storage Account |
| SQL Server（Logical） | 1～63 | 小寫英數字及 `-`；不可 `-` 開頭或結尾 | 全球 |
| SQL Database | 1～128 | 不可含 `<>*%&:\/?` 或控制字元；不可以 `.` 或空格結尾 | SQL Server |
| Cosmos DB Account | 3～44 | 小寫英數字及 `-`；小寫字母或數字開頭 | 全球 |
| Redis Cache（Microsoft.Cache/Redis） | 1～63 | 英數字及 `-`；英數字首尾，不可連續 `-` | 全球 |
| Backup Vault（Microsoft.DataProtection）/ Recovery Services Vault | 2～50 | 英數字及 `-`；字母開頭 | RG |
| Key Vault | 3～24 | 英數字及 `-`；字母開頭、英數字結尾，不可連續 `-` | 全球 |
| User-assigned Managed Identity | 3～128 | 英數字、`-`、`_`；英數字開頭 | RG |
| Log Analytics Workspace | 4～63 | 英數字及 `-`；英數字首尾 | RG |
| Application Insights | 1～260 | 不可含 `%&\/?` 或控制字元；不可以 `.` 或空格結尾 | RG |
| Resource Group | 1～90 | Unicode 字母/數字、`_`、`-`、`.`、`(`、`)`；不可 `.` 結尾 | Subscription |
| Management Group（ID，不是 display name） | 1～90 | 英數字、`-`、`_`、`.`、`(`、`)`；英數字開頭，不可 `.` 結尾 | Tenant |
| Policy Definition / Assignment（resource name） | 1～64；管理群組 Assignment 1～24 | 不可含 `#<>%&:\?/` 或控制字元；不可 `.` 或空格結尾 | 定義 / 指派的 scope |

### 5.1 元件組合與長度預算

以 org 2～6 碼、wl 3～6 碼、env 3～4 碼、region 3 碼、inst 3 碼計算（含分隔符）：

| 格式 | 本專案最短～最長 | 結論 |
|------|------|------|
| `{type}-{org}-{wl}-{env}-{region}-{inst}` | `len(type) + 19`～`len(type) + 27` | 第 3 節使用此格式的類型，除 Key Vault 外皆在各自上限內；Cosmos 最長 33、App / Function 最長 30 / 31 |
| `st{org}{wl}{env}{inst}` / `dls{org}{wl}{env}{inst}` | 13～21 / 14～22 | 保留完整環境代碼，均不超過 24 |
| `cr{org}{wl}{env}` | 10～18 | 不含空格，符合 5～50 |
| `np{wl}{env-short}{inst}` / `np{env-short}{inst}` | 9～12 / 6 | 分別符合 Linux / Windows 上限 |
| `vm{wl}{env-short}{inst}`（主機名稱） | 9～12 | 符合 Windows 15 與 Linux 64 上限 |
| `kv-{org}-{wl}-{env}-{region}-{inst}` | 未加總長限制時為 21～29 | 必須再套用第 3.4 節的 org + wl 預算；超過 24 即拒絕 |

含自由文字的 Subnet、Private Endpoint、Identity、Management Group、Policy、Blob Container、NSG / Firewall Rule 等格式，必須在串接完成後檢查總長；不能因前綴或單一元件合法就視為通過。需縮短時使用已登記且唯一的別名，不可自動截斷。

### 5.2 唯一性、保留字與部署前檢查

全球名稱衝突時，應在既有格式及長度預算內調整已登記的 org / wl 或 instance，並重新檢查；不要一律附加隨機字串。Container Registry 格式沒有 instance 時，可先登記不同的 org / wl 別名。

具可存取端點的資源不可使用服務保留字或商標，即使端點尚未啟用也可能被拒絕；例如 `MICROSOFT` / `WINDOWS` 不可作為子字串，`LOGIN` 不可作為名稱開頭。完整清單與適用判定見 [ReservedResourceName](https://learn.microsoft.com/azure/azure-resource-manager/templates/error-reserved-resource-name)。固定子網路與 `$root` 等服務指定名稱則依服務例外處理。

Function App 雖允許 60 字元，預設 host ID 只取名稱前 32 字元；若共用 Storage Account，需確認 host ID 不碰撞。本文件格式最長 31 字元，但修改格式或附加字尾後仍須重新檢查。來源：[Microsoft.Web 限制](https://learn.microsoft.com/azure/azure-resource-manager/management/resource-name-rules#microsoftweb)。

本文件確認靜態格式、範例與長度邊界；**未連線查詢 Azure 名稱可用性，亦未實際部署**。全球名稱是否已被占用、已刪除名稱的保留狀態、服務的保留字判定及 VMSS 最終產生的主機名稱，仍須在目標雲端與部署設定下確認；符合此表不等於保證可建立。

---

## 6. Tag（標籤）強制規範

命名無法承載所有資訊，搭配以下強制 Tag 進行資源治理與成本分配：

| Tag 鍵 | 範例值 | 說明 |
|--------|-------|------|
| `Environment` | `prod` / `stg` / `dev` / `test` | 環境識別，短名稱也使用完整 Tag 值 |
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

Azure 強制規則為 **1～80 字元、同一 NSG 內名稱唯一**；允許英數字、`_`、`.`、`-`，以英數字開頭、英數字或 `_` 結尾。本專案採較嚴格的小寫與連字號格式，且自訂名稱上限 63；`source` / `destination` 放角色別名，不直接放含 `/` 的 CIDR 或 `*`。名稱中的 `any` 只是可讀標籤，不能代替實際規則欄位。

### 7.2 優先序規劃建議（醫院分層）

下列分段與 900 拒絕規則是本專案設計，不是 Azure 強制命名規則。Azure 自訂規則 priority 為 100～4096；名稱中的 priority 必須與實際設定一致，並維持同方向內 priority 唯一。來源：[NSG security rules](https://learn.microsoft.com/azure/virtual-network/network-security-groups-overview#security-rules)。

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

Firewall Policy 的 Azure 名稱限制為 1～80 字元、RG 內唯一，字元與首尾規則同網路資源。RCG、RC 與 Individual Rule 在本專案皆採 1～63 字元小寫英數字及單一連字號、英數字首尾，並分別在父 Policy、RCG、RC 內維持名稱唯一；此為專案約束，不宣稱是所有 Firewall Policy 子欄位的官方完整限制。

官方 [classic rule collection](https://learn.microsoft.com/azure/firewall/rule-processing#rule-processing-by-using-classic-rules) 明列 RC 名稱最多 80 字元及英數字、`_`、`.`、`-` 的首尾規則；但 [Firewall Policy RCG API](https://learn.microsoft.com/azure/templates/microsoft.network/firewallpolicies/rulecollectiongroups#property-values) 對 RCG / RC / Rule 的 name 僅列字串，未列完整字元及長度限制，不能直接把 classic 或 `firewallPolicies/ruleGroups` 的限制當成這些欄位的硬性規則。此部分仍需在部署時由實際 API 驗證。

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

`source`、`destination`、`fqdn-category` 使用簡短角色或分類別名（例：`app`、`sql`），不要直接把 FQDN、URL、CIDR、萬用字元或逗號清單嵌入名稱。名稱表達意圖，實際協定、連接埠與流量動作仍以規則設定為準；串接後須再檢查總長。
