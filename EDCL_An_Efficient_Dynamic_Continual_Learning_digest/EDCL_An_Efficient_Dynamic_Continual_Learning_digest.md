# 論文摘要：《EDCL: An Efficient Dynamic Continual Learning Framework for IoT Systems》

> 分析日期：2026-05-29 ｜ 原始論文：11389162.pdf  
> 發表於：IEEE Transactions on Computers, Vol. 75, No. 5, May 2026

---

## 一、整體內容

### 研究問題

（第 1 頁，第 11–31 行）現有的持續學習（continual learning, CL）方法在解決模型遺忘（catastrophic forgetting）問題時，普遍假設訓練過程可獨佔資源，而忽略了邊緣裝置（edge device）上多應用程式同時執行所導致的 GPU 記憶體競爭問題。當低優先級的 CL 訓練程式與高優先級的推論程式（inference program）共存時，固定資源分配會使推論失敗率高達 50% 以上，嚴重影響用戶體驗。本文旨在設計一個能動態調整訓練資源、同時維持模型精度的高效 CL 框架。

### 研究背景

（第 2 頁，第 65–97 行）隨著 IoT 設備數量增長，邊緣裝置上的智慧應用（如智慧家居感測、工業數據分析、醫療穿戴監控）需要在隱私和頻寬限制下於裝置端進行模型訓練，且同時必須持續執行推論任務。傳統 CL 方法（包括正則化方法與回放方法）在分配固定資源的情況下，無法因應高優先級推論程式動態變化的記憶體需求，導致資源衝突頻繁。

（第 3 頁，第 125–160 行）現有邊緣端排程研究（如 Ekya [21]、Zeng et al. [22]）僅考慮運算資源競爭，忽略 GPU 記憶體競爭這一硬性約束；且主要聚焦於 domain-incremental 設定，對 class-incremental 和 task-incremental 情境的支援有限。

### 核心貢獻

（第 4 頁，第 218–224 行）

1. 透過離線分析（offline profiling）建立批次大小（batch size）、訓練時間與記憶體消耗之間的量化關係三元組（triplet）配置檔，作為線上搜尋依據。
2. 提出動態持續學習模組（Dynamic Continual Learning, DCL），基於高優先級推論程式的即時記憶體占用動態調整訓練策略，防止相互干擾。
3. 提出自適應分層緩衝區交換模組（Adaptive Hierarchical Buffer Swap, AHBS），利用熵（entropy）導引的重要性評分在記憶體緩衝區與磁碟緩衝區間動態交換樣本，降低歷史知識遺忘。

### 研究範疇

（第 3–4 頁，第 162–190 行）本文聚焦於邊緣伺服器和具有中等運算能力的邊緣裝置（如 NVIDIA Jetson 系列、工業邊緣閘道）上的 GPU 記憶體資源競爭問題，適用於 class-incremental 與 task-incremental 兩種 CL 設定。CPU、I/O、網路頻寬等軟性約束不在主要優化範疇內。

---

## 二、研究方法

### 方法概述

（第 7 頁，第 421–446 行）EDCL 是一個可與現有基於回放（replay-based）的 CL 方法無縫整合的動態框架，核心思想是：透過離線建立批次策略配置檔，在線上動態監控高優先級推論程式的資源消耗，進而即時選擇滿足記憶體限制的最優批次策略，同時用熵導引的分層緩衝區交換來緩解遺忘。

### 架構說明

（第 8 頁，第 490–492 行，Figure 5）本圖呈現了 EDCL 的兩階段框架：
- **離線階段（Offline Stage）**：對不同批次大小下的 CL 方法進行資源分析，產生訓練時間、記憶體用量的配置文件。
- **線上階段（Online Stage）**：包含 DCL 模組（資源監控 + 排程器）與 AHBS 模組（熵評分 + 自適應交換比率），前者確保推論不被干擾，後者維持回放緩衝區的樣本多樣性與重要性。

### 關鍵模組

