---
layout: default
title: 04：Token 配額與隔離驗證
parent: Azure AI 資安與治理實戰
grand_parent: Workshop
nav_order: 5
permalink: /workshop/azure-ai-security-governance/token-governance/
---

# 04：Token 配額與隔離驗證

**目標**：每個應用有獨立計數器，能分辨每分鐘限流與期間配額。約 45 分鐘；先確認[Gateway](../ai-gateway/)呼叫為 `200`。

## 先設計配額

TPM 是每分鐘 Token 數，quota 是某段期間允許的累計量。模型部署配額與 Gateway 限額不同：Gateway 不會增加模型容量。先查可用部署 TPM，再分配應用額度並保留餘裕。

| 應用代號 | TPM（範例） | 每日 quota（範例） | 負責角色 |
|----------|------------|-------------------|----------|
| app-a | 2,000 | 20,000 | 教學應用負責人 |
| app-b | 2,000 | 20,000 | 研究應用負責人 |

數值只是便於測試的示例，不是正式建議。估算成本時要分開考量輸入與輸出 Token 單價；每日 quota 也不是精確帳單上限。

## 操作一：Product 範圍的限額

1. 保留 `lab-teaching` 原有 policy。
2. 在 **Products → lab-teaching → Policies** 的 `inbound` 中，保留 `<base />`，加入以下元素。這是**片段**，不要取代整份 policy。
3. 確認 API 有繼承 Product policy；在其他範圍自訂過 policy 時，要檢查有效 policy，不能只看編輯畫面。

```xml
<llm-token-limit
    counter-key="@(context.Subscription.Id)"
    tokens-per-minute="2000"
    token-quota="20000"
    token-quota-period="Daily"
    estimate-prompt-tokens="true"
    remaining-quota-tokens-header-name="x-remaining-quota"
    remaining-tokens-header-name="x-remaining-tpm"
    tokens-consumed-header-name="x-tokens-consumed" />
```

此片段以 APIM subscription ID 分別計數，前提是 API 要求有效訂用。在同一 Product 再建立第二個 **Product 範圍**訂用 `lab-app-b`，不可共用 app-a 的金鑰。

`llm-token-limit` 不支援 Consumption tier；SKU、API schema 與 gateway 支援以[官方 policy 文件](https://learn.microsoft.com/en-us/azure/api-management/llm-token-limit-policy)為準。同一 `counter-key` 跨範圍會共享計數；在 v2 層級跨範圍重複使用時，TPM 必須一致。本課只在一個範圍設定。

## 操作二：有限次數的驗證

沿用 Gateway 的 `$gatewayUri` 與 `$body`，先確認沒有其他流量使用這兩個訂用。避免無限重試或高併發壓測。

```powershell
$keyA = Read-Host "輸入 lab-app-a 的金鑰" -MaskInput
$keyB = Read-Host "輸入 lab-app-b 的金鑰" -MaskInput
try {
  foreach ($i in 1..20) {
    $r = Invoke-WebRequest -Uri $gatewayUri -Method Post `
      -Headers @{ "Ocp-Apim-Subscription-Key" = $keyA } `
      -ContentType "application/json" -Body $body -SkipHttpErrorCheck
    "A: HTTP {0}; retry-after={1}" -f [int]$r.StatusCode, ($r.Headers["Retry-After"] -join ",")
    if ([int]$r.StatusCode -ne 200) { break }
  }
  $r = Invoke-WebRequest -Uri $gatewayUri -Method Post `
    -Headers @{ "Ocp-Apim-Subscription-Key" = $keyB } `
    -ContentType "application/json" -Body $body -SkipHttpErrorCheck
  "B: HTTP {0}" -f [int]$r.StatusCode
} finally {
  Remove-Variable keyA, keyB
}
```

短提示與有限請求**不保證**觸發門檻。若仍全為 `200`，先檢查訂用範圍與有效 policy，再暫時調低沙盒 TPM（例如 `100`）重新測試；勿直接放大呼叫量。改 policy 不代表既有計數器立即歸零，必要時等待期間重置。

| 測試 | 預期 | 排除誤判 |
|------|------|----------|
| A 超過 TPM | `429`，觀察 `Retry-After` | 後端也可能回 `429`，需用日誌 / trace 確認來源 |
| A 被限流時呼叫 B | B 仍 `200` | 若部署總容量耗盡，B 也可能失敗，不表示訂用計數共用 |
| A 累計 quota 耗盡 | `403` | 後端授權錯誤也可能是 `403`，檢查 policy 來源 |

為單獨測試 quota，可暫時把 TPM 設為部署允許的較高值，把 quota 降為小額並設 `Hourly`，再做相同的有限測試。記錄設定、消耗與失敗原因；測完還原。不要同時把兩個門檻降得很低而無法判斷觸發來源。

## 限制與營運

Daily 期間依 **UTC 日界線**重置，在台北時間為每日 08:00。Token 估算、串流與併發會影響計數，可能暫時超出限額；不同 gateway 的計數也不是全域精確帳單總額。

客戶端收到 `429` 應遵守 `Retry-After`，並設重試次數與時間上限。配額變更要有申請、負責人與複核日期；臨時例外的到期應另建追蹤流程，不假設 Product 自動到期。

## 驗收與回復

留存匿名化的 A 限流、B 可用與 quota 耗盡三種結果；若未觸發或來源未確認，標記為未完成而不是通過。還原原 policy、移除不再需要的測試訂用；保留低成本測試流量供[監控 Lab](../monitoring/)使用。
