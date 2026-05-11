---
layout: default
title: M3：Azure Firewall + Bastion
parent: Azure Landing Zone 醫療雲端
nav_order: 3
---

# M3：Azure Firewall + Bastion

{: .highlight }
**預估時間**：Demo 35 分鐘 ＋ Lab 40 分鐘 ｜ **部署區域**：Taiwan North

---

## 一、核心概念說明

### 1.1 Azure Firewall 流量路徑圖

Azure Firewall 作為 Hub-Spoke 架構中的集中式流量管控節點，依流向可分為三類：

**南北向流量（對外）**
```
Spoke VM → UDR → Firewall (10.0.1.4) → Application Rules（過濾）→ Internet
                                      ↓（規則不符）
                                   拒絕並記錄 Log
```

**東西向流量（Spoke 之間）**
```
Spoke-EMR → UDR → Firewall → Network Rules（過濾）→ Spoke-PACS
                            ↓（規則不符）
                         拒絕並記錄 Log
```

**管理流量（工程師連線）**
```
工程師瀏覽器 → Azure Portal → Bastion (HTTPS/443) → VM (RDP/SSH)
（無任何 Public IP，無 RDP/SSH 端口暴露）
```

---

### 1.2 Firewall 規則類型對照

| 規則類型 | 匹配依據 | 適用場景 | 支援 IDPS |
|---------|---------|---------|-----------|
| Network Rules | IP + Port + Protocol | Spoke 間流量、DICOM 傳輸 | Premium ✓ |
| Application Rules | FQDN + URL | 對外 HTTP/HTTPS 管控 | Premium ✓ |
| DNAT Rules | 公開 IP:Port → 內部 IP:Port | 對外發布服務（罕用） | ✗ |

---

### 1.3 Firewall Premium vs Standard 對照

| 功能 | Standard | Premium | 醫療場景需求 |
|------|---------|---------|-------------|
| Network Rules | ✓ | ✓ | 基本流量管控 |
| Application Rules（FQDN） | ✓ | ✓ | 對外連線管控 |
| IDPS（入侵偵測防禦） | ✗ | ✓ | **強烈建議（醫療）** |
| TLS 檢測 | ✗ | ✓ | 加密流量檢查 |
| URL 過濾（非 FQDN） | ✗ | ✓ | 細粒度 URL 管控 |
| Web Categories | ✗ | ✓ | 一鍵封鎖社交媒體 |
| 月費估算 | ~NT$18,000 | ~NT$36,000 | **醫療建議 Premium** |

---

### 1.4 Network 規則設計（醫療版）

| 規則名稱 | 來源 | 目的地 | Port | 動作 | 說明 |
|---------|------|--------|------|------|------|
| Allow-EMR-DB | 10.1.1.0/24 | 10.1.2.0/26 | TCP:1433 | Allow | EMR App → 病歷 DB |
| Allow-PACS-DICOM | 10.2.0.0/16 | 院內 VPN IP | TCP:104,11112 | Allow | DICOM 影像傳輸 |
| Allow-DNS-Internal | All Spokes | 10.0.3.4 | UDP:53 | Allow | 內部 DNS 查詢 |
| Deny-EMR-to-PACS | 10.1.0.0/16 | 10.2.0.0/16 | Any | Deny | EMR 不直接存取 PACS |

---

### 1.5 Application 規則設計（醫療版）

**允許清單（Allowlist）：**
- `*.microsoft.com`, `*.windowsupdate.com` → Windows 更新
- `*.azure.com`, `*.azurewebsites.net` → Azure 服務
- `api.nhi.gov.tw` → 健保署 API

**拒絕清單（Denylist）：**
- `*.facebook.com`, `*.youtube.com`, `*.netflix.com` → 社交媒體 / 娛樂
- Web Categories: Social Networking → 一鍵封鎖

---

## 二、Demo 操作步驟

**總時間：35 分鐘 ｜ 區域：Taiwan North**

### 步驟 1：部署 Azure Firewall（等待 5–10 分鐘）

**操作路徑**：Portal → 防火牆 → 建立

