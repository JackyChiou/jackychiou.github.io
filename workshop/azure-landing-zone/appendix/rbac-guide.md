---
layout: default
title: 附錄：RBAC 規劃指南
parent: Azure Landing Zone 醫療雲端
nav_order: 8
---

# Azure RBAC 醫療院所角色權限規劃指南

{: .highlight }
**版本**：v1.0 ｜ **日期**：2026-04-29 ｜ **依據**：Azure CAF + Zero Trust

---

## 1. Azure RBAC 概念架構

### 1.1 三大核心元件

| 元件 | 說明 | 醫院場景舉例 |
|------|------|------------|
| Security Principal（身份） | 執行操作的對象：使用者、群組、Service Principal、Managed Identity | 資源服務帳號（App Service Managed Identity） |
| Role Definition（角色定義） | 一組允許/拒絕的 Actions 清單，定義「能做什麼」 | Reader、Contributor、Storage Blob Data Reader、自訂 HIS 管理員 |
| Scope（作用範圍） | RBAC 生效的層級：MG > Sub > RG > Resource | `rg-hosp-his-prod-twn`（只能操作 HIS 生產環境 RG 內的資源） |

### 1.2 Scope 層級與繼承規則

Azure RBAC 採「向下繼承」原則：父層指派的角色，子層自動套用。建議在最精細的層級指派，避免過度授權：

| 層級 | Scope 範例 | 建議指派對象 | 注意事項 |
|------|-----------|------------|---------|
| Management Group | `mg-hosp-corp` | 全院管理者（Owner）、資安長（Reader） | 繼承至所有 Subscription，謹慎指派 |
| Subscription | `sub-hosp-prod` | 訂閱管理員 IT 主管 | 避免直接指派 Contributor，改用 RG 層級 |
| Resource Group | `rg-hosp-his-prod-twn` | HIS 系統管理員、HIS 開發團隊 | 最常用層級，依系統/環境區分 RG |
| Resource | `kv-hosp-his-prod-twn-001`（Key Vault） | 特定人員的存取金鑰權限 | 適用高敏感資源（Key Vault、Storage） |

---

## 2. 醫院管理群組與 Subscription 設計

### 2.1 建議的管理群組架構

依據 Azure CAF Landing Zone 設計，醫院建議採用以下管理群組層次：

| 層級 | 管理群組 / Subscription | CAF 對應 | RBAC 設計重點 |
|------|----------------------|---------|--------------|
| L1 | `mg-hosp-root` | Root MG | 僅 Global Admin 及 Cloud Architect 有 Owner；所有 Policy 從此繼承 |
| L2 | `mg-hosp-platform` | Platform MG | 網路、身份、監控等平台訂閱；IT 基礎架構團隊 Contributor |
| L2 | `mg-hosp-landingzone` | Landing Zone MG | 各系統工作負載訂閱（HIS、PACS、Portal），依系統分 Sub-MG |
| L3 | `mg-hosp-his` / `mg-hosp-pacs` | Workload MG | 各系統管理員在此層或 Subscription 層取得 Contributor；稽核人員取得 Reader |
| L2 | `mg-hosp-sandbox` | Sandbox MG | 開發者自由探索用；與正式環境完全隔離，可放寬 Contributor 指派 |

### 2.2 Subscription 規劃建議

| Subscription | 用途 | RBAC 原則 |
|---|---|---|
| `sub-hosp-identity` | Entra ID、PIM、RBAC 集中管理 | 僅 IT 資安主管及身份管理員有 User Access Administrator |
| `sub-hosp-management` | Log Analytics、監控、備份 | IT 維運團隊 Contributor；各系統管理員 Reader（看 Log） |
| `sub-hosp-network` | Hub VNet、Firewall、DNS | 網路工程師 Network Contributor；其他人 Reader |
| `sub-hosp-his-prod` | HIS 生產環境 | HIS 管理員 Contributor（RG 層）；PIM 保護 Sub 層存取 |
| `sub-hosp-pacs-prod` | PACS 生產環境 | 影像管理員 Custom Role；DICOM 服務 Managed Identity |
| `sub-hosp-dev` | 所有系統開發 / 測試環境 | 開發人員 Contributor；不允許存取 prod Subscription |

---

## 3. Azure 內建角色說明與醫院適用建議

### 3.1 通用角色（General Roles）

| 角色名稱 | 權限範圍 | 醫院建議用法 |
|---------|---------|------------|
| Owner | 完整控制 + 指派角色 | 僅限雲端架構師及 IT 主管；建議透過 PIM 啟用，禁止長期持有 |
| Contributor | 資源完整管理，無法指派角色 | 系統管理員於對應 RG 層級；生產環境需 PIM 審核 |
| Reader | 唯讀檢視所有資源 | 稽核人員、臨床資訊需求科室主管查看資源狀態 |
| User Access Administrator | 管理角色指派 | 僅 IT 資安主管；必須配合 PIM + 審核流程，嚴格控管 |

