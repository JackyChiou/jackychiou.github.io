---
layout: default
title: M4：Private DNS / DNS 架構
parent: Azure Landing Zone 醫療雲端
nav_order: 4
---

# M4：Private DNS / DNS 架構

{: .highlight }
**預估時間**：Demo 25 分鐘 ＋ Lab 45 分鐘 ｜ **部署區域**：Taiwan North

---

## 一、核心概念說明

### 1.1 Private DNS 解析流程對照圖

#### 危險：無 Private Endpoint

```
EMR VM --> DNS 查詢：emr-db.database.windows.net
       --> Azure DNS 公開解析
       --> 回傳：104.x.x.x（公開 IP）
       --> 流量路徑：VM --> Internet --> Azure SQL 公開端點
       --> 風險：任何人可從網際網路嘗試連線！
```

#### 安全：有 Private Endpoint + Private DNS Zone

```
EMR VM --> DNS 查詢：emr-db.database.windows.net
       --> Azure DNS (168.63.129.16)
       --> 查找 Private DNS Zone：privatelink.database.windows.net
       --> 找到 A Record：emr-db --> 10.1.2.5
       --> 回傳：10.1.2.5（私有 IP）
       --> 流量路徑：VM --> Private Endpoint（VNet 內）--> Azure SQL
       --> 完全不離開 Azure 骨幹網路！
```

---

### 1.2 院內 DNS 整合架構（Taiwan North）

院內 Active Directory DNS 透過 ExpressRoute/VPN 與 Azure DNS Resolver 整合：

```
院內環境（On-Premises）
┌─────────────────────────────────┐
│  院內 Active Directory DNS      │
│  172.16.1.1                     │
│                                 │
│  條件轉發規則：                  │
│  *.database.windows.net         │
│    -> 10.0.3.4（DNS Resolver）  │
│  *.blob.core.windows.net        │
│    -> 10.0.3.4                  │
│  *.vaultcore.azure.net          │
│    -> 10.0.3.4                  │
└──────────────┬──────────────────┘
               │ ExpressRoute / VPN（Taiwan North）
               ↓
Azure Hub VNet 10.0.0.0/16（Taiwan North）
┌─────────────────────────────────┐
│  DNS Resolver（ManagementSubnet）│
│  Inbound EP：10.0.3.4  <-- 院內 DNS 轉發到此
│  Outbound EP：10.0.3.5  --> Azure Private DNS 查詢
│                                 │
│  Private DNS Zones（已連結）：   │
│  privatelink.database.windows.net
│  privatelink.blob.core.windows.net
│  privatelink.vaultcore.azure.net
└─────────────────────────────────┘
```

---

### 1.3 醫療服務 Private DNS Zone 對照表

| 服務 | Private DNS Zone | 記錄名稱範例 | 解析 IP |
|------|-----------------|-------------|---------|
| Azure SQL | `privatelink.database.windows.net` | `emr-db` | 10.1.2.5 |
| Azure Blob | `privatelink.blob.core.windows.net` | `pacsstorage` | 10.1.3.4 |
| Azure Key Vault | `privatelink.vaultcore.azure.net` | `medkv` | 10.0.3.6 |
| Azure Container Registry | `privatelink.azurecr.io` | `medacr` | 10.0.3.7 |
| Azure Monitor | `privatelink.monitor.azure.com` | — | 系統自動 |
| Azure API Mgmt | `privatelink.azure-api.net` | `fhirapi` | 10.3.2.4 |

---

### 1.4 費用估算

Taiwan North 區域 Private DNS 相關元件月費參考（2026 年 4 月）：

| 元件 | 費用估算 | 備注 |
|------|---------|------|
| Private Endpoint | ~NT$200/月/個 | 含 5GB 資料傳輸 |
| Private DNS Zone | ~NT$15/月/Zone + NT$0.6/百萬查詢 | 極低 |
| DNS Resolver | ~NT$50/月/端點 | Inbound + Outbound 各一個 |
| 超出 5GB 傳輸 | ~NT$28/GB | — |

