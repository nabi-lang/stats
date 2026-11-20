# Changelog

All notable changes to this package. Versions follow semver, independent of the Nabi toolchain.

## 0.1.0 (unreleased)

First release.

- Descriptive statistics along any axis, NaN-aware forms (`stats::nan`),
  quantiles by the nine Hyndman-Fan definitions and the lower, higher,
  nearest and midpoint rules.
- Covariance and Pearson, Spearman and Kendall correlation; histograms with
  the common bin rules.
- 19 distributions (continuous, discrete and multivariate) with densities,
  quantiles, sampling and maximum-likelihood fits.
- Hypothesis tests: t-tests, chi-square, Fisher exact, Kolmogorov-Smirnov,
  Shapiro-Wilk, Mann-Whitney U, Wilcoxon, one-way ANOVA, correlation tests.
- Regression: `ols`, `wls`, `logistic`, `polyfit`; the bootstrap.
- The `ops` feature (default): `~`, `±` and `pct`.
- Pure Nabi, every target including wasm. Needs Nabi 0.1 or later; depends
  on `linalg` and `special`.
