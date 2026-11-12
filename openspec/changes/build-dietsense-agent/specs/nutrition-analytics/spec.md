## Purpose

以確定性、可重現的公式計算熱量、巨量營養素、不確定性區間與個人能量需求，確保所有數值不由語言模型自由生成。

## ADDED Requirements

### Requirement: 數值只來自確定性計算

所有熱量、營養素與能量需求數值 MUST 由確定性計算產生；語言模型產生的文字中出現的數值 MUST 與計算結果一致，否則系統 SHALL 以計算結果取代。

#### Scenario: 相同輸入相同輸出

- **WHEN** 以相同品項與克數重複計算
- **THEN** 系統回傳完全相同的數值

### Requirement: 熱量與營養素計算

系統 SHALL 以「克數 × 每 100 g 營養值 ÷ 100」計算每個食物的熱量、蛋白質、碳水化合物、脂肪與膳食纖維，複合料理為各食材加總。

#### Scenario: 單一食物計算

- **WHEN** 品項每 100 g 為 150 kcal，份量為 200 g
- **THEN** 熱量為 300 kcal

### Requirement: 不確定性區間

系統 SHALL 對每筆紀錄輸出低、中、高三個熱量值，由份量的低、中、高克數推算；當日總和 SHALL 同時呈現區間。

#### Scenario: 顯示區間

- **WHEN** 牛肉麵份量為 450–650 g，中位 550 g
- **THEN** 系統以對應的三個熱量值呈現，例如「約 X kcal（範圍 L–H）」

### Requirement: BMR 與 TDEE

系統 SHALL 以 Mifflin–St Jeor 公式依體重、身高、年齡、生理性別計算 BMR，乘以使用者選擇的活動係數得 TDEE，並 MUST 標示其為估計值。

#### Scenario: 計算 TDEE

- **WHEN** 使用者為 30 歲女性、60 kg、160 cm、活動係數 1.375
- **THEN** BMR = 10×60 + 6.25×160 − 5×30 − 161 = 1,289 kcal，TDEE ≈ 1,772 kcal

### Requirement: 能量缺口

系統 SHALL 計算當日攝取（中位值）與 TDEE 的差值，並提供 7 日平均差值。

#### Scenario: 每日缺口

- **WHEN** 使用者當日攝取中位值 1,500 kcal、TDEE 1,772 kcal
- **THEN** 系統回報缺口約 272 kcal
