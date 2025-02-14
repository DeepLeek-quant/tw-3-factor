# TW3FactorModel

A quantitative trading strategy based on the three factors from 台股研究室 by Prof. 葉怡成, with progressive improvements.

## Base Strategy

The base strategy uses three fundamental factors:
1. ROE (Return on Equity)
2. 3-Month Momentum
3. Price-to-Book Ratio (inverted)

These factors, originally proposed by Prof. 葉怡成 from 台股研究室, are combined to create a composite score for stock selection, rebalancing quarterly.
```python
roe = bts.get_factor('roe')
mtm = bts.get_factor('mtm_3m')
pbr = bts.get_factor('股價淨值比', asc=False)
factor = bts.get_factor(roe+mtm+pbr)

tw3factor = bts.backtesting(factor>=0.99, rebalance='Q')
tw3factor.summary
```
- Annual Return: 24.07%
- Total Return: 7525.18%
- Max Drawdown: -72.55%
- Annual Volatility: 28.59%
- Sharpe Ratio: 0.84
- Calmar Ratio: 0.33
- Beta: 0.59

```python
tw3factor._plot_equity_curve()
```

<img src="images/org_strategy.png" width="800"/>



## Improving stock pool, rebalancing dates

We made two key adjustments to improve the strategy:

1. Universe Selection:
   - Added profitability filter (positive net income)
   - Added technical filter (price above 60-day moving average)
   
   These filters help focus on financially healthier companies with positive price momentum.

2. Rebalance Timing:
   - Changed from calendar quarters ('Q') to quarterly financial report release dates ('QR')
   - This ensures we trade on actual financial data after it becomes publicly available
   - Helps capture the market reaction to new fundamental information

The results show improved performance with higher returns (33.25% vs 22.79% annually) while maintaining similar risk metrics. However, the large maximum drawdown remains a concern that will be addressed in the next version.

```python
universe = (bts.get_data('稅後淨利')>=0) &\
    (bts.get_data('收盤價')>=bts.get_data('收盤價').rolling(60).mean())

roe = bts.get_factor('roe', universe=universe)
mtm = bts.get_factor('mtm_3m', universe=universe)
pbr = bts.get_factor('股價淨值比', asc=False, universe=universe)
factor = bts.get_factor(roe+mtm+pbr)

adjusted_tw3factor = bts.backtesting(factor>=0.99, rebalance='QR')
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

<img src="images/improved_strategy.png" width="800"/>

## Converging drawdown with MAE/ MFE analysis
as can be seen from the MFE distribution, the maximum loss for loss trades at Q3 is 15.16%, which means that the loss exceed 15.16% is mostly cannot recover.

```python
adjusted_tw3factor._plot_maemfe()
```

<img src="images/improved_maemfe.png" width="800"/>

so we can add a stop loss at 15% to improve the strategy

```python
stop_lossed_tw3factor = bts.backtesting(factor>=0.99, rebalance='QR', stop_loss=0.15, stop_at='intraday')
stop_lossed_tw3factor._plot_maemfe()
```

<img src="images/stop_lossed_maemfe.png" width="800"/>

As can be seen from the GMFE/ MAE, the stop loss is effective, the maximum loss is reduced to ~20%, which is originally ~40%.

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


the MDD is reduced to 47%, which is originally 61%, with the slight sacrifice of annual return from 33.25% to 27.24%.

```python
stop_lossed_tw3factor._plot_equity_curve()
```

<img src="images/stop_lossed_strategy.png" width="800"/>
