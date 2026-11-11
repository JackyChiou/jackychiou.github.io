---
layout: default
title: 05：監控、警示與成本
parent: Azure AI 資安與治理實戰
grand_parent: Workshop
nav_order: 6
permalink: /workshop/azure-ai-security-governance/monitoring/
---

# 05：監控、警示與成本

**目標**：查到應用用量與限額事件，讓通知真正送達。約 45 分鐘，不含資料匯入等待；先完成[Token Lab](../token-governance/)。

## 操作一：啟用必要日誌，不記錄內容

1. 準備專用 Log Analytics workspace，先設定必要的存取範圍與保留期間。
2. APIM → **Monitoring → Diagnostic settings**，啟用 `GatewayLogs` 與 `GatewayLlmLogs`（介面可能顯示對應說明名稱），傳送至 workspace，選 **Resource specific**。
3. 依[LLM 日誌設定文件](https://learn.microsoft.com/en-us/azure/api-management/api-management-howto-llm-logs)檢查 API 的診斷設定，保持 prompt / completion 訊息記錄關閉；也不要記錄憑證標頭或 request / response body。
4. 送出幾次無敏感內容的測試呼叫，等待匯入。不要為了取得 Token 統計而開啟提示內容記錄。

| 資料表 | 主要用途 |
|--------|----------|
| `ApiManagementGatewayLogs` | 訂用、API、HTTP 狀態、耗時與關聯 ID |
| `ApiManagementGatewayLlmLog` | 模型、Token 與關聯 ID；不能假設有訂用欄位 |

## 操作二：用量與限額查詢

在 workspace → **Logs** 查詢。先確認實際 schema 與資料範圍；空白或資料表不存在表示要查設定、支援與匯入狀況，不代表「沒有異常」。

以下將同一關聯 ID 先彙整，再關聯訂用，避免多筆日誌造成重複加總。這是營運觀測值，不是計費帳務。

```kusto
let usage = ApiManagementGatewayLlmLog
    | where TimeGenerated > ago(1h)
    | where isnotempty(CorrelationId)
    | summarize Tokens = max(TotalTokens) by CorrelationId;
let gateway = ApiManagementGatewayLogs
    | where TimeGenerated > ago(1h)
    | where isnotempty(CorrelationId)
    | summarize ApimSubscriptionId = take_any(ApimSubscriptionId) by CorrelationId;
usage
| join kind=inner gateway on CorrelationId
| summarize TotalTokens = sum(Tokens), Requests = count() by ApimSubscriptionId
```

再查看 `429` 與 `403` 的分布：

```kusto
ApiManagementGatewayLogs
| where TimeGenerated > ago(1h)
| where ResponseCode in (429, 403)
| summarize Events = count()
    by bin(TimeGenerated, 5m), ApimSubscriptionId, ResponseCode
| order by TimeGenerated desc
```

這只呈現狀態碼，不足以判定根因；對照 API policy、後端狀態與安全處理的 trace，區分 quota、身分驗證與模型端節流。不要公開原始 trace。

## 操作三：建立並測試通知

1. 在 Azure Monitor 建立專用 **Action Group**，使用獲准的通知對象並執行 Test。
2. 先在 Logs 驗證下方查詢。建立 log search alert，以 **15 分鐘視窗、5 分鐘評估**為練習值，結果資料列數大於 0 時通知。
3. 將門檻暫時調低到少量沙盒流量可觸發的程度；確認 alert instance 及通知收件結果，再還原門檻。

```kusto
ApiManagementGatewayLogs
| where TimeGenerated > ago(15m) and ResponseCode == 429
| summarize Events = count() by ApimSubscriptionId
| where Events > 5
```

門檻 `5` 是示範值。資料匯入、評估與通知有延遲，不能保證 15 分鐘內送達；記錄實測耗時，依需求調整與估算費用。

另在沙盒訂用帳戶建立 **Activity Log alert**，監看 `Microsoft.Authorization/roleAssignments/write`，由獲授權管理者新增一次暫時的 Reader 指派以驗證，再移除該指派。可同樣規劃刪除角色與 Policy 變更的監控。

Azure 訂用帳戶 Activity Log 不等同 Entra 稽核日誌；GA 的提升存取事件須依[身分治理頁](../entra-governance/)另外檢視。

## 操作四：預算與繞過檢視

在 Cost Management → **Budgets**，選專用資源群組或適當訂用帳戶範圍，建立每月預算及多門檻通知（例如 50% / 80% / 100%），確認收件人。

{: .warning }
預算只通知，不會自動停用資源或中止推論。成本資料與評估有延遲；即時控管應靠 Gateway 限額、應用重試上限與可執行的應變流程。

定期比對 Gateway 與模型端的請求 / Token 趨勢。差異可能來自直連，也可能是匯入延遲、估算或統計範圍不同，先調整成相同時間窗與模型範圍，不把差異直接當作外洩證據。

## 驗收與清理

應看到自己的測試訂用、限額事件與實際到達的通知。記錄資料延遲、規則範圍、測試時間與回復結果，不繳交帳號、IP 或收件人。

刪除本次測試警示、暫時 Reader 指派與不再使用的預算 / Action Group；避免刪到共用設定。完成所有 Lab 後依[清理指引](../prerequisites/#課後清理)刪除專用資源，並確認日誌保留政策。

[下一步：Entra 高權限治理](../entra-governance/)
