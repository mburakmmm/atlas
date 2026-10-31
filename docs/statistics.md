# Summary Statistics & Numerical Stability

Atlas provides high-precision, numerically stable statistical estimators for single columns and grouped partitions.

## 1. Formulas & Estimators

### Mean
$$\bar{x} = \frac{1}{n} \sum_{i=1}^n x_i$$
Non-missing numeric observations are summed and divided by count $n$. If $n = 0$, raises `AggregationError`.

### Median
- For an odd number of observations: the middle value of the sorted series.
- For an even number of observations: the arithmetic mean of the two middle values.

### Variance & Standard Deviation (Welford's Algorithm)
Atlas deliberately avoids the naive one-pass calculation $E[x^2] - E[x]^2$, which is notoriously susceptible to catastrophic cancellation when computing variance on large floating-point numbers.

Instead, Atlas implements **Welford's algorithm**:
- Updates running mean and sum of squared differences $M_2$ online in $O(N)$ time with minimal rounding error.
- **Sample Variance** ($s^2$):
  $$s^2 = \frac{M_2}{n - 1} \quad (n \ge 2)$$
- **Population Variance** ($\sigma^2$):
  $$\sigma^2 = \frac{M_2}{n} \quad (n \ge 1)$$
- **Standard Deviation**:
  $$s = \sqrt{s^2}, \quad \sigma = \sqrt{\sigma^2}$$
  Computed using `nox.mathx.sqrt()`.
