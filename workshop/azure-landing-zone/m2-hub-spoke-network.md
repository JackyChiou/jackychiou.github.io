---
layout: default
title: M2：Hub-Spoke 網路架構
parent: Azure Landing Zone 醫療雲端
nav_order: 2
---

# M2：Hub-Spoke 網路架構

{: .highlight }
**預估時間**：Demo 30 分鐘 ＋ Lab 50 分鐘 ｜ **部署區域**：Taiwan North

---

## 一、核心概念說明

### 1.1 Hub-Spoke 完整架構圖（醫療版）

```
                    +--------------------------------------------+
                    |        Hub VNet  10.0.0.0/16               |
                    |                                            |
  院内網路            |  GatewaySubnet        10.0.0.0/27          |
  --VPN/ER--------->|  AzureFirewallSubnet   10.0.1.0/26  <-- 所有流量過這裡
                    |  AzureBastionSubnet    10.0.2.0/27  <-- 管理入口
                    |  ManagementSubnet      10.0.3.0/28  <-- Private EP / DNS
                    +--------------------+-------------------+---+
                                         |  VNet Peering（雙向）  |
              +-------------------------+-+--------+-------------+
              |                                    |
  +-----------+-------+    +-------+--------+    +-----------+----------+
  |  Spoke-EMR        |    |  Spoke-PACS    |    |  Spoke-Web (DMZ)     |
  |  10.1.0.0/16      |    |  10.2.0.0/16   |    |  10.3.0.0/16         |
  |                   |    |                |    |                      |
  |  AppSubnet /24    |    |  Imaging /24   |    |  WebSubnet /24       |
  |  DBSubnet /26     |    |  Archive /24   |    |  APISubnet /24       |
  +-------------------+    +----------------+    +----------------------+
  （HIS/EMR 病歷）          （DICOM 影像）          （掛號 App / FHIR）
  Taiwan North              Taiwan North           Taiwan North
```

---

### 1.2 子網路設計對照表

| 子網路 | IP 範圍 | 用途 | NSG | 路由表 | 備注 |
|--------|---------|------|-----|--------|------|
| GatewaySubnet | 10.0.0.0/27 | VPN/ER Gateway | 不可加 | 不可加 | Azure 限制 |
| AzureFirewallSubnet | 10.0.1.0/26 | Firewall | 不可加 | 不可加 | 名稱固定 |
| AzureBastionSubnet | 10.0.2.0/27 | Bastion | 不可加 | 不可加 | 名稱固定 |
| ManagementSubnet | 10.0.3.0/28 | DNS Resolver / Private EP | 可加 | 可加 | — |
| AppSubnet（EMR） | 10.1.1.0/24 | EMR 應用程式層 | 可加 | 可加 | — |
| DBSubnet（EMR） | 10.1.2.0/26 | 病歷資料庫 | 可加（最嚴格） | 可加 | — |
| ImagingSubnet（PACS） | 10.2.1.0/24 | DICOM 傳輸 | 可加 | 可加 | — |
| WebSubnet（DMZ） | 10.3.1.0/24 | 對外服務 | 可加（WAF） | 可加 | — |

---

### 1.3 NSG 規則設計

#### nsg-emr-db（病歷 DB 最嚴格）

| 優先順序 | 方向 | 來源 | 目的地 | Port | 動作 | 說明 |
|---------|------|------|--------|------|------|------|
| 100 | Inbound | 10.1.1.0/24（AppSubnet） | Any | TCP:1433 | Allow | EMR App → DB |
| 200 | Inbound | 10.0.3.0/28（Management） | Any | TCP:1433 | Allow | DBA 維護連線 |
| 4096 | Inbound | Any | Any | Any | Deny | 拒絕其他一切 |
| 100 | Outbound | Any | 10.1.1.0/24 | Any | Allow | DB 回應 App |
| 4096 | Outbound | Any | Any | Any | Deny | 拒絕其他出站 |

#### nsg-pacs-imaging（DICOM 影像）

| 優先順序 | 方向 | 來源 | Port | 動作 | 說明 |
|---------|------|------|------|------|------|
| 100 | Inbound | 院內 VPN IP Pool | TCP:104,11112 | Allow | DICOM 傳輸 |
| 200 | Inbound | 10.0.0.0/8 | TCP:443 | Allow | 管理存取 |
| 4096 | Inbound | Any | Any | Deny | 拒絕其他 |

---

## 二、Demo 操作步驟

**總時間：30 分鐘 ｜ 區域：Taiwan North ｜ 操作環境：Azure Portal（繁體中文介面）**

### 步驟 1：建立 Hub VNet（8 分鐘）

**操作路徑**：Portal → 虛擬網路 → 建立

