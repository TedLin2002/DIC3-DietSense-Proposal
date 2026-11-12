# DESIGN.md — DietSense

> **DietSense: A Data-Grounded and Safety-Aware Conversational AI Agent for Personalized Dietary Assessment and Weight Management**
>
> 文件狀態：Proposal 階段（v0.2, 2026-11-12）。本文件為系統設計的「單一事實來源」，並已拆解為 OpenSpec 工作流程，見 `openspec/changes/build-dietsense-agent/`。所有數值目標皆為 **Expected Outcomes**，非實驗結果。

依課程規範，本文件依序回答：Problem → Requirements → Architecture → Components → Data → AI/Analytics → Domain Knowledge → Algorithm → Implementation → How to Run → Results。

---

## 1. Problem

| # | 問題 | 對使用者的影響 |
|---|---|---|
| P1 | 手動記錄負擔高：需搜尋食物、選份量、拆解複合菜 | 記錄中斷、長期遵循率隨時間下降 |
| P2 | 自報攝取誤差大（約 30% 受試者低報、熱量低估約 15%；全國調查錯報 > 50%），份量、隱藏油脂、湯底擴大不確定性 | 熱量數字看似精確但不可信 |
| P3 | 臺灣在地料理（夜市小吃、便當、滷味、湯麵）在國際資料庫覆蓋不足 | 找不到或對錯品項 |
| P4 | 通用 LLM 直接回答熱量會幻覺、虛構數值、給出未經驗證的建議 | 錯誤或不安全的節食建議 |

**一句話定位**：把「被動、繁瑣的熱量記錄」轉為「以自然語言驅動、資料錨定、安全守門的個人化體重管理 Agent」。

## 2. Requirements

### 2.1 四大核心能力（Core Capabilities, C1–C4）

| 編號 | 能力 | 說明 | 驗證指標 |
|---|---|---|---|
| C1 | 自然語言輸入 Natural Language Input | 以日常繁中（文字/語音）記錄飲食 | 每筆記錄秒數、完成率、實體抽取 F1 |
| C2 | 主動式智慧互動 Proactive Agentic Interaction | 模糊時主動追問、提醒未記錄餐次 | 澄清後誤差下降幅度、追問次數、提醒接受率 |
| C3 | 情境感知 Context Awareness | 依個人檔案、時段、活動量、歷史給建議 | 建議適切性評分、建議接受率 |
| C4 | 大數據驅動 Big Data–Driven Intelligence | 多來源營養資料 + 在地料理庫 + 歷史行為 | 臺式料理涵蓋率、熱量 MAE、來源揭露率 |

### 2.2 功能需求（FR）

- FR1 解析餐點描述 → 食物實體（名稱、份量、烹調方式、餐次、時間）
- FR2 從多來源資料庫檢索候選並排名，附信心分數
- FR3 信心不足或份量模糊時觸發 Clarification Loop（上限 2 輪）
- FR4 以確定性公式計算熱量與巨量營養素，輸出 **低 / 中 / 高** 區間
- FR5 以 Mifflin–St Jeor × 活動係數估算 BMR / TDEE，計算攝取缺口
- FR6 產生替代建議與每日/每週趨勢
- FR7 安全守門：極低熱量（女性 < 1,200、男性 < 1,500 kcal/日，連續 3 日）、極端節食語句、飲食失調風險、特殊族群 → 降級並轉介
- FR8 每個數值附資料來源（資料庫名稱 + 品項 ID）

### 2.3 非功能需求（NFR）

- NFR1 **LLM 不得直接產生營養數值**，所有數值必須來自工具回傳（可稽核）
- NFR2 單次記錄回應時間 < 5 秒（不含使用者追問）
- NFR3 個資最小化：健康資料僅存本地 / 使用者帳號下，不用於模型訓練
- NFR4 所有建議附「非醫療診斷」聲明

## 3. Architecture（① – ⑧）

```text
 ① User / Scenario ──► ② Query / Input ──► ③ Planner & Reasoning Agent (LLM)
   App / Web Chat        Whisper STT           意圖分類、任務拆解、工具選擇
   文字 / 語音            繁簡正規化             Clarification Loop 控制
                         意圖分類                     │  whitelisted function calls
                                                      ▼
 ④ Data Retrieval ◄──────────────────────────────────┤
   USDA FDC API · 臺灣食品營養成分資料庫 · 臺式料理向量庫(RAG) · 使用者歷史
                                                      ▼
 ⑤ Analytics Engine（確定性）── E = g × kcal/100g ÷ 100、巨量營養素、TDEE、區間
                                                      ▼
 ⑥ Domain Knowledge Integration ── 每日飲食指南、DRIs/AMDR、份量對照表、安全規則庫
                                                      ▼
 ⑦ Decision / Recommendation ── 缺口分析、替代建議、安全降級
                                                      ▼
 ⑧ Output & Interaction ── 文字回覆 + 區間 + 來源 + 圖表 + 下一步引導
```