### 3.2 服務專用內建角色（醫院常用）

| 角色名稱 | 適用服務 | 醫院使用場景 |
|---------|---------|------------|
| Storage Blob Data Reader | Azure Storage | PACS 影像查閱服務（唯讀 DICOM 影像） |
| Storage Blob Data Contributor | Azure Storage | HIS 上傳報告、檢驗結果至 Blob（寫入但不刪除） |
| Key Vault Secrets User | Azure Key Vault | 應用程式取用連線字串、API 金鑰（僅讀取 Secret） |
| Key Vault Secrets Officer | Azure Key Vault | IT 工程師管理 Secret 的讀寫（不含憑證/金鑰管理） |
| Key Vault Administrator | Azure Key Vault | 資安長管理所有金鑰、憑證，配合 PIM 使用 |
| SQL DB Contributor | Azure SQL | DBA 管理資料庫結構，不包含 Subscription 層操作 |
| AKS Cluster Admin | Azure Kubernetes | PACS/HIS 容器平台管理員，配合 PIM 使用 |
| AKS RBAC Reader | Azure Kubernetes | 開發人員唯讀 Pod 狀態，不可 exec 進容器 |
| Monitoring Reader | Azure Monitor | 維運人員及臨床資訊科查閱 Log 與 Alerts |
| Backup Contributor | Azure Backup | 備份管理員設定 Recovery Services Vault 排程 |
| Network Contributor | Azure Networking | 網路工程師管理 VNet、NSG、Route Table |

---

## 4. 自訂角色（Custom Role）設計

### 4.1 何時需要自訂角色

當內建角色無法符合最小權限原則時，應設計自訂角色。醫院常見情境：

- 內建 Contributor 權限過大，但又需要超過 Reader 的操作能力（例如：只能重啟 VM，不能新增/刪除）
- 需要跨多個服務的特定組合權限（例如：HIS 管理員需要 VM + SQL + Storage，但不需要 Network）
- PACS 影像管理員只需 Storage Blob 讀寫，不需要整個 Storage Account 管理權
- 稽核人員需要 Log 查閱但完全禁止資源修改

### 4.2 醫院自訂角色範例

#### 角色一：HIS 系統管理員（Custom-HIS-Admin）

| 屬性 | 值 |
|------|---|
| 適用對象 | HIS 系統維運工程師 |
| Assignable Scope | `rg-hosp-his-prod-twn`、`rg-hosp-his-dev-twn` |

允許 Actions：
```
Microsoft.Compute/virtualMachines/start/action
Microsoft.Compute/virtualMachines/restart/action
Microsoft.Compute/virtualMachines/read
Microsoft.SqlVirtualMachine/*
Microsoft.Sql/servers/databases/read
Microsoft.Storage/storageAccounts/blobServices/containers/read
Microsoft.Insights/*/read  （監控唯讀）
```

拒絕 NotActions：
```
Microsoft.Compute/virtualMachines/delete
Microsoft.Storage/storageAccounts/delete
Microsoft.Authorization/*  （禁止改 RBAC）
```

---

#### 角色二：PACS 影像管理員（Custom-PACS-Imaging-Admin）

| 屬性 | 值 |
|------|---|
| 適用對象 | 放射科及 IT PACS 系統管理員 |
| Assignable Scope | `rg-hosp-pacs-prod-twn`（st-pacs Storage Account 專用） |

允許 Actions：
```
Microsoft.Storage/storageAccounts/blobServices/containers/*
Microsoft.Storage/storageAccounts/blobServices/generateUserDelegationKey/action
Microsoft.Storage/storageAccounts/read  （只讀帳戶資訊，不可改設定）
Microsoft.Compute/virtualMachines/read
Microsoft.Compute/virtualMachines/restart/action
```

特別說明：
- 禁止 `storageAccounts/write`（不可修改 Storage Account 設定）
- 禁止 `storageAccounts/delete`
- 禁止 `Microsoft.Authorization/*`

---

#### 角色三：醫院稽核員（Custom-Hospital-Auditor）

| 屬性 | 值 |
|------|---|
| 適用對象 | 內部稽核單位、外部合規稽核師 |
| Assignable Scope | `mg-hosp-corp`（全院，唯讀） |

允許 Actions：
```
*/read                            （所有資源唯讀）
Microsoft.Insights/*/read
Microsoft.OperationalInsights/*/read
Microsoft.Authorization/*/read   （RBAC 指派查閱）
Microsoft.PolicyInsights/*/read  （Policy 合規報告）
```

明確拒絕 NotActions：
```
Microsoft.Authorization/*/write
Microsoft.Authorization/*/delete
所有 */write、*/delete、*/action 操作
```

---

