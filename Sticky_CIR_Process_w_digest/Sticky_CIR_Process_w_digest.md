# 論文摘要：《Sticky CIR Process with Potential: Invariant Measure and Exact Sampling》

**作者**：Tony Shardlow（University of Bath）
**來源**：arXiv:2605.13648v2 [math.PR]，2026 年 5 月 18 日
**分析日期**：2026-05-30

---

## 一、整體內容

### 研究問題

（第 1 頁，第 1–20 行）本文研究一維**黏性 Cox–Ingersoll–Ross（CIR）過程**（sticky CIR process），即定義在 $[0,\infty)$ 上、在原點具有黏性邊界條件（sticky boundary condition）的擴散過程。核心問題包含三層：
1. 在 CIR 參數 $\delta \in (1,2)$ 的範圍內，過程是否適定（well-posed）？
2. 其唯一不變測度的顯式形式是什麼？
3. 如何針對帶勢能函數 G 的不變測度構建精確或近似的採樣算法？

### 研究背景

（第 1–2 頁）動機源自 Cheltsov et al. (2026) 提出的**稀疏貝葉斯推斷框架**（sparse Bayesian inference）——Hadamard–Langevin 動力學。該框架寫 $\mathbf{X} = \mathbf{u} \odot \mathbf{v}$（Hadamard 積），並以 CIR 過程對 $\mathbf{u}$ 分量抽樣。原始 $\delta = 2$ 時邊界不可達，樣本永遠不等於零，無法實現真正的稀疏性（sparsity）。將 $\delta$ 降至 $(1,2)$ 後，排斥漂移項 $(\delta-1)/(\beta u)$ 減弱，過程能在有限時間內到達原點；黏性條件阻止吸收，使不變測度在原點攜帶**點質量（point mass）**，與連續密度混合，形成「**尖峰-板塊先驗（spike-and-slab prior）**」的結構。

### 核心貢獻

1. （第 1 頁）**適定性與不變測度唯一性**：為 $\delta \in (1,2)$ 的黏性 CIR 過程證明適定性（定理 3.1）及不變測度唯一性（定理 3.2），該測度為原點的點質量與帶權 Gamma 型連續密度的混合。
2. （第 1 頁）**顯式 Green 函數**：以合流超幾何函數（confluent hypergeometric functions）推導反射 CIR 求解算子的顯式 Green 函數（引理 4.1–4.3，定理 4.4）。
3. （第 1 頁）**G=0 精確採樣器**：利用 Green 函數構建零勢能情形的精確採樣算法（Algorithm 1）。
4. （第 1 頁）**帶勢能的採樣**：透過 Girsanov 變換（Girsanov change of measure）確立傾斜不變測度的存在唯一性（定理 5.1–5.2），並發展兩種算法：帶 Metropolis–Hastings 修正的精確採樣器（Algorithm 2）與帶 $O(h)$ 偏差的非調整 Langevin 算法（ULA，Algorithm 3）。

### 研究範疇

（第 1–2 頁，第 2 頁第 30–45 行）聚焦**一維**黏性 CIR 過程，$\delta \in (1,2)$，定義在 $[0,\infty)$。數值實驗使用基礎參數 $\lambda=1,\,\beta=2,\,\delta=1.5$，測試三種勢能函數（$u^2/2$、$(u-1)^2/2$、$u^3/3$）。完整 $d$ 維採樣器（$\mathbf{u}$ 與 $\mathbf{v}$ 的聯合採樣）留待未來工作。

---

## 二、研究方法

### 方法概述

（第 2–16 頁）以 **Itô–McKean 時間變換**為基礎建立黏性邊界，以**合流超幾何函數**解析求解 Green 函數，再以 **Lie–Trotter 算子分裂**（splitting）結合黏性 CIR 求解算子構建採樣方案；勢能 G 通過 Girsanov 變換引入，並以 Metropolis–Hastings 或 ULA 處理。

### 架構說明

（第 2 頁，Section 1.2）論文以四層遞進結構展開：

1. **反射 CIR**（Section 2）：建立基礎過程，$m(\{0\}) = 0$，邊界瞬時反射。
2. **黏性 CIR，$G=0$**（Section 3）：以時間變換 $u_t = \tilde{u}_{T_t}$（其中 $T_t = A_t^{-1}$，$A_t = t + \mu^{-1}\ell_t^0(\tilde{u})$）引入原點的逗留（sojourn）行為。
3. **Green 函數與精確採樣**（Section 4）：通過 Kummer 方程的解析解建立顯式 Green 函數，據此構建三成分混合採樣器（Algorithm 1）。
4. **帶勢能 G 的擴展**（Section 5）：Girsanov 變換確立不變測度，Euler 步驟 + 黏性 CIR 求解算子 = 提議核心，MH 修正（Algorithm 2）或去掉修正的 ULA（Algorithm 3）。

