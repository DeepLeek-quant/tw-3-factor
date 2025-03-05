# 台灣三因子策略

基於葉怡成教授在台股研究室提出的三因子量化交易策略，並進行改良。

## 基礎策略

基礎策略使用三個基本的因子：
1. ROE（股東權益報酬率）
2. 3個月動能
3. 股價淨值比（反向）

這些因子最初由葉怡成教授在台股研究室提出，將其結合以創建綜合評分用於選股，每季度再平衡。

```python
import quantdev.backtest as bts
roe = get_factor('roe')
mtm = get_factor('mtm_3m')
pbr = get_factor('股價淨值比', asc=False)
factor = get_factor(roe+mtm+pbr)

tw3factor = backtesting(factor>=0.99, rebalance='Q')
tw3factor.summary
```
- Annual Return: 22.79%
- Total Return: 6086.99%
- Max Drawdown: -58.24%
- Annual Volatility: 19.66%
- Sharpe Ratio: 1.2
- Calmar Ratio: 0.39
- Beta: 0.57

```python
tw3factor._plot_equity_curve()
```

<img src="figure/org_strategy.png" width="800"/>



## 改進股票池和再平衡日期

我們進行了兩個關鍵調整以改進策略：

1. 選股範圍：
   - 增加獲利能力篩選（正淨利）
   - 增加技術面篩選（價格高於60日移動平均線）
   
   這些篩選有助於聚焦於財務更健康且具有正向價格動能的公司。

2. 再平衡時機：
   - 從季度（'Q'）改為季度財報發布日期（'QR'）
   - 確保我們使用最新的財務數據進行交易，而不是過時的資訊

結果顯示在保持類似風險指標的同時，績效有所提升（年化報酬率從22.79%提升至33.25%）。然而，較大的最大回撤仍然是一個需要在下一版本中解決的問題。

```python
universe = (get_data('稅後淨利')>=0) &\
    (get_data('收盤價')>=get_data('收盤價').rolling(60).mean())

roe = get_factor('roe', universe=universe)
mtm = get_factor('mtm_3m', universe=universe)
pbr = get_factor('股價淨值比', asc=False, universe=universe)
factor = get_factor(roe+mtm+pbr)

adjusted_tw3factor = backtesting(factor>=0.99, rebalance='QR')
adjusted_tw3factor.summary
```

- Annual return: 33.25%
- Total return: 33221.64%
- Max drawdown: -61.43%
- Annual volatility: 29.03%
- Sharpe ratio: 1.1
- Calmar ratio: 0.54
- Beta: 0.6

```python
adjusted_tw3factor._plot_equity_curve()
```

<img src="figure/improved_strategy.png" width="800"/>

## 透過MAE/MFE分析收斂回撤
從MFE分布可以看出，虧損交易在75%百分位(Q3)的最大損失為15.16%，這意味著超過15.16%的損失大多是無法回復的。

```python
adjusted_tw3factor._plot_maemfe()
```

<img src="figure/improved_maemfe.png" width="800"/>

因此，我們可以在15%虧損時增加停損來改善策略。

```python
stop_lossed_tw3factor = backtesting(factor>=0.99, rebalance='QR', stop_loss=0.15, stop_at='intraday')
stop_lossed_tw3factor._plot_maemfe()
```

<img src="figure/stop_lossed_maemfe.png" width="800"/>

從GMFE/MAE可以看出，停損是有效的，最大損失從原來的40%減少到20%左右。

```python
stop_lossed_tw3factor.summary
```

- Annual return: 27.24%
- Total return: 12994.90%
- Max drawdown: -46.91%
- Annual volatility: 24.30%
- Sharpe ratio: 1.1
- Calmar ratio: 0.58
- Beta: 0.37

最大回撤從原本的61%降低至47%，年化報酬率則從33.25%略微下降至27.24%。

```python
stop_lossed_tw3factor._plot_equity_curve()
```

<img src="figure/stop_lossed_strategy.png" width="800"/>


## 結論
| | 原始策略 | 改良策略 | 停損策略 |
|----------|---------------|--------------|--------------|
| Annual return | 22.79% | 33.25% | 27.24% |
| Total return | 6086.99% | 33221.64% | 12994.90% |
| Max drawdown | -58.24% | -61.43% | -46.91% |
| Annual volatility | 19.66% | 29.03% | 24.30% |
| Sharpe ratio | 1.20 | 1.10 | 1.10 |
| Calmar ratio | 0.39 | 0.54 | 0.58 |
| Beta | 0.57 | 0.60 | 0.37 |

本研究透過三個階段優化台股動能策略:
1. 原始策略使用簡單的價值/獲利/動能三因子，雖然有不錯的報酬但波動較大
2. 改良策略只納入獲利能力為正且收盤價在季線之上的股票，並使用財報發布日期再平衡，年化報酬率提升至33.25%，但最大回撤仍然較大
3. 最後加入15%停損機制，成功將最大回撤從61%降至47%，雖然年化報酬略為下降，但整體風險調整後報酬(卡瑪比率)反而提升

整體而言，最終版本的策略在風險與報酬上都有明顯改善。

