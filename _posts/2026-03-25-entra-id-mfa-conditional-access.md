---
layout: post
title: "CIS 基準 2.2.5：為所有使用者建立多因素驗證（MFA）條件式存取原則——完整設定指南"
date: 2026-03-25
categories: [Azure, Security, CIS Benchmark]
---

在 Microsoft 365 與 Azure 的安全性基準中，**CIS Benchmark 2.2.5** 要求組織為所有使用者建立多因素驗證（MFA）條件式存取原則。自 2024 年 10 月起，Microsoft 已強制所有使用者（包括 Break Glass 帳號）啟用 MFA。本文將詳細說明如何透過 Microsoft Entra ID Conditional Access 完成這項設定。

## 為什麼需要 MFA 條件式存取原則？

**風險：** 帳號密碼外洩是最常見的攻擊途徑。僅靠密碼保護的帳號一旦被入侵，攻擊者即可存取組織所有雲端資源。

**效益：** 啟用 MFA 後，即使密碼被竊取，攻擊者仍需要第二個驗證因素（如手機驗證、FIDO2 金鑰）才能登入，大幅降低帳號被入侵的風險。

> **CIS 等級：** Level 2
> **前提條件：** 需要 Microsoft Entra ID P1 或 P2 授權

---

## 第一部分：稽核現有原則

在建立新原則之前，先確認組織是否已有相關的 MFA 原則。

### 步驟 1：進入 Microsoft Entra ID

