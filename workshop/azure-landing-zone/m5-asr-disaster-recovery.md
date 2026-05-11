---
layout: default
title: M5：ASR 災難復原
parent: Azure Landing Zone 醫療雲端
nav_order: 5
---

# M5：ASR 災難復原（Azure Site Recovery）

{: .highlight }
**預估時間**：Demo 40 分鐘 ＋ Lab 45 分鐘 ｜ **主要區域**：Taiwan North ｜ **DR 區域**：Taiwan Northwest

---

## 一、核心概念說明

### 1.1 ASR 複製架構圖

持續非同步複製架構（Taiwan North → Taiwan Northwest）：

```
主要區域：Taiwan North（生產環境）
┌──────────────────────────────────────────┐
│ spoke-emr-vnet                           │
│  EMR-AppServer (10.1.1.4)         ──┐    │
│  EMR-DBServer (10.1.2.4)          ──┤    │  持續非同步複製
│                                    │    │  RPO ≈ 30 秒
│ spoke-pacs-vnet                    │    │
│  PACS-Server (10.2.1.4)           ──┤    │
└──────────────────────────────────────────┘ │
                                             ↓
DR 區域：Taiwan Northwest（災難復原站）
┌──────────────────────────────────────────┐
│ Recovery Services Vault                  │
│  rsv-medcenter-dr                        │
│                                          │
│  複製磁碟（關機狀態，費用低）：             │
│  EMR-AppServer-replica                   │
│  EMR-DBServer-replica                    │
│  PACS-Server-replica                     │
│                                          │
│ Test Failover VNet（隔離測試用）：         │
│  test-failover-vnet (10.99.0.0/16)       │
└──────────────────────────────────────────┘
```

---

### 1.2 RTO / RPO 目標與 ASR 能力對照

| 醫療系統 | 目標 RTO | 目標 RPO | ASR 可達到 | 補充措施 |
|---------|---------|---------|-----------|---------|
| EMR（急診） | 15 分鐘 | 5 分鐘 | ✅ RTO ~10-15 分鐘 | Traffic Manager 自動切換 |
| HIS（門診） | 1 小時 | 30 分鐘 | ✅ RTO < 1 小時 | Recovery Plan 自動化 |
| PACS（影像） | 4 小時 | 1 小時 | ✅ RTO < 4 小時 | App Consistent 快照 |
| 研究 DB | 24 小時 | 4 小時 | ✅ RTO < 24 小時 | 定期備份即可 |

---

### 1.3 Recovery Plan 啟動順序

當 Failover 從 Taiwan North 觸發至 Taiwan Northwest 時，Recovery Plan 依以下群組順序自動執行：

```
Failover 觸發（Taiwan North → Taiwan Northwest）
│
▼
Group 1：基礎網路（並行，等待 5 分鐘）
  • DNS 伺服器啟動
  • 確認 VNet / Peering（Taiwan Northwest）就緒
│
▼
Group 2：資料層
  • EMR-DBServer 啟動
  • Pre-Action：停止應用程式服務（避免資料不一致）
  • Post-Action：執行 DB SQL 健康檢查
│（等待 DB 就緒後）
▼
Group 3：應用程式層
  • EMR-AppServer 啟動
  • Post-Action：呼叫健康檢查 URL
│（應用程式就緒後）
▼
Group 4：對外服務
  • Web Server / Load Balancer 啟動
  • Post-Action：更新 DNS 指向 DR 端點（Taiwan Northwest）
│
▼
Failover 完成 → 記錄完成時間 → 計算實際 RTO
```

---

### 1.4 ASR 費用估算

以 10 台 VM 為基準，每月預估費用（實際費用依 VM 規格與儲存容量而異）：

| 費用項目 | 計算方式 | 估算（10 台 VM） |
|---------|---------|----------------|
| ASR 保護費 | ~NT$500 / VM / 月 | NT$5,000 / 月 |
| 目標 Storage（Taiwan Northwest） | ~NT$28 / GB / 月 | 視 VM 大小而定 |
| 複製 VM 費用（平時） | 不計算 VM 費用 | NT$0 |
| Failover 後 VM 費用 | 啟動後才計費 | 依 VM 規格 |

{: .important }
複製期間磁碟儲存費用持續計算；Failover 後 VM 才開始計費 VM 費用。建議定期評估儲存層級以控制成本。

---

## 二、Demo 操作步驟

**總時間：40 分鐘 ｜ 主要區域：Taiwan North ｜ DR 區域：Taiwan Northwest**

### 步驟 1：建立 Recovery Services Vault（5 分鐘）

**操作路徑**：Portal → Recovery Services 保存庫 → 建立

| 設定項目 | 值 |
|---|---|
| 名稱 | `rsv-medcenter-dr` |
| 區域 | **Taiwan Northwest** ⚠️ 必須與來源不同區域！ |
| 資源群組 | `rg-dr-management` |

**複寫原則設定：**
- 復原點保留：24 小時
- App 一致快照頻率：4 小時

---

### 步驟 2：啟用 VM 複製（5 分鐘操作，15-20 分鐘等待）

**操作路徑**：rsv-medcenter-dr → Site Recovery → 複製

| 設定項目 | 值 |
|---|---|
| 來源類型 | Azure |
| 來源區域 | Taiwan North |
| 目標區域 | Taiwan Northwest |
| 目標資源群組 | `rg-dr-emr-tnw` |
| 目標虛擬網路 | `dr-spoke-emr-vnet`（預先建立於 Taiwan Northwest） |
| 選取 VM | EMR-AppServer, EMR-DBServer |
| ☑ 多 VM 一致性 | 啟用 Multi-VM Consistency |

