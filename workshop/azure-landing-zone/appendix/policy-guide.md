---
layout: default
title: 附錄：Azure Policy 規劃指南
parent: Azure Landing Zone 醫療雲端
nav_order: 9
---

# Azure Policy 醫療院所環境規劃與使用指南

{: .highlight }
**版本**：v1.0 ｜ **日期**：2026-03-28 ｜ **依據**：HIPAA、台灣個資法、ISO 27001

---

## 1. Azure Policy 概念架構

### 1.1 四大核心元件

| 元件 | 說明 | 醫院場景 |
|------|------|---------|
| Policy Definition（原則定義） | 單一規則：定義「資源應符合什麼條件」以及不符合時的 Effect | 「所有 Storage Account 必須啟用 HTTPS」、「VM 磁碟必須加密」 |
| Policy Initiative（原則計畫） | 多個 Definition 的組合包，用於一次套用多條相關規則（舊稱 Policy Set） | 「醫院資安基準」Initiative 包含 20 條安全性相關 Policy |
| Policy Assignment（原則指派） | 將 Definition 或 Initiative 指派至特定 Scope（MG / Sub / RG / Resource） | 將「Tag 強制」Policy 指派至 `rg-hosp-his-prod-twn` |
| Compliance State（合規狀態） | Azure 自動評估每個資源是否符合規則，回傳 Compliant / Non-Compliant | 儀表板顯示 HIS 生產環境 87% 合規，清單列出不合規資源供修正 |

### 1.2 Policy Effect（效果）說明

Effect 決定 Policy 在不符合規則時「採取什麼動作」：

| Effect | 強制程度 | 建議階段 | 說明與醫院用法 |
|--------|---------|---------|--------------|
| Audit | 低（只記錄） | 第一階段 | 不阻止操作，但將不合規資源標記於 Activity Log。用於了解現況、不影響業務的起步階段 |
| AuditIfNotExists | 低（只記錄） | 第一階段 | 當關聯資源不存在時才標記（例如：VM 未安裝 Log Analytics Agent） |
| Deny | 高（直接拒絕） | 第三階段 | 直接阻止不符合規則的資源建立/修改。確認不影響業務後才啟用，避免突然封鎖正常操作 |
| DeployIfNotExists | 中（自動補建） | 第二階段 | 當關聯資源不存在時自動部署（例如：自動安裝 Defender Agent）。最省人力的自動化方式 |
| Modify | 中（自動修改） | 第二階段 | 自動修改資源屬性（例如：自動加上必填 Tag、強制開啟 HTTPS） |
| Append | 低（新增設定） | 第二階段 | 在請求中附加額外的屬性（例如：建立資源時自動加入特定 Tag 值） |
| Disabled | 無 | — | 暫時停用 Policy 而不刪除定義，適合測試期間使用 |

---

## 2. 醫院 Policy 指派層級規劃

### 2.1 依管理群組層級指派策略

Policy 從父層 Scope 向下繼承。應在最高適合的層級指派，避免重複設定：

| Scope | 建議指派的 Policy 類型 | 醫院範例 |
|-------|-------------------|---------|
| `mg-hosp-root`（根管理群組） | 全院通用強制規則 | 禁止建立 Classic 資源、強制所有資源加上 Tag、禁止高風險區域（限東亞/日本） |
| `mg-hosp-platform`（平台訂閱） | 平台層安全基準 | Log Analytics 強制連接、Defender for Cloud 計畫強制啟用、網路安全群組記錄 |
| `mg-hosp-landingzone`（工作負載訂閱） | 工作負載通用安全規則 | SQL 加密強制、Storage HTTPS、Key Vault 清除保護、VM 磁碟加密 |
| `mg-hosp-his` / `mg-hosp-pacs`（系統層） | 系統專屬合規規則 | HIS：HIPAA 相關 Policy Initiative；PACS：Storage 複寫設定、DICOM 加密 |
| `rg-hosp-his-prod-twn`（資源群組層） | 環境專屬例外或加強規則 | Prod RG：Deny 效果；Dev RG：Audit 效果（允許較彈性的開發操作） |

### 2.2 Exemption（豁免）使用原則

當特定資源因業務需要而無法符合 Policy 時，應使用 Exemption 機制，而非停用整條 Policy：

