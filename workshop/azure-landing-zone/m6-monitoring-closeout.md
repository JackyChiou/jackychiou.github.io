---
layout: default
title: M6：監控管理與課程收尾
parent: Azure Landing Zone 醫療雲端
nav_order: 6
---

# M6：監控管理與課程收尾

{: .highlight }
**預估時間**：Demo 30 分鐘 ＋ Lab 45 分鐘 ｜ **部署區域**：Taiwan North

---

## 一、核心概念說明

### 1.1 監控架構分層圖

Azure 醫療雲端環境中三個監控層次的資料流向：

```
Layer 1：資料收集（被動）— Taiwan North
┌────────────────────────────────────────────────────┐
│ Azure Resources → Diagnostic Logs（診斷日誌）        │
│ ├── VM Security Events, Syslog, Perf Counters      │
│ ├── SQL Audit Logs, Query Performance              │
│ ├── Storage Access Logs, Transaction Logs          │
│ ├── Firewall Network/App Rule Logs                 │
│ └── Activity Log（所有控制面板操作記錄）              │
└──────────────────────────┬─────────────────────────┘
                           │ 傳送至
                           ↓
Layer 2：集中儲存與分析
┌──────────────────────────▼─────────────────────────┐
│ Log Analytics Workspace（ws-medcenter-logs）        │
│ ├── Defender for Cloud（威脅分析）                  │
│ ├── Azure Monitor（效能與可用性）                   │
│ └── Network Watcher（連線品質）                     │
└──────────────────────────┬─────────────────────────┘
                           │ 觸發
                           ↓
Layer 3：警示與回應（主動）
┌──────────────────────────▼─────────────────────────┐
│ Alert Rules → Action Group                         │
│ 電子郵件 → 資安長、IT 主管                           │
│ 簡訊     → 值班工程師手機                            │
│ Teams    → IT 警示頻道                              │
│ ITSM     → ServiceNow（自動建單）                   │
└────────────────────────────────────────────────────┘
```

---

### 1.2 Defender for Cloud 方案優先順序

| 方案 | 月費估算 | 醫療優先度 | 核心偵測能力 |
|------|---------|-----------|-------------|
| Defender for Servers | ~NT$500/VM | ★★★★★ | 勒索軟體、異常程序、FIM 檔案完整性 |
| Defender for Storage | 按用量 | ★★★★★ | PACS 影像異常存取、惡意上傳 |
| Defender for SQL | ~NT$300/核心 | ★★★★★ | SQL Injection、異常查詢、暴力破解 |
| Defender for Key Vault | ~NT$200 | ★★★★☆ | 金鑰異常存取、憑證外洩 |
| Defender for DNS | 含在 Server | ★★★☆☆ | DNS Tunneling、C2 通訊偵測 |

---

### 1.3 Alert 規則設計對照

| Alert 名稱 | 訊號來源 | 觸發條件 | 嚴重性 | 通知對象 |
|-----------|---------|---------|-------|---------|
| 高風險安全事件 | Defender for Cloud | 高嚴重性警示 > 0 | Critical | 資安長 + 值班 |
| 異常登入 | Entra ID Sign-in | 失敗 > 10/分鐘 | High | 值班工程師 |
| EMR DB 連線失敗 | Connection Monitor | 封包遺失 > 5% | High | IT 主管 |
| VM CPU 過高 | Azure Monitor | CPU > 90%，5 分鐘 | Medium | 系統工程師 |
| PACS 存儲異常 | Defender for Storage | 任何高嚴重性 | Critical | 資安長 |

---

### 1.4 醫療合規對照總表

| Azure 元件 | 對應法規要求 | 稽核證明 |
|-----------|------------|---------|
| Management Group + RBAC | 個資法第18條「適當安全措施」 | IAM 設定截圖 |
| Azure Policy（地區限制） | 個資法「資料不境外傳輸」 | Policy 合規報告 |
| Activity Log（90 天） | 醫療法「操作稽核紀錄」 | Log Analytics 查詢 |
| NSG + Azure Firewall | 醫療法施行細則「存取管控」 | Firewall Log |
| Private Endpoint | 零信任「最小存取原則」 | DNS 解析驗證 |
| Azure Bastion | 「消除特權存取風險」 | 無 Public IP 驗證 |
| ASR + Test Failover | 電子病歷管理辦法「備份機制」 | DR 演練記錄 |
| Defender + Alert | HIPAA Technical Safeguard「稽核控制」 | Defender 警示記錄 |

---

## 二、Demo 操作步驟

**總時間：30 分鐘 ｜ 區域：Taiwan North**

### 步驟 1：啟用 Defender for Cloud（5 分鐘）

**操作路徑**：Portal → 適用於雲端的 Microsoft Defender → 環境設定

選取您的訂閱，啟用方案：
- 伺服器（Servers）
- 儲存體（Storage）
- SQL
- Key Vault

代理程式設定：
- 自動佈建 AMA 代理程式
- Log Analytics：`ws-medcenter-logs`（Taiwan North）

---

### 步驟 2：查看並修復安全建議（8 分鐘）

**操作路徑**：Defender for Cloud → 建議 → 依嚴重性排序

選取高嚴重性項目示範修復「應啟用 SQL 伺服器稽核」：
- 快速修正 → 套用至 `emr-sqlserver`
- 稽核記錄傳送至 Log Analytics

