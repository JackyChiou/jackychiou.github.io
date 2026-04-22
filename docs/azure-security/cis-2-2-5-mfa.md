---
layout: default
title: "CIS 2.2.5 — MFA 條件式存取"
parent: Azure 資安合規
nav_order: 1
permalink: /docs/azure-security/cis-2-2-5-mfa/
---

# CIS 2.2.5 — MFA 條件式存取原則
{: .no_toc }

<details open markdown="block">
  <summary>目錄</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## 基本資訊

| 項目 | 說明 |
|------|------|
| **CIS 等級** | Level 2 |
| **前提條件** | Microsoft Entra ID P1 或 P2 |
| **影響範圍** | 所有使用者（含 Break Glass 帳號） |
| **強制日期** | 2024 年 10 月（Microsoft 強制啟用） |

{: .warning }
自 **2024 年 10 月**起，Microsoft 已強制要求所有 Azure 使用者啟用 MFA，包括 Break Glass（緊急存取）帳號。未遵從的組織將無法登入 Azure Portal。

---

## 為什麼需要 MFA？

帳號密碼外洩是最常見的攻擊途徑。MFA 在密碼之外增加第二驗證因素，即使密碼遭竊，攻擊者仍無法登入。

**支援的 MFA 方法：**
- Microsoft Authenticator App
- FIDO2 實體安全金鑰（推薦用於 Break Glass 帳號）
- 憑證式驗證（儲存於安全可移除儲存裝置）
- SMS / 電話（不建議，安全性較低）

---

## 稽核步驟

1. **Azure Portal** → 左上角 ☰ → **Microsoft Entra ID**
2. 左側選單 → **Security**
3. **Conditional Access** → **Policies**
4. 找到現有的 MFA 原則 → 點入查看
5. 確認 **Users > Include** 已包含 `All users`
6. 確認 **Grant** 已勾選 `Require multifactor authentication`

---

## 設定步驟

### 步驟 1：建立新原則

```
Azure Portal → Microsoft Entra ID → Security → Conditional Access → Policies → + New policy
```

輸入原則名稱，例如：`MFA - Require for All Users`

### 步驟 2：設定使用者範圍

| 設定 | 值 |
|------|-----|
| **Include** | All users |
| **Exclude** | 指定豁免帳號（如 Break Glass 帳號所在群組） |

{: .highlight }
Break Glass 帳號必須在 **Exclude** 中排除，但需另行確保其使用 FIDO2 金鑰或憑證。

### 步驟 3：設定目標資源

| 設定 | 值 |
|------|-----|
| **Target resources** | All cloud apps |

### 步驟 4：設定授權控制

| 設定 | 值 |
|------|-----|
| **Grant access** | ✅ Require multifactor authentication |

### 步驟 5：啟用原則

{: .important }
**務必先以 Report-only 模式測試 7–14 天**，確認不影響正常業務，再切換為 `On`。

| 階段 | Enable policy 設定 |
|------|-------------------|
| 測試期 | `Report-only` |
| 正式啟用 | `On` |

---

## Break Glass 帳號處理

Break Glass 帳號是緊急存取帳號，必須在所有 Conditional Access 原則的 Exclude 清單中，但仍需符合 MFA 要求：

| 方法 | 說明 |
|------|------|
| **FIDO2 安全金鑰** | 推薦。實體金鑰存放於安全、有文件記錄的實體位置 |
| **憑證式驗證** | 憑證儲存於安全可移除儲存裝置（如加密 USB） |

{: .warning }
Break Glass 帳號的實體金鑰或憑證存放位置應有正式文件記錄，並定期測試存取是否正常。

---

## 相關資源

- [完整設定指南（Blog 文章）](/2026/03/25/entra-id-mfa-conditional-access/){: .btn .btn-primary }
- [Microsoft：Conditional Access 文件](https://learn.microsoft.com/en-us/entra/identity/conditional-access/overview)
- [Microsoft：Break Glass 帳號最佳實踐](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/security-emergency-access)
