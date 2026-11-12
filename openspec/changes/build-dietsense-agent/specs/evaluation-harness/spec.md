## Purpose

以標註資料集、可比較的 baseline 與明確指標，重現性地評估 DietSense 的準確度、不確定性、效率與安全性，並區分預期目標與實測結果。

## ADDED Requirements

### Requirement: 標註資料集

評估 SHALL 使用 500 筆臺式餐點描述資料集，涵蓋夜市小吃、便當、湯麵、複合料理、飲料與家常菜，每筆 MUST 附參考熱量、營養素、計算來源與至少一名人工複核紀錄。

#### Scenario: 資料集完整性檢查

- **WHEN** 執行資料集檢查
- **THEN** 報告各類別筆數，並列出缺少參考值或來源的資料

### Requirement: Baseline 與 Ablation

評估 SHALL 至少比較：通用 LLM 直接回答、手動搜尋記錄流程、DietSense 完整版、DietSense 移除追問版。

#### Scenario: 執行比較

- **WHEN** 對同一資料集執行評估
- **THEN** 產出各系統在相同指標下的比較表

### Requirement: 評估指標

評估 SHALL 報告熱量 MAE、MAPE、±10% 與 ±20% 達標率、巨量營養素誤差、區間涵蓋率與寬度、記錄時間、安全攔截率與誤攔率。

#### Scenario: 產出報告

- **WHEN** 評估完成
- **THEN** 輸出含上述指標的報告檔，並記錄所用模型版本、資料版本與隨機種子

### Requirement: 目標與結果分離

所有文件 MUST 將預期目標（如 ±20% 內 ≥ 75%）標示為 Expected，僅能以實測報告的數值作為 Results。

#### Scenario: 撰寫結果

- **WHEN** 更新 DESIGN.md 的 Results 章節
- **THEN** 數值必須引用實際評估報告檔