| 模組名稱 | 輸入 | 處理 | 輸出 | 位置 |
|---------|------|------|------|------|
| 離線分析（Offline Profiling） | 不同批次大小的訓練過程 | 記錄每個批次大小對應的訓練時間與 GPU 記憶體用量，構建三元組集合 T | 優化後的配置文件 T' | 第 8 頁，第 448–522 行 |
| 動態 CL 模組（DCL） | 高優先級推論程式的記憶體占用序列 S | 計算可用記憶體上限 mavailable，從 T' 搜尋訓練時間最短且記憶體滿足限制的批次策略 | 動態調整後的批次大小 b | 第 9 頁，第 524–583 行 |
| 自適應分層緩衝區交換（AHBS） | 記憶體緩衝區 Bm 中的樣本及模型輸出 | 計算每個樣本的熵重要性評分，動態調整交換比率，以非同步程序從磁碟緩衝區 Bd 換入高重要性樣本 | 更新後的記憶體緩衝區 Bm | 第 10–11 頁，第 604–735 行 |

### 核心公式

**公式 (5)（第 9 頁，第 551–560 行）**：可用記憶體上限計算

$$m_{\text{available}} = (M - m_{\text{max}}) \times \alpha$$

- $M$：裝置的 GPU 記憶體容量
- $m_{\text{max}}$：時間窗口內推論程式的最大 GPU 記憶體占用量
- $\alpha$：收縮係數（empirically set to 0.8），用於為高優先級任務保留安全裕量，防止測量誤差導致衝突
- 直覺解釋：EDCL 不直接使用理論上限，而是預留 20% 安全邊距，確保即使資源測量有偏差也不會干擾推論。

**公式 (6)（第 10 頁，第 593–599 行）**：樣本熵計算

$$H(f(x_i; \theta)) = -\sum_{j=1}^{c} (f_j(x_i; \theta) \cdot \log(f_j(x_i; \theta)))$$

- $f_j(x_i; \theta)$：模型對樣本 $x_i$ 在第 $j$ 個類別的預測機率
- $c$：類別總數
- 直覺解釋：熵越高表示模型對該樣本分類越不確定，需要優先保留在緩衝區中以維持決策邊界。

**公式 (7)（第 11 頁，第 665–672 行）**：自適應交換比率計算

$$r = r_{\min} + \frac{(r_{\max} - r_{\min})}{1 + e^{-\lambda(\bar{H}_W - \bar{H}_{X_b})}}$$

- $r_{\min}, r_{\max}$：交換比率的最小值和最大值（設為 0.2 和 0.8）
- $\bar{H}_W$：滑動窗口 W 中的平均熵
- $\bar{H}_{X_b}$：當前批次 $X_b$ 的平均熵
- $\lambda$：溫度係數（設為 2）
- 直覺解釋：當前批次熵高於歷史平均時（資料較難），降低交換比率保留重要樣本；反之則提高交換率引入新樣本。

**公式 (8)（第 11 頁，第 685–694 行）**：重要性評分

$$\sigma(x_i) = \begin{cases} \frac{1}{2u} \cdot H(f(x_i; \theta)), & f(x_i; \theta) = y_i \quad \text{（分類正確）} \\ \frac{1}{2u} \cdot [2u - H(f(x_i; \theta))], & f(x_i; \theta) \neq y_i \quad \text{（分類錯誤）} \end{cases}$$

- $u$：最大熵值
- 直覺解釋：分類正確的樣本中熵越高越重要（高不確定性）；分類錯誤的樣本中熵越低越重要（模型有把握地錯了）。

### 訓練策略

（第 13 頁，第 987–992 行）所有方法（包含 EDCL 和基準線）均使用 SGD 優化器（optimizer），學習率從 {0.01, 0.03, 0.1} 中 grid search 選擇。EDCL 超參數設定：滑動窗口大小 $k_w = 3$，最小/最大交換比率 $r_{\min} = 0.2, r_{\max} = 0.8$，溫度係數 $\lambda = 2$。CIFAR-10 訓練 50 個 epoch，CIFAR-100 和 Tiny-ImageNet 訓練 100 個 epoch。

---

## 三、主要結果

### 實驗設定

（第 13 頁，第 751–1004 行）

**資料集（dataset）**：CIFAR-10（5 個任務，各含 2 類）、CIFAR-100（10 個任務，各含 20 類）、Tiny-ImageNet（10 個任務，各含 20 類）

**評估指標（evaluation metric）**：
- 模型精度（Classification Accuracy）：Class-IL 與 Task-IL 兩種設定
- 訓練時間（Clock Time, 秒）
- 高優先級推論程式的**成功率（Success Rate, SR）**：推論程式執行不失敗的比例
- 高優先級推論程式的**吞吐量完成率（Completion Rate, CR）**：達成目標吞吐量的比例

