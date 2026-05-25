# 每日營養需求計算器：核心演算法與科學依據 (Methodology & Proof)

本文件旨在詳細說明「每日營養需求計算器」背後的數學模型、科學文獻來源以及底層假設，以確保計算結果的嚴謹性與可重複驗證性。

---

## 一、 基礎代謝率 (BMR) 公式證明

本系統採用 **Mifflin-St Jeor (1990)** 公式，而非傳統的 Harris-Benedict (1919) 公式。

* **原因**：根據多項現代臨床研究（如 *American Dietetic Association* 評估），Mifflin-St Jeor 公式在現代肥胖與非肥胖人口中，預估靜止代謝率（RMR/BMR）的準確度最高（誤差通常在 $\pm 10\%$ 以內），比舊版 Harris-Benedict 公式更不易高估熱量。
* **數學公式**：
  $$BMR = 10 \times W + 6.25 \times H - 5 \times A + s$$
  其中 $W$ 為體重 (kg)，$H$ 為身高 (cm)，$A$ 為年齡 (歲)，性別修正項 $s$ 在男性為 $+5$，女性為 $-161$。

---

## 二、 24 小時時間與熱量拆解模型 (TDEE)

傳統 TDEE 計算直接將 BMR 乘以一個粗略的活動係數（如 1.375），這會將「運動熱量」與「日常靜態熱量」混淆。本系統為了提高精確度，將一天 24 小時嚴格拆解為三部分：

1. **睡眠消耗 ($E_{sleep}$)**：
   * **原理**：睡眠時的代謝率略低於清醒時的 BMR。文獻指出約為 BMR 的 $0.95$ 倍。
   * **公式**：$E_{sleep} = 0.95 \times BMR \times \frac{8}{24}$（固定假設每日基礎睡眠為 8 小時）。
2. **運動消耗 ($E_{ex}$)**：
   * **原理**：依據**美國運動醫學會 (ACSM)** 的代謝當量 (MET) 定義。$1\,\text{MET}$ 代表每公斤體重每小時消耗 $1\,\text{kcal}$（基於氧氣消耗量 $3.5\,\text{mL/kg/min}$）。
   * **公式**：$E_{ex} = \sum (\text{MET}_i \times W \times \text{Hours}_i)$
3. **靜態清醒消耗 ($E_{rest}$)**：
   * **原理**：扣除睡眠 8 小時與運動時間 ($T_{ex}$) 後的剩餘時間，視為基礎清醒狀態（基礎代謝率 1.0 倍）。
   * **公式**：$E_{rest} = BMR \times \frac{16 - T_{ex}}{24}$

### 【總結 TDEE 公式】
將上述三者相加，並化簡後得到系統底層的運算式：
$$TDEE = BMR \times \frac{23.6 - T_{ex}}{24} + \sum (\text{MET}_i \times W \times \text{Hours}_i)$$

*此模型成功避免了「邊騎腳踏車邊計算靜態 BMR」的雙重計費（Double Counting）誤差。*

---

## 三、 動態蛋白質係數 ($f_{prot}$) 設計邏輯

本系統不使用固定的蛋白質攝取比例，而是根據**每日總運動時數 ($T_{ex}$)** 進行動態階梯式配置：

| 總運動時數 ($T_{ex}$) | 蛋白質係數 ($f_{prot}$) | 科學依據與目標 |
| :--- | :--- | :--- |
| $T_{ex} = 0\,\text{hr}$ | $0.8\,\text{g/kg}$ | **WHO 建議最低維持量**：一般健康成人維持氮平衡之底線。 |
| $0 < T_{ex} \le 1\,\text{hr}$ | $1.2\,\text{g/kg}$ | **ACSM / ISSN 輕度運動指引**：適合規律運動、耐力訓練初學者。 |
| $1 < T_{ex} \le 2\,\text{hr}$ | $1.5\,\text{g/kg}$ | **高強度/阻力訓練指引**：滿足肌肉修復與合成的進階需求。 |
| $T_{ex} > 2\,\text{hr}$ | $2.0\,\text{g/kg}$ | **運動員/極端訓練標準**：防止高消耗下的肌肉流失。 |

*系統實作安全閥：`C_prot = Math.max((0.15 * TDEE) / 4, W * f_prot)`，確保熱量極高時，蛋白質攝取不會低於總熱量的 15%。*

---

## 四、 微量元素 (DRI) 分級依據

本系統的維生素與礦物質推薦量，完全對照衛福部國民健康署之**「膳食營養素參考攝取量」(DRIs)** 與 WHO 指引，並透過 JavaScript 邏輯進行精準的分流（Branching）：
* **年齡分流**：例如鈣質（Calcium）在 $A \ge 71$ 歲時，為了預防骨質疏鬆，系統會自動將建議量由 $1000\,\text{mg}$ 調升至 $1200\,\text{mg}$。
* **性別分流**：例如鐵質（Iron），生育年齡女性（$A < 51$ 歲）因生理期流失，系統會將需求量設為 $18\,\text{mg}$，而男性與停經後女性則自動調降為 $8\,\text{mg}$。

---

## 📚 主要參考文獻

1. Mifflin, M. D., et al. (1990). "A new predictive equation for resting energy expenditure in healthy individuals." *The American Journal of Clinical Nutrition*.
2. American College of Sports Medicine (ACSM). (2018). *ACSM's Guidelines for Exercise Testing and Prescription*.
3. International Society of Sports Nutrition (ISSN). (2017). "International Society of Sports Nutrition Position Stand: protein and exercise."
4. 衛生福利部國民健康署．《膳食營養素參考攝取量 (DRIs)》。