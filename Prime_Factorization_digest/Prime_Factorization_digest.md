# 論文摘要：《Prime Factorization Using Partially Constrained Multiple Quantum Annealing With Analytical and Pattern-Based Variable Reduction》

**來源**：IEEE Transactions on Computers, Vol. 75, No. 4, April 2026  
**作者**：Xinyi Guo, Geguang Miao, Shinichi Nishizawa, Shinji Kimura, Takashi Sato  
**分析日期**：2026-05-30 ｜ 原始論文：11355721.pdf

---

## 一、整體內容

### 研究問題

（第 1 頁，Abstract）大型半素數（semiprime, N = p × q）的質因數分解是一個困難問題，也是公鑰密碼系統的安全基礎。現有以量子退火（Quantum Annealing, QA）為基礎的分解方法受限於 QUBO 變數數量過多，只能處理最多 21-bit 的半素數；本文旨在設計新的 QUBO 建模技術，大幅減少變數數量，從而使 QA 方法能夠分解更大的半素數。

### 研究背景

（第 1 頁，Introduction）質因數分解（Prime Factorization, PF）是公鑰密碼學的安全基礎——RSA 加密系統依賴於對大型半素數進行因數分解的計算困難性。Shor's algorithm 雖然能在量子電腦上以多項式時間解決 PF，但受限於 NISQ（Noisy Intermediate-Scale Quantum）硬體的短相干時間、高錯誤率，目前僅能分解 N=21 等小數字。另一途徑是絕熱量子計算（Adiabatic Quantum Computing, AQC）和量子退火（QA），其 QUBO 公式化方式對 NISQ 硬體更友善，但現有方法的 QUBO 變數數量過多，最大僅能處理約 20-bit 的半素數（D-Wave 2000Q 的極限）。

### 核心貢獻

1. （第 3 頁，Section IV-A）提出 **MulBlocks**（多區塊多次退火）：將乘法表分成子問題 Block I 和 Block III 分別退火，再組合候選解，打破單次 QUBO 的規模限制。
2. （第 4 頁，Section IV-B）提出 **VarLSB**（LSB 端分析變數縮減）：利用低有效位元（LSB）的乘法模式分析，直接推導變數等式，消除冗餘的 QUBO 變數與高階項。
3. （第 4 頁，Section IV-C）提出 **VarMSB**（MSB 端分析變數縮減）：透過與 √N 比較，分析高有效位元（MSB）的數值範圍約束，演算法式地固定部分因子位元。
4. （第 5 頁，Section IV-D）提出 **SpePattern**（特殊模式）：針對奇位元寬且具有長 MSB 側連續零序列的半素數，利用結構性特徵直接確定因子位元，大幅縮減 Block III 的變數數量。
5. （第 5 頁，Section IV-E）提出 **OBB**（最優區塊平衡）：動態調整 Block I 和 Block III 的列寬，最小化兩塊的項數不平衡度，提升退火求解效果。

### 研究範疇

（第 6 頁，Section V-A）本文聚焦於半素數（semiprime）的質因數分解問題，假設兩個質因子 p 和 q 的位元寬度相等，且最低有效位元固定為 1（奇數質因子）。實驗在 Fixstars' Amplify AE（GPU-based 退火模擬機）上進行，部分 SOTA 方法在同平台上重新實現以確保公平比較。

---

## 二、研究方法

### 方法概述

（第 3 頁，Section IV 引言）本文提出五種互補的 QUBO 建模技術，核心思想是：透過分塊退火（MulBlocks）降低單次問題規模，再結合 LSB/MSB 端的分析推導（VarLSB、VarMSB）與結構性模式利用（SpePattern），以及動態負載平衡（OBB）來最小化 QUBO 變數數量，使量子退火機能在現有硬體限制下解決更大的半素數分解。

### 架構說明

（第 3 頁，Table II；第 4 頁，Section IV-A）乘法表（multiplication table）被分成三個 Block，忽略變數最多且最複雜的中間 Block II，僅對 Block I（LSB 側欄）和 Block III（MSB 側欄）建立各自的 QUBO 並獨立退火。退火結果（候選因子對）最後合併，通過窮舉驗證找到正確的質因子。