| 豁免類型 | 適用情境 | 醫院使用規範 |
|---------|---------|------------|
| Waiver（放棄合規） | 資源永久無法符合規則，已知且接受風險 | 需資安長簽核；設定 6 個月到期後重新審查；必須填寫豁免理由 |
| Mitigated（已緩解） | 資源用其他方式達到相同安全目標 | 例：舊版 PACS 系統無法啟用 TLS 1.2，但已部署 WAF 前置防護；需附上替代控制措施說明 |

{: .important }
所有 Exemption 必須：設定到期日（最長 1 年）、記錄豁免理由、每半年重新審查一次，並存入 ITSM 工單系統備查。

---

## 3. 醫院建議啟用的內建 Policy

### 3.1 身份與存取管理

| Policy 名稱 | Effect | 指派層級 | 醫院說明 |
|------------|--------|---------|---------|
| MFA should be enabled for accounts with owner permissions | Audit | Sub | 訂閱擁有者必須啟用 MFA |
| MFA should be enabled on accounts with write permissions | Audit | Sub | 具寫入權限帳號強制 MFA |
| A maximum of 3 owners should be designated for your subscription | Audit | Sub | 限制 Owner 數量，降低過度授權 |
| There should be more than one owner assigned to your subscription | Audit | Sub | 確保至少 2 名 Owner（避免單點失效） |
| Deprecated accounts should be removed from your subscription | AuditIfNotExists | Sub | 自動偵測並告警停用帳號 |
| External accounts with owner permissions should be removed | Audit | Sub | 偵測 Guest User 持有 Owner 角色 |
| Guest accounts with write permissions should be removed | Audit | Sub | 偵測廠商 Guest 具不當寫入權限 |

### 3.2 資料保護與加密

| Policy 名稱 | Effect | 指派層級 | 醫院說明 |
|------------|--------|---------|---------|
| Secure transfer to storage accounts should be enabled | Deny | MG | 強制 Storage 只接受 HTTPS（禁止 HTTP） |
| Storage accounts should prevent shared key access | Deny | MG | 禁止 Shared Key 存取，改用 Entra ID 認證 |
| Azure SQL Database should have Azure Defender enabled | AuditIfNotExists | LZ MG | SQL DB 啟用 Defender 掃描 |
| Transparent data encryption on SQL databases should be enabled | Deny | LZ MG | SQL Database 強制啟用 TDE 透明加密 |
| Azure Key Vault should have purge protection enabled | Deny | LZ MG | Key Vault 禁止永久刪除，防止誤刪或勒索 |
| Key vaults should have soft delete enabled | Deny | LZ MG | Key Vault 啟用軟刪除（30 天緩衝） |
| Disk encryption should be applied on virtual machines | AuditIfNotExists | LZ MG | VM 磁碟（OS + Data）必須加密 |
| Azure Backup should be enabled for Virtual Machines | AuditIfNotExists | LZ MG | 所有 VM 必須設定備份原則 |

### 3.3 網路安全

| Policy 名稱 | Effect | 指派層級 | 醫院說明 |
|------------|--------|---------|---------|
| Network Watcher should be enabled | AuditIfNotExists | Platform | 確保每個 Region 都有 Network Watcher |
| Flow logs should be configured for every network security group | Audit | LZ MG | NSG Flow Log 必須開啟，供資安分析 |
| Azure DDoS Protection should be enabled | Audit | Sub | 提醒開啟 DDoS 防護（尤其是 Portal 系統） |
| All network ports should be restricted on NSGs | Audit | LZ MG | 偵測過度開放的 NSG 規則（如 0.0.0.0/0） |
| RDP access from the Internet should be blocked | Deny | LZ MG | 禁止公開 Internet 直接 RDP 連線至 VM |
| SSH access from the Internet should be blocked | Deny | LZ MG | 禁止公開 Internet 直接 SSH 連線至 VM |
| Storage accounts should restrict network access using VNet rules | Deny | LZ MG | Storage Account 僅允許 VNet 存取 |
| Azure SQL Managed Instance should avoid public endpoints | Deny | LZ MG | SQL 不直接暴露公開端點 |

### 3.4 稽核與監控