設計原則：**LLM 負責理解與協調（③），工具負責取數與計算（④⑤），規則負責最終安全判斷（⑥⑦）。**

## 4. Components

| 模組 | 責任 | 輸入 → 輸出 | 對應能力 |
|---|---|---|---|
| ① User/Scenario | 文字/語音介面、個人檔案設定 | 使用者 → 原始輸入 | C1 |
| ② Input Processing | STT、正規化、意圖分類（log_food / query_nutrition / analyze / recommend） | 原始輸入 → Q_norm, intent | C1 |
| ③ Planner Agent | 任務拆解、工具呼叫序列、對話狀態、追問 | Q_norm → tool plan | C2 |
| ④ Retrieval | 多來源候選檢索、排名、信心分數 | 食物實體 → candidates, score | C4 |
| ⑤ Analytics | 熱量/營養素/TDEE/區間計算 | match + 份量 → 數值區間 | C4 |
| ⑥ Domain KB | 指南、份量對照、安全規則 | 數值 + 檔案 → 規則判定 | C3 |
| ⑦ Decision | 缺口、建議、安全降級 | 判定 → 建議 | C3 |
| ⑧ Output | 回覆格式化、圖表、來源揭露 | 結果 → 回應 R | C1/C2 |

### 4.1 Tool Whitelist（Function Calling）

| Tool | 用途 | 確定性 |
|---|---|---|
| `search_food(name, context)` | 跨資料庫檢索候選 | 是 |
| `estimate_portion(desc, food_id)` | 家用量詞 → 克數區間（查份量對照表） | 是 |
| `compute_nutrition(food_id, grams_range)` | 計算熱量與營養素區間 | 是 |
| `compute_tdee(profile)` | Mifflin–St Jeor × 活動係數 | 是 |
| `log_meal(entry)` / `get_daily_log(date)` | 讀寫日誌 | 是 |
| `check_safety(log, profile, text)` | 規則庫判定 | 是 |

LLM 只能呼叫上表工具；任何不在白名單的呼叫或 LLM 自行產生的數值皆被拒絕。

## 5. Data

| 資料來源 | 內容 | 用途 | 備註 |
|---|---|---|---|
| USDA FoodData Central | 美國官方食品營養資料（API） | 國際食材、原料 | 公有領域 |
| 臺灣食品營養成分資料庫（TFDA） | 臺灣常見食材營養素 | 在地食材 | 需確認最新版本與授權 |
| 臺式料理向量庫（自建） | 料理 → 食材組成 + 份量 + 來源 | 複合料理拆解（RAG） | 由前兩者組成計算 |
| 份量對照表（自建） | 一碗/一份/一顆 → 克數區間 | 份量推估 | 參考每日飲食指南份量 |
| 使用者日誌 | 歷史飲食、體重、活動 | 情境感知、趨勢 | 本地儲存 |
| 評估資料集（自建，500 筆） | 臺式餐點描述 + 參考熱量 | 實驗 | 參考值由營養資料庫計算並人工複核 |

## 6. AI / Analytics

- **LLM**：負責意圖分類、實體抽取、追問生成、回覆潤飾；採 function calling，temperature 低。
- **RAG**：臺式料理向量庫，以料理名稱 + 描述嵌入檢索，回傳結構化組成。
- **熱量**：`E = portion(g) × kcal_per_100g / 100`；複合料理 = Σ 食材。
- **BMR (Mifflin–St Jeor)**：男 `10W + 6.25H − 5A + 5`；女 `10W + 6.25H − 5A − 161`；`TDEE = BMR × activity_factor`。
- **不確定性**：份量取 (低, 中, 高) 克數 → 傳遞至熱量區間；信心分數 < 0.80 觸發追問。
  - 注意：此為**啟發式估計區間**，非統計校準信賴區間；實驗階段將以校準度（coverage）檢驗。

## 7. Domain Knowledge

1. 能量平衡（Energy Balance）與動態體重模型
2. 巨量營養素組成（Atwater 係數 4/4/9/7 kcal/g；AMDR）
3. 份量估計與自報誤差
4. 個體差異（BMR、代謝反應差異）
5. 飲食行為情境（情緒性進食、時段、社交）
6. 安全邊界：極低熱量飲食、飲食失調風險、孕婦/慢性病/用藥族群轉介

詳見 `docs/01-deep-research.md`（5 核心思維 / 3 專業爭論 / 10 深度問題）。

