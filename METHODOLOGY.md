# 分析手法と数理モデル仕様（METHODOLOGY）
> **Mathematical Formulations & Quantitative Models for Crypto Utility Trajectory**

本ドキュメントでは、ダッシュボード内で使用されている各金融指標、4象限境界アルゴリズム、およびストレステスト判定ロジックの数理的定式化を記載します。

---

## 1. コア指標の定義と算出式

### ① 実需指標（Utility Metric: $Y_t$）
プロトコルや暗号資産の種類に応じ、経済活動の裏付けとなるオンチェーン数値を定義します。
- **L1 / DeFi / P2E 銘柄（ETH, SOL, UNI, AAVE, LINK, AXS）**:
  $$Y_t = \text{月間手数料収益（Monthly Protocol Fees in USD）}$$
  ※ DefiLlama `fees` エンドポイントより取得した月間累計値。
- **ステーブルコイン（USDT, USDC）**:
  $$Y_t = \text{月末流通供給量（Circulating Supply in USD）}$$
  ※ 決済流動性レールとしての規模を示すため、DefiLlama `stablecoins` エンドポイントより取得。
- **ミームコイン（DOGE, SHIB）**:
  $$Y_t = \text{床値（Floor Value: \$100 / 月）}$$
  ※ プロトコル手数料が発生しないため、対数軸表示（Log-scale）のための下限床値として処理。

### ② 実需密度（Utility Density: $D_t$）
時価総額（Market Cap: $M_t$）に対して、どれだけ密度の高い実需手数料を生み出しているかを示す資本効率指標です。
$$D_t = \frac{Y_t}{M_t}$$
- 実需密度が高いほど、投機的なバブルに依存せず、ファンダメンタルズに裏打ちされた経済活動が存在することを示します。
- ステーブルコイン（USDT, USDC）は価格が一定（ペッグ）であり流通量そのものが時価総額に近似するため、$D_t \approx 1.0$（100%）に規準化されます。

### ③ ボラティリティ（Volatility: $\sigma_t$）
該当月における日次収益率（Daily Log Returns）の標準偏差を算出し、年率換算した指標です。
$$R_{d} = \ln\left(\frac{P_d}{P_{d-1}}\right)$$
$$\sigma_{\text{daily}} = \sqrt{\frac{1}{N-1}\sum_{d=1}^{N}(R_d - \bar{R})^2}$$
$$\sigma_t = \sigma_{\text{daily}} \times \sqrt{365}$$
※ 価格変動リスクの尺度としてX軸に配置されます。

### ④ 月間回転率（Monthly Turnover: $T_t$）
該当月の実取引高の合計を月末時価総額で除した指標です。
$$T_t = \frac{\sum_{d=1}^N \text{Daily Volume}_d}{M_t}$$
※ 投機熱や流動性の活発度を測る代替X軸として利用されます。

---

## 2. 動的4象限境界アルゴリズム（市場中央値）

本分析では、特定の閾値を主観的に固定するのではなく、各月（$t$）における観測銘柄群の **「市場中央値（Median）」** を境界線として動的に算出します。

$$X_{\text{bound}, t} = \text{Median}(\{X_{i, t}\}_{i=1}^K)$$
$$Y_{\text{bound}, t} = \text{Median}(\{Y_{i, t}\}_{i=1}^K)$$

これにより、強気相場（ブル相場）や弱気相場（ベア相場）といったマクロ環境の変化に応じて境界が自律的に伸縮し、その時点の市場全体における相対的な立ち位置（社会インフラ / 成長過渡期 / 投機過熱 / 淘汰）を客観的に判定します。

---

## 3. ストレステスト定量評価モデル

過去の3大市場ショック（$t_{\text{event}}$）を基準とし、前後各6ヶ月（合計12ヶ月）の平均値を比較します。

$$\bar{Y}_{\text{pre}} = \frac{1}{6}\sum_{k=1}^6 Y_{t_{\text{event}}-k}, \quad \bar{Y}_{\text{post}} = \frac{1}{6}\sum_{k=1}^6 Y_{t_{\text{event}}+k}$$

### ① 評価メトリクス
1. **実需維持率（Demand Retention Ratio）**:
   $$R_{\text{demand}} = \frac{\bar{Y}_{\text{post}}}{\bar{Y}_{\text{pre}}}$$
2. **密度維持率（Density Retention Ratio）**:
   $$R_{\text{density}} = \frac{\bar{D}_{\text{post}}}{\bar{D}_{\text{pre}}}$$
3. **ボラティリティ変化率（Volatility Change Ratio）**:
   $$R_{\text{vol}} = \frac{\bar{\sigma}_{\text{post}}}{\bar{\sigma}_{\text{pre}}}$$

### ② 判定アルゴリズム
- **社会インフラ定着**:
  - 実需維持率 $\ge 70\%$ かつ 密度維持率 $\ge 70\%$
  - もしくは、実需維持率 $\ge 40\%$ かつ 密度維持率 $\ge 100\%$（時価総額の縮小以上に利用が粘り強く残存した場合）
- **中立・過渡期**:
  - 実需維持率が $30\% \sim 70\%$ の範囲で推移し、完全な崩壊には至っていない状態。
- **淘汰**:
  - 実需維持率 $< 10\%$、あるいは破綻イベントにより時価総額・実需が不可逆的に喪失した場合（LUNA, FTT等）。
- **実需ゼロ(投機)**:
  - 手数料収益が存在しないミームコイン等（DOGE, SHIB）。
- **判定不能**:
  - 該当期間にデータ系列が存在しない、または欠損している場合（初期のLINK等）。

---

## 4. レイヤー2（L2）合算ロジック

Ethereum（ETH）においては、EIP-4844等のスケーリング技術によりトランザクションがL2に移行した実態を捉えるため、L1単体系列に加えて以下の主要L2ロールアップ手数料を合算した「incl_l2」バリアントを算出しています。

$$Y_{\text{ETH+L2}, t} = Y_{\text{ETH, L1}, t} + Y_{\text{Arbitrum}, t} + Y_{\text{Optimism}, t} + Y_{\text{Base}, t}$$

これにより、L1手数料単体の低下が「エコシステムの衰退」ではなく「低コスト化による大衆インフラ化」であることを分離して分析可能としています。