**整體流程**：輸入半素數 N → 建立乘法表 → MulBlocks 切分 Block I / Block III → 各自應用 VarLSB（Block I）/ VarMSB + SpePattern（Block III）→ OBB 調整寬度 → 各塊獨立退火 → 組合候選 → 驗證輸出 p, q。

### 關鍵模組

| 模組名稱 | 輸入 | 處理 | 輸出 | 位置 |
|---------|------|------|------|------|
| MulBlocks | 半素數 N、乘法表 | 分成 Block I（LSB 側）、Block III（MSB 側），各建 QUBO 獨立退火 | 候選因子對集合 | 第 3 頁，Section IV-A |
| VarLSB | 乘法表 LSB 端欄位（2¹, 2²... 欄） | 分析低位元 4/8 種模式，推導因子位元等式，消除高階 QUBO 項 | 變數縮減後的 Block I QUBO | 第 4 頁，Section IV-B |
| VarMSB | 因子的 MSB 端位元 | 透過 Algorithm 1 逐位比較 √N，固定可確定的高位元為 0 或 1 | 固定的 MSB 位元值、縮減的 Block III QUBO | 第 4 頁，Section IV-C |
| SpePattern | 奇位寬且 MSB 端有連續零的半素數 | 識別結構性零模式，直接確定對應因子位元 | 大量固定的因子位元（大幅減少 Block III 變數） | 第 5 頁，Section IV-D |
| OBB | Block I 和 III 的初始欄寬 w₁, w₃ | Algorithm 2 遍歷 x，計算使 \|T₁(w₁')-T₃(w₃')\| 最小的最優偏移量 x* | 平衡後的欄寬 w₁*, w₃*，使兩塊項數相近 | 第 5 頁，Section IV-E |

### 核心公式

**QUBO 目標函數**（第 2 頁，Section II-C）：

$$f = \sum_i Q_{i,i} x_i + \sum_{i<j} Q_{i,j} x_i x_j$$

- $Q_{i,i}$：線性係數（linear coefficient）
- $Q_{i,j}$：二次係數（quadratic coefficient）
- $x_i, x_j \in \{0,1\}$：二元決策變數（binary decision variables）
- 直覺解釋：QA 求解器找使 f 最小化的變數賦值，即找乘法等式約束同時成立的因子位元組合。

**高階項消除（懲罰轉換）**（第 2 頁，Section II-C）：

$$x_1 x_2 x_3 = y x_3 + P \cdot (x_1 x_2 - 2x_1 y - 2x_2 y + 3y)$$

- $y = x_1 x_2$：輔助變數（auxiliary variable），編碼高階積
- $P$：懲罰係數（penalty coefficient），強制 y = x₁x₂
- 直覺解釋：三次項透過引入輔助變數 y 和懲罰項轉換為二次 QUBO 形式；P 過大或過小都會劣化搜尋效果。

**MulBlocks 組合目標函數**（第 4 頁，Section IV-A）：

$$f_1 = \sigma_1 \times f_{11} + \sigma_2 \times f_{12}$$

- $f_{11}, f_{12}$：Block I 切成兩個子塊（pieces）的各自平方誤差目標函數
- $\sigma_1, \sigma_2$：各子塊的調整係數（tunable piece coefficients）
- 直覺解釋：係數 σ 調整兩子塊對 Hamiltonian 最大能量的相對貢獻，影響退火時的搜尋偏向。

### 訓練策略

（第 6 頁，Section V-A；第 9 頁，Section V-E）

- **退火機器**：Fixstars' Amplify AE（GPU-based 模擬量子退火），支援 10 萬個以上的全連接 QUBO 變數
- **退火時間**：每塊固定 10 秒；MulBlocks 總共退火兩次（Block I 和 Block III 各一次），等效總時間 20 秒，與 SOTA 方法比較基準一致
- **懲罰方案**：PA-1（三次項 P=2，二次項 P²=4）作為主要方案；PA-2 懲罰值為 PA-1 的兩倍，用於極端測試
- **重複次數**：每個實例重複 100 次，取最佳成功率（success rate）
- **驗證方式**：候選因子對窮舉相乘，與目標半素數 N 比對；所有係數（piece coefficients）σ 預設設為 1

