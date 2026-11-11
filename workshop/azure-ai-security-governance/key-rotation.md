---
layout: default
title: 演練：模型與 APIM 雙 Key 輪替
parent: Azure AI 資安與治理實戰
grand_parent: Workshop
nav_order: 8
permalink: /workshop/azure-ai-security-governance/key-rotation/
---

# 演練：模型與 APIM 雙 Key 輪替

**對象**：Azure Admin ｜ **時間**：約 30 分鐘，不含設定傳播 ｜ **狀態**：操作教材，未於 Azure 沙盒實跑

{: .warning }
重新產生金鑰不可還原舊值，會中斷仍使用舊值的用戶端。只使用**既有、已核准、無正式流量**的專用沙盒；沒有這個環境就做桌上演練，不建立付費資源、不輪替真實服務的 Key。本課未代你執行任何雲端變更。

## 兩類 Key 與止血決策

| 類型 | 影響範圍 | 日常輪替 | 已知／疑似外洩 |
|------|----------|----------|----------------|
| 模型服務 Key1 / Key2 | 同一模型資源下的金鑰存取，不是每位使用者身分 | 先更新未使用的備用 Key，遷移所有使用者，再更新原 Key | 不等待完整遷移才止血；停用本機驗證或立即重新產生洩漏的 Key，可能需要中斷；無法排除另一把也外洩時處理兩把 |
| APIM subscription Primary / Secondary | 該 API 訂用的兩把憑證，共用同一訂用計數與狀態 | 同樣先備用、後切換、再更新舊 Key | 先 Suspend 受影響訂用，使兩把 Key 都無法新進入；重新產生兩把、受控配發，核准後恢復 Active |

日常輪替中的「不中斷」有前提：備用 Key 未遭洩漏、所有用戶端可逐一切換，而且已觀察到成功；不是服務保證。**刪除 Git 中的 Key、刪除本機變數或 Unpublish Product，都不會撤銷已外流的 Key。**重新產生一把 Key 也不會撤銷另一把。

模型主線維持 `disableLocalAuth=true`，Gateway 後端用 MI。此處模型 Key 演練只為必要的舊式例外；最後仍應遷移成無金鑰。

## 前置與共用驗收函式

使用 PowerShell 7、Azure CLI 與[前置頁](../prerequisites/)的沙盒登入。需有目標資源的金鑰重新產生權限；APIM 需有目標訂用的管理權，不需 Entra P1 / P2。停止終端機錄影與 transcript，不列印 CLI 金鑰輸出，不使用 `--debug`，不要記錄憑證標頭。

將下列函式貼在同一終端機。只回傳 HTTP 狀態碼，不印出內容、Key 或完整端點。輸入只用無個資短提示；所有表格中的碼都是**預期**，不是實測。

```powershell
function Test-LabKey {
  param(
    [Parameter(Mandatory)][string]$Uri,
    [Parameter(Mandatory)][ValidateSet("api-key","Ocp-Apim-Subscription-Key")]
    [string]$HeaderName,
    [Parameter(Mandatory)][string]$Key
  )
  if ($Uri -match "<|>" -or -not $Uri.StartsWith("https://")) {
    throw "請先設定自己的 HTTPS 沙盒端點。"
  }
  $body = @{
    messages = @(@{ role = "user"; content = "ping" })
    max_tokens = 16
  } | ConvertTo-Json -Depth 5
  $r = Invoke-WebRequest -Uri $Uri -Method Post `
    -Headers @{ $HeaderName = $Key } -ContentType "application/json" `
    -Body $body -SkipHttpErrorCheck
  [int]$r.StatusCode
}
```

`401` 不能單獨證明輪替成功：必須先證明同一路徑的舊 Key 為 `200`，再證明重新產生後舊值遭拒、新值與未輪替的備用值可用。遇到 `429`、網路或部署錯誤先停止，不將其算成撤銷成功。

## A：模型 Key1 / Key2 的日常輪替

只使用原先已啟用金鑰的專用對照資源，**不能填主線 Foundry 資源**。若沒有對照資源，跳過 A，記為未執行。

1. 在 Portal 的 Keys and Endpoint 確認自己只管理該對照資源，所有測試用戶端目前使用 Key1。盤點不完整時，不可宣稱無中斷輪替。
2. 設定專用資源、驗證原 Key1，再重新產生**未使用的 Key2**。CLI 用 `--output none`，避免成功結果帶出兩把 Key。

```powershell
$legacyName = "<dedicated-key-enabled-comparison-resource>"
$legacyEndpoint = "https://<comparison-resource>.openai.azure.com"
$legacyDeployment = "<existing-comparison-deployment>"
$modelUri = "$legacyEndpoint/openai/deployments/$legacyDeployment/chat/completions?api-version=2024-10-21"
if ((Read-Host "輸入專用對照資源名稱，確認允許輪替") -cne $legacyName) {
  throw "目標未確認，停止。"
}
$localAuthDisabled = az cognitiveservices account show --subscription $subscriptionId `
  --resource-group $rg --name $legacyName --query properties.disableLocalAuth -o tsv
