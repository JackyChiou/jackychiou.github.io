---
layout: default
title: 06：Entra 高權限治理
parent: Azure AI 資安與治理實戰
grand_parent: Workshop
nav_order: 7
permalink: /workshop/azure-ai-security-governance/entra-governance/
---

# 06：Entra 高權限治理

**目標**：建立最小權限、緊急存取與複核流程。測試租戶演練約 60 分鐘；沒有測試租戶就僅做盤點與桌上演練。

{: .warning }
此 Lab 可能影響整個租戶的登入與管理權。不得在公司或學校正式租戶直接照做。變更前需有核准、另一位可用管理者、已驗證的緊急存取與回復計畫；不批次刪除管理角色，也不移除自己最後的可用管理途徑。

## 先分清楚權限與授權

Global Administrator 管理 Entra 目錄，不自動等於 Azure Owner；但可藉由 **elevate access** 取得根範圍的 User Access Administrator，因此必須另外監控。

| 能力 | 規劃方式 |
|------|----------|
| Entra Free 基本保護 | 評估 Security defaults、分離管理身分、人工角色複核與緊急存取 |
| Conditional Access | 需符合 P1 或相應授權，不當成 Free 的既有能力 |
| PIM 與進階治理 | 依 P2 / ID Governance 等實際功能授權確認，不以本課代替授權評估 |
| Azure RBAC 安全性群組 | 與 Entra 目錄的 role-assignable group 不同；後者有額外授權要求 |

## 操作一：先驗證緊急存取

1. 在測試租戶確認至少 **兩個**僅雲端的緊急存取帳號，使用該租戶的 `onmicrosoft.com` 網域，不依賴地端同步或聯盟。
2. 依[官方緊急存取指引](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/security-emergency-access)配置 Global Administrator 的永久啟用指派及抗網路釣魚驗證，例如 FIDO2 安全金鑰。不要把緊急帳號誤設為免 MFA。
3. 憑證與日常管理驗證方式應有不同失效依賴，安全分開保管，限制授權取用者；不要公開帳號全名、保管位置或序號。
4. 從指定安全工作站測試實際登入與管理作業，確認通知與稽核流程。至少每 90 天及人員 / 設定變更後再次驗證。

若使用 Conditional Access，依官方指引排除會封鎖緊急登入的原則，同時保有抗網路釣魚驗證、監控與定期測試。**Security defaults 不支援逐帳號例外**，不要照搬 CA 排除步驟或為了測試而關閉 MFA。

## 操作二：評估 MFA 保護

1. 在測試租戶檢查目前授權、既有 Conditional Access 與驗證政策，不建立互相衝突的控制。
2. Free 情境可在 Entra → **Properties → Manage security defaults** 評估啟用。先閱讀[影響範圍](https://learn.microsoft.com/en-us/entra/fundamentals/security-defaults)：會影響使用者 MFA 註冊、管理者驗證與舊式驗證。
3. 在變更前備妥註冊與支援方式、確認緊急帳號相容；記錄原設定後再於測試租戶啟用。
4. 用一般測試使用者與管理者實際登入驗證，不只檢查 Enabled 字樣。已有 P1 / P2 原則的環境應依核准設計檢視 CA，不直接切換為 Security defaults。

## 操作三：先加後減

使用[盤點頁](../inventory/)的角色責任表，為日常管理建立專用管理身分與必要的任務角色，避免把日常信箱直接當最高權限帳號。

1. 選一位**非必要、非自己**的測試管理者，記錄原角色與範圍。
2. 依[最小權限角色對照](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/delegate-by-task)新增所需角色，不直接授予 GA。
3. 由該身分實際測試「應該能做」與「不應該能做」的操作。
4. 確認其他管理途徑可用，才移除這位測試管理者不再需要的 GA 指派；再次驗證工作可完成。
5. Azure 端用相同方式，把日常作業授權縮到資源群組或資源，先測試再減少訂用帳戶 Owner；不要改動非本 Lab 的既有指派。

檢視人數時可參照 Microsoft 最佳實務：GA 少於 5 位、訂用帳戶 Owner 最多 3 位。這是減少過度授權的參考，不是為了湊數而刪除必要管理者；群組、服務身分與緊急帳號也須納入責任檢視。

## 操作四：驗證可觀測性

檢查角色異動的 Entra 稽核紀錄與緊急帳號登入紀錄。Free 的可用記錄保留時間有限，依[官方保留表](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/reference-reports-data-retention)安排檢視；需要日誌匯出或自動告警時先確認授權與成本，不能假設 Free 已提供完整告警。

提升存取應依[官方文件](https://learn.microsoft.com/en-us/azure/role-based-access-control/elevate-access-global-admin)檢視**租戶層級**事件與必要委派；不要為了課堂截圖而在正式租戶提升存取，也不要把訂用帳戶角色異動告警當成已完整涵蓋此事件。若有核准的測試提升，結束後移除其根範圍權限並複查。

## 驗收與回復

完成一份不含個資的結果表：緊急存取已測試、MFA 實際驗證、任務角色正負向測試、角色變更有紀錄，以及下次複核日期。

若測試角色不足，由另一位授權管理者依原紀錄還原**該筆**指派。登入異常先停止變更，以已驗證的恢復途徑處理，不繼續移除角色。回復安全設定需評估保護缺口，不以長期停用 MFA 為解法；刪除測試帳號前確認它不承擔必要的緊急管理功能。

[課程收尾：30 天行動計畫](../action-plan/)
