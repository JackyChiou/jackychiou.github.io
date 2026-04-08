---
layout: post
title: "Azure Migrate Collector：無需 Azure 連線即可探索伺服器與工作負載"
date: 2026-04-08
categories: [Azure, Migration]
---

在企業規劃雲端遷移時，第一步往往是盤點現有的 IT 資產——伺服器數量、效能指標、已安裝軟體、資料庫實例等等。過去這個探索流程通常需要在目標環境部署一台連線到 Azure 的 Appliance，但對於高度監管、網路隔離或尚未取得雲端連線核准的環境來說，這就成了一道門檻。

Microsoft 推出的 **Azure Migrate Collector** 正是為了解決這個問題：**它可以在完全沒有 Azure 連線的環境中進行伺服器探索與資料收集**，事後再將收集到的資料匯出並上傳至 Azure Migrate 專案，產生商業案例與評估報告。

## 核心特色：離線探索與資料收集

Azure Migrate Collector 的最大亮點就是**不需要直接連線到 Azure**。整個工作流程分為兩個階段：

1. **離線收集（Offline Collection）**：在本地環境安裝 Collector，掃描 VMware 環境或實體/虛擬伺服器，所有資料儲存在本機。
2. **線上上傳（Online Upload）**：將收集到的 ZIP 檔案帶到有網路連線的環境，匯入到 Azure Migrate 專案。

這種設計省去了在隔離網路中建立複雜網路通道或申請雲端存取權限的需求，對政府機關與高度監管產業特別實用。

## 可以離線安裝嗎？

根據官方文件，Collector 的安裝方式如下：

1. 從 Azure Migrate 專案中選擇 **Discover → using Collector** 並下載安裝程式，或直接透過連結 [https://aka.ms/Migrate/DownloadCollector](https://aka.ms/Migrate/DownloadCollector) 下載。
2. 將安裝程式 ZIP 檔解壓縮到目標伺服器上的資料夾。
3. 以管理員權限啟動 PowerShell，切換到解壓縮資料夾後執行：

```powershell
.\AzureMigratecollector.ps1
```

安裝程式本身是一個 PowerShell 腳本，會安裝所需的 Agent 與 Web 應用程式、啟用 Windows 功能（如 Windows Activation Service、Web-Server、Web-Mgmt-Service）。**只要事先將安裝程式 ZIP 檔下載好，就可以在離線環境中進行安裝與執行**——這正是 Collector 與傳統 Appliance 最大的差異。

> **重點：** 收集的資料會儲存在 `C:\ProgramData\Microsoft Azure\OfflineData`，不需要網路連線即可完成整個探索流程。

## 支援的探索範圍

### VMware 環境

| 項目 | 說明 |
|------|------|
| **支援的 vCenter 版本** | 8.0、7.0、6.7、6.5、6.0、5.5 |
| **網路需求（vCenter）** | Collector 需連線到 vCenter，TCP 443 |
| **網路需求（ESXi）** | Collector 需連線到所有 ESXi 主機，TCP 443 |
| **Guest 資料收集** | 透過 VMware Tools 經由 ESXi Host 進行，不需要直接連線到 Guest VM |
| **收集內容** | 伺服器組態、效能指標、已安裝軟體、SQL/PostgreSQL 資料庫實例、Web 應用程式（.NET on IIS、Java on Tomcat） |

### 實體伺服器 / 其他 Hypervisor

同一個 Collector 可以透過切換 Fabric Type 來探索實體伺服器，不限於 VMware。通訊埠：

- **Windows 伺服器**：WinRM HTTPS (5986) 或 HTTP (5985)
- **Linux 伺服器**：SSH (TCP 22)

## 系統需求

| 項目 | 最低需求 |
|------|----------|
| **作業系統** | Windows Server 2019 / 2022 / 2025 |
| **CPU** | 8 vCPU |
| **記憶體** | 16 GB RAM |
| **磁碟** | 約 80 GB |

## 資料收集與匯出流程

### 1. 設定認證

- **vCenter 帳號**：提供唯讀加 Guest Operations 權限的帳號
- **Windows Guest**：網域帳號或本機管理員帳號
- **Linux Guest**：Root 帳號
- **SQL 認證**：Windows 或 SQL Server 驗證

可以參考 [最低權限帳號建議](https://learn.microsoft.com/en-us/azure/migrate/best-practices-least-privileged-account?view=migrate) 設定客製化的最低權限帳號。

### 2. 啟動資料收集

選擇 **Start Data Collection** 後，依環境規模與認證數量，收集過程約需 2–4 小時。收集完成後可檢視摘要，下載 CSV 檔案查看每台伺服器的詳細錯誤訊息。

### 3. 增量收集

如果部分伺服器先前收集失敗，可啟用 **Incremental data** 選項，僅針對失敗的工作負載重新嘗試，不需要重跑整個探索流程。

### 4. 匯出 ZIP 檔

收集完成後選擇 **Export**，系統會在以下路徑產生 ZIP 檔：

```
C:\ProgramData\Microsoft Azure\OfflineData\Azure-Migrate-Discovery-YYYY-MM-DD-HH-MM-SS.zip
```

### 5. 上傳至 Azure Migrate

在有網路連線的環境中：

1. 建立新的 Azure Migrate 專案（Collector 匯入僅支援新建專案）
2. 選擇 **Start discovery using collector**
3. 瀏覽並選擇 ZIP 檔，進行匯入
4. 等待約 30 分鐘完成探索資料處理
5. 上傳成功後等待 45 分鐘，即可建立商業案例與評估報告

> **注意：** 同一個專案可以匯入多個不同 Hypervisor 類型（VMware、Physical）的 ZIP 檔。

## 目前已知限制

- 不支援伺服器相依性識別（Dependency Analysis）
- 不支援 MySQL 與 PostgreSQL 實例的深度探索
- 收集 SQL Readiness 資料時，Collector 需要網路連線到目標 SQL 實例（自訂 TCP 埠）

## 適用情境

| 情境 | 說明 |
|------|------|
| **Air-gapped / 隔離網路** | 無需 Azure 連線即可完成資產盤點 |
| **尚未核准雲端存取** | 先收集資料產生報告，再用報告爭取預算與核准 |
| **多環境盤點** | 單一 Collector 支援最多 10 個 vCenter，也支援實體伺服器 |
| **快速 PoC** | 不需複雜的 Appliance 部署，PowerShell 腳本安裝即可啟動 |

## 結語

Azure Migrate Collector 提供了一條輕量、離線優先的遷移評估路徑。對於過去因為網路限制而無法順利進行 Azure Migrate 評估的環境，現在可以用「先離線收集、後線上上傳」的兩階段模式完成整個流程。搭配先前介紹的 [Microsoft 主權雲離線能力](/2026/03/25/microsoft-sovereign-cloud-taiwan/)，Microsoft 在離線與主權情境下的工具鏈越來越完整。

---

**參考資料：**
- [Discover servers and workloads using Azure Migrate collector](https://learn.microsoft.com/en-us/azure/migrate/how-to-discover-using-collector?view=migrate)
- [Create a new Azure Migrate project](https://learn.microsoft.com/en-us/azure/migrate/quickstart-create-project?view=migrate)
- [Best practices for least privileged accounts](https://learn.microsoft.com/en-us/azure/migrate/best-practices-least-privileged-account?view=migrate)