| Policy 名稱 | Effect | 指派層級 | 醫院說明 |
|------------|--------|---------|---------|
| Diagnostic logs in Key Vault should be enabled | AuditIfNotExists | LZ MG | Key Vault 操作日誌必須傳送至 Log Analytics |
| Auditing on SQL server should be enabled | AuditIfNotExists | LZ MG | SQL Server 稽核日誌啟用，保留 90 天 |
| Azure Monitor log profile should collect logs for all categories | AuditIfNotExists | Sub | Activity Log 收集所有類別（含 RBAC、Security） |
| Log Analytics agent should be installed on your virtual machine | DeployIfNotExists | LZ MG | 自動在所有 VM 安裝 Log Analytics Agent |
| Auto provisioning of the Log Analytics agent should be enabled | AuditIfNotExists | Sub | 啟用 Defender for Cloud 自動布建 Agent |
| Azure subscriptions should have a log profile for Activity Log | AuditIfNotExists | Sub | 確保 Activity Log 匯出設定存在 |

---

## 4. 醫院自訂 Policy 設計

### 4.1 強制命名規範（Naming Convention）

確保所有資源命名符合本院《Azure 資源命名規則》文件：

| 資源類型 | 命名模式（Regex） | Effect | 說明 |
|---------|----------------|--------|------|
| Resource Group | `^rg-[a-z]+-[a-z]+-[a-z]+-[a-z]+$` | Deny | 例：`rg-hosp-his-prod-twn`；不符合即拒絕建立 |
| Virtual Network | `^vnet-[a-z]+-[a-z]+-[a-z]+-[a-z]+-\d{3}$` | Deny | 例：`vnet-hosp-his-prod-twn-001` |
| Storage Account | `^st[a-z0-9]{5,18}$` | Deny | 無連字號，24 字元內全小寫英數字 |
| Key Vault | `^kv-[a-z]+-[a-z]+-[a-z]+-[a-z]+-\d{3}$` | Deny | 例：`kv-hosp-his-prod-twn-001` |
| Virtual Machine | `^vm-[a-z]+-[a-z]+-[a-z]+-[a-z]+-\d{3}$` | Deny | 例：`vm-hosp-his-prod-twn-001` |

{: .note }
命名規範 Policy 建議分階段啟用：先 **Audit** 2 週觀察現有不符合資源，再切換為 **Deny**；避免突然封鎖既有工作負載。

### 4.2 強制 Tag（資源標籤）

所有資源必須附帶以下強制 Tag，才能進行成本分攤與合規稽核：

| Tag 鍵 | Effect | 內建 Policy 可用 | 建議設定 |
|--------|--------|----------------|---------|
| `Environment` | Modify（自動補） | 是 | 若未填寫，從 RG 繼承 Environment Tag；RG 層 Deny 未填寫的部署 |
| `Workload` | Deny | 否（自訂） | 資源群組必填 Workload Tag；可用 Append Policy 繼承至子資源 |
| `Owner` | Deny | 否（自訂） | 必須填寫負責人 email；格式驗證包含 @ 符號 |
| `CostCenter` | Deny | 否（自訂） | 對應財務部門成本中心代碼，必填才能進行月度帳單分攤 |
| `Compliance` | Audit | 否（自訂） | 標示適用法規（hipaa/gdpr/twhca）；Audit 效果供稽核查閱 |

### 4.3 禁止或限制特定資源類型

| 規則 | Effect | 醫院理由 |
|------|--------|---------|
| 禁止建立 Classic（傳統）VM 與 Storage | Deny | Classic 資源已停止支援，安全性不足且無法套用現代 RBAC |
| 禁止使用非允許 Azure Region | Deny | 僅允許 Taiwan North、Japan East；資料落地合規（個資法不出亞太） |
| 禁止建立公開 IP 位址（非 AGW / Firewall 用途） | Deny | VM 不得有直接公開 IP；所有外部流量必須經過 AGW / Azure Firewall |
| 禁止非受管理磁碟（Unmanaged Disk） | Deny | 非受管理磁碟無法套用磁碟加密 Policy |
| 禁止 Storage Account 公開存取（匿名讀取） | Deny | 避免 DICOM 影像或病歷資料因設定疏失被公開存取 |
| 限制 VM SKU 至允許清單 | Deny | 避免誤建高成本 GPU VM；生產環境限制 SKU 範圍 |
| 禁止從非核准清單安裝 VM 延伸模組 | Audit | 防止攻擊者安裝惡意 VM Extension 取得持久性存取 |

