---
layout: default
title: 02：無金鑰存取與 Policy
parent: Azure AI 資安與治理實戰
grand_parent: Workshop
nav_order: 3
permalink: /workshop/azure-ai-security-governance/keyless-policy/
---

# 02：無金鑰存取與 Policy

**目標**：先證明 Entra 推論可用，再停用模型金鑰並建立部署護欄。約 45 分鐘，不含權限與設定傳播。

{: .warning }
停用本機驗證會中斷仍使用金鑰的應用。只操作專用沙盒資源；先記錄原設定與回復方式，不在正式服務直接切換。

## 概念與前提

人員使用 Entra 身分，Azure 上的服務優先使用 Managed Identity。模型端金鑰與 APIM 訂用金鑰是不同憑證，不能互相代用。Contributor 等管理平面角色也不保證有資料平面推論權限。

完成[盤點](../inventory/)，準備支援 Chat Completions 的模型部署與 PowerShell 7。由具指派權限的管理者在**模型資源**的 IAM，暫時為測試使用者指派 `Cognitive Services OpenAI User`。本課測試 Azure OpenAI 相容端點；其他 Foundry API 的角色與權杖 audience 請依其文件確認。

## 操作一：不帶金鑰的呼叫

沿用[前置頁](../prerequisites/)的變數，以獲授權的測試使用者執行 `az login`，確認在正確租用戶與訂用帳戶。

```powershell
$token = az account get-access-token `
  --resource https://cognitiveservices.azure.com `
  --query accessToken -o tsv
if ($LASTEXITCODE -ne 0 -or [string]::IsNullOrWhiteSpace($token)) {
  throw "無法取得存取權杖，請確認登入身分。"
}

$uri = "$modelEndpoint/openai/deployments/$deployment/chat/completions?api-version=2024-10-21"
$body = @{
  messages = @(@{ role = "user"; content = "ping" })
  max_tokens = 16
} | ConvertTo-Json -Depth 5

try {
  $response = Invoke-WebRequest -Uri $uri -Method Post `
    -Headers @{ Authorization = "Bearer $token" } `
    -ContentType "application/json" -Body $body -SkipHttpErrorCheck
  "HTTP {0}" -f [int]$response.StatusCode
} finally {
  Remove-Variable token
}
```

預期 `200`。`401` 先檢查權杖 audience、登入租用戶與端點；`403` 再檢查角色範圍、網路及傳播狀態。`404` 通常要核對部署名稱或 API 路徑。失敗時不要直接擴大為 Owner。

## 操作二：停用本機金鑰

1. 在 AI 資源 → **JSON View** 記錄原本的 `disableLocalAuth`。
2. 依[官方停用方式](https://learn.microsoft.com/en-us/azure/ai-services/disable-local-auth)設定該屬性為 `true`。入口網站的驗證設定位置會隨資源類型不同；找不到時，請管理者依官方方法操作。
3. 確認資源屬性為 `true`，重新執行上方 Entra 測試，預期仍為 `200`。
4. 若有**原本已啟用金鑰的專用對照資源**，在切換前後各做一次下方測試；沒有就記為未執行，不為了演示而重新開啟正式資源金鑰。

```powershell
$key = Read-Host "輸入沙盒模型金鑰（不會顯示）" -MaskInput
try {
  $response = Invoke-WebRequest -Uri $uri -Method Post `
    -Headers @{ "api-key" = $key } `
    -ContentType "application/json" -Body $body -SkipHttpErrorCheck
  "HTTP {0}" -f [int]$response.StatusCode
} finally {
  Remove-Variable key
}
```

停用前應先證明舊金鑰可呼叫，停用並傳播後預期 `401`。只看到失敗不代表控制有效：網路封鎖、無效金鑰與路徑錯誤也會失敗。設定傳播可能延遲，記錄觀測時間；不要宣稱設定完成就立即阻擋。

## 操作三：Azure Policy 先 Audit 再 Deny

1. Portal → **Policy → Definitions**，搜尋停用 AI Services 本機驗證的內建定義，讀取其適用資源、參數與支援效果。
2. 在**專用資源群組**指派 `Audit`，檢視 Compliance。若查不到相符定義，停止此部分並記下資源種類，不隨意套用其他 Policy。
3. 檢查非合規項目與應用依賴；確認無金鑰測試通過後，才把這個沙盒指派改為 `Deny`。
4. 在相同範圍嘗試建立一個啟用本機驗證的測試資源（不必部署付費模型），預期 `RequestDisallowedByPolicy`。若成功，確認指派範圍、排除項目與傳播，再清理該測試資源。

`Deny` 是對不符合條件的建立或更新操作設護欄，不會自動修復既有資源，也不負責撤銷 RBAC。不要把非合規狀態誤認為存取已遭封鎖。

## 驗收與回復

記錄四項結果：Entra 成功、停用設定為真、舊金鑰遭拒（或註明未做對照）、Policy 拒絕非合規部署。以另一個沒有資料平面角色的測試身分執行呼叫，應遭拒；確認它沒有繼承或群組授權。

移除本次建立的沙盒 Policy 指派與測試資源，保留模型供下一個 Lab 使用。若因核准的沙盒回復需要重啟本機驗證，先移除會阻擋此變更的測試 Deny，還原原設定並處理金鑰輪替；正式導入不應以長期重啟金鑰當作解法。

[下一步：建立 AI Gateway](../ai-gateway/)