## 8. Algorithm

```text
Algorithm 1: DietSense Meal-Logging & Advisory Agent
Input : user query Q, profile P, daily log D
Output: response R, updated log D'
01: Q' ← normalize(STT(Q))                         ▷ ②
02: intent ← classify_intent(Q')                    ▷ ②
03: if intent = log_food then
04:   F ← extract_food_entities(Q')                 ▷ ③ LLM, structured output
05:   for each f in F do
06:     C, s ← search_food(f)                       ▷ ④ USDA / TFDA / RAG
07:     k ← 0
08:     while (s < 0.80 or ambiguous(f.portion)) and k < 2 do
09:       f ← f ⊕ ask_user(clarify(f, C)); C, s ← search_food(f); k ← k + 1
10:     end while
11:     g_lo, g_mid, g_hi ← estimate_portion(f, C*)  ▷ ⑤ 份量對照表
12:     E ← compute_nutrition(C*, g_lo, g_mid, g_hi) ▷ ⑤ 確定性
13:     D ← D ∪ {(f, C*, E, source(C*), s)}
14:   end for
15: else if intent ∈ {analyze, recommend} then
16:   T ← compute_tdee(P); Δ ← Σ D.E_mid − T          ▷ ⑤
17:   A ← apply_guidelines(D, P, Δ)                   ▷ ⑥ 指南 + AMDR
18: end if
19: σ ← check_safety(D, P, Q')                        ▷ ⑥ 規則庫
20: if σ = DANGER then A ← safe_fallback_and_referral(P)  ▷ ⑦
21: R ← render(D, E-ranges, sources, A, charts)       ▷ ⑧
22: return R, D
```

## 9. Implementation（規劃）

- 實作步驟拆解於 `openspec/changes/build-dietsense-agent/tasks.md`。
- 建議技術棧（可替換）：Python 3.12、FastAPI、SQLite、Chroma/FAISS 向量庫、任一支援 function calling 的 LLM、Streamlit 或 Web Chat 前端。

## 10. How to Run

尚未實作。實作完成後補上安裝、環境變數、執行與測試指令。

## 11. Results / Evaluation Plan

| 項目 | 設計 |
|---|---|
| Dataset | 500 筆臺式餐點描述（夜市小吃、便當、湯麵、複合料理、飲料、家常菜） |
| Baselines | (B1) 通用 LLM 直接回答熱量；(B2) MyFitnessPal 類手動搜尋流程；(B3) DietSense 去除追問（ablation） |
| Accuracy | 熱量 MAE / MAPE、±10% 與 ±20% 達標率、巨量營養素誤差 |
| Uncertainty | 區間涵蓋率（參考值落在 [lo, hi] 比例）、區間寬度 |
| Efficiency | 每筆記錄時間、點擊/輸入次數 |
| Safety | 高風險語句攔截率、誤攔率 |
| User Study | 20 人 × 7 天；SUS、記錄完成率、中斷率 |
| Expected（非結果） | ±20% 內 ≥ 75%（且比通用 LLM 基準高 ≥ 15 個百分點）、記錄時間 −50%、SUS ≥ 70 |

## 12. Decision Log

| 日期 | 決策 | 理由 |
|---|---|---|
| 2026-11-12 | 準確率目標由「±20% 內 > 85%」下修為「≥ 75% 且比通用 LLM 高 ≥ 15 個百分點」 | 臺灣實測 GPT-4 僅 58–61% 落在 ±20% 內 [A2]；以相對基準設目標較可辯護 |
| 2026-11-12 | 記錄時間目標由 −60% 改為 −50% | 原 60% 無文獻依據，採較保守值 |
| 2026-11-12 | 極低熱量門檻依性別區分：女性 1,200、男性 1,500 kcal/日，連續 3 日觸發 | 常見臨床實務的低熱量飲食下限；單日偏低不觸發，以減少誤攔 |
| 2026-11-12 | 追問上限 2 輪 | 對應 Deep Research Trade-off 2，以 ablation 驗證 |
| 2026-11-12 | 本階段只做規劃產出（OpenSpec proposal/specs/design/tasks），不實作程式 | Proposal 階段的評分重點是設計與可行性 |
| 2026-11-12 | 技術棧維持 FastAPI + SQLite + 向量庫 + Streamlit | 可離線、部署簡單，適合課程 Demo |

## 13. Limitations

- 純文字描述無法觀測真實份量，誤差下限受限於使用者描述品質。
- 臺式料理食譜差異大（店家、油量），參考熱量本身有不確定性。
- 500 筆資料集與 20 人研究規模小，僅能作可行性驗證。
- 本系統不提供醫療診斷；特殊族群一律轉介專業人員。