登入 [Azure Portal](https://portal.azure.com)，點擊左上角的 **Portal Menu（☰）**，選擇 **Microsoft Entra ID**。

```
Azure Portal
└── ☰ Portal Menu
    └── Microsoft Entra ID
```

### 步驟 2：進入條件式存取

在左側選單中向下捲動，依序點擊：

1. **Security（安全性）**
2. **Conditional Access（條件式存取）**
3. **Policies（原則）**

```
Microsoft Entra ID
├── Overview
├── Users
├── Groups
└── Security
    └── Conditional Access
        └── Policies  ← 在這裡
```

### 步驟 3：檢查現有原則

在 Policies 清單頁面中，檢查是否已有針對「All users」要求 MFA 的原則：

- 點擊要稽核的原則名稱
- 檢查 **Users** 區段的 **Include** 是否涵蓋所有使用者
- 檢查 **Exclude** 是否有不當排除的帳號
- 檢查 **Grant** 區段是否要求 **Require multifactor authentication**

> **稽核重點：** 如果沒有找到涵蓋所有使用者的 MFA 原則，或現有原則的涵蓋範圍不完整，請依照下方步驟建立新原則。

---

## 第二部分：建立 MFA 條件式存取原則

### 步驟 1：新增原則

在 **Conditional Access → Policies** 頁面，點擊上方的 **＋ New policy**。

```
┌──────────────────────────────────────────────┐
│  Conditional Access | Policies               │
│                                              │
│  [+ New policy]  [↓ New policy from template]│
│                                              │
│  Policy Name        State       Last Modified│
│  ─────────────────────────────────────────── │
│  (existing policies listed here)             │
└──────────────────────────────────────────────┘
```

### 步驟 2：命名原則

在 **Name** 欄位輸入清晰的原則名稱，例如：

```
CIS 2.2.5 - Require MFA for All Users
```

> **命名建議：** 加上 CIS 編號方便日後對照稽核。

### 步驟 3：設定使用者範圍

點擊 **Users** 下方的藍色文字連結進入設定：

**Include 頁籤：**
- 選擇 **All users**

```
┌─ Users ──────────────────────────────────────┐
│                                              │
│  Include                    Exclude          │
│  ────────                   ───────          │
│  ○ None                                      │
│  ○ Select users and groups                   │
│  ● All users  ← 選擇這個                     │
│                                              │
└──────────────────────────────────────────────┘
```

**Exclude 頁籤：**
- 勾選 **Users and groups**
- 選擇需要排除的帳號（如 Break Glass 緊急存取帳號）

```
┌─ Exclude ────────────────────────────────────┐
│                                              │
│  ☑ Users and groups                          │
│                                              │
│  Selected: 2 users                           │
│  ├── BreakGlass-Account-01                   │
│  └── BreakGlass-Account-02                   │
│                                              │
│  [Select]                                    │
└──────────────────────────────────────────────┘
```

> **重要提醒：** 自 2024 年 10 月起，Microsoft 要求所有帳號（包括 Break Glass 帳號）都必須啟用 MFA。若排除 Break Glass 帳號，應為其設定 **FIDO2 實體安全金鑰** 或存放在安全實體位置的憑證，以滿足 MFA 要求。

### 步驟 4：設定目標資源

點擊 **Target resources** 下方的藍色文字連結：

- 選擇 **All cloud apps**

```
┌─ Target resources ───────────────────────────┐
│                                              │
│  Select what this policy applies to:         │
│  Cloud apps                                  │
│                                              │
│  Include                    Exclude          │
│  ────────                   ───────          │
│  ○ None                                      │
│  ● All cloud apps  ← 選擇這個                │
│  ○ Select apps                               │
│                                              │
└──────────────────────────────────────────────┘
```

### 步驟 5：設定授與控制

點擊 **Grant** 下方的藍色文字連結：

- 選擇 **Grant access**
- 勾選 **Require multifactor authentication**
- 點擊 **Select**

```
┌─ Grant ──────────────────────────────────────┐
│                                              │
│  ○ Block access                              │
│  ● Grant access                              │
│                                              │
│  Require one of the selected controls:       │
│  ☑ Require multifactor authentication        │
│  ☐ Require authentication strength           │
│  ☐ Require device to be marked as compliant  │
│  ☐ Require Microsoft Entra hybrid joined     │
│  ☐ Require approved client app               │
│  ☐ Require app protection policy             │
│  ☐ Require password change                   │
│                                              │
│  [Select]                                    │
└──────────────────────────────────────────────┘
```

### 步驟 6：先以報告模式啟用

在頁面底部的 **Enable policy** 區段：

- 先選擇 **Report-only**（僅報告模式）
- 點擊 **Create** 建立原則

```
┌─ Enable policy ──────────────────────────────┐
│                                              │
│  ● Report-only  ← 先選擇這個進行測試          │
│  ○ On                                        │
│  ○ Off                                       │
│                                              │
│  [Create]                                    │
└──────────────────────────────────────────────┘
```

> **為什麼要用 Report-only？** 直接啟用可能導致大量使用者無法登入。Report-only 模式會記錄原則「如果啟用會影響哪些登入」，讓你在正式上線前確認影響範圍。

---

## 第三部分：測試與正式啟用

### 檢視 Report-only 結果

1. 前往 **Microsoft Entra ID → Security → Conditional Access → Insights and reporting**
2. 選擇時間範圍（建議至少觀察 **3-7 天**）
3. 篩選你剛建立的原則
4. 檢視以下項目：
   - **會被要求 MFA 的使用者數量**
   - **可能被阻擋的登入嘗試**
   - **尚未註冊 MFA 的使用者**

### 正式啟用原則

確認 Report-only 結果無異常後：

1. 回到 **Conditional Access → Policies**
2. 點擊原則名稱
3. 將 **Enable policy** 從 **Report-only** 改為 **On**
4. 點擊 **Save**

```
Enable policy: Report-only  →  On ✓
```

---

## 第四部分：Break Glass 帳號的 MFA 處理

由於 Microsoft 自 2024 年底已強制所有帳號啟用 MFA，Break Glass（緊急存取）帳號也需要 MFA 方案：

| 方案 | 說明 | 建議 |
|------|------|------|
| **FIDO2 實體安全金鑰** | USB/NFC 硬體金鑰 | ✅ 推薦：將金鑰存放在保險箱等安全位置 |
| **憑證（Certificate）** | 存放於安全的可移除儲存媒體 | ✅ 適用於更高安全等級的環境 |
| **手機驗證** | Authenticator App 或 SMS | ⚠️ 不建議：Break Glass 帳號不應依賴個人裝置 |

> **最佳實務：** 為 Break Glass 帳號準備 **至少兩把 FIDO2 金鑰**，分別存放在不同的安全實體位置，並記錄於組織的災難復原文件中。

---

## 常見問題

### Q：需要什麼授權？

條件式存取原則需要 **Microsoft Entra ID P1** 或 **P2** 授權。如果組織使用 Microsoft 365 E3/E5 或 EMS E3/E5，已包含此授權。

### Q：使用者會受到什麼影響？

首次觸發 MFA 時，尚未註冊的使用者會被引導完成 MFA 註冊流程。建議在啟用前透過內部公告提前通知使用者。

### Q：如何確認所有使用者都已註冊 MFA？

前往 **Microsoft Entra ID → Security → Authentication methods → User registration details**，可查看各使用者的 MFA 註冊狀態。

### Q：Report-only 模式要觀察多久？

建議至少 **3-7 天**，涵蓋正常工作日與假日，確保所有使用者的登入模式都被納入評估。

---

## 總結

| 項目 | 設定值 |
|------|--------|
| **原則名稱** | CIS 2.2.5 - Require MFA for All Users |
| **Users - Include** | All users |
| **Users - Exclude** | Break Glass 帳號（需另外設定 FIDO2） |
| **Target resources** | All cloud apps |
| **Grant** | Require multifactor authentication |
| **啟用流程** | Report-only → 觀察 3-7 天 → On |

完成上述設定後，組織即符合 CIS Benchmark 2.2.5 的要求，為所有使用者建立了 MFA 條件式存取原則，有效降低帳號遭入侵的風險。

---

**參考資料：**
- [CIS Microsoft 365 Foundations Benchmark](https://www.cisecurity.org/benchmark/microsoft_365)
- [Microsoft Entra Conditional Access documentation](https://learn.microsoft.com/en-us/entra/identity/conditional-access/)
- [Manage emergency access accounts in Microsoft Entra ID](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/security-emergency-access)
- [Plan a Conditional Access deployment](https://learn.microsoft.com/en-us/entra/identity/conditional-access/plan-conditional-access)