等待複製狀態變化：
```
Not Protected → Replicating → Protected
```
預估等待時間：15 ~ 20 分鐘（依 VM 磁碟大小而異）

---

### 步驟 3：執行 Test Failover（10 分鐘）

**操作路徑**：複製項目 → EMR-AppServer → 測試容錯移轉

| 設定項目 | 值 |
|---|---|
| 還原點 | 最近的 App Consistent |
| Azure VNet | `test-failover-vnet` ⚠️ 必須使用隔離 VNet！ |

**RTO 記錄表：**

| 時間點 | 記錄 |
|--------|------|
| Failover 開始時間 | ___:___:___ |
| VM 啟動完成時間 | ___:___:___ |
| 應用程式就緒時間 | ___:___:___ |
| 實際 RTO | _______ 分鐘 |

**驗證項目：**
- ☐ VM 在 Taiwan Northwest 成功啟動
- ☐ 資料庫可連線
- ☐ 應用程式健康檢查通過

{: .important }
完成驗證後，務必執行「**清除測試容錯移轉**」，以刪除 Taiwan Northwest 上的測試 VM 並釋放資源。

---

### 步驟 4：建立 Recovery Plan（10 分鐘）

**操作路徑**：rsv-medcenter-dr → 復原計畫 → 建立

| 設定項目 | 值 |
|---|---|
| 名稱 | `rp-emr-full` |
| 來源 | Taiwan North |
| 目標 | Taiwan Northwest |
| VM | EMR-AppServer, EMR-DBServer |

**自訂群組設定：**
- Group 1（預設）：EMR-DBServer — Post-Action：DB 健康確認腳本
- Group 2：EMR-AppServer — Post-Action：健康檢查 URL 呼叫

```
# Recovery Plan 群組執行邏輯
Group 1 完成後 → 等待 DB 健康確認（最長等待 300 秒）
Group 2 啟動 → 呼叫 https://<DR-AppServer>/health
             → 預期回應：HTTP 200 OK
```

---

## 三、Lab 練習指引

**預估時間：45 分鐘 ｜ 主要區域：Taiwan North ｜ DR 區域：Taiwan Northwest**

### Task 1：啟用複製（20 分鐘等待）

- 對您的測試 VM（Taiwan North）啟用 ASR 複製至 Taiwan Northwest
- 等待狀態變為「受保護（Protected）」
- **截圖**：複製健康狀態頁面

確認複製狀態指令（Azure CLI）：

```bash
az site-recovery replication-protected-item list \
  --resource-group rg-dr-management \
  --vault-name rsv-medcenter-dr \
  --query "[].{name:name, health:properties.replicationHealth}"
```

---

### Task 2：Test Failover（15 分鐘）

- 執行 Test Failover（使用隔離 `test-failover-vnet`）
- 填寫以下 DR 演練記錄表：

| 欄位 | 記錄 |
|------|------|
| 演練日期 | |
| 系統名稱 | EMR-AppServer |
| 還原點時間 | |
| Failover 開始時間 | |
| VM 啟動完成時間 | |
| 應用程式就緒時間 | |
| 實際 RTO（分鐘） | |
| 目標 RTO | 15 分鐘 |
| 是否達標 | ☐ 是 ☐ 否 |

- **截圖**：Taiwan Northwest 的 VM 運行中狀態
- 執行「清除測試容錯移轉」

---

### Task 3：Recovery Plan 設計（10 分鐘）

- 建立 Recovery Plan，加入您的 VM
- 調整啟動群組順序：DB 先於 App

```
# 正確的群組順序
Group 1: EMR-DBServer
    Post-Action: DB 健康確認
Group 2: EMR-AppServer
    Post-Action: 健康檢查 URL

# 錯誤的順序（避免）
# Group 1: EMR-AppServer ← 應用程式先啟動但 DB 未就緒，會導致連線失敗
```

- **截圖**：Recovery Plan 設計頁面

---

## 附錄：常見問題 FAQ

| 問題 | 解答 |
|------|------|
| Q：複製狀態卡在「Replicating」很久？ | 屬正常現象（初次複製整顆磁碟）。確認 Storage Account 無 RBAC 問題，Mobility Service Agent 已安裝。 |
| Q：Test Failover 後 VM IP 不同？ | Test Failover 使用 test-failover-vnet 的 DHCP，IP 會與生產不同，屬預期行為。 |
| Q：如何確認 RPO 是否達標？ | 在 rsv-medcenter-dr → 複製項目 → 監控，查看「最後一致復原點」時間戳記。 |
| Q：Failback 怎麼做？ | Failover 完成後，在復原計畫執行「重新保護（Re-protect）」，再觸發反向 Failover 回 Taiwan North。 |
| Q：多 VM 一致性（Multi-VM）的影響？ | 啟用後 RPO 略增（10-15 分鐘），但確保 DB + App 的崩潰一致性。急診 EMR 建議啟用。 |

---

## 術語對照表

| 術語 | 說明 |
|------|------|
| ASR | Azure Site Recovery — Azure 原生 BCDR 服務 |
| RTO | Recovery Time Objective — 可接受的最長停機時間 |
| RPO | Recovery Point Objective — 可接受的最大資料遺失時間 |
| BCDR | Business Continuity & Disaster Recovery |
| Failover | 將工作負載切換至 DR 站（Taiwan Northwest） |
| Failback | 從 DR 站切回主要區域（Taiwan North） |
| Re-protect | Failover 後重新設定反向複製，為 Failback 準備 |
| App Consistent | VSS 快照，確保應用程式（DB）資料一致性 |
| Crash Consistent | OS 層一致性快照（不含 DB 交易日誌） |
| Test Failover | 隔離環境測試容錯移轉，不影響生產 |