if ($LASTEXITCODE -ne 0 -or $localAuthDisabled -ne "false") {
  throw "未確認是已啟用金鑰的對照資源；不改動其驗證設定。"
}
$oldKey1 = Read-Host "輸入目前的模型 Key1" -MaskInput
if ((Test-LabKey -Uri $modelUri -HeaderName "api-key" -Key $oldKey1) -ne 200) {
  throw "原 Key1 未成功，停止輪替。"
}
az cognitiveservices account keys regenerate --subscription $subscriptionId `
  --resource-group $rg --name $legacyName --key-name Key2 --output none
if ($LASTEXITCODE -ne 0) { throw "Key2 重新產生失敗，停止。" }
```

3. 由受控的 Portal 取得新 Key2，以遮罩輸入測試。將所有測試用戶端／秘密儲存改用新 Key2，觀察其成功率；**確認全部切換**後才執行下一段重新產生 Key1。不要只是把舊 Key 複製到另一個欄位。

```powershell
$newKey2 = Read-Host "輸入新模型 Key2" -MaskInput
if ((Test-LabKey -Uri $modelUri -HeaderName "api-key" -Key $newKey2) -ne 200) {
  throw "新 Key2 未成功，保留原 Key1，停止。"
}
if ((Read-Host "所有測試用戶端已切換 Key2 並驗證成功？輸入 SWITCHED") -cne "SWITCHED") {
  throw "尚未完成用戶端切換，不更新 Key1。"
}
az cognitiveservices account keys regenerate --subscription $subscriptionId `
  --resource-group $rg --name $legacyName --key-name Key1 --output none
if ($LASTEXITCODE -ne 0) { throw "Key1 重新產生失敗，停止。" }
$newKey1 = Read-Host "輸入新模型 Key1" -MaskInput
try {
  "舊 Key1：HTTP {0}" -f (Test-LabKey -Uri $modelUri -HeaderName "api-key" -Key $oldKey1)
  "新 Key1：HTTP {0}" -f (Test-LabKey -Uri $modelUri -HeaderName "api-key" -Key $newKey1)
  "新 Key2：HTTP {0}" -f (Test-LabKey -Uri $modelUri -HeaderName "api-key" -Key $newKey2)
} finally {
  Remove-Variable oldKey1, newKey1, newKey2
}
```

預期依序 `401 / 200 / 200`，確認新 Key2 的用戶端在更新 Key1 期間仍可使用。未達預期時記為失敗，停止繼續輪替，查核實際目標、請求與設定傳播，不對外宣稱通過。若前段中止而尚未執行 finally，關閉這個專用終端機以清除其記憶體變數。

## B：APIM Primary / Secondary 的日常輪替

使用 [Gateway Lab](../ai-gateway/)的 `lab-app-a` Product 範圍訂用；`lab-app-b` 不改動。先確定沒有 open Product 可匿名存取，API 也要求訂用。兩把 Key 仍對應同一個訂用，**輪替不應重置配額**；quota 已耗盡時先等重置，不用全新訂用規避額度。

1. APIM → **Subscriptions → lab-app-a**，確認目前用戶端只用 Primary，Secondary 沒有在使用。
2. 執行以下設定。`$sid` 是訂用 entity 的 **ID**，不一定等於 Display name；從目標訂用的資源資訊確認，不猜名稱，也不填 Azure 訂用帳戶 ID。

```powershell
$apimName = "<existing-dedicated-lab-apim>"
$sid = "<lab-app-a-subscription-entity-id>"
$gatewayUri = "https://<apim-name>.azure-api.net/ai/openai/deployments/<deployment>/chat/completions?api-version=2024-10-21"
$subscriptionResource = "https://management.azure.com/subscriptions/$subscriptionId/resourceGroups/$rg/providers/Microsoft.ApiManagement/service/$apimName/subscriptions/$sid"
if ((Read-Host "確認專用訂用 ID，輸入該 ID 以允許輪替") -cne $sid) {
  throw "訂用未確認，停止。"
}
$oldPrimary = Read-Host "輸入 A 目前的 Primary" -MaskInput
if ((Test-LabKey -Uri $gatewayUri -HeaderName "Ocp-Apim-Subscription-Key" -Key $oldPrimary) -ne 200) {
  throw "原 Primary 未成功，停止輪替。"
}
az rest --method post `
  --url "$subscriptionResource/regenerateSecondaryKey?api-version=2024-05-01" --output none
if ($LASTEXITCODE -ne 0) { throw "Secondary 重新產生失敗，停止。" }
```

3. Portal 重新取得 Secondary，測試並遷移所有 A 用戶端。使用與本課 API 一致的預設 `Ocp-Apim-Subscription-Key` 標頭；若自訂過需同步調整函式與 API。