**硬體環境**：Intel Xeon Platinum 8474C CPU + NVIDIA RTX-4090 GPU（24 GB 記憶體）；邊緣裝置驗證使用 NVIDIA Jetson XAVIER NX（8 GB 統一記憶體）

### 主要比較結果

（第 14 頁，第 1006–1074 行，Table I）

| 方法 | CIFAR-10 Class-IL | CIFAR-10 Task-IL | 訓練時間(s) | Tiny-ImageNet Class-IL | 位置 |
|-----|-----|-----|-----|-----|------|
| ER-ACE_4GB | 62.23% | 91.66% | 8,880 | 31.02% | 第 14 頁，Table I |
| ER-ACE_16GB | 64.91% | 91.89% | 33,120 | 35.62% | 第 14 頁，Table I |
| ER-ACE_Ekya | 64.28% | 92.22% | 9,540 | 31.15% | 第 14 頁，Table I |
| **ER-ACE_EDCL** | **82.20%** | **96.70%** | **6,538** | **49.23%** | 第 14 頁，Table I |
| DER++_16GB | 65.88% | 91.75% | 77,700 | 23.32% | 第 14 頁，Table I |
| DER++_Ekya | 66.63% | 91.28% | 22,020 | 29.83% | 第 14 頁，Table I |
| **DER++_EDCL** | **77.62%** | **96.34%** | **15,433** | **50.10%** | 第 14 頁，Table I |

- ER-ACE_EDCL 在 CIFAR-10 Class-IL 上比最佳基準線高出 **25.13%**，比 Ekya 平均高出 **22.19%**（第 14 頁，第 1017–1021 行）
- ER-ACE_EDCL 訓練時間比 4 GB 版本減少 **26.37%**，比 16 GB 版本減少 **80.26%**（第 15 頁，第 1047–1051 行）
- 在 Tiny-ImageNet 上，EDCL 的 Class-IL 精度平均比 Ekya 高出 **63%**（第 15 頁，第 1055 行）

**Jetson XAVIER NX 邊緣裝置驗證**（第 18 頁，第 1302–1316 行）：
- EDCL 訓練時間比 Ekya 少 **42.92%**，比 Static 訓練少 **42%**
- EDCL 尾延遲（tail latency）比 Ekya 低 **67.09%**（僅比 Static 訓練高 3.77%）
- EDCL 能耗比 Ekya 少 **8.1%**，比 Static 訓練少 **0.22%**

**推論程式影響**（第 17 頁，第 1214–1225 行）：ER-ACE_EDCL 和 DER++_EDCL 均達到 **SR = 100%**（零推論失敗）；EDCL 的 SR 平均比 Ekya 高 7.49%，CR 平均比 Ekya 高 30.11%。

### 消融實驗（Ablation Study）

（第 17 頁，第 1227–1258 行，Figures 15–17）

- **AHBS 模組消融**：Base（無交換）→ EIS（熵導引選擇，固定交換比率）→ EIS+ASR（完整 AHBS）。EIS 和 EIS+ASR 均顯著優於 Base；EIS+ASR 在所有交換預算下均優於 EIS，驗證了自適應比率調整的有效性。
- **DCL 模組消融**（第 17 頁，第 1241–1251 行）：移除 DCL 後（EDCL-w/o DCL），SR 和 CR 顯著下降，且隨高記憶體消耗程式比例增加急劇惡化，而 EDCL 保持穩定高性能。
- **計算開銷**（第 17 頁，第 1253–1258 行）：AHBS 中交換比率和排序計算僅占訓練時間的 **2% 和 1.7%**，開銷可忽略不計。
- **端到端延遲**（第 17 頁，第 1259–1291 行）：首次分析需約 10.2 秒（含 8 秒資源監控窗口）；後續快取命中情況下延遲降至約 **1 秒**。

### 主要發現

（第 19 頁，第 1424–1433 行）EDCL 在 IoT 邊緣場景下成功實現了三個目標的同時優化：(1) 確保高優先級推論程式不中斷（SR 達 100%）；(2) 顯著提升 CL 模型精度（比靜態方法最高提升 25.13%）；(3) 大幅縮短訓練時間（比固定 16 GB 方案縮短 80.26%）。

### 侷限性（作者自述）

（第 3 頁，第 161–190 行）作者明確指出 EDCL 目前聚焦於 GPU 記憶體這一硬性約束，對 CPU、I/O、網路頻寬等軟性約束的優化不在本文範疇內。

