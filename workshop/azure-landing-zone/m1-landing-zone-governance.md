---
layout: default
title: M1：Landing Zone 治理設計
parent: Azure Landing Zone 醫療雲端
nav_order: 1
---

# M1：Landing Zone 治理設計

{: .highlight }
**預估時間**：Demo 25 分鐘 ＋ Lab 40 分鐘 ｜ **部署區域**：Taiwan North

---

## 一、核心概念說明

### 1.1 Management Group 架構圖（醫療版）

Azure Policy 在醫療集團 MG 層套用後，會自動繼承至所有子 MG 與 Subscription。

```
Tenant Root Group
│
+-- [MG] 醫療集團 MG  <-- Azure Policy（在此層套用，往下全部繼承）
    │
    +-- [MG] Prod-MG（生產環境）
    │   +-- Sub-EMR-Prod    -> HIS / 電子病歷
    │   +-- Sub-PACS-Prod   -> 醫療影像 / DICOM
    │   +-- Sub-WebApp-Prod -> 對外掛號 App
    │
    +-- [MG] NonProd-MG（測試環境）
    │   +-- Sub-Dev-Test    -> 開發 / UAT / 壓力測試
    │
    +-- [MG] Platform-MG（管理平台）
        +-- Sub-Platform    -> Hub 網路 / Log Analytics / DR Vault
```

{: .note }
**架構說明**：Prod-MG 套用最嚴格 Policy；NonProd-MG 允許較寬鬆的 SKU；Platform-MG 管理共用基礎服務，含日誌與 DR。

---

### 1.2 Subscription 策略對照表

| Subscription | 用途 | 資料分類 | 建議 Policy | 計費負責人 |
|---|---|---|---|---|
| Sub-EMR-Prod | HIS / 病歷 | 個人敏感資料 | 最嚴格 | 資訊室 |
| Sub-PACS-Prod | 影像 / DICOM | 醫療影像資料 | 嚴格 + 大容量 | 放射科 |
| Sub-WebApp-Prod | 對外 App | 病患基本資料 | DMZ 管控 | 掛號中心 |
| Sub-Dev-Test | 開發測試 | 去識別化 | 寬鬆 | IT 開發團隊 |
| Sub-Platform | 共用服務 | 日誌 / 設定 | 最嚴格 | 資安長 |

---

### 1.3 RBAC 角色設計矩陣

| 人員 | 角色 | 授權範圍 | 可做 | 不可做 |
|---|---|---|---|---|
| 全院管理師 | Owner | Subscription | 一切操作 | — |
| 網路工程師 | Network Contributor | Resource Group（網路） | 建立/修改網路資源 | 建立 VM、改 RBAC |
| 系統工程師 | Virtual Machine Contributor | Resource Group（VM） | 管理 VM | 改網路、改 IAM |
| 資安稽核員 | Reader | MG 層（繼承） | 查看一切 | 任何修改 |
| 財務人員 | Billing Reader | MG 層 | 查看帳單 | 查看資源設定 |

---

### 1.4 Azure Policy 必套清單

| Policy 名稱 | 效果 | 醫療必要性 | 套用層級 |
|---|---|---|---|
| Allowed locations | Deny | 資料不出台灣（個資法） | Prod-MG |
| Require tag: DataClassification | Audit | 資料分類稽核 | 醫療集團 MG |
| Require tag: CostCenter | Audit | 成本歸屬管理 | 醫療集團 MG |
| No public IP on VMs | Deny | 消除 RDP/SSH 暴露 | Prod-MG |
| Audit diagnostic settings | Audit | 確保所有資源啟用日誌 | 醫療集團 MG |
| Allowed VM SKUs | Deny | 防止超規格建立 | NonProd-MG |

---

## 二、Demo 操作步驟

**總時間：25 分鐘 ｜ 部署區域：Taiwan North**

### 步驟 1：建立 Management Group 階層（5 分鐘）

**操作路徑**：Portal → 搜尋「Management groups」→ 新增

建立順序：
1. `MedCenter-Root`（根 MG，掛在 Tenant Root 下）
2. `MedCenter-Prod`（子 MG，掛在 Root 下）
3. `MedCenter-NonProd`
4. `MedCenter-Platform`

{: .warning }
MG 建立後 **15 分鐘內**才能移動訂閱，請耐心等待。

---

### 步驟 2：將 Subscription 移入對應 MG（3 分鐘）

**操作路徑**：MedCenter-Prod → 詳細資料 → 新增訂閱

- 選取 `Sub-EMR-Prod`
- 選取 `Sub-PACS-Prod`

---

