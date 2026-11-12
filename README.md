# DietSense — 對話式熱量 Domain AI Agent

> DIC3 期末專題 · Proposal Refactor with OpenSpec（2026-11-12）

DietSense 讓使用者用日常中文描述飲食（例如「一碗加蛋牛肉麵，湯喝一半」），由 LLM 理解與協調、領域工具負責檢索與計算、規則負責安全判斷，輸出**可追溯來源、含不確定性區間**的熱量與個人化體重管理建議。

## 四大核心能力

| | 能力 | 一句話 |
|---|---|---|
| C1 | 自然語言輸入 | 用講的就能記錄，不必搜尋、秤重 |
| C2 | 主動式智慧互動 | 不確定就主動問，最多兩輪 |
| C3 | 情境感知 | 依個人檔案、時段、歷史給建議 |
| C4 | 大數據驅動 | USDA + 臺灣食品資料庫 + 臺式料理知識庫 |

## Repository 結構

```text
DESIGN.md                       系統設計（Problem → … → Results），單一事實來源
docs/
  00-abstract.md                摘要（A–F，中文 + IEEE 英文）
  01-deep-research.md           5 核心思維 · 3 專業爭論 · 10 深度問題
  02-introduction.md            緒論（A–F）
  03-related-work.md            相關研究（AI ½ + Domain ½ + Article Summary Table）
  references.md                 已查證參考文獻
openspec/
  config.yaml                   專案背景與撰寫規則
  changes/build-dietsense-agent/
    proposal.md                 Why / What / Capabilities
    specs/<capability>/spec.md  7 個能力的行為規格（SHALL + Scenario）
    design.md                   技術決策 D1–D7、風險
    tasks.md                    實作任務清單（含驗證方式）
deliverables/
  DietSense_Proposal_v2.docx    繳交用 Proposal 文件（由 docs/ + DESIGN.md 產生）
  DietSense_Proposal_Presentation_v2.pptx  繳交用簡報（22 頁，已更新數據與文獻）
source/                         課堂原始素材（docx、pptx、PDF；舊版，含未查證文獻）
```

## Spec-Driven Workflow（OpenSpec）

```bash
npm install -g @fission-ai/openspec   # 安裝
openspec list                         # 查看進行中的變更
openspec show build-dietsense-agent   # 檢視提案
openspec validate --all --strict      # 驗證規格
# 實作：在 Claude Code 中執行 /openspec-apply-change build-dietsense-agent
openspec archive build-dietsense-agent  # 完成後將 specs 併入 openspec/specs/
```

流程：`DESIGN.md` → `proposal.md`（為什麼）→ `specs/`（系統必須做什麼）→ `design.md`（怎麼做）→ `tasks.md`（步驟）→ apply → archive。

## How to Run

尚未實作（Proposal 階段）。實作完成後補上。

## 狀態聲明

所有數值目標（±20% 內 ≥ 75%（且比通用 LLM 基準高 ≥ 15 個百分點）、記錄時間 −50%、SUS ≥ 70）皆為 **Expected Outcomes**，尚非實驗結果。