### 關鍵模組

| 模組名稱 | 輸入 | 處理 | 輸出 | 位置 |
|---------|------|------|------|------|
| 反射 CIR（Reflecting CIR） | 初始值 $x>0$，$\delta\in(1,2)$ | 將 SDE 化為帶權 Bessel 過程（Itô 公式） | 非負強解 $\tilde{u}_t$，邊界瞬時反射 | 第 4 頁，定理 2.5 |
| 黏性 CIR（Sticky CIR, $G=0$） | $\tilde{u}_t$ + 黏性參數 $\mu$ | Itô–McKean 時間變換：速度測度加點質量 $\mu^{-1}\delta_0$ | 弱解 $u_t$，可在邊界逗留 | 第 5 頁，定理 3.1 |
| Green 函數（Green's function） | 生成算子 $\mathcal{L}$，頻率 $\alpha$ | 求解 Kummer 合流超幾何 ODE，邊界匹配 | 解析核 $G_\alpha^{\rm sticky}(x,y)$ | 第 7–9 頁 |
| 精確採樣器（Algorithm 1） | 當前狀態 $u_k$ | 三成分混合分佈：原點 / $(0,x)$ / $(x,\infty)$ | 下一狀態 $u_{k+1}$，精確 | 第 12–13 頁 |
| MCMC 採樣器（Algorithm 2） | $u_k$，勢能 $G$ | clamped Euler 步 $\phi_h(x)$ + 黏性 CIR 提議 + MH 修正 | 精確目標 $\pi$ 的樣本 | 第 14–16 頁 |
| ULA（Algorithm 3） | $u_k$，勢能 $G$ | 同 Algorithm 2，去掉接受步 | 帶 $O(h)$ 偏差的樣本 | 第 17 頁 |

### 核心公式

**公式 (1.1)（第 1 頁）— 黏性 CIR SDE**