| 設定項目 | 值 |
|---|---|
| 名稱 | `hub-vnet` |
| 區域 | Taiwan North |
| 位址空間 | `10.0.0.0/16` |

子網路（逐一新增）：

| 子網路名稱 | IP 範圍 | 備注 |
|---|---|---|
| `AzureFirewallSubnet` | `10.0.1.0/26` | **名稱完全照打！（Azure 限制）** |
| `AzureBastionSubnet` | `10.0.2.0/27` | **名稱完全照打！（Azure 限制）** |
| `GatewaySubnet` | `10.0.0.0/27` | **名稱完全照打！（Azure 限制）** |
| `ManagementSubnet` | `10.0.3.0/28` | 可自訂名稱 |

---

### 步驟 2：建立 Spoke VNets（5 分鐘）

| VNet 名稱 | 位址空間 | 子網路 | 區域 |
|---|---|---|---|
| `spoke-emr-vnet` | `10.1.0.0/16` | AppSubnet（10.1.1.0/24）、DBSubnet（10.1.2.0/26） | Taiwan North |
| `spoke-pacs-vnet` | `10.2.0.0/16` | ImagingSubnet（10.2.1.0/24）、ArchiveSubnet（10.2.2.0/24） | Taiwan North |

---

### 步驟 3：設定 VNet Peering（8 分鐘）

**操作路徑**：hub-vnet → Peering → 新增

| Peering 連結名稱 | VNet 對象 | 重要設定 | 狀態 |
|---|---|---|---|
| `hub-to-emr` | hub-vnet → spoke-emr-vnet | 允許閘道傳輸（Allow gateway transit） | 已啟用 |
| `emr-to-hub` | spoke-emr-vnet → hub-vnet | 使用遠端閘道（Use remote gateways） | 已啟用 |

重複以上步驟完成 Hub ↔ Spoke-PACS

{: .note }
**驗證**：Peering 狀態 = Connected

---

### 步驟 4：建立 NSG 並套用（5 分鐘）

**操作路徑**：建立 NSG `nsg-emr-db`

新增 Inbound 規則：
- 優先順序：100
- 來源：IP Addresses → `10.1.1.0/24`
- 服務：MS SQL（自動填入 TCP:1433）
- 動作：Allow

新增 Deny All：
- 優先順序：4096，All → Any → Deny
- 套用 NSG 至 DBSubnet

---

### 步驟 5：建立路由表（4 分鐘）

建立 Route Table：`rt-spoke-emr`

路由設定：

| 設定項目 | 值 |
|---|---|
| 名稱 | `default-to-firewall` |
| 位址前置詞 | `0.0.0.0/0` |
| 下一個躍點類型 | 虛擬設備（Virtual appliance） |
| 下一個躍點位址 | `10.0.1.4`（Firewall IP，M3 再確認） |

套用至：spoke-emr-vnet 的 `AppSubnet` + `DBSubnet`

---

## 三、Lab 練習指引

**Lab 目標：完成 Hub-Spoke 網路架構建置 ｜ 預估時間：50 分鐘**

**前置條件**：已完成 M1 基礎設定，具備訂閱貢獻者權限

---

### Task 1：建立 Hub-Spoke 架構（25 分鐘）

- 依照 IP 規劃建立 Hub + 2 個 Spoke VNet（Taiwan North）
- 確認每個 VNet 的位址空間不重疊
- 完成 Hub ↔ EMR、Hub ↔ PACS 的雙向 Peering
- **驗證**：Peering 狀態 = Connected

---

### Task 2：NSG 設計（15 分鐘）

- 建立 `nsg-emr-db`（只允許 AppSubnet TCP:1433）
- 套用至 DBSubnet

使用「IP Flow Verify」驗證：

```
// 測試 1：應通過
來源：10.1.1.10（AppSubnet IP）
目的地：10.1.2.10（DBSubnet IP）
Port：1433
預期：Allow ✓

// 測試 2：應拒絕
來源：10.2.1.10（PACS Subnet）
目的地：10.1.2.10（EMR DB）
Port：1433
預期：Deny ✓
```

---

### Task 3：路由驗證（10 分鐘）

- 建立路由表並套用至 AppSubnet
- 在 AppSubnet 建立測試 VM（無 Public IP，Taiwan North）
- 用 Network Watcher → Effective Routes 確認：
  - `0.0.0.0/0` 的 Next Hop = `10.0.1.4`（Firewall）

---

### 加分題

嘗試在 Spoke-EMR 的 AppSubnet 建立 VM，測試是否可以 ping 到 Spoke-PACS 的 VM。

{: .note }
此時 Firewall 尚未部署，記錄結果。M3 套用 Firewall 規則後再測試一次，比較差異。