### 4.4 自訂 Policy 範例（Tag 強制 + Deny）

強制 Resource Group 必須填寫 `Environment` Tag 的自訂 Policy JSON：

```json
{
  "name": "require-tag-environment-on-rg",
  "properties": {
    "displayName": "Resource Group 必須填寫 Environment Tag",
    "description": "醫院合規要求：所有 RG 必須有 Environment 標籤（prod/stg/dev/test）",
    "policyType": "Custom",
    "mode": "Indexed",
    "parameters": {
      "tagValues": {
        "type": "Array",
        "allowedValues": ["prod", "stg", "dev", "test"]
      }
    },
    "policyRule": {
      "if": {
        "allOf": [
          {
            "field": "type",
            "equals": "Microsoft.Resources/resourceGroups"
          },
          {
            "not": {
              "field": "tags['Environment']",
              "in": "[parameters('tagValues')]"
            }
          }
        ]
      },
      "then": {
        "effect": "deny"
      }
    }
  }
}
```

---

## 5. 醫院合規 Initiative（Policy Set）設計

Initiative 將多條相關 Policy 打包為一個可管理的合規套件，一次指派、集中監控合規率：

| Initiative 名稱 | 對應法規 | 包含主要 Policy（範例） |
|---------------|---------|----------------------|
| **Hospital-Security-Baseline**（醫院資安基準） | ISO 27001、TWGCSF | MFA 強制、Owner 數量限制、RDP/SSH 封鎖、磁碟加密、Key Vault 軟刪除、NSG Flow Log、Log Analytics Agent、VM 備份、Storage HTTPS、禁止公開 IP（共 15–20 條） |
| **Hospital-PHI-Protection**（個人健康資訊保護） | 台灣個資法、HIPAA | Storage 禁止匿名存取、SQL TDE、Key Vault 清除保護、稽核日誌保留 90 天、Defender for SQL、儲存帳號 Entra 認證、資料分類標籤（共 10–15 條） |
| **Hospital-Cost-Governance**（成本治理） | 內部財務規範 | 強制 Tag（Environment/CostCenter/Owner）、限制 VM SKU、禁止 Classic 資源、禁止非允許區域、Advisor 成本建議（共 8–12 條） |

{: .note }
Azure 內建的「HIPAA HITRUST 9.2」及「ISO 27001:2013」Initiative 也可直接使用（含 300+ 條 Policy），但建議先以 Audit 模式了解現況缺口，再逐步收緊為 Deny。

---

## 6. 漸進式導入策略

醫院環境不能一次性將所有 Policy 切換為 Deny，必須採「先觀察、再自動修、後強制拒絕」的三階段策略：

| 階段 | 期間（建議） | 主要 Effect | 工作內容 |
|------|-----------|-----------|---------|
| **第一階段（觀察）** | 第 1–4 週 | Audit、AuditIfNotExists | 1. 部署全部 Policy 但設定為 Audit 2. 匯出合規報告，了解現有不合規資源清單 3. 與各系統廠商溝通即將強制的規則 4. 識別需要 Exemption 的例外資源 |
| **第二階段（自動修）** | 第 5–8 週 | DeployIfNotExists、Modify、Append | 1. 將 Log Agent 安裝、Tag 繼承等改為 DeployIfNotExists / Modify 2. 解決自動可修復的不合規項目 3. 建立 Exemption 並完成簽核 4. 目標：合規率達 80% 以上 |
| **第三階段（強制）** | 第 9 週起 | Deny（逐條啟用） | 1. 先對 Dev/Test 環境切換為 Deny，觀察 2 週 2. 確認無業務影響後，對 Prod 環境啟用 Deny 3. 每條 Policy 分別切換，不一次全部改 4. 建立緊急豁免申請 SOP（1 小時內核准機制） |

{: .important }
**緊急豁免 SOP**：當 Deny Policy 影響緊急維護作業時，IT 主管可透過 Exemption API 在 15 分鐘內建立 2 小時臨時豁免，事後補填工單。此流程需於正式啟用 Deny 前完成測試演練。

---

## 7. 合規監控、儀表板與報告

### 7.1 Azure Policy 合規儀表板

