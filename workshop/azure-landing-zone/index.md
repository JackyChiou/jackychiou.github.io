---
layout: default
title: Azure Landing Zone 醫療雲端
parent: Workshop
has_children: true
nav_order: 1
permalink: /workshop/azure-landing-zone/
---

# Azure Landing Zone 醫療雲端 Workshop

{: .highlight }
**課程系列**：Azure 醫療雲端架構與治理 ｜ **對象**：Healthcare 產業技術人員 ｜ **部署區域**：Taiwan North

本 Workshop 以醫療院所為情境，帶領學員從零開始建立符合醫療法規（個資法、HIPAA）且具備完整災難復原能力的 Azure Landing Zone。課程包含六個模組，由治理設計到監控收尾，並附有三份參考指南供實作參考。

---

## 課程架構

```
Azure 醫療雲端 Landing Zone
│
├── M1：Landing Zone 治理設計          ← Management Group / RBAC / Policy
├── M2：Hub-Spoke 網路架構             ← VNet / Subnet / NSG / Route Table
├── M3：Azure Firewall + Bastion       ← 集中式防火牆 / 安全管理連線
├── M4：Private DNS / DNS 架構         ← Private Endpoint / DNS Resolver
├── M5：ASR 災難復原                   ← Site Recovery / RTO/RPO / Recovery Plan
├── M6：監控管理 + 課程收尾            ← Defender for Cloud / Alert / KQL
│
└── 附錄
    ├── 命名規則規範                   ← CAF 命名格式、各資源類型縮寫
    ├── RBAC 規劃指南                  ← 自訂角色、PIM、存取審查
    └── Azure Policy 規劃指南          ← Policy 效果、Initiative、合規策略
```

---

## 模組總覽

| 模組 | 主題 | 核心技術 | Demo | Lab |
|------|------|----------|------|-----|
| M1 | Landing Zone 治理設計 | Management Group、RBAC、Azure Policy | 25 分鐘 | 40 分鐘 |
| M2 | Hub-Spoke 網路架構 | VNet、Subnet、NSG、Route Table | 30 分鐘 | 50 分鐘 |
| M3 | Azure Firewall + Bastion | Firewall Premium、Bastion Standard | 35 分鐘 | 40 分鐘 |
| M4 | Private DNS / DNS 架構 | Private Endpoint、DNS Resolver | 25 分鐘 | 45 分鐘 |
| M5 | ASR 災難復原 | Site Recovery、Recovery Plan | 40 分鐘 | 45 分鐘 |
| M6 | 監控管理 + 課程收尾 | Defender for Cloud、Alert、KQL | 30 分鐘 | 45 分鐘 |

---

## 前置需求

{: .important }
開始 Workshop 前，請確認以下環境已就緒。

- Azure 訂閱（Owner 角色）
- 已準備兩個帳號：工程師帳號 + 稽核帳號
- 部署區域：**Taiwan North**（主要）、**Taiwan Northwest**（DR，M5 使用）
- Azure Portal 可存取（建議使用 Edge 或 Chrome）

---

## 醫療情境說明

本 Workshop 以一家中型醫院為範例，部署以下四套關鍵系統至 Azure Taiwan North：

| # | 系統名稱 | 說明 | 資料分類 |
|---|----------|------|----------|
| 1 | HIS（醫院資訊系統） | 含病歷，個人敏感資料 | 最高等級保護 |
| 2 | PACS（醫療影像） | DICOM 影像，每月新增 ~5TB | 大容量儲存 |
| 3 | 門診 App | 對外服務，病患可存取 | DMZ 隔離 + WAF |
| 4 | 研究資料庫 | 去識別化資料，供學術研究 | 較寬鬆 Policy |

---

## 附錄參考指南

| 文件 | 說明 |
|------|------|
| [命名規則規範](./appendix/naming-convention/) | Azure 資源命名格式、各類型縮寫對照 |
| [RBAC 規劃指南](./appendix/rbac-guide/) | 自訂角色設計、PIM 設定、存取審查 |
| [Azure Policy 規劃指南](./appendix/policy-guide/) | Policy 效果、Initiative 設計、合規策略 |