```powershell
$newSecondary = Read-Host "輸入 A 新 Secondary" -MaskInput
if ((Test-LabKey -Uri $gatewayUri -HeaderName "Ocp-Apim-Subscription-Key" -Key $newSecondary) -ne 200) {
  throw "新 Secondary 未成功，保留原 Primary，停止。"
}
if ((Read-Host "所有 A 用戶端已切換 Secondary 並驗證成功？輸入 SWITCHED") -cne "SWITCHED") {
  throw "尚未切換，不更新 Primary。"
}
az rest --method post `
  --url "$subscriptionResource/regeneratePrimaryKey?api-version=2024-05-01" --output none
if ($LASTEXITCODE -ne 0) { throw "Primary 重新產生失敗，停止。" }
$newPrimary = Read-Host "輸入 A 新 Primary" -MaskInput
try {
  "舊 Primary：HTTP {0}" -f (Test-LabKey -Uri $gatewayUri -HeaderName "Ocp-Apim-Subscription-Key" -Key $oldPrimary)
  "新 Primary：HTTP {0}" -f (Test-LabKey -Uri $gatewayUri -HeaderName "Ocp-Apim-Subscription-Key" -Key $newPrimary)
  "新 Secondary：HTTP {0}" -f (Test-LabKey -Uri $gatewayUri -HeaderName "Ocp-Apim-Subscription-Key" -Key $newSecondary)
} finally {
  Remove-Variable oldPrimary, newPrimary, newSecondary
}
```

預期依序 `401 / 200 / 200`。再用 B 的原憑證呼叫，預期 `200`，證明沒有誤改另一訂用。只改應用設定或只隱藏 Portal 的 Key，不能使舊 Key 失效。

## C：外洩止血演練，不等待無中斷遷移

只在相同專用沙盒以虛構事件演練，**不要真的把 Key 洩漏出去**。

| 順序 | 模型 Key 疑似外洩 | APIM 訂用 Key 疑似外洩 |
|------|-----------------|-----------------------|
| 1 | 保留必要的匿名化事件證據，停止分享；以既有可信管理身分處置 | 同左，確認僅 A 受影響 |
| 2 | 在該對照資源依[停用指引](../keyless-policy/#操作二停用本機金鑰)將 `disableLocalAuth` 設為 true，或立即重新產生受影響 Key；這不是等全員切換的日常輪替 | Portal → Subscriptions → A → **Suspend**；Unpublish Product 不算止血 |
| 3 | 用原本成功的舊 Key1 與 Key2 驗證停止金鑰入口；停用傳播期間不能假設已封鎖，必要時依既有應變計畫加網路限制 | 用原先成功的 Primary／Secondary 各呼叫一次，預期皆遭訂用驗證拒絕；B 仍可用。已在途請求可能完成 |
| 4 | 兩把都重新產生，清查程式碼與其他存放位置。保持本機驗證停用，改驗證 Entra／MI 可用，不為看見新 Key 的 200 而重啟 | 保持 Suspended，使用 B 段的兩個 regenerate 操作重新產生兩把；此時新 Key 也應遭拒。安全配發新值，經核准改回 **Active** |
| 5 | 檢視未授權用量、角色與影響，記錄後移交；A 段已單獨驗證過日常輪替的新舊 Key 行為 | 用輪替前兩個舊值測試皆遭拒，新值成功、B 成功；若有匿名路徑、繼承 policy 或傳播問題，先排除再宣告完成 |

撤銷影響的是後續驗證，不會取消已執行的推論，也不會撤銷另外授權的 Entra 身分。若發現管理身分也受影響，依事件流程處置，不能只輪替 API Key。

## 驗收紀錄與收尾

只記「類型、步驟、匿名代號、時間、狀態碼、是否通過」，不存任何金鑰、端點、租用戶／訂用帳戶或物件識別碼。日常流程至少驗證舊值先 200 後 401、兩個新值 200；外洩流程另驗證停用／暫停期間拒絕與恢復條件。未執行或只做桌上演練要明列，不能以預期表格當成實測。

不能復原舊 Key；回復是受控重新配發**新值**，不是把舊值設回去。關閉專用終端機、移除測試用戶端中的秘密，並依[課後清理](../prerequisites/#課後清理)處理資源。主線 Foundry 保持無金鑰；如保留 APIM 訂用，將後續輪替日期與負責人納入營運。

官方依據：[模型雙 Key 輪替](https://learn.microsoft.com/en-us/azure/ai-services/rotate-keys)、[模型 CLI regenerate](https://learn.microsoft.com/en-us/cli/azure/cognitiveservices/account/keys#az-cognitiveservices-account-keys-regenerate)、[APIM 訂用與金鑰生命週期](https://learn.microsoft.com/en-us/azure/api-management/api-management-subscriptions)、[Primary REST](https://learn.microsoft.com/en-us/rest/api/apimanagement/subscription/regenerate-primary-key?view=rest-apimanagement-2024-05-01)、[Secondary REST](https://learn.microsoft.com/en-us/rest/api/apimanagement/subscription/regenerate-secondary-key?view=rest-apimanagement-2024-05-01)。