$$du_t = \left[\frac{\delta-1}{\beta u_t} - \lambda u_t - G'(u_t)\right]dt + \sqrt{\frac{2}{\beta}}\,dW_t$$

$$\mathbf{1}_{\{u_t=0\}}\,dt = \frac{\exp(-\beta G(0))}{\mu}\,d\ell_t^0(u)$$

- $\delta \in (1,2)$：CIR 參數，控制邊界可達性
- $\mu > 0$：黏性參數，$\mu\to 0$ 對應吸收，$\mu\to\infty$ 對應瞬時反射
- $\lambda > 0$：耗散（dissipation）參數；$\beta > 0$：逆溫度（inverse temperature）
- $\ell_t^0(u)$：擴散局部時間（diffusion local time），第二行為「逗留條件（sojourn condition）」

**公式 (定理 3.2)（第 5 頁）— 零勢能不變測度**

$$\pi_0(dx) = \frac{1}{Z}\left[\frac{1}{\mu}\delta_0(dx) + \beta x^{\delta-1}e^{-\lambda\beta x^2/2}\,dx\right],\quad Z = \frac{1}{\mu} + \int_0^\infty \beta x^{\delta-1}e^{-\lambda\beta x^2/2}\,dx$$

- 第一項：原點的點質量，比例為 $1/(\mu Z)$
- 第二項：帶權 Gamma 型密度（weighted gamma-type density）

**公式 (4.6)（第 7 頁）— Green 函數**

$$G_\alpha(x,y) = \frac{f_0(x\wedge y)\cdot U(a,b,z_{x\vee y})}{|\mathcal{W}|}$$

$$|\mathcal{W}| = \lambda\beta\,\frac{\Gamma(b)}{\Gamma(a)}\left(\frac{\lambda\beta}{2}\right)^{-b}$$

- $M(a,b,z) = {}_1F_1(a;b;z)$：第一類合流超幾何函數，在 $z=0$ 正則
- $U(a,b,z)$：第二類合流超幾何函數，在 $z\to\infty$ 衰減
- $a = \alpha/(2\lambda)$，$b = \delta/2$，$z_x = \lambda\beta x^2/2$
- $|\mathcal{W}|$：Wronskian 歸一化常數

**定理 5.2（第 14 頁）— 帶勢能的不變測度**

$$\frac{d\pi}{d\pi_0}(u) = \frac{1}{Z}e^{-\beta G(u)},\quad Z = \int_{[0,\infty)} e^{-\beta G(u)}\,\pi_0(du)$$

- Gibbsian 重加權：以 $e^{-\beta G}$ 因子同時作用於內部密度與邊界點質量

### 訓練策略

（第 12 頁，Remark 4.7 / 第 13 頁 Algorithm 1）不涉及神經網絡訓練。計算成本分析：
- **預計算**：在網格 $[0, y_{\max}]$ 上 $O(N)$ 次 $M(a,b,\cdot)$ 和 $U(a,b,\cdot)$ 評估
- **每步代價**：$O(\log N)$ CDF 反演（二元搜索）+ 當前狀態的 2 次超幾何函數評估
- 同一網格可供所有 $\mu$ 值複用；僅需更新標量 $c_\mu$ 和 $p_{\rm leave}$

---

## 三、主要結果

### 實驗設定

（第 19 頁）基礎參數：$\lambda=1$，$\beta=2$，$\delta=1.5$。

**實驗 1**：$G \in \{u^2/2,\,(u-1)^2/2,\,u^3/3\}$，$\mu \in \{0.5, 1.0, 2.0\}$，$\alpha \in \{2, 5, 10\}$，200,000 步，4 條鏈，10,000 步預熱。評估指標：邊界質量誤差、ESS/秒、內部密度。

**實驗 2**：$\mu=1$，$G \in \{0,\,u^2/2,\,(u-1)^2/2,\,2u\}$，$\alpha=5$，$h=0.2$，30,000 步（MCMC）；以及 10,000 步/α 的 ULA 偏差研究。

### 主要比較結果

| 算法 | 邊界質量誤差 | ESS/秒 | 是否精確採樣 | 位置 |
|-----|------------|--------|------------|------|
| Algorithm 2（MCMC） | ≈ Monte Carlo 雜訊，各設定無系統偏差 | 較低（多數設定） | 是 | 第 20 頁，Fig. 1–2 |
| Algorithm 3（ULA） | $O(h)$ 偏差，$\alpha$ 增大時降低 | 較高（多數設定） | 否 | 第 20 頁，Fig. 1–2 |

**理論邊界質量 $\pi(\{0\})$（$\lambda=1, \beta=2, \delta=1.5, \mu=1$）**（第 20 頁，Table 1）：

| 勢能 G | $\pi(\{0\})$ |
|-------|-------------|
| $G = 0$ | 0.449 |
| $G = u^2/2$ | 0.579 |
| $G = (u-1)^2/2$ | 0.275 |
| $G = 2u$ | 0.844 |

**MCMC 接受率（$\alpha=5$，$h=0.2$）**（第 23 頁，Fig. 4）：

| 移動類型 | $G=0$ | $G=u^2/2$ | $G=(u-1)^2/2$ | $G=2u$ |
|--------|-------|----------|--------------|-------|
| 內部→內部 | 1.00 | 0.87 | 0.91 | 0.75 |
| 內部→邊界 | 1.00 | 0.93 | 0.70 | 0.82 |
| 邊界→內部 | 1.00 | 0.96 | 0.86 | 0.77 |

### 消融實驗（Ablation Study）

本文未進行傳統消融研究。以下等價觀察替代：

（第 20–22 頁）以「移除 MH 修正步驟」為分界：Algorithm 2（保留）精確，Algorithm 3（去除）引入 $O(h)$ 偏差。偏差在 $G=2u$（$G'(0)>0$）情形最為顯著；其他勢能在測試步長範圍內偏差較小（$<0.03$），難以與 Monte Carlo 雜訊區分。

### 主要發現

（第 21 頁）黏性 CIR 過程在 $\delta \in (1,2)$ 的不變測度具有「原子＋密度」結構，滿足稀疏先驗所需。顯式 Green 函數使 $G\equiv 0$ 的精確採樣可行（Algorithm 1）。Gibbsian 重加權（定理 5.2）被 Metropolis–Hastings 採樣器精確目標（Algorithm 2），或由 ULA 以 $O(h)$ 偏差近似（Algorithm 3，定理 5.6）。

（第 20 頁）數值實驗確認理論預測：$G=(u-1)^2/2,\,\mu=1,\,\alpha=20$ 下，MCMC 給出 $\hat{\pi}(\{0\})=0.277$，ULA 給出 $0.275$，理論值 $0.275$。

### 侷限性（作者自述）

1. （第 21 頁，第 5–8 行）當前分析限於一維；完整 $d$ 維 Hadamard–Langevin 動力學需處理 $\mathbf{u}$ 與 $\mathbf{v}$ 分量的耦合，留待未來工作。
2. （第 21 頁，第 8–15 行）希望移除 Assumption 5.5（$\pi_0$-相對密度正則性）與相容條件 Eq. (5.8)；黏性邊界情形下的 Malliavin 微積分（Malliavin calculus）或半群導數估計尚未在文獻中建立。
3. （第 21 頁，第 15–20 行）Foster–Lyapunov 漂移條件已均勻建立，但均勻幾何遍歷性（geometric ergodicity uniform in $h$）所需的 minorisation 估計目前仍缺失。

---

## 四、英文關鍵字

**Author Keywords**（作者提供，第 1 頁）

`sticky diffusion`, `CIR process`, `invariant measure`, `MCMC`, `unadjusted Langevin algorithm`, `sparse Bayesian inference`

**Method Keywords**（方法相關）

- `sticky boundary condition` — (第 1 頁)
- `confluent hypergeometric functions` — (第 7 頁)
- `Metropolis-Hastings correction` — (第 14 頁)
- `Itô-McKean time change` — (第 5 頁)
- `Girsanov change of measure` — (第 13 頁)
- `Lie-Trotter operator splitting` — (第 16 頁)
- `Green's function / resolvent kernel` — (第 7 頁)

**Topic Keywords**（領域相關）

- `stochastic differential equations` — (第 1 頁)
- `Markov chain Monte Carlo` — (第 1 頁)
- `sparse Bayesian inference` — (第 1 頁)
- `diffusion processes` — (第 2 頁)
- `spike-and-slab prior` — (第 2 頁)
- `Wentzell boundary condition` — (第 6 頁)

---

## 五、值得改進之處

### 作者已承認的不足

1. （第 21 頁，第 5–8 行）分析限於一維；完整 $d$ 維採樣器需處理 $u$ 和 $v$ 分量間的耦合，尚待開發。
2. （第 21 頁，第 8–15 行）ULA 偏差分析依賴未經嚴格證明的 Assumption 5.5（$\pi_0$-相對密度的均勻有界性），作者明確指出黏性邊界情形的 Malliavin 微積分估計在文獻中尚未建立。
3. （第 21 頁，第 15–20 行）均勻幾何收斂速率所需的 minorisation 步驟仍為開放問題。

### 獨立分析觀察

| 維度 | 潛在問題 | 改進建議 |
|------|---------|---------|
| 方法論（Methodology） | Algorithm 1 精確採樣依賴對 $(α, λ, β, δ)$ 預計算固定網格；參數變動需重算整個網格。另外 $c_\mu$ 在 $\alpha \to 0$（$h \to \infty$）時數值不穩定（分母 $\to 0$）。 | 研究自適應網格策略；對 $h$ 大的情形考慮替代提議分佈。 |
| 實驗設計（Experimental Design） | 實驗固定 $\delta=1.5$，未系統測試 $\delta$ 接近邊界值（$\delta\to 1^+$ 或 $\delta\to 2^-$）的行為；也未測試多峰（multimodal）勢能。 | 擴展數值實驗至多個 $\delta$ 值；測試非凸勢能，評估 MCMC 混合速度（mixing rate）的退化情形。 |
| 可重現性（Reproducibility） | 論文未提供代碼或數據鏈接；合流超幾何函數在 $a \gg 1$（對應 $h \to 0$、$\alpha \to \infty$）時的數值精度未討論。 | 開源 Python 實現；在 Remark 4.7 中補充 `scipy.special.hyp1f1` 和 `hyperu` 在大 $a$ 時的精度警告。 |
| 泛化能力（Generalizability） | 假設 $|G'(u)| \leq C(1+u)$（線性增長），對超線性增長的勢能（如 $G(u) = u^4$）不直接適用；$d$ 維情形下乘積結構是否保留黏性屬性尚不明確。 | 研究 $G'$ 超線性增長的 Novikov 條件替代方案；分析 $d$ 維 Hadamard 積結構下各分量的解耦條件。 |
| 理論支撐（Theoretical Foundation） | Assumption 5.5 的必要性未被討論——是否存在反例使得偏差不是 $O(h)$？兼容性條件 Eq. (5.8) 的物理意義未詳述，限制了定理 5.6 的實際可用範圍。 | 補充反例或充分條件以澄清 Assumption 5.5 的可省略情形；對相容性條件給出更直觀的概率解釋。 |

---

*由 Paper Digest Skill 自動生成 ｜ 所有頁碼行號引用基於原始 PDF*