（第 19 頁，第 1431–1433 行）未來工作計畫將 EDCL 延伸至多租戶（multi-tenant）設定和線上 CL 場景（online CL scenarios）。

（第 19 頁，第 1400–1422 行）目前的 EDCL 設計尚未整合線上類別發現（online category discovery）功能，需搭配 OOD 檢測器和教師模型才能實現端到端的開放世界 CL。

---

## 四、英文關鍵字

### Author Keywords（作者提供，第 1 頁，第 32–33 行）

`Continual learning`, `Internet of Things`, `machine learning`, `edge computing`

### Method Keywords（方法相關）

- `replay-based continual learning` — (第 3 頁，第 113–116 行)
- `offline profiling` — (第 8 頁，第 448 行)
- `dynamic batch size selection` — (第 9 頁，第 524–570 行)
- `entropy-based importance scoring` — (第 10 頁，第 679–715 行)
- `hierarchical memory buffer swap` — (第 10 頁，第 604–617 行)
- `adaptive swap ratio` — (第 11 頁，第 635–672 行)

### Topic Keywords（領域相關）

- `catastrophic forgetting` — (第 2 頁，第 94 行)
- `edge intelligence` — (第 2 頁，第 73 行)
- `resource-constrained learning` — (第 4 頁，第 162–166 行)
- `class-incremental learning` — (第 5 頁，第 271–277 行)
- `GPU memory management` — (第 4 頁，第 183–190 行)
- `inference-training co-scheduling` — (第 9 頁，第 524–530 行)

---

## 五、值得改進之處

### 作者已承認的不足

1. （第 4 頁，第 183–190 行）作者明確指出本文聚焦於 GPU 記憶體競爭，未處理 CPU、I/O、網路頻寬等軟性資源的競爭，這限制了框架在更廣泛邊緣場景下的適用性。
2. （第 19 頁，第 1431–1433 行）目前僅驗證了單台設備場景，尚未延伸至多租戶（multi-tenant）設定，無法處理多個 CL 應用共存時的資源協調問題。
3. （第 19 頁，第 1400–1422 行）EDCL 尚不支援線上類別發現，無法自動識別新任務的出現，仍依賴預先定義的任務切換通知。
4. （第 17 頁，第 1263–1267 行）首次處理新推論工作負載時存在約 10.2 秒的高延遲（其中 8 秒為資源監控窗口），對於頻繁且不規律切換推論任務的場景可能仍有影響。

### 獨立分析觀察

| 維度 | 潛在問題 | 改進建議 |
|------|---------|---------|
| 方法論（Methodology） | 安全係數 α=0.8 為 empirically 設定，缺乏自適應機制；不同邊緣裝置的記憶體碎片化程度不同，固定 α 可能過於保守或不足 | 設計基於歷史資源測量誤差分佈自適應調整 α 的機制 |
| 實驗設計（Experimental Design） | 僅比較了 ER、ER-ACE、DER++ 三種 replay 方法，未包含近年的強力 CL 基準（如 DualPrompt、CODA-Prompt）；推論任務全部基於 ResNet18，缺乏多樣性 | 加入更多 state-of-the-art CL 方法和更多樣的推論任務（目標偵測、語音辨識等） |
| 可重現性（Reproducibility） | 論文未提及代碼開源，超參數設定僅有最終值，缺少敏感性分析（如 α、λ、kw 對結果的影響） | 開源代碼並補充超參數敏感性分析實驗 |
| 泛化能力（Generalizability） | 所有實驗均在影像分類任務上；邊緣裝置僅驗證了 Jetson XAVIER NX 一款；設備僅有單 GPU，未考慮多 GPU 或 NPU 場景 | 在 NLP、時序數據等其他模態以及不同硬體平台（如 Raspberry Pi、Intel NCS）上驗證 |
| 理論支撐（Theoretical Foundation） | 熵導引的重要性評分方案（公式 8）雖直覺合理，但缺乏理論保證（如遺忘上界的理論分析）；AHBS 對最終精度提升的貢獻也缺少收斂性分析 | 從 PAC 學習或 online learning 理論角度分析 AHBS 的遺忘上界 |

---

*由 Paper Digest Skill 自動生成 ｜ 所有頁碼行號引用基於 pdftotext 提取的文字行號*