---

## 三、主要結果

### 實驗設定

（第 6 頁，Section V-A；第 7 頁，Table V）

- **資料集**：8–26 bit 半素數（與 SOTA 比較）；26–57 bit 半素數（提出方法擴展測試）；特殊結構的 61, 101, 389, 415, 655, 1185, 2049-bit 半素數（Table VII 極端測試）
- **評估指標**：成功率（SR, Success Rate）= 100 次試驗中正確分解的比例；QUBO 變數數（#Var, #VarI, #VarIII）；候選解數量（#CandI, #CandIII）；驗證時間（VerTime, 秒）

### 主要比較結果

（第 7 頁，Table V）

| 方法 | 最大穩定可解位元 | 19-bit 變數數 | 20-bit 成功率 | 26-bit 成功率 | 位置 |
|-----|----------------|-------------|-------------|-------------|------|
| Jiang et al. (SOTA) | ~20 bit | 95 | 75% | 1% | 第 7 頁，Table V |
| Peng et al. (SOTA) | ~20 bit | 90 | 94% | 4% | 第 7 頁，Table V |
| MulBlocks | 40 bit | 19 | 100% | 100% | 第 7 頁，Table V |
| MulVar | 47 bit | 12 | 100% | 100% | 第 7 頁，Table V |
| MulVarSpeObb | 2049 bit（特殊） | 11 | 100% | 100% | 第 8 頁，Table VII |

補充：19-bit 半素數 376,289（=571×659）中，SOTA 需約 90-95 個變數；MulBlocks 僅需 19 個，MulVar 降至 12 個，MulVarSpeObb 降至 11 個（第 7 頁，Table V）。

### 消融實驗（Ablation Study）

（第 9 頁，Section V-E）

- **懲罰值（Penalty Value）影響**：在 51-bit 半素數（1,495,000,453,129,013）上，懲罰值掃描範圍 10⁰ 到 10¹⁰。Block I 最優區間約 [23,400, 37,000]，候選數從 87 激增至 24,300；Block III 最優區間 [6,170, 10,000]，在 6,160 時僅 87 候選，6,170 時達 11,851。整體成功率因此從 19% 提升至 81%。
- **退火時間（Annealing Time）影響**：退火時間 10–100 秒（10 秒遞增）測試於 51-bit 半素數。成功率與候選數量均隨退火時間增加而提升並趨於飽和（第 9 頁，Figure 3–4）。
- **子塊數（Number of Pieces）影響**：2-piece vs. 3-piece 在 46, 47, 51, 53, 58-bit 半素數上比較（第 10 頁，Table VIII）。3-piece 成功率均低於 2-piece，原因是三塊化後丟失了鄰欄變數間積的關係，導致資訊損失並增加輔助變數。
- **係數（Coefficients for Pieces）影響**：在 40-bit 半素數（723,320,469,463）上測試 σ₁/σ₂ 從 1 到 100,000 的組合，整體成功率波動約 40%–60%，σ₁ 較大時略優；當比值超過 ~77,842 或低於 1/3,149 時，成功率驟降至零（第 10 頁，Section V-E-4）。

### 主要發現

（第 11 頁，Conclusion）本文所提的 MulBlocks 和 MulVar 在變數縮減和可分解半素數位元寬度方面均顯著超越 SOTA 方法。MulBlocks 達到 40-bit 半素數 100% 成功率；MulVar 延伸至 47-bit；MulVarSpeObb 在奇位寬且有長 MSB 零序列的特殊結構條件下，成功分解最大 2049-bit 的半素數，展示出超越現有學術基準的工程可擴展性。

### 侷限性（作者自述）

