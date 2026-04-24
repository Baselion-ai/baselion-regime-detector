# baselion-regime-detector

> **Lightweight, pip-installable market-regime classifier: LATERAL, STRONG_TREND, VOLATILE.**

A small, focused package for tagging each bar of an OHLCV series with one of three labels:

* `LATERAL` — mean-reverting, low volatility
* `STRONG_TREND` — directional, momentum regime
* `VOLATILE` — high-variance, no clear direction

Used internally in the [FOX OMEGA Engine](https://baselion.ai) to gate which sub-model's signal is live. Released open-source because labelled regime data is useful far beyond our setup.

## Why three labels, not five?

Because "momentum vs mean-reversion vs volatile" is where most strategy-selection gains come from empirically. More labels means tighter buckets means less data per bucket means noisier estimates. Three is the sweet spot we keep landing on.

## Install

```bash
pip install baselion-regime-detector
```

## 20-second example

```python
from baselion_regime import RegimeDetector
import pandas as pd

prices = pd.read_parquet("BTCUSDT_1h.parquet")["close"]

det = RegimeDetector(method="gmm", n_states=3)
det.fit(prices)

labels = det.predict(prices)          # Series of LATERAL / STRONG_TREND / VOLATILE
probs = det.predict_proba(prices)      # DataFrame with per-state probabilities
```

## Methods supported

| Method | When to use |
|--------|-------------|
| `gmm` | Default. Fast, interpretable, works on returns + realised-vol features. |
| `hmm` | When you need Markov transitions instead of IID assignments. |
| `rule_based` | Simple ATR + ADX heuristic. Useful as a sanity baseline. |

## Benchmarks

Fit on BTC/ETH/SOL 1-hour bars 2020-01-01 → 2026-03-01:

| Pair | GMM regime purity* | Rule baseline | GMM vs rule ΔPnL (SMA crossover) |
|------|-------------------:|--------------:|---------------------------------:|
| BTC  | 0.71 | 0.58 | **+18 %** |
| ETH  | 0.68 | 0.54 | **+23 %** |
| SOL  | 0.66 | 0.49 | **+14 %** |

*Regime purity = fraction of bars labelled consistently with a next-N-bar directional outcome, higher is better.

Re-run the full benchmark with `python -m baselion_regime.benchmarks`.

## Notebooks

* `notebooks/01_three_pair_benchmark.ipynb` — full 6-year run on BTC/ETH/SOL
* `notebooks/02_gmm_vs_hmm.ipynb` — when Markov transitions help *(coming soon)*

## References

* Hamilton, J. D. (1989). *A new approach to the economic analysis of nonstationary time series and the business cycle.* Econometrica. (HMM)
* Ang, A., & Bekaert, G. (2002). *Regime switches in interest rates.* Journal of Business & Economic Statistics.
* Bishop, C. M. (2006). *Pattern Recognition and Machine Learning.* Springer. (GMM chapter)

## Status

Alpha, API likely to shift. Contributions welcome, especially additional regime methods and benchmark datasets.

## License

MIT.

---

## About Baselion

Baselion is an institutional-grade quantitative trading intelligence platform, powered by the proprietary FOX OMEGA Engine. Learn more at [baselion.ai](https://baselion.ai).