## 5. 各部門角色指派矩陣

| 部門 / 角色 | RG: HIS Prod | RG: PACS Prod | RG: Shared | RG: Network | Key Vault | Monitor / Log | Sub 層 |
|-----------|-------------|--------------|-----------|------------|----------|--------------|-------|
| IT 雲端架構師 | PIM | PIM | PIM | PIM | PIM | Reader | Owner（PIM） |
| IT 資安主管 | Reader | Reader | Contributor | Reader | Custom | Contributor | User Access Admin（PIM） |
| HIS 系統管理員 | Custom | N/A | Reader | Reader | Custom | Reader | N/A |
| PACS 系統管理員 | N/A | Custom | Reader | Reader | Custom | Reader | N/A |
| DBA（資料庫管理） | Contributor | Contributor | N/A | N/A | Custom | Reader | N/A |
| 網路工程師 | Reader | Reader | Reader | Contributor | N/A | Reader | N/A |
| 應用開發人員（Dev） | N/A | N/A | N/A | N/A | Custom | N/A | N/A |
| 維運人員（Ops） | Reader | Reader | Reader | Reader | Reader | Contributor | N/A |
| 稽核人員 | Custom | Custom | Custom | Custom | Reader | Reader | Custom（Reader 全院） |
| 臨床資訊科主任 | Reader | Reader | N/A | N/A | N/A | Reader | N/A |

{: .important }
應用開發人員（Dev）僅能存取 dev/test 環境的 RG，嚴格禁止指派任何 prod RG 的 Contributor 以上角色。

---

## 6. Privileged Identity Management（PIM）設定

### 6.1 為什麼醫院需要 PIM

PIM 讓高權限角色只在需要時才啟用（Just-in-Time），顯著降低帳號被入侵後的爆炸半徑（blast radius）：

- 病患資料屬於高敏感個資，持久性 Owner/Contributor 帳號是重大風險
- 台灣個資法及 HIPAA 要求存取控管與稽核日誌，PIM 提供完整的申請、核准、使用記錄
- 勒索病毒攻擊中，PIM 可阻止攻擊者橫向移動到雲端管理面（Control Plane）

### 6.2 PIM 設定建議

| 角色 | 啟用上限 | 需要核准者 | 設定建議 |
|------|---------|----------|---------|
| Owner（全 Sub/MG） | 1 小時 | 是（IT 主管） | 需填寫工單說明；核准後自動發通知；操作結束後立即停用 |
| User Access Admin | 2 小時 | 是（資安主管） | 同上；所有角色指派變更須記錄並每季審查 |
| Contributor（Prod RG） | 4 小時 | 是（Team Lead） | 適用緊急維護；非緊急操作必須在業務時間內申請 |
| Key Vault Admin | 1 小時 | 是（資安主管） | 每次啟用皆寄送告警郵件至資安部門 |
| AKS Cluster Admin | 4 小時 | 是（Team Lead） | 申請時需填寫 CR（Change Request）單號 |
| Custom-HIS-Admin | 8 小時 | 否（自動核准） | 適用日常維運；但超過 8 小時需重新申請 |

{: .note }
PIM 需要 Microsoft Entra ID P2 授權（或 Microsoft Entra ID Governance）。如果院所尚未啟用，也應至少開啟 Conditional Access（條件式存取）要求 MFA 登入，作為過渡方案。

---

## 7. 實作設定步驟

### 7.1 透過 Azure Portal 指派角色

1. 進入目標 Resource Group：Azure Portal → Resource Groups → `rg-hosp-his-prod-twn`
2. 開啟 Access Control（IAM）：左側選單 → Access control (IAM) → Add → Add role assignment
3. 選擇角色：搜尋角色名稱（例如 Contributor 或自訂的 Custom-HIS-Admin）→ Next
4. 指定成員：Select members → 搜尋 Entra ID 群組（推薦）或個人帳號 → Next
5. 填寫說明：Description 欄位填入變更原因（供稽核追蹤）→ Review + assign
6. 驗證：Assignments 分頁確認新增成功；建議指派「安全性群組」而非個人

### 7.2 透過 Azure CLI 自動化指派（推薦）

```bash
# 查詢角色定義 ID
az role definition list --name 'Custom-HIS-Admin' --query '[].id' -o tsv

# 指派角色給 Entra ID 群組
az role assignment create \
  --assignee-object-id <Entra-Group-Object-ID> \
  --role 'Custom-HIS-Admin' \
  --scope /subscriptions/<sub-id>/resourceGroups/rg-hosp-his-prod-twn \
  --assignee-principal-type Group
```

### 7.3 Bicep 自訂角色定義範例