### 步驟 3：指派 RBAC（8 分鐘）

**操作路徑**：Sub-EMR-Prod → 存取控制 (IAM) → 新增角色指派

**指派 Owner：**
- 指派對象：您的帳號（示範用）
- 範圍：Subscription

**指派 Reader：**
- 指派對象：`audit-user@hospital.com`
- 範圍：MedCenter-Prod MG（繼承下去）

**驗證步驟**：切換到稽核帳號 → 確認只能 Read，無法建立資源

---

### 步驟 4：啟用 Activity Log 診斷設定（5 分鐘）

**操作路徑**：訂閱 → 活動記錄 → 診斷設定 → 新增診斷設定

| 設定項目 | 值 |
|---|---|
| 名稱 | `diag-activity-log-emr-prod` |
| 傳送至 | Log Analytics Workspace（`ws-medcenter-logs`） |
| 選擇類別 | Administrative、Security、Alert |
| 保留天數 | 90 天 |
| 區域 | Taiwan North |

---

### 步驟 5：套用 Azure Policy（4 分鐘）

**操作路徑**：MedCenter-Prod MG → Policy → 指派

**Policy 1：Allowed locations**
- 效果：Deny
- 允許位置：Taiwan North、Taiwan North 2

**Policy 2：Require tag on resources**
- 效果：Audit
- 標籤名稱：`DataClassification`

---

## 三、Lab 練習指引

**Lab 目標：完成醫療院所的基礎治理框架 ｜ 預估時間：40 分鐘**

### 前置條件

- 已有 Azure 訂閱（Owner 角色）
- 已準備好 2 個帳號（工程師帳號 + 稽核帳號）
- 區域設定：Taiwan North

---

### Task 1：建立 MG 階層（15 分鐘）

- 建立 4 個 MG，架構如課程圖示
- 將您的訂閱移入 Prod-MG
- **截圖**：MG 階層頁面

---

### Task 2：RBAC 指派（10 分鐘）

- 在 Subscription 層指派 Owner（自己）
- 在 MG 層指派 Reader（稽核帳號）
- 驗證：用稽核帳號登入 → 確認無法建立資源
- **截圖**：IAM 角色指派頁面

---

### Task 3：Activity Log 診斷（5 分鐘）

- 啟用診斷設定，傳送至 Log Analytics
- 執行任意操作（如新增標籤）
- 等 2 分鐘後查看 Log Analytics，執行以下 KQL 查詢：

```kql
AzureActivity
| take 10
```

- **截圖**：Log Analytics 查詢結果

---

### Task 4：Azure Policy（10 分鐘）

- 套用「Allowed locations」= Taiwan North 與 Taiwan Northwest
- 嘗試在 East Asia 建立資源 → 應被拒絕
- **截圖**：Policy 違規畫面

### 完成標準

4 張截圖 + Activity Log 查詢結果顯示操作記錄

---

## 四、分組討論：治理設計決策

**時間分配**：討論時間 30 分鐘 ｜ 每組報告 3 分鐘

### 情境說明

您的醫院即將將以下系統搬上 Azure（部署於 Taiwan North）：

| # | 系統名稱 | 說明 |
|---|----------|------|
| 1 | HIS（醫院資訊系統） | 含病歷，屬個人敏感資料，法規要求最高等級保護 |
| 2 | PACS（醫療影像） | DICOM 影像，每月新增約 5TB，需大容量儲存方案 |
| 3 | 門診 App | 對外服務，病患可存取，需 DMZ 隔離與 WAF 防護 |
| 4 | 研究資料庫 | 去識別化資料，供學術研究，可採較寬鬆 Policy |

### 討論題目

**Q1：您會設計幾個 Subscription？如何命名？**
> 提示：考慮資料分類邊界、計費歸屬、安全隔離需求。是否應為研究資料庫設立獨立訂閱？

**Q2：各角色人員應有什麼 Azure 角色？**
> 提示：IT 工程師、放射科技術人員、資安長、財務主管、外部學術合作者，各自的最小權限為何？

**Q3：哪些 Azure Policy 是醫院「必套用」的？**
> 提示：個資法合規、HIPAA 要求、醫院內部資安規範，優先順序如何？Deny vs. Audit 的考量？

### 報告格式

| 項目 | 說明 |
|------|------|
| 報告時間 | 每組 3 分鐘（含 Q&A 1 分鐘） |
| 報告形式 | 口頭報告，可使用白板或 Azure Portal 示意 |
| 評分重點 | 合理性、完整性、對個資法的考量 |
| 參考架構 | 可參考課程第 1.1 節的架構圖作為起點 |
