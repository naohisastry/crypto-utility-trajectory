# データソースと補正仕様（DATA_SOURCES）
> **Data Lineage, API Specifications, and Data Cleaning Notes**

本分析の透明性と再現性を担保するため、採用しているデータソース、取得APIエンドポイント、および欠損値・外れ値の補正処理方針を開示します。

---

## 1. データソース一覧とAPI仕様

| データ系列 | 提供元 | エンドポイント・取得手法 | 取得頻度・単位 |
| :--- | :--- | :--- | :--- |
| **日次価格・出来高** | **Yahoo Finance API** | `v8/finance/chart/{SYMBOL}?interval=1d&range=5y` | 日次確定値（USD） |
| **月間プロトコル手数料** | **DefiLlama Fees API** | `https://api.llama.fi/summary/fees/{protocol}` | 月間累計値（USD） |
| **ステーブルコイン流通量** | **DefiLlama Stablecoins**| `https://stablecoins.llama.fi/stablecoincharts/all` | 月末スナップショット（USD） |
| **月末時価総額** | **CoinCodex Historical** | `https://coincodex.com/api/coincodex/get_coin_history/` | 月末確定実績値（USD） |

---

## 2. 銘柄別データ取得マッピングと公式リファレンス

1. **ETH (Ethereum)**
   - 手数料: DefiLlama Ethereum Fees (`https://defillama.com/fees/ethereum`)
   - L2合算: Arbitrum, Optimism, Base 各手数料系列を合算
2. **SOL (Solana)**
   - 手数料: DefiLlama Solana Fees (`https://defillama.com/fees/solana`)
3. **USDT (Tether USD)**
   - 実需: DefiLlama Stablecoin 流通供給量 (`https://defillama.com/stablecoins`)
4. **USDC (USD Coin)**
   - 実需: DefiLlama Stablecoin 流通供給量 (`https://defillama.com/stablecoins`)
5. **UNI (Uniswap)**
   - 手数料: DefiLlama Uniswap Fees (`https://defillama.com/fees/uniswap`)
6. **AAVE (Aave)**
   - 手数料: DefiLlama Aave Fees (`https://defillama.com/fees/aave`)
7. **LINK (Chainlink)**
   - 手数料: DefiLlama Chainlink Fees (`https://defillama.com/fees/chainlink`)
8. **DOGE (Dogecoin)**
   - 実需: プロトコル手数料未集計（床値 $100/月 処理）
9. **SHIB (Shiba Inu)**
   - 実需: プロトコル手数料未集計（床値 $100/月 処理）
10. **LUNA (Terra Classic)**
    - 手数料: 破綻前はDefiLlama実績値、破綻後（2022-06以降）は残存床値
11. **FTT (FTX Token)**
    - 手数料: 破綻前は取引所推計値、破綻後（2022-12以降）は残存床値
12. **AXS (Axie Infinity)**
    - 手数料: DefiLlama Axie Infinity Fees (`https://defillama.com/fees/axie-infinity`)

---

## 3. 欠損・床値（Floor Value）の処理方針

学術的・分析的な整合性を維持するため、以下の補正処理を実施しています：

1. **対数軸表示のための床値設定**:
   - プロトコル手数料がゼロまたは未計測のミーム銘柄（DOGE, SHIB）について、対数グラフ上で計算不能（$\log(0) = -\infty$）となることを防ぐため、下限床値（Floor Value = \$100 / 月）を設定。
2. **破綻銘柄の残存処理（LUNA, FTT）**:
   - 破綻後のトークン残骸取引によって生じる見かけの取引高や時価総額の歪みを排除するため、破綻確定月以降の時価総額・実需を最低水準に固定し、第4象限に退場した状態を正確に表現。
3. **初期データ欠損の適正処理（LINK, AAVE）**:
   - DefiLlamaにおける計測開始前の期間（例: LINKの2021〜2022年初期等）については、恣意的なゼロ補間を避け、`missing` フラグを付与して軌跡の分断を防止。
4. **観測終点の確定月基準**:
   - 途中の未確定月（月半ばの数値）の混入を避けるため、確定月（2026年8月）を最新観測時点として採用。
