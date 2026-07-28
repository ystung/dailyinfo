# dailyinfo

以 Hermes agent 為核心的「每日基礎設施情報收集」專案，針對 Unix / Linux 大規模維運場景設計。

適用環境重點：
- OpenShift
- Red Hat Enterprise Linux (RHEL)
- Dell PowerEdge / iDRAC / Firmware
- 超過千台伺服器
- 五個廠區的維運與變更風險控管

---

## 1) 專案目的

本專案目標是把每天分散在各官方來源的更新（版本、資安公告、已知問題、維運實務）轉成**可執行的維運建議**，幫助你：

- 每日快速掌握高風險變更
- 在 patching / firmware 升級前完成風險判讀
- 跨廠區協調維護時段與依賴團隊（Network / Storage / App）
- 將資訊直接落地為工單級行動項目

---

## 2) 目錄結構

```text
.
├─ skills/
│  └─ hermes-daily-unix-sre.md      # Hermes skill 規格（核心）
├─ templates/
│  └─ daily-report-template.md      # 每日報告模板
├─ configs/
│  ├─ sources.yml                   # 情報來源與收集規則
│  └─ site-profile.yml              # 五廠區設定與優先級策略
└─ README.md
```

---

## 3) 如何設定 Hermes 每日 / 每週排程

目前建議排程（Asia/Taipei）：
- 每日報告：`30 8 * * *`（08:30）
- 每週彙整：`0 9 * * 1`（週一 09:00）

對應檔案：`configs/sources.yml`

建議執行方式：
1. 在 Hermes 建立一個「Daily Infra Intel」任務。
2. 任務載入 `skills/hermes-daily-unix-sre.md` 作為固定指令。
3. 將排程設定為上述 cron。
4. 輸出時套用 `templates/daily-report-template.md`。

> 若你的 Hermes 平台有環境變數或工作流設定，可再補 `TIMEZONE=Asia/Taipei` 與通知管道（如 Slack/Email）。

---

## 4) 輸入來源與驗證規則

主要來源在 `configs/sources.yml`，分為：
- OpenShift
- RHEL
- Dell
- Security（NVD / CISA KEV）

收集規則（預設）：
- `cve_min_cvss: 7.0`
- `require_cross_validation: true`
- `label_verification_status: true`
- `prioritize_control_plane_impact: true`

實務建議：
- 優先採官方來源與可信來源
- 同事件盡量做雙來源交叉檢核
- 明確標示「已確認 / 待確認」避免誤判

---

## 5) 報告輸出格式

請固定使用 `templates/daily-report-template.md`，內容包含：

- A. 今日三大重點
- B. 重大變更與風險清單
- C. CVE 快速判讀
- D. 本日建議行動 Top 5
- E. Watchlist
- Site Impact Matrix（Site-1 ~ Site-5）

輸出原則：
- 避免純貼新聞
- 每項資訊都要附「建議行動」
- 能直接轉為工單敘述（可指派、可追蹤）

---

## 6) 五廠區維運策略與優先級

站點設定在 `configs/site-profile.yml`，包含：
- 各站 criticality
- 維護時段 maintenance_window
- 依賴團隊 dependencies

優先級策略（預設）：
- **P1**：正在被利用 / 影響控制平面 / 影響多廠區核心服務（SLA 24h）
- **P2**：高風險但可於 72h 內處理（SLA 72h）
- **P3**：納入一般維護週期

建議流程：
- 先在低風險站點做 canary（如 5~10% 節點）
- 觀察後再擴展至高 criticality 站點
- 控制平面或共用基礎服務一律走變更會議

---

## 7) 日常操作流程（Runbook）

每日建議流程：
1. 08:30 取得 Hermes 報告。
2. 先看 A + B，快速判斷是否有 P1。
3. 針對 C（CVE）確認是否影響：OpenShift node / RHEL / Dell 管理面。
4. 將 D（Top 5）轉成工單，指定站點與時段。
5. 更新 E（Watchlist）並追蹤次日進度。

每週建議流程（週一）：
1. 匯總上週 P1/P2 事件。
2. 檢查延遲處理項目與阻塞原因。
3. 調整各站 maintenance window 與變更策略。

---

## 8) 後續擴充建議

可逐步增加：
- Slack / Email 自動通知
- 與 ITSM / 工單系統串接（ServiceNow / Jira）
- 建立 CVE 影響資產映射（CMDB）
- 將「已確認/待確認」狀態寫回知識庫
- 導入每月 KPI（MTTR、準時修補率、風險暴露時間）

---

## 快速開始（TL;DR）

1. 使用 `skills/hermes-daily-unix-sre.md` 作為 Hermes 任務指令。  
2. 用 `configs/sources.yml` 設定來源與每日/每週排程。  
3. 用 `templates/daily-report-template.md` 固定輸出格式。  
4. 依 `configs/site-profile.yml` 執行五廠區分級維運。  
