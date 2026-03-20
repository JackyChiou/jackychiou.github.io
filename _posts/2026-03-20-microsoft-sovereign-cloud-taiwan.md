---
layout: post
title: "Microsoft 主權雲全面升級：離線 AI、治理與生產力工具如何回應台灣資料主權需求"
date: 2026-03-20
categories: [Azure, Sovereign Cloud]
---

2026 年 2 月，Microsoft 宣布 Sovereign Cloud 重大更新——Azure Local、Microsoft 365 Local 與 Foundry Local 三大元件全面支援**完全離線（Air-gapped）**部署，讓政府與高度監管產業在斷網環境下也能運行大型 AI 模型、協作工具及關鍵基礎設施。這篇文章將整理這次更新的核心內容，並從台灣法規與政策面切入，探討對本地公部門與企業的影響。

## 三大核心更新

### 1. Azure Local Disconnected Operations（已正式上線）

Azure Local 現在支援在**完全無雲端連線**的環境下運行工作負載，同時保有 Azure 一致的治理與原則控制。

- 管理平面（Control Plane）以地端 appliance VM 形式運行，不依賴外部雲端
- 政策執行、工作負載部署與管理全部在客戶場域內完成
- 從小規模部署到大型 AI 驅動工作負載皆可彈性擴展
- 維持與公有雲 Azure 一致的操作模型

**對台灣的意義：** 對於國防、關鍵基礎設施或涉及機敏資料的機關而言，這意味著不必為了安全而犧牲現代化的雲管理能力。

### 2. Microsoft 365 Local Disconnected（已正式上線）

核心生產力工作負載——Exchange Server、SharePoint Server、Skype for Business Server——可完全在客戶的主權私有雲內運行。

- 支援保證至少延續到 **2035 年**
- 團隊可在受控邊界內安全地通訊、分享資訊與協作
- 資料韌性、存取權限與合規性完全由客戶自主控制

**對台灣的意義：** 公部門內部的電子郵件、文件共享與通訊不再需要經過境外伺服器，直接滿足資料在地化需求。

### 3. Foundry Local：地端大型 AI 模型

Foundry Local 將多模態大型 AI 模型帶入離線環境：

- 支援在客戶自有硬體上部署大規模多模態模型
- 透過 NVIDIA 等合作夥伴的 GPU 基礎設施進行本地推論
- API 完全在客戶控制的資料邊界內運作
- Microsoft 提供部署、更新與運營健康的全面支援

**對台灣的意義：** 政府或金融機構可以在不將資料送出境外的前提下，運用最先進的 AI 能力處理機敏文件分析、風險評估等任務。

## Sovereign Private Cloud：統一架構

Microsoft 將上述三大元件整合為 **Sovereign Private Cloud** 架構，提供兩種運作模式：

| 模式 | 控制平面 | 適用場景 |
|------|---------|---------|
| **Connected** | 雲端區域 + 地端連線 | 一般政府機關、受規範企業 |
| **Disconnected** | 地端 Appliance VM | 國防、情報、高機敏環境 |

兩種模式共享一致的管理體驗與政策框架，組織可依個別工作負載選擇適當的控制層級。

## 台灣法規與政策面的對應

### 《資通安全管理法》修正案（2025 年 9 月公布）

台灣《資通安全管理法》於 2025 年完成首次重大修正，重點包括：

- **禁用危害國安產品：** 公務機關不得使用危害國家資通安全的產品（已提升至法律位階）
- **強化稽核聯防：** 資通安全署可定期或不定期稽核納管機關的資通安全維護計畫
- **擴大納管範圍：** 涵蓋關鍵基礎設施提供者、公營事業及特定法人
- **提高罰則：** 未依法通報資安事件罰鍰上限提高至 NT$1,000 萬

Microsoft Sovereign Cloud 的完全離線能力，正好對應了高安全等級機關在合規上的需要——既能使用現代化工具，又能確保資料與管理流程不離開受控環境。

### 衛福部「主權雲八大指導方針」（2026 年 1 月）

衛福部於 2026 年 1 月發布的主權雲採購指導方針，進一步明確了跨國雲端業者在台灣政府採購中應遵循的核心原則：

1. **數據在地（Data Residency）**：所有原始資料與備份必須儲存於台灣境內
2. **人員主權（Personnel Sovereignty）**：高權限管理員須具中華民國國籍並通過背景審核
3. **法治優先（Legal Primacy）**：完全遵循《資通安全管理法》與《個資法》
4. **數位韌性（Resilience）**：雙活架構，確保零中斷運作
5. **透明稽核（Transparency）**：具合約約束力的持續審計權

| 台灣要求 | Microsoft Sovereign Cloud 對應能力 |
|----------|----------------------------------|
| 數據在地 | Azure Local Disconnected 全地端部署 |
| 人員主權 | 客戶自主管理，不依賴境外人員 |
| 法治優先 | 工作負載與資料不經過境外法域 |
| 數位韌性 | 離線模式下仍可持續運作 |
| 透明稽核 | Azure 一致的治理與政策控制 |

## 實務建議

如果你的組織正在評估主權雲方案，以下幾點值得考量：

1. **盤點工作負載的機敏等級：** 並非所有系統都需要 air-gapped，依敏感程度選擇 Connected 或 Disconnected 模式
2. **對照法規要求：** 確認組織屬於《資通安全管理法》哪一級納管對象，相應的安全維護計畫是否需要地端部署
3. **評估 AI 需求：** 如果有機敏資料的 AI 應用需求（如醫療影像分析、司法文件處理），Foundry Local 提供了不必將資料送出的選項
4. **規劃硬體投資：** 離線部署需要自有硬體與 GPU，需評估 TCO 與維運能力

## 結語

Microsoft Sovereign Cloud 的這次更新，不只是技術能力的擴展，更是對全球數位主權趨勢的直接回應。對台灣而言，在《資通安全管理法》修正案與衛福部主權雲指導方針的雙重驅動下，公部門與關鍵基礎設施業者正面臨「安全」與「現代化」兼得的迫切需求。完全離線的主權私有雲架構，為這個需求提供了一個具體可行的答案。

---

**參考資料：**
- [Microsoft Sovereign Cloud adds governance, productivity and support for large AI models securely running even when completely disconnected - The Official Microsoft Blog](https://blogs.microsoft.com/blog/2026/02/24/microsoft-sovereign-cloud-adds-governance-productivity-and-support-for-large-ai-models-securely-running-even-when-completely-disconnected/)
- [Microsoft strengthens sovereign cloud capabilities with new services - Azure Blog](https://azure.microsoft.com/en-us/blog/microsoft-strengthens-sovereign-cloud-capabilities-with-new-services/)
- [資通安全管理法 - 全國法規資料庫](https://law.moj.gov.tw/LawClass/LawAll.aspx?pcode=A0030297)
- [行政院通過「資通安全管理法」修正草案](https://www.ey.gov.tw/Page/9277F759E41CCD91/9c1b42cb-ebb2-4647-b2d6-2431ff1dfcb1)
- [衛福部「主權雲應用發表會」，公布政府雲端採購新基準](https://geneonline.news/mohw-sovereign-cloud-2026)
