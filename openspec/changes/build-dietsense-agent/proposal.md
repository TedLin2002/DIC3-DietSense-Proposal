# Proposal

## Why

傳統飲食記錄 App 需手動搜尋、秤重與拆解複合料理，負擔高而難以持續；通用 LLM 雖能理解自然語言，直接回答熱量卻會幻覺與虛構數值，且對臺式料理覆蓋不足。DietSense 以「LLM 負責協調、領域工具負責計算、規則負責安全」的 Domain AI Agent，讓使用者以日常繁中描述飲食即可獲得可追溯、含不確定性、且安全的熱量與體重管理建議（詳見專案根目錄 `DESIGN.md` §1）。

## What Changes

- 新增自然語言飲食記錄：文字/語音 → 食物實體（名稱、份量、烹調、餐次）。
- 新增主動澄清迴圈：信心不足或份量模糊時追問，最多 2 輪。
- 新增多來源營養檢索：USDA FoodData Central、臺灣食品營養成分資料庫、自建臺式料理向量庫，所有數值附來源。
- 新增確定性營養分析引擎：熱量/巨量營養素區間、Mifflin–St Jeor BMR/TDEE、攝取缺口。
- 新增情境感知個人化：個人檔案、時段、7 日趨勢、替代建議與提醒。
- 新增安全守門：極低熱量、極端節食語句、飲食失調風險、特殊族群轉介。
- 新增評估框架：500 筆臺式餐點資料集、baseline 與 ablation、使用者研究。

## Capabilities

### New Capabilities

- `meal-logging`: 自然語言（文字/語音）飲食描述解析與每日飲食日誌的寫入、修改、查詢。
- `clarification-dialog`: 在食物辨識或份量不確定時，主動產生追問並整合回覆的對話行為。
- `nutrition-retrieval`: 跨營養資料庫與臺式料理知識庫的候選檢索、排名、信心分數與來源揭露。
- `nutrition-analytics`: 確定性的熱量、巨量營養素、不確定性區間、BMR/TDEE 與能量缺口計算。
- `personalized-guidance`: 依個人檔案、歷史日誌與情境產生趨勢摘要、替代建議與提醒。
- `safety-guardrails`: 對不安全的攝取模式、語句與族群進行偵測、建議降級與專業轉介。
- `evaluation-harness`: 以標註資料集、baseline 與指標重現性地評估系統準確度、效率與安全性。

### Modified Capabilities

（無，本專案首次建立規格。）

## Impact

- 新程式碼：Agent 協調層、工具服務、資料匯入腳本、前端聊天介面、評估腳本。
- 外部依賴：LLM API（需支援 function calling）、語音轉文字服務、USDA FDC API Key、TFDA 開放資料。
- 資料：使用者健康資料（體重、飲食）屬敏感個資，需本地儲存與最小化蒐集。
- 文件：`DESIGN.md` 為系統設計來源，與本變更的 `design.md` 保持同步。
