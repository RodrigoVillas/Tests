# ON Moirai Distribution
#### Linked notebook: MOIRAI_Time_And_Shock_Analyzes.ipynb

### Scope and data

This analysis examines the one- to ten-business-day return forecasts for the Nikkei 225 Index (`^N225`). Daily closing prices were downloaded with `yfinance` for approximately two years, from 1 September 2024 through 1 September 2026 (the end date is exclusive in the download call). Daily log returns were computed as `log1p(pct_change(Close))`, equivalent to the log ratio of consecutive closing prices.

### Forecast model and return treatment

Forecasts were generated with the pretrained `Salesforce/moirai-1.0-R-base` time-series foundation model. The full observed return history was provided as context; the forecast horizon was 10 business days, and 2,000 simulated forecast paths were requested.

The sampled forecasts include extraordinarily large values. Converting cumulative log returns to simple returns with `expm1` can therefore produce extreme or numerically non-finite values. This makes moment-based summaries, particularly variance and higher moments, unreliable or impossible to interpret. For simulated-path plots and the shock analysis, the notebook neutralizes any individual forecast log return outside [-100, 100] by replacing it with zero before cumulating. The extreme observation is removed rather than clipped to the threshold. Nevetheless, the moments of the forcasted distribution are still unstable even after applying this contraint. 

Moment plots from the uncapped forecasts should therefore be treated cautiously. However, the lower-tail VaR and ES estimates are surprisingly steady risk summaries in this analysis: the notebook produces finite lower-tail values despite the unstable extreme forecasts. This robustness is empirical for these forecasts, not a general guarantee against all numerical problems.

### Horizon behavior of VaR and ES

VaR is calculated at the 1%, 2.5%, 5%, and 7.5% return quantiles; ES is calculated as the mean return at or below the 2.5%, 5%, 7.5%, and 10% quantiles. These are expressed in return convention, not positive-loss convention: lower, more negative values indicate worse outcomes. In particular, the notebook's ES is a lower-tail conditional mean, often called expected shortfall.

Across forecast horizons from one to ten business days, the unadjusted tail measures evolve with horizon, while their magnitudes divided by $\sqrt{T}$ are comparatively stable. The notebook plots this normalization for all VaR and ES levels. This is consistent with an approximately square-root-of-time scaling of downside-return risk over the short horizons examined. It should be read as an empirical pattern in these model simulations, rather than a universal scaling law or a formal statistical test.

### Shock design and motivation

To test sensitivity to the most recent market observation, the analysis replaces the final historical daily log return with a sequence of hypothetical shock inputs from -0.50 to +0.20 log-return units (including zero and intermediate values). These are not literal simple-return percentages. Each shocked history is passed through the same predictor, generating 2,000 simulated paths per scenario. The resulting forecasts receive the same ±100 log-return neutralization before cumulative simple returns and summary metrics are computed.

This intervention isolates how the model's forecast distribution responds when its latest input return is changed, while keeping the earlier history and model fixed. It is a conditional sensitivity experiment, not a claim that the shocks are likely or that the model has been retrained under each scenario.

### One-day shock sensitivity

The one-day VaR and ES shock plots show an asymmetric response. For positive shocks, the lower-tail measures move toward a plateau: beyond positive shock levels, additional increases have little further effect and the risk metrics approach a roughly constant level. For negative shocks, both VaR and ES change approximately linearly with shock size over the tested range, becoming more adverse as the imposed return becomes more negative. The negative-shock response is therefore more directly transmitted into the forecast downside tail, whereas positive shocks exhibit diminishing sensitivity.

VaR and ES display the same broad directional pattern, with ES representing the more extreme conditional tail average. The charts support these qualitative comparisons; the notebook does not report fitted slopes, uncertainty intervals, or formal tests of linearity or saturation.

### Effect of shocks across forecast time

The horizon plots compare the time profiles for different shock inputs, including VaR and ES normalized by $\sqrt{T}$. The scenarios change the level of the downside-risk measures, but the broad horizon structure remains similar: the normalized profiles are comparatively stable over the one- to ten-day window. Negative shocks produce a stronger adverse shift, while positive-shock effects level off in keeping with the one-day saturation pattern. Thus, the shock mainly changes the severity of the forecast tail rather than eliminating the observed square-root-of-time pattern.

### Interpretation and limitations

These results describe simulated forecasts conditional on one price series, one historical window, one pretrained model, and a particular outlier-neutralization rule. The very large forecast samples limit interpretation of means, variances, skewness, and kurtosis. The cap is also asymmetric in its practical effect because out-of-range observations are set to zero. Finally, the observed $\sqrt{T}$ stability and the asymmetric shock response are visual, qualitative findings; no confidence intervals, calibration backtest, or hypothesis tests are provided. The tail-risk patterns should therefore be treated as exploratory model sensitivities, not validated market-risk forecasts.
