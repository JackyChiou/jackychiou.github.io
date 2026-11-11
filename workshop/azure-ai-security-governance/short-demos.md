---
layout: default
title: 展示：Foundry 到 Gateway 三個短 Demo
parent: Azure AI 資安與治理實戰
grand_parent: Workshop
nav_order: 9
permalink: /workshop/azure-ai-security-governance/short-demos/
---

# 展示：Foundry 到 Gateway 三個短 Demo

**對象**：Azure Admin ｜ **形式**：實際截圖導讀 + 三個短 Demo ｜ **狀態**：以下皆為可執行流程與預期結果，未於本課 Azure 沙盒實跑

不必把整套 Lab 在課堂上重新部署。先備妥環境，依序展示「Foundry 部署 → Gateway 請求 → 應用隔離」，進一步的兩類 Key 輪替另依[專頁](../key-rotation/)操作。

## 課前準備（不計入展示時間）

只使用管理者事先核准且已存在的專用沙盒。模型推論與 APIM 會計費；若尚無環境、權限或額度，**只導讀官方截圖與流程，標記未執行**，不要自行建立付費資源或沿用正式環境金鑰。

1. 登入 [Microsoft Foundry](https://ai.azure.com/)，從既有 Project 的 Overview／管理資訊確認它所屬的 **Foundry 資源**與所在訂用帳戶。介面可能不同，找不到時先請環境管理者確認。
2. 辨識三層：**Foundry 資源**是本課模型端點、停用本機驗證與 MI 授權的目標；**Project**是團隊開發與組態脈絡；**模型部署**是呼叫時使用的部署別名，對應模型與版本，不是直接把模型商品名稱填入 URL。不要把每個 Project 誤當成獨立的金鑰或容量隔離邊界。
3. 在該資源／Project 的模型部署清單開啟既有部署，確認支援 **Azure OpenAI Chat Completions**、`2024-10-21` 與 `max_tokens`。抄下部署的**推論端點與部署名稱**，不使用含 `/api/projects/...` 的 Project 管理端點來做本課推論。官方說明見[模型端點與部署](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/concepts/endpoints)。
4. 依[無金鑰 Lab](../keyless-policy/)完成授權身分的直連測試，保持 Foundry `disableLocalAuth=true`。測試使用者的直接推論權限只保留到 Demo 1 完成，正式目標僅授權 Gateway MI 與必要管理者。
5. 依[Gateway Lab](../ai-gateway/)在 APIM 選取**同一個 Foundry 資源**，相容模式用 Azure OpenAI、Base path `ai`。檢查後端 MI、資源範圍 IAM、有效 policy 與 operation；不使用主線以外的另一個 OpenAI 資源替代。
6. 同一個 `lab-teaching` Product 建立 `lab-app-a`、`lab-app-b` 兩個 Product 範圍訂用，API 與 Product 都要求訂用，不存在 open Product。套用[Token policy](../token-governance/#操作一product-範圍的限額)，先用低量請求測試。
7. 依[監控 Lab](../monitoring/)備妥日誌；先確認低量流量不會耗盡模型容量。勿使用 APIM Test 的 all-access Key 演示 Product 限額。

課前預演成功不代表授權可永久沿用；開始時再次確認環境與角色。只展示遮蔽後的畫面，不錄製秘密輸入；官方兩張[實際 Portal 截圖](../ai-gateway/#實際-portal-截圖)可用於說明匯入與相容模式，但不是下列 Demo 已成功的證據。

## Demo 1：Foundry 的部署、端點與無金鑰（5 分鐘）

**展示次序**：Foundry Project 與所屬資源 → 模型部署名稱／版本 → 推論端點與 Project 端點的差別 → Entra 呼叫。

先貼上[前置頁](../prerequisites/)的沙盒變數，再執行[無金鑰頁的操作一](../keyless-policy/#操作一不帶金鑰的呼叫)完整 PowerShell 區塊。該區塊會建立後面使用的 `$uri` 與 `$body`，只顯示 HTTP 狀態並清除權杖。

**預期**：`200`，不使用任何模型 Key；畫面上的 deployment 與 URL 使用同一個別名。失敗就停止，先查 audience、角色、網路與部署，不把增加 Owner 當修復方法。Demo 1 的端點不可直接接上 APIM Base path，兩者分開設定。

## Demo 2：Gateway 能呼叫，直連不再授權（6 分鐘）

**展示次序**：APIM backend MI 與模型 IAM → Product 訂用 A → 有 Key 呼叫 → 無 Key 呼叫 → 移除本次暫時的直接推論權限。

保留 Demo 1 的 `$body`，在 PowerShell 7 執行：

```powershell
$gatewayBase = "https://<apim-name>.azure-api.net/ai"
$gatewayUri = "$gatewayBase/openai/deployments/$deployment/chat/completions?api-version=2024-10-21"
$keyA = Read-Host "輸入 A 的 Product 訂用金鑰" -MaskInput
try {
  $withKey = Invoke-WebRequest -Uri $gatewayUri -Method Post `
    -Headers @{ "Ocp-Apim-Subscription-Key" = $keyA } `
    -ContentType "application/json" -Body $body -SkipHttpErrorCheck
  $withoutKey = Invoke-WebRequest -Uri $gatewayUri -Method Post `
    -ContentType "application/json" -Body $body -SkipHttpErrorCheck
  "Gateway A：HTTP {0}" -f [int]$withKey.StatusCode
  "Gateway 無 Key：HTTP {0}" -f [int]$withoutKey.StatusCode
} finally {
  Remove-Variable keyA
}
```

**預期**：`200 / 401`。確認 `401` 來自缺少訂用 Key；不能只看見任意失敗就算通過。

接著由授權管理者依[Gateway 驗收段](../ai-gateway/#驗收與防繞過)只移除本課暫時給測試使用者的直接推論角色，保留 APIM MI。待權限傳播並取得新權杖後，重新執行 Demo 1 的呼叫，預期 `401`／`403`；再執行上方 Gateway 呼叫仍為 `200`。等待不計入展示時間，未觀察到拒絕就標為待確認，不能假稱即時生效。不能移除共用或原有角色。

## Demo 3：A 被限流，B 仍可用（8 分鐘）

**展示次序**：有效 Product policy → 兩個獨立訂用 → A 觸發 `429` → B `200` → 日誌核對來源。

1. 課前記錄原 policy，僅在專用 Product 暫設較低 TPM（例如 `100`）、較高 daily quota（例如 `20000`），避免先觸發 quota。計數器保持 `@(context.Subscription.Id)`，不改成共用固定字串。
2. 在同一終端機執行[Token 頁「有限次數的驗證」](../token-governance/#操作二有限次數的驗證)的 PowerShell 區塊；沿用 Demo 2 的 `$gatewayUri` 與 Demo 1 的 `$body`。迴圈最多 20 次、不重試失敗，不改用真實大型提示或無限壓測。
3. **預期** A 由 `200` 轉成 `429`，B 仍 `200`。短提示不保證 20 次內觸發；未觸發時先查有效 policy、Product 範圍與計數，必要時調整沙盒門檻或等待，不宣稱成功。
4. 在 APIM 日誌確認限流來自 Gateway policy 而非模型後端；若 B 也失敗，先查共用部署容量，不把後端節流當成隔離成功。日誌有延遲，可於預演後準備匿名化結果，但需標示觀測時間，不能用虛構輸出代替。
5. 還原原 Product policy。quota `403` 的獨立實驗依[Token Lab](../token-governance/)做，不在同一短 Demo 同時改低兩個門檻。

## 展示驗收與限制

| 項目 | 通過條件 | 本教材目前狀態 |
|------|----------|----------------|
| Demo 1 | Entra 推論 `200`，部署／端點對應正確 | 未雲端實跑 |
| Demo 2 | Gateway 有 Key `200`、無 Key `401`；移除暫時直連角色後直連遭拒而 MI 仍成功 | 未雲端實跑 |
| Demo 3 | Gateway policy 對 A 限流，B 成功，來源已確認，policy 已還原 | 未雲端實跑 |

預期結果、官方截圖與語法檢查都不能替代雲端實測。學員可在自己的核准沙盒填入觀測時間、匿名化狀態碼與結果；沒有環境就明列未執行。結束後清除本機秘密並依[課後清理](../prerequisites/#課後清理)處理專用資源。
