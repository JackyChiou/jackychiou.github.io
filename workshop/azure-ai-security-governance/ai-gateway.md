---
layout: default
title: 03：建立 AI Gateway
parent: Azure AI 資安與治理實戰
grand_parent: Workshop
nav_order: 4
permalink: /workshop/azure-ai-security-governance/ai-gateway/
---

# 03：建立 AI Gateway

**目標**：用 APIM 作為應用入口，後端使用 Managed Identity，不發放模型金鑰。約 45 分鐘，不含佈建等待。

## 前置條件與架構取捨

完成[無金鑰 Lab](../keyless-policy/)，保留已停用本機驗證的模型部署。準備支援 AI policy 的 APIM（本課以 Basic v2 為例），以及建立身分與角色指派所需的權限。

這是沙盒功能練習，不是完整網路隔離架構。若模型端只有私人端點，APIM 必須具備相容的網路方案、DNS 與後端連線能力；不可直接套用本頁的公網練習假設。

## 操作一：APIM 與模型後端

1. 在專用資源群組建立 APIM，先確認區域、SKU 與費用。發行者與通知資料填入自己的受控資訊，不要提交到教材。
2. 開啟 APIM 的 **Identity**，確認系統指派 Managed Identity 已啟用。
3. 在模型資源 IAM，將 `Cognitive Services OpenAI User` 指派給 **APIM 的 Managed Identity**，不是學員的日常帳號。
4. APIM → **APIs → Add API**，依[官方匯入流程](https://learn.microsoft.com/en-us/azure/api-management/azure-ai-foundry-api)選取 Foundry / Azure OpenAI 資源。
5. 本課選 **Azure OpenAI** 相容模式，Base path 設為 `ai`。檢查產生的 operations 是否包含 `/openai/deployments/{deployment}/chat/completions`。
6. 檢視 backend 與有效 policy，確認後端為自己的模型端點，採 Managed Identity 驗證，且有後端轉送設定。若精靈已自動授權，不重複建立指派；若未設定，依官方驗證文件補齊。

編輯 policy 前保留原設定；不要以一份空白 policy 蓋掉匯入所需的 `set-backend-service` 或驗證設定。

## 操作二：建立每應用訂用

1. APIM → **Products** 建立 `lab-teaching`，設定 Published、**Requires subscription**，加入剛匯入的 API。
2. 在 API 的設定中確認也要求 subscription，檢查沒有其他不需要訂用的 Product 暴露此 API。
3. APIM → **Subscriptions** 建立 `lab-app-a`，Scope 明確選 **Product → lab-teaching**。
4. 不使用 All APIs、單一 API 範圍或內建 all-access 金鑰。這些金鑰不能用來驗證 Product 範圍的限額。

這裡的 APIM subscription 是 API 存取憑證，**不是 Azure 訂用帳戶**。每個應用建立獨立訂用才能隔離計數；多人共用一把金鑰無法代表每位使用者身分。進階環境可加上 Entra JWT 驗證，但仍需獨立規劃後端 MI。

## 操作三：經 Gateway 呼叫

在 PowerShell 7 設定以下值。Base URL 取自 APIM Overview 的實際 gateway URL，再加上 API Base path；部署名稱沿用模型端。

```powershell
$gatewayBase = "https://<apim-name>.azure-api.net/ai"
$deployment = "<chat-completions-deployment-name>"
$gatewayKey = Read-Host "輸入 lab-app-a 的 Product 訂用金鑰" -MaskInput
$gatewayUri = "$gatewayBase/openai/deployments/$deployment/chat/completions?api-version=2024-10-21"
$body = @{
  messages = @(@{ role = "user"; content = "ping" })
  max_tokens = 16
} | ConvertTo-Json -Depth 5

try {
  $result = Invoke-WebRequest -Uri $gatewayUri -Method Post `
    -Headers @{ "Ocp-Apim-Subscription-Key" = $gatewayKey } `
    -ContentType "application/json" -Body $body -SkipHttpErrorCheck
  "HTTP {0}" -f [int]$result.StatusCode
} finally {
  Remove-Variable gatewayKey
}
```

預期 `200`。訂用標頭名稱若已修改，需以 API 設定為準。不要把模型 `api-key` 一起送給 Gateway。

## 驗收與防繞過

| 測試 | 預期與判讀 |
|------|------------|
| 有效 Product 訂用呼叫 Gateway | `200` |
| 不帶訂用金鑰呼叫相同 Gateway 路徑 | `401`，確認原因是缺少 subscription key |
| 舊模型金鑰直接呼叫模型端 | 本機驗證停用並傳播後遭拒 |
| 沒有資料平面角色的身分直連 | 遭拒；確認不是部署或網路錯誤造成 |

完成 Gateway 測試後，由管理者移除上一個 Lab 暫時給測試使用者的**直接推論角色**，再測一次該使用者不能直連而 Gateway 仍可用。不要移除共用或原有的角色指派。

若 Gateway 為 `401` / `403`，檢查是入口訂用驗證還是後端 MI 失敗；若為 `404`，檢查 Base path、operation 與 deployment。等待角色傳播，不透過啟用模型金鑰繞過問題。

## 回復與清理

保留 APIM 供後續 Lab 使用。課後移除本次建立的訂用、Product 與 APIM MI 角色指派，再刪除專用 APIM。刪除 API 或停用 Gateway **不等於刪除 APIM，也不會停止其執行個體費用**。

[下一步：Token 配額與隔離驗證](../token-governance/)