展示：安全分數（Secure Score）修復後的提升

---

### 步驟 3：建立 Connection Monitor（7 分鐘）

**操作路徑**：Network Watcher → 連線監視器 → 建立

| 設定參數 | 值 |
|---|---|
| 名稱 | `cm-emr-db-health` |
| 區域 | Taiwan North |
| 來源 | EMR-AppServer（VM） |
| 目的地 | EMR-DB Private Endpoint IP（10.1.2.5） |
| 通訊協定 | TCP，Port：1433 |
| 測試頻率 | 30 秒 |
| 失敗閾值 | 連線遺失百分比 > 5%，延遲 > 100ms |

---

### 步驟 4：建立 Action Group + Alert（8 分鐘）

**Action Group 設定：**

| 設定項目 | 值 |
|---|---|
| 名稱 | `ag-medcenter-oncall` |
| 顯示名稱 | 醫療中心值班 |
| 通知① | 電子郵件（it-oncall@hospital.com.tw） |
| 通知② | 簡訊（+886-9xxxxxxxx） |
| 動作 | Teams Webhook URL |

**Alert Rule 設定：**

| 設定項目 | 值 |
|---|---|
| 訊號 | Defender for Cloud Security Alerts |
| 條件 | 嚴重性 = High or Critical，Count > 0 |
| 評估頻率 | 5 分鐘 |
| 動作群組 | `ag-medcenter-oncall` |
| 名稱 | `alert-high-security-events` |

---

### 步驟 5：模擬觸發 Alert（Live Demo！）

透過 Bastion 連線至 EMR-AppVM，執行以下指令觸發 Defender 警示：

```cmd
net user hacker P@ssw0rd123 /add
```

等待約 3-5 分鐘 → Defender 偵測「在 Azure VM 上偵測到可疑的使用者建立活動」→ Teams 頻道收到通知（現場展示）

示範完成後執行清理：

```cmd
net user hacker /delete
```

---

## 三、Lab 練習指引

**預估時間：45 分鐘 ｜ 區域：Taiwan North**

### Task 1：Defender for Cloud（15 分鐘）

- 啟用 Servers + Storage + SQL 方案
- 查看安全建議，找出 3 項高嚴重性項目
- 修復其中 1 項（選擇最簡單的）
- 記錄完成結果：

| 項目 | 數值 |
|------|------|
| 修復前安全分數 | |
| 修復後安全分數 | |
| 修復的建議名稱 | |

---

### Task 2：Connection Monitor（10 分鐘）

- 建立 EMR App → DB 的連線監視器（Taiwan North）
- 等待 2 分鐘後查看監控圖表
- 確認延遲 < 10ms（私有網路應非常低）
- **截圖**：連線監視器健康狀態圖

---

### Task 3：Alert 實測（20 分鐘）

- 建立 Action Group（包含自己的 Email）
- 建立 Alert Rule：Defender for Cloud 高嚴重性警示
- 在 VM 內執行觸發測試
- 確認收到 Email 通知
- **截圖**：Defender 警示 + Email 通知

---

### Task 4：KQL 查詢（加分，10 分鐘）

執行以下 Log Analytics 查詢：

**查詢 1：最近的安全事件**

```kql
SecurityEvent
| where TimeGenerated > ago(1h)
| summarize count() by EventID, Activity
| order by count_ desc
```

**查詢 2：Firewall 阻擋的連線**

```kql
AzureDiagnostics
| where Category == "AzureFirewallApplicationRule"
| where Action_s == "Deny"
| summarize count() by Fqdn_s
| order by count_ desc
```

---

## 四、課程收尾：成果驗收清單

### 4.1 今日完成項目確認

**治理層**
- ☐ Management Group 階層建立完成
- ☐ RBAC 角色指派完成（Owner / Network Contributor / Reader）
- ☐ Activity Log 診斷設定啟用（Taiwan North，傳送至 Log Analytics）
- ☐ Azure Policy 套用（Taiwan North + Taiwan Northwest 地區限制）

**網路層**
- ☐ Hub-Spoke VNet Peering 設定正確（狀態：Connected）
- ☐ NSG 規則套用並驗證（IP Flow Verify）
- ☐ Azure Firewall 規則生效（Log 可見流量記錄）
- ☐ Bastion 連線測試成功（無需 Public IP）

**安全層**
- ☐ Private DNS Zone 解析正確（nslookup 驗證）
- ☐ ASR 複製狀態顯示「受保護」（Taiwan North → Taiwan Northwest）
- ☐ Test Failover 已執行並記錄 RTO
- ☐ Defender for Cloud 已啟用
- ☐ Alert 測試通知已收到

---

### 4.2 繳交作業

| 文件 | 說明 |
|------|------|
| 架構圖截圖 | Hub-Spoke 完整架構截圖 |
| RBAC 設計表 | 角色指派截圖 |
| Firewall 規則匯出 | fwpol-medcenter JSON 匯出 |
| DR 演練記錄 | Test Failover RTO 記錄表 |
| Alert 觸發截圖 | Defender 警示 + 通知截圖 |
| Git Repo 連結 | 以上截圖上傳 Repo |

{: .important }
**截止時間**：課後 3 天內
