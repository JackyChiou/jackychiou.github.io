---
layout: default
title: 官方文件與名詞
parent: Azure AI 資安與治理實戰
grand_parent: Workshop
nav_order: 11
permalink: /workshop/azure-ai-security-governance/references/
---

# 官方文件與名詞

以下皆為公開的 Microsoft 官方文件或官方範例入口。功能、授權與介面可能更新；請在開始實作時確認當前支援，不把範例值視為服務承諾。

## 驗證、角色與護欄

| 文件 | 用途 |
|------|------|
| [停用 AI Services 本機驗證](https://learn.microsoft.com/en-us/azure/ai-services/disable-local-auth) | 設定方式、傳播與金鑰停用驗證 |
| [AI 與機器學習內建角色](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles/ai-machine-learning) | 核對實際角色 Actions / DataActions，不只看名稱 |
| [AI Services 內建 Policy](https://learn.microsoft.com/en-us/azure/ai-services/policy-reference) | 指派前確認定義、效果與適用資源 |
| [Azure RBAC 最佳實務](https://learn.microsoft.com/en-us/azure/role-based-access-control/best-practices) | 最小權限與指派範圍 |
| [模型服務雙 Key 輪替](https://learn.microsoft.com/en-us/azure/ai-services/rotate-keys) | 先更新備用 Key，再切換及更新原 Key |
| [模型 Key 重新產生 CLI](https://learn.microsoft.com/en-us/cli/azure/cognitiveservices/account/keys#az-cognitiveservices-account-keys-regenerate) | `Key1` / `Key2` 與輸出控制 |
| [Foundry 模型端點與部署](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/endpoints) | 部署別名、資源推論端點與 API 相容性 |

## Gateway、Token 與觀測

| 文件 | 用途 |
|------|------|
| [APIM AI Gateway 能力](https://learn.microsoft.com/en-us/azure/api-management/genai-gateway-capabilities) | 功能與支援總覽 |
| [匯入 Foundry API](https://learn.microsoft.com/en-us/azure/api-management/azure-ai-foundry-api) | 匯入模式、路徑與後端驗證 |
| [AI API 驗證與授權](https://learn.microsoft.com/en-us/azure/api-management/api-management-authenticate-authorize-ai-apis) | Managed Identity 與 Gateway 驗證 |
| [APIM subscriptions](https://learn.microsoft.com/en-us/azure/api-management/api-management-subscriptions) | Product 範圍與 all-access 的差異 |
| [APIM Primary 重新產生](https://learn.microsoft.com/en-us/rest/api/apimanagement/subscription/regenerate-primary-key?view=rest-apimanagement-2024-05-01) | 指定訂用的日常輪替與外洩處置 |
| [APIM Secondary 重新產生](https://learn.microsoft.com/en-us/rest/api/apimanagement/subscription/regenerate-secondary-key?view=rest-apimanagement-2024-05-01) | 備用 Key 輪替 |
| [llm-token-limit](https://learn.microsoft.com/en-us/azure/api-management/llm-token-limit-policy) | 429 / 403、計數器、期間與估算限制 |
| [LLM 日誌設定](https://learn.microsoft.com/en-us/azure/api-management/api-management-howto-llm-logs) | 診斷、內容記錄與隱私 |
| [Gateway 日誌 schema](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/apimanagementgatewaylogs) | HTTP 狀態與訂用欄位 |
| [LLM 日誌 schema](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/apimanagementgatewayllmlog) | Token 與關聯欄位 |
| [APIM v2 層級](https://learn.microsoft.com/en-us/azure/api-management/v2-service-tiers-overview) | SKU 與網路能力 |
| [APIM 區域可用性](https://learn.microsoft.com/en-us/azure/api-management/api-management-region-availability) | 建立資源前查核 |
| [Action Groups](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/action-groups) | 通知與測試 |
| [Azure Budgets](https://learn.microsoft.com/en-us/azure/cost-management-billing/costs/tutorial-acm-create-budgets) | 預算通知與資料延遲 |
| [官方 AI Gateway 範例](https://github.com/Azure-Samples/AI-Gateway) | 課後延伸；部署前先審閱授權、費用與環境參數 |

## Entra 管理

| 文件 | 用途 |
|------|------|
| [Entra 角色最佳實務](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/best-practices) | 特權最小化與管理帳號分離 |
| [緊急存取帳號](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/security-emergency-access) | 雲端帳號、抗網路釣魚驗證、監控與定期驗證 |
| [Security defaults](https://learn.microsoft.com/en-us/entra/fundamentals/security-defaults) | Free 基本保護與租戶影響 |
| [依任務選擇最小權限](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/delegate-by-task) | 管理角色選擇 |
| [Entra 授權](https://learn.microsoft.com/en-us/entra/fundamentals/licensing) | Free / P1 / P2 與功能規劃 |
| [日誌保留期間](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/reference-reports-data-retention) | 檢視與保留排程 |
| [Global Administrator 提升存取](https://learn.microsoft.com/en-us/azure/role-based-access-control/elevate-access-global-admin) | 租戶層級事件與移除提升權限 |

## 截圖來源與授權

[Gateway 頁](../ai-gateway/#實際-portal-截圖)使用 Microsoft 與文件貢獻者提供的兩張真實 Portal 截圖，取自公開的 MicrosoftDocs/azure-docs 儲存庫，依其 [Legal Notices](https://github.com/MicrosoftDocs/azure-docs/blob/main/ThirdPartyNotices.md) 與 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 使用。原圖連結、修改說明與無擔保聲明均置於該頁圖說。黑色區塊為追加的識別資訊遮蔽；輸出 PNG 已移除嵌入中繼資料。截圖不代表本課 Demo 已在 Azure 執行，也不表示 Microsoft 為本課背書。

## 常用名詞

| 名詞 | 本課用法 |
|------|----------|
| Managed Identity（MI） | Azure 代管的工作負載身分，不由應用保存模型金鑰 |
| RBAC | 以角色及範圍限制權限；Azure 與 Entra 角色不能混為一談 |
| TPM | 每分鐘 Token 額度，與期間累計 quota 分別管理 |
| APIM subscription | API 存取訂用；不同於 Azure 訂用帳戶 |
| Product | APIM 的 API 發佈與 policy 範圍；本課每個應用使用 Product 範圍訂用 |
| disableLocalAuth | 停用資源本機金鑰驗證的屬性，不等於撤銷 Entra 權限或關閉公網 |
| Break-glass | 平常不用、緊急時經核准啟用的管理存取流程與帳號 |

[回到課程首頁](../)