```bicep
resource customRole 'Microsoft.Authorization/roleDefinitions@2022-04-01' = {
  name: guid(subscription().id, 'Custom-HIS-Admin')
  properties: {
    roleName: 'Custom-HIS-Admin'
    description: 'HIS 系統管理員 - 允許 VM 重啟、SQL 讀取、Blob 讀取'
    assignableScopes: [ resourceGroup().id ]
    permissions: [{
      actions: [
        'Microsoft.Compute/virtualMachines/restart/action'
        'Microsoft.Sql/servers/databases/read'
        'Microsoft.Storage/storageAccounts/blobServices/containers/read'
        'Microsoft.Insights/*/read'
      ]
      notActions: [ 'Microsoft.Authorization/*' ]
    }]
  }
}
```

---

## 8. 監控、稽核與定期存取審查

### 8.1 必要的監控項目

| 監控項目 | 工具 | 建議告警條件 |
|---------|------|------------|
| 角色指派新增/移除 | Azure Activity Log | 任何 Owner 或 User Access Admin 的指派變更立即告警至資安單位 |
| Owner 角色持有時間 | Microsoft Entra PIM | 啟用超過設定上限（例如 1 小時未停用）自動強制停用 |
| 異常登入後的操作 | Microsoft Entra ID 登入日誌 | 高風險登入（異地、不明裝置）後的 RBAC 操作立即告警 |
| Key Vault 存取記錄 | Azure Monitor + KV Diagnostic | 非預期 IP 或帳號存取 Secret 立即告警 |
| Policy 合規狀態 | Azure Policy | 每週產出合規報告；不合規的 RBAC 設定 1 工作日內修正 |
| 服務主體憑證到期 | Entra ID App Registration | 90 天前發告警；30 天前發嚴重告警；0 天前通知 IT 主管 |

### 8.2 定期存取審查（Access Review）排程

| 審查類型 | 頻率 | 審查者 | 未響應的處理 |
|---------|------|--------|------------|
| Owner / User Access Admin | 每季（90 天） | IT 主管 + 資安長 | 超過 7 天未確認 → 自動移除角色 |
| Contributor（Prod 環境） | 每半年 | 直屬主管 | 超過 14 天未確認 → 降級為 Reader |
| Custom 角色成員 | 每半年 | 系統負責人 | 超過 14 天未確認 → 暫停存取 |
| Reader（全院） | 每年 | 部門主管 | 超過 30 天未確認 → 自動移除 |
| Managed Identity | 每年 | IT 架構師 | 確認服務是否仍使用；廢棄服務立即刪除 |
| 離職員工帳號 | 即時（HR 通知） | IT 帳號管理員 | 收到離職通知 24 小時內停用帳號及移除所有角色 |

### 8.3 合規法規對應

| 法規 / 標準 | 相關 RBAC 要求 | 對應設定 |
|-----------|--------------|---------|
| 台灣個資法 | 個資存取限制、存取日誌保存 | RBAC 最小權限 + Activity Log 保存 90 天以上 |
| HIPAA（美國） | PHI 存取控管、帳號唯一識別、稽核軌跡 | 個人帳號指派（禁止 shared account）+ PIM 啟用日誌 |
| TWGCSF（台灣政府） | 權限分離、定期審查 | 職責分離（HIS 管理員不可同時是稽核員）+ 每季審查 |
| ISO 27001 | A.9 存取控管、A.12.4 日誌記錄 | Azure Policy 強制 RBAC + Log Analytics 保存 |

---

## 9. 外部廠商 Guest User（Azure B2B）管理

### 9.1 適用場景與機制說明

醫院資訊系統（HIS、PACS、RIS、LIS 等）通常委由不同廠商維護。透過 Azure Entra ID B2B Guest Invite，可讓外部廠商工程師以其自有帳號進入醫院 Azure 環境，但僅能存取指定的資源範圍：

| 方式 | 適用對象 | 醫院建議 |
|------|---------|---------|
| B2B Guest Invite（本文主要方式） | 廠商工程師以自有公司帳號（例如 user@partner.com）存取 | 廠商有自己的 Entra ID；工程師用既有帳號登入；醫院側指派最小 RBAC 角色至指定 RG |
| Azure Lighthouse | 受委託的 MSP / 系統整合商管理整個 Subscription | 適合長期、跨 Sub 的委外管理合約；廠商不需要被 invite，透過 Lighthouse delegation 存取 |
| Managed Identity（服務帳號） | 廠商部署的應用程式或自動化腳本 | 服務帳號不要用 Guest User；改用 Managed Identity |

### 9.2 各廠商系統存取範圍原則

依系統劃分廠商存取範圍，每家廠商只能存取其負責維護的 Resource Group，不可跨系統。建議設定：
- 最小角色：Custom 角色（限定操作）或 Contributor（限定 RG）
- 必須啟用 MFA（Conditional Access 強制）
- 設定存取有效期（最長 6 個月），到期前提醒重新評估
- 所有操作透過 Bastion，禁止直接 Public IP 存取
