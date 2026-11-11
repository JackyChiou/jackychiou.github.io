---
layout: default
title: 00：前置準備與安全守則
parent: Azure AI 資安與治理實戰
grand_parent: Workshop
nav_order: 1
permalink: /workshop/azure-ai-security-governance/prerequisites/
---

# 00：前置準備與安全守則

## 必要環境

準備 Azure 沙盒訂用帳戶、一個**專用**資源群組，以及可使用的 Foundry / Azure OpenAI 模型部署。模型與 APIM 的區域、SKU 和配額要分別確認，不預設所有區域都可用。

| 操作 | 權限需求 |
|------|----------|
| 盤點 Azure 資源 | 目標範圍 Reader |
| 建立或修改資源 | 專用資源群組 Contributor，必要時由管理員預先建立 |
| Azure 角色指派 | 目標範圍具角色指派權限，如 Role Based Access Control Administrator；Contributor 不足 |
| Policy 指派 | 目標範圍 Resource Policy Contributor 或相應權限 |
| Entra 角色盤點 | 可讀取目錄角色，如 Global Reader；不必為盤點授予 Global Administrator |
| Entra 設定變更 | 僅測試租戶，依任務分配 User / Authentication Policy / Conditional Access / Privileged Role Administrator 等角色 |

Azure RBAC 與 Entra 目錄角色是兩套權限。訂用帳戶 Owner 不等於 Entra Global Administrator，也不應將 Owner 當作每位學員的預設需求。

## 工具與本機設定

入口網站即可完成主要設定；呼叫測試採 **PowerShell 7**，使用 `-MaskInput` 與 `-SkipHttpErrorCheck`，不適用 Windows PowerShell 5.1。另備妥 Azure CLI；不執行陌生來源腳本。

以下佔位符必須換成自己的**沙盒**值；每次新開終端機重新設定。變數只保留在本機，不提交到 Git。

```powershell
$subscriptionId = "<sandbox-subscription-id>"
$rg = "<dedicated-lab-resource-group>"
$modelEndpoint = "https://<model-resource-name>.openai.azure.com"
$deployment = "<chat-completions-deployment-name>"

az login
az account set --subscription $subscriptionId
if ($LASTEXITCODE -ne 0) { throw "無法選取沙盒訂用帳戶，請停止操作。" }
az account show --query "{name:name, id:id}" -o table
```

在 Azure Portal 建立 `$rg`，並於其中準備可使用 **Azure OpenAI Chat Completions** 的模型部署。從資源頁複製實際端點，確認部署名稱、支援的 API 版本與一次少量呼叫可用。後續範例採 `2024-10-21`；若模型不支援此版本或 `max_tokens`，請依模型文件調整，勿混用 Responses API。

本課的 **Foundry 主線**使用 Foundry 資源下的 Project 與模型部署，不以另一個獨立 Azure OpenAI 資源代替。資源、Project、部署名稱與推論端點的銜接見[短 Demo 的環境準備](../short-demos/#課前準備不計入展示時間)。僅使用已獲授權且預先備妥的沙盒；沒有模型或 APIM 時先做閱讀，不因本課自行建立付費資源。

## 成本與資料界線

在開始前記下資源擁有者、預算與清理時間。APIM 選用支援所需 policy 的 SKU；本課以 Basic v2 沙盒為例，**不是免費資源**。不要在未確認費用時建立服務。

練習一律使用無個資的短提示，例如 `ping`。不要輸入學生資料、研究內容或公司內部文件。模型金鑰與 APIM 訂用金鑰都是機密；只在需要的測試階段以遮罩輸入，不出現在命令歷史、截圖或檔案。

截圖與繳交資料只保留設定狀態、HTTP 狀態碼與匿名化結果。遮蔽姓名、UPN、租用戶 / 訂用帳戶 / 物件 ID、實際端點、IP 與通知收件人；緊急帳號憑證、金鑰序號與保管位置也不能公開。

## 開始前驗收

確認目前訂用帳戶為沙盒、資源群組專供本課使用，且你知道誰能協助角色指派。若權限或模型不足，先做[唯讀盤點](../inventory/)，不要藉由擴大權限繞過限制。

## 課後清理

先確認後續 Lab 不再需要資源。在 Portal 檢視專用資源群組的**所有內容**，確認沒有共用資源後再刪除。APIM、模型部署、Log Analytics、警示與預算要逐項確認；單純停用 API 或 Gateway 不會停止 APIM 計費。

資源群組以外的角色、Entra 測試帳號、群組與 Policy 指派不一定隨之刪除，依各 Lab 回復。軟刪除可能使名稱暫時無法重用，勿為了重用名稱而任意清除保留資料。

[下一步：現況盤點](../inventory/)