| 設定項目 | 值 |
|---|---|
| 名稱 | `fw-hub-medcenter` |
| 區域 | Taiwan North |
| SKU | **Premium**（醫療建議） |
| 防火牆原則 | 新建 `fwpol-medcenter` |
| 虛擬網路 | `hub-vnet` |
| 公用 IP | 新建 `pip-fw-hub`（靜態） |

{: .note }
提示：部署需要 5–10 分鐘，期間可繼續步驟 2。

---

### 步驟 2：建立 Firewall Policy 規則（10 分鐘）

**操作路徑**：fwpol-medcenter → 規則 → 新增規則集合

**Network 規則集合：**
- 名稱：`net-rules-medical`
- 優先順序：100
- 規則 `Allow-EMR-DB`：來源 `10.1.1.0/24` → 目的地 `10.1.2.0/26` → TCP:1433 → Allow
- 規則 `Deny-EMR-to-PACS`：來源 `10.1.0.0/16` → 目的地 `10.2.0.0/16` → Any → Deny

**Application 規則集合：**
- 名稱：`app-rules-medical`
- 優先順序：200
- 規則 `Allow-Essential`：來源 `10.0.0.0/8` → FQDN: `*.microsoft.com`, `api.nhi.gov.tw` → HTTPS:443 → Allow
- 規則 `Deny-Social`：來源 `10.0.0.0/8` → Web Categories: Social Networking → Deny

---

### 步驟 3：更新路由表 Next Hop IP（2 分鐘）

確認 Firewall Private IP：
- `fw-hub-medcenter` → 概觀 → 私人 IP 位址（應為 `10.0.1.4`）

更新 `rt-spoke-emr` 路由：
- `0.0.0.0/0` → 虛擬設備 → `10.0.1.4`（確認 IP）

---

### 步驟 4：部署 Bastion（等待 3 分鐘）

| 設定項目 | 值 |
|---|---|
| 名稱 | `bas-hub-medcenter` |
| SKU | Standard |
| 虛擬網路 | `hub-vnet` |
| 子網路 | `AzureBastionSubnet`（自動偵測） |
| 公用 IP | 新建 `pip-bas-hub` |
| 區域 | Taiwan North |

---

### 步驟 5：流量驗證（Live Demo）

透過 Bastion 連線至 EMR-AppVM（示範無 Public IP），在 VM 內執行下列測試：

```bash
# 測試 1：被封鎖的社交媒體
curl -I https://www.facebook.com
# 預期：連線逾時（Firewall 阻擋）

# 測試 2：允許的 Microsoft 服務
curl -I https://www.microsoft.com
# 預期：HTTP/2 200
```

查看 Firewall Log（KQL）：

```kql
AzureDiagnostics
| where Category == "AzureFirewallApplicationRule"
| take 20
```

---

## 三、Lab 練習指引

**預估時間：40 分鐘 ｜ 區域：Taiwan North**

### Task 1：Firewall 規則驗證（20 分鐘）

- 確認 Firewall 已部署且狀態 = Running
- 在 AppSubnet 建立測試 VM（無 Public IP，Taiwan North）
- 用 Bastion 連線進入 VM
- 執行測試並截圖記錄：

```bash
# 測試 A
curl -I https://www.youtube.com
# 預期：Timeout / 403

# 測試 B
curl -I https://www.microsoft.com
# 預期：200 OK

# 測試 C：ping 10.2.1.4（PACS Subnet VM）
ping 10.2.1.4
# 預期：Timeout（Deny-EMR-to-PACS 規則）
```

---

### Task 2：Bastion Shareable Link（10 分鐘）

- 確認 Bastion SKU = Standard
- 進入 VM → 連線 → Bastion → 建立可分享連結
- 設定有效期：4 小時
- 將連結傳給旁邊的學員，測試對方能否無需帳號連線

---

### Task 3：查看 Firewall Log（10 分鐘）

執行 KQL 查詢找出被阻擋連線：

```kql
AzureDiagnostics
| where Category == "AzureFirewallApplicationRule"
| where Action_s == "Deny"
| project TimeGenerated, msg_s
| take 10
```

**截圖**：顯示 YouTube 連線被阻擋的記錄