{: .note }
**ROI 說明**：一次資料外洩醫療業平均損失 US$11M；10 個 Private Endpoint 月費僅 NT$2,000。

---

## 二、Demo 操作步驟

**總時間：25 分鐘 ｜ 區域：Taiwan North**

### 步驟 1：建立 Azure SQL（若已有可跳過）（5 分鐘）

**操作路徑**：Portal → SQL 資料庫 → 建立

| 設定項目 | 值 |
|---|---|
| 名稱 | `emr-db` |
| 伺服器 | `emr-sqlserver` |
| 區域 | Taiwan North |
| 網路 | 公用端點（暫時開啟，步驟 3 會關閉） |

---

### 步驟 2：建立 Private Endpoint（8 分鐘）

**操作路徑**：emr-sqlserver → 網路 → 私人存取 → 建立私人端點

| 設定項目 | 值 |
|---|---|
| 名稱 | `pe-emr-sql` |
| 區域 | Taiwan North |
| 資源類型 | `Microsoft.Sql/servers` |
| 資源 | `emr-sqlserver` |
| 目標子資源 | `sqlServer` |
| VNet | `spoke-emr-vnet` |
| Subnet | `DBSubnet` |
| ☑ 與私人 DNS 區域整合 | 自動建立 DNS Zone |

---

### 步驟 3：關閉公開存取（2 分鐘）

**操作路徑**：emr-sqlserver → 網路 → 公用網路存取：**停用**

確認訊息：「只有私人端點連線可存取」

---

### 步驟 4：驗證 DNS 解析（5 分鐘）

透過 Bastion 連線至 EMR-AppVM，執行以下指令：

```bash
nslookup emr-sqlserver.database.windows.net
```

- ✅ 預期結果：`Address: 10.1.x.x`（私有 IP）
- ❌ 若看到公開 IP：
  - 確認 Private DNS Zone 是否連結到此 VNet
  - Private DNS Zone → 虛擬網路連結 → 確認已連結

---

### 步驟 5：建立 DNS Resolver（5 分鐘）

**操作路徑**：Portal → DNS 私人解析程式 → 建立

| 設定項目 | 值 |
|---|---|
| 名稱 | `dnsresolver-hub` |
| 區域 | Taiwan North |
| VNet | `hub-vnet` |
| Inbound 端點 | ManagementSubnet → IP `10.0.3.4`（靜態） |
| Outbound 端點 | ManagementSubnet → IP `10.0.3.5`（靜態） |

---

## 三、Lab 練習指引

**預估時間：45 分鐘 ｜ 區域：Taiwan North**

### Task 1：Private Endpoint 建立（20 分鐘）

- 為您的 Azure SQL 建立 Private Endpoint（Taiwan North）
- 確認 Private DNS Zone 已自動建立並連結
- 關閉 SQL Server 的公開網路存取
- 驗證（在 VM 內）：

```bash
nslookup [您的 SQL FQDN]
# 確認解析結果是私有 IP（10.1.x.x），非公開 IP
```

---

### Task 2：多 VNet DNS Zone 連結（10 分鐘）

- 在 Private DNS Zone 新增 VNet 連結：
  - 連結 `spoke-pacs-vnet`（讓 PACS 也能存取 SQL）
- 從 PACS Subnet VM 執行同樣的 nslookup
- 確認不同 VNet 都能解析到同一私有 IP

---

### Task 3：DNS Resolver 設定（15 分鐘）

- 建立 DNS Resolver（Taiwan North，Inbound：`10.0.3.4`）
- 模擬院內 DNS 條件轉發測試：

```bash
nslookup [SQL FQDN] 10.0.3.4
# 確認透過 DNS Resolver 也能正確解析
```

---

### 加分題

嘗試為 Azure Key Vault 也建立 Private Endpoint：
- Zone：`privatelink.vaultcore.azure.net`
- 驗證 Key Vault FQDN 同樣解析為私有 IP
