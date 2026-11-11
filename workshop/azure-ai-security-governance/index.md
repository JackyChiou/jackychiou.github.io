---
layout: default
title: Azure AI 資安與治理實戰
parent: Workshop
has_children: true
nav_order: 2
permalink: /workshop/azure-ai-security-governance/
description: 以無金鑰存取、AI Gateway、Token 配額與 Entra 最小權限建立 Azure AI 治理護欄。
---

# Azure AI 資安與治理實戰

{: .highlight }
**學員公開版** ｜ **對象**：Azure 管理者、平台工程師與資安協作者 ｜ **情境**：校園與多團隊共用 AI 平台

當教學、研究與行政應用共用 AI 服務，如何避免共用金鑰外洩、單一專案耗盡額度，以及管理權限長期累積？本課以 Microsoft Foundry（部分介面仍使用 Azure AI Foundry 名稱）與 Azure API Management（APIM）的 AI Gateway 為主軸，將治理要求轉成可驗證的操作。

## 學習目標

完成後，你將能盤點 AI 資源與高權限、以 Microsoft Entra ID / Managed Identity 取代模型金鑰、建立 Gateway 統一入口、為不同應用設定 Token 限額，並用日誌與權限複核形成營運閉環。

| 風險 | 控制措施 | 學員驗收成果 |
|------|----------|--------------|
| 模型金鑰外洩 | 無金鑰驗證、停用本機驗證、Azure Policy | 授權身分呼叫成功，舊金鑰不能呼叫 |
| Token 異常消耗 | Gateway 每應用 TPM / quota、監控與預算 | 區分限流與配額耗盡，確認應用間隔離 |
| 高權限累積 | 最小權限、緊急存取、MFA、定期複核 | 角色責任表、測試租戶演練與回復計畫 |

## 學習路線

建議先以約 60 分鐘閱讀概念與操作流程，再依下表分次實作。各 Lab 時間為估計，不含資源佈建、權限傳播與疑難排解。

| 順序 | 內容 | 預估實作時間 |
|------|------|--------------|
| 00 | [前置準備與安全守則](./prerequisites/) | 30 分鐘 |
| 01 | [現況盤點](./inventory/) | 30 分鐘 |
| 02 | [無金鑰存取與 Policy](./keyless-policy/) | 45 分鐘 |
| 03 | [建立 AI Gateway](./ai-gateway/) | 45 分鐘 |
| 04 | [Token 配額與隔離驗證](./token-governance/) | 45 分鐘 |
| 05 | [監控、警示與成本](./monitoring/) | 45 分鐘 |
| 06 | [Entra 高權限治理](./entra-governance/) | 60 分鐘，限測試租戶 |
| 收尾 | [30 天行動計畫與檢核](./action-plan/) | 依導入規模安排 |
| 參考 | [官方文件與名詞](./references/) | 隨查隨用 |

## 參考架構與邊界

```text
教學 / 研究應用
    |
    | 每應用 Gateway 憑證（不發放模型金鑰）
    v
API Management：驗證 -> 每應用 Token 限額 -> 後端轉送
    |                                      |
    | Managed Identity                     +-> Log Analytics / 警示
    v
Foundry 模型部署：停用本機金鑰 + 最小權限 + 適當網路限制

Azure Policy：稽核與部署護欄
Microsoft Entra ID：身分與管理角色治理
```

Gateway 不會自動阻止直連。必須一起處理模型端金鑰、資料平面角色與網路存取。停用金鑰也不會撤銷已授權的 Entra 身分。

本課採 **APIM 匯入模型 API** 路線，讓每個應用都有明確的 Product 訂用與計數器；Foundry 入口內建 Gateway 可作為延伸閱讀，不作為必要前提。不同 API 相容模式的端點、驗證 audience 與 policy 支援可能不同，不要混用。

## 使用限制

{: .warning }
只在自有且獲授權的沙盒執行。APIM、模型推論與日誌可能持續計費；預算通知不是硬性停機機制。Entra 設定可能影響整個租戶，沒有測試租戶就只做盤點與桌上演練。

教材使用虛構的應用名稱、範例數值與佔位符，不含真實環境識別資料、未遮蔽截圖或講師操作稿。不要將實作產生的身分名單、金鑰或輸出上傳到公開網站。

服務名稱、授權、區域、模型可用性與預覽功能會變動，操作時請對照[官方文件](./references/)。本教材不是正式合規認證或生產環境部署範本。