| 設定項目 | 建議作法 |
|---------|---------|
| 合規率 KPI 目標 | Security-Baseline Initiative：≥ 95%；PHI-Protection Initiative：**100%**（強制，不允許任何不合規）；Cost-Governance：≥ 90% |
| 每週自動報告 | Policy → Compliance → Export（匯出 CSV）排程；或透過 Azure Monitor Workbook 建立視覺化儀表板 |
| 不合規資源通知 | 建立 Activity Log Alert：當 Policy 合規率低於閾值，自動發送 Email / Teams 通知至資安部門 |
| Defender for Cloud 整合 | Policy 合規狀態自動整合至 Defender for Cloud Security Score，提供統一的資安評分 |
| 稽核報告匯出 | 每季匯出一次完整合規報告（PDF/Excel），附上 Exemption 清單，留存供稽核備查 |

### 7.2 Policy 與其他服務的整合

| 整合對象 | 整合方式與效益 |
|---------|--------------|
| RBAC | Policy 強制 Tag 可輔助 RBAC：只有帶有正確 Tag 的資源才允許特定角色存取（Condition）；搭配使用可達到屬性為基礎的存取控制（ABAC） |
| Defender for Cloud | Defender 的建議事項（Recommendations）直接對應 Policy 定義；啟用 Defender 計畫後，相關 Policy 自動指派並追蹤修補進度 |
| Azure Monitor / Log Analytics | DeployIfNotExists Policy 自動在新 VM 安裝 Log Agent；確保監控覆蓋率 100%，不遺漏任何新建資源 |
| 命名規範文件 | Policy 的命名規範 Deny 規則直接引用《Azure 資源命名規則》文件定義的 Regex Pattern；兩份文件需同步更新 |
| Azure Blueprints（或 Bicep） | 將 Policy Assignment + RBAC + Resource Group 打包為一個可重複部署的 Blueprint；確保每次新建環境都自動套用所有 Policy |

---

## 8. 導入檢查清單（Checklist）

依序完成以下設定，確保 Azure Policy 完整落地於醫院環境：

| 類別 | 項目 |
|------|------|
| ☐ 架構 | 確認 Management Group 層次，規劃每層要指派哪類 Policy（全院 / 平台 / 工作負載 / 系統層） |
| ☐ 初期評估 | 以 Audit 模式部署全部 Policy，匯出合規報告，了解現況缺口（第一階段） |
| ☐ Initiative | 建立 Hospital-Security-Baseline Initiative，包含所有資安基準 Policy |
| ☐ Initiative | 建立 Hospital-PHI-Protection Initiative，涵蓋個資保護相關 Policy |
| ☐ Initiative | 建立 Hospital-Cost-Governance Initiative，強制 Tag 與資源類型限制 |
| ☐ 自訂 Policy | 建立命名規範 Policy（Resource Group / VNet / VM / Storage / Key Vault），先 Audit 後 Deny |
| ☐ 自訂 Policy | 建立 Tag 強制 Policy（Environment / Workload / Owner / CostCenter），搭配 Modify 自動繼承 |
| ☐ 自訂 Policy | 建立禁止特定資源 Policy：Classic 資源、非允許 Region、公開 IP、Storage 匿名存取 |
| ☐ 導入策略 | 第二階段：DeployIfNotExists / Modify Policy 上線（自動安裝 Agent、自動補 Tag） |
| ☐ 導入策略 | 第三階段：先 Dev/Test 環境切 Deny → 確認 2 週 → Prod 環境切 Deny（逐條推進） |
| ☐ 豁免管理 | 建立 Exemption 申請 SOP 及緊急豁免流程（IT 主管可 15 分鐘內核准臨時豁免） |
| ☐ 監控 | 設定合規率低於閾值的 Alert（Security：<95%，PHI：<100%）通知資安部門 |
| ☐ 報告 | 建立每週自動合規報告（Azure Monitor Workbook 或 CSV 匯出） |
| ☐ 稽核 | 排程每季一次完整合規報告匯出，附 Exemption 清單，留存 3 年供稽核備查 |
| ☐ 整合 | 確認 Policy 定義與命名規範文件的 Regex Pattern 保持同步 |

{: .warning }
Policy 導入初期最常見的問題是「Deny 太早，封鎖業務」或「Audit 太久，從未收緊」。建議設定明確的三個月導入時程，並由 IT 主管定期確認推進狀況。