1. （第 10 頁，Section V-F）懲罰值的選取缺乏系統化方法，必須針對每個半素數實例經驗性調整，是 QA-based 方法標準化的主要障礙。
2. （第 10 頁，Section V-F）即使退火時間較長，若懲罰參數設定不當，系統仍可能收斂至次優解，退火時間無法完全補償不良的 QUBO 設計。
3. （第 11 頁，Conclusion）改善 QUBO 設計、權重校正與參數最佳化以解決更大一般性實例留待未來工作。

---

## 四、英文關鍵字

### Author Keywords（作者提供，第 1 頁，Abstract）

`Quantum annealing`, `QUBO`, `prime factorization`, `combinatorial optimization problem`, `multiplication table`, `addition of partial products`

### Method Keywords（方法相關）

- `multiple annealing` — (第 3 頁，Section IV-A)
- `partial block decomposition` — (第 3 頁，Section IV-A)
- `analytical variable reduction` — (第 4 頁，Section IV-B/C)
- `pattern-based variable reduction` — (第 5 頁，Section IV-D)
- `optimal block balancing` — (第 5 頁，Section IV-E)
- `QUBO converter` — (第 2 頁，Introduction)

### Topic Keywords（領域相關）

- `quantum annealing` — (第 1 頁，Abstract)
- `prime factorization` — (第 1 頁，Introduction)
- `QUBO` — (第 2 頁，Section II-C)
- `public key cryptography` — (第 1 頁，Introduction)
- `NISQ` — (第 1 頁，Introduction)
- `adiabatic quantum computing` — (第 2 頁，Section II-A)

---

## 五、值得改進之處

### 作者已承認的不足

1. （第 10 頁，Section V-F）懲罰值選取缺乏系統化框架，每個問題實例需經驗性調整，限制了方法的通用性與自動化程度。
2. （第 10 頁，Section V-F）對 54-bit 以上的大型問題，搜尋空間急劇縮小，精確懲罰值調整的難度更高，目前僅測試少數懲罰值，不足以覆蓋最優範圍。
3. （第 11 頁，Conclusion）實驗平台限於 Amplify AE（GPU 模擬），未來應透過改進 QUBO 設計、權重校正、參數最佳化來拓展至更大一般性實例。

### 獨立分析觀察

| 維度 | 潛在問題 | 改進建議 |
|------|---------|---------|
| 方法論（Methodology） | SpePattern 僅適用於奇位元寬且具長 MSB 零序列的半素數，覆蓋範圍十分有限；2049-bit 成功不代表對一般 RSA 模數的突破 | 探索更通用的結構性特徵識別方法，或結合機器學習輔助識別可利用的位元模式；研究偶位元寬的情形 |
| 實驗設計（Experimental Design） | 所有實驗均在 Amplify AE（GPU 模擬）上進行，缺乏在真實 D-Wave Advantage 量子退火機上的驗證；兩者硬體連接圖拓撲不同，QUBO 嵌入方式有差異 | 在 D-Wave Advantage（Pegasus 15-way）上進行相同實驗，評估真實量子硬體圖拓撲的影響 |
| 可重現性（Reproducibility） | 未提及開源代碼；PyQUBO 版本、Amplify AE 具體配置、懲罰值搜尋策略未完整文件化 | 公開 QUBO 轉換器代碼（含所有超參數設定），以及標準測試半素數集合，方便他人重現 |
| 泛化能力（Generalizability） | 方法假設兩個質因子位元寬度相等，不適用於實際 RSA 中位元寬差異大的情形；且僅在 Amplify AE 上驗證，跨平台泛化性未知 | 探索不等寬質因子分解的 QUBO 建模，並在多種退火平台上驗證方法的可移植性 |
| 理論支撐（Theoretical Foundation） | 成功率與懲罰值、候選數量之間的關係缺乏理論分析；懲罰值最優區間的存在性與寬度僅為經驗觀察，無理論解釋 | 建立 QUBO Hamiltonian 能量景觀（energy landscape）的理論模型，分析懲罰值對能量 gap（正確解與最近錯誤解之間的能量差）的影響 |

---

*由 Paper Digest Skill 自動生成 ｜ 所有頁碼引用基於原始 PDF 物理頁碼（第 1–13 頁）*
