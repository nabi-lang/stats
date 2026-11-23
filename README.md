# stats

[![CI](https://github.com/nabi-lang/stats/actions/workflows/ci.yml/badge.svg)](https://github.com/nabi-lang/stats/actions/workflows/ci.yml) [![packages.nabi-lang.org](https://img.shields.io/github/v/tag/nabi-lang/stats?label=packages.nabi-lang.org&sort=semver)](https://packages.nabi-lang.org/packages/stats) [![License: MIT OR Apache-2.0](https://img.shields.io/badge/license-MIT%20OR%20Apache--2.0-blue)](#license)

Statistics for Nabi on core's `Array` and `List`: descriptive statistics
along any axis, quantiles by all nine Hyndman-Fan definitions (and the
lower, higher, nearest and midpoint rules), correlation, histograms, 19
distributions with densities, quantiles, sampling and fits, the common
hypothesis tests, linear and logistic regression and the bootstrap. Pure
Nabi (linear algebra from `linalg`, the special functions behind the
distributions from `special`), so it runs on every target, wasm
included.

## Add it

```sh
nabi add stats
```

Needs Nabi 0.1 or later.

## Usage

```nabi
use stats::{Continuous, Normal, StudentT};
use core::array::Matrix;

let x = [5.1, 4.9, 5.6, 5.8, 6.0, 5.4, 5.2, 5.7];
println(f"{stats::mean(x)} {stats::std(x, ddof: 1)} {stats::median(x)}");
let q = try x |> stats::quantile(0.9);           // NumPy's default (type 7)
let s = stats::describe(x);                      // n, mean, quartiles, shape

let m: Matrix<f64> = [[1, 2, 9], [4, 6, 8]];
let col_means = try stats::axis::mean(m, 0);     // [2.5, 4.0, 8.5]

let d = try Normal::new(100.0, 15.0);
println(f"{d.cdf(130.0)} {d.ppf(0.975)} {d.interval(95pct)}");
let t = try StudentT::new(9.0);
println(f"{t.isf(2.5pct)}");                     // 2.262157162798205
```

Tests return the statistic and p-value; t-tests also the estimate and
its standard error, so a confidence interval is one call:

```nabi
let a = [0.7, -1.6, -0.2, -1.2, -0.1, 3.4, 3.7, 0.8, 0.0, 2.0];
let b = [1.9, 0.8, 1.1, 0.1, -0.1, 4.4, 5.5, 1.6, 4.6, 3.4];
let w = try stats::ttest_ind(a, b);              // Welch by default
let p = try stats::ttest_rel(a, b);
println(f"{w} {p.ci(95pct)} {p.result().rejects(1pct)}");
let sw = try stats::shapiro(a);
let u = try stats::mannwhitneyu(a, b);
let ws = try stats::wilcoxon(a);                 // one sample
let wp = try stats::wilcoxon(a, [1.9, 0.8, 1.1, 0.1, -0.1, 4.4, 5.5, 1.6, 4.6, 3.4]);  // pairs
let gof = try stats::chisquare([16.0, 18.0, 16.0, 14.0, 12.0, 12.0]);
```

Regression goes through linalg's Householder QR, never `X^T X`:

```nabi
use core::array::{Matrix, Vector};

let x: Matrix<f64> = [[1, 0], [2, 1], [3, 1], [4, 3], [5, 2], [6, 4]];
let y: Vector<f64> = [3.1, 4.0, 6.2, 5.9, 9.1, 9.0];
let fit = try stats::ols(x, y);
println(f"{fit.coef:?} se {fit.se:?} p {fit.p:?} R^2 {fit.r2}");
let next: Matrix<f64> = [[7, 4]];
let yhat = try fit.predict(next);
```

Errors are one `StatsError` (`TooFew`, `InvalidParameter`,
`InvalidArgument`, `ShapeMismatch`, `NotFinite`, `Singular`,
`NoConvergence`); each message says what to pass instead. Descriptive
statistics do not throw: empty input gives NaN, as NumPy, and
`stats::nan::` skips NaN.

## Notation

With the `ops` feature (on by default) code reads like the notation
of a statistics text:

| Form        | Meaning                                                   | Named form                     |
|-------------|-----------------------------------------------------------|--------------------------------|
| `g ~ d`     | a draw from `d` with the `Sampler` `g` ("distributed as") | `d.sample(ref g)`              |
| `g ~ d.iid(n)` | `n` independent draws                                  | `d.samples(ref g, n)`          |
| `y ~ x`     | least-squares fit of `y` on `x` with an intercept (R's `lm(y ~ x)`) | `stats::ols(x, y)`   |
| `m ± h`     | the interval `[m - h, m + h]`                             | `Interval::around(m, h)`       |
| `95pct`     | the number 0.95                                           | `0.95`                         |
| `x in ci`   | interval membership (core's `∈`)                          | `ci.lo <= x && x <= ci.hi`     |

`~` has a second spelling, `∼` (U+223C TILDE OPERATOR); both bind like
`==`, and `g ~ d` advances `g`. `±` binds like `+`, so `m ± 1.96 * se`
is `m ± (1.96 * se)`. Operators cannot throw, so `y ~ x` is a `Result`:

```nabi
use stats::{Distribution, Normal, Sampler};
use core::array::Vector;

let g = Sampler::seed(42);
let z = g ~ Normal::standard();
let xs = g ~ (try Normal::new(10.0, 2.0)).iid(1000);
let se = stats::sem(xs);
let ci = stats::mean(xs) ± 1.96 * se;              // an Interval
println(f"{z} {ci} {10.0 in ci}");

let t: Vector<f64> = [1, 2, 3, 4];
let y: Vector<f64> = [3.1, 4.9, 7.2, 8.8];
let line = try (y ~ t).get();
```

Prefix `~` is still bitwise not. `%` is a fixed operator, so a
percentage is written `95pct`, not `95%`. Build without the feature
(`default_features: false`) if `~`, `±` or `pct` clash with another
package's.

## What is in it

| Area            | Items |
|-----------------|-------|
| Descriptive     | `mean`, `wmean`, `sum`, `var`, `std`, `sem`, `median`, `quantile`, `quantiles`, `percentile` (`QuantileMethod`: the 9 Hyndman-Fan types plus `Lower`, `Higher`, `Nearest`, `Midpoint`), `mode`, `min`, `max`, `range`, `iqr`, `skew`, `kurtosis`, `moment`, `zscore`, `describe` |
| Along an axis   | `axis::{mean, sum, var, std, min, max, median, quantile, skew, kurtosis, apply}` |
| NaN-aware       | `nan::{count, sum, mean, var, std, median, quantile, min, max}` |
| Association     | `cov`, `corr` (`CorrMethod::Pearson`, `Spearman`, `Kendall`), `pearson`, `spearman`, `kendall`, `rankdata` (`Ties`) |
| Histograms      | `histogram`, `bin_edges` (`Bins::Count`, `Width`, `Edges`, `Auto`, `Sturges`, `Scott`, `Fd`, `Sqrt`, `Rice`, `Doane`) |
| Special         | `special::{lgamma, gamma, digamma, trigamma, gammainc, gammaincc, gammaincinv, gammainccinv, beta, lbeta, betainc, betaincc, betaincinv, erf, erfc, erfinv, erfcinv, ndtr, ndtri}` |
| Continuous      | `Normal`, `LogNormal`, `Uniform`, `Exponential`, `Gamma`, `Beta`, `ChiSquared`, `StudentT`, `FisherF`, `Cauchy`, `Laplace`, `Weibull` (`Continuous`: `pdf`, `logpdf`, `cdf`, `sf`, `ppf`, `isf`, `mean`, `var`, `median`, `interval`) |
| Discrete        | `Bernoulli`, `Binomial`, `Poisson`, `Geometric`, `NegativeBinomial` (`Discrete`: `pmf`, `logpmf`, `cdf`, `sf`, `ppf`, `mean`, `var`) |
| Multivariate    | `Multinomial`, `MultivariateNormal` |
| Sampling        | `Distribution::{sample, samples, iid}` with any `Rng`, `Sampler` |
| Fits (MLE)      | `fit` on `Normal`, `LogNormal`, `Uniform`, `Exponential`, `Gamma`, `Laplace`, `Bernoulli`, `Binomial`, `Poisson`, `Geometric`, `Multinomial`, `MultivariateNormal` |
| Tests           | `ttest_1samp`, `ttest_ind` (Welch or pooled), `ttest_rel`, `chisquare`, `chi2_contingency`, `fisher_exact`, `kstest`, `ks_2samp`, `shapiro`, `mannwhitneyu`, `wilcoxon` (one sample or pairs), `f_oneway`, `pearsonr`, `spearmanr`, `kendalltau` (`Alternative`) |
| Regression      | `ols`, `wls` (`Ols`: coefficients, standard errors, t, p, R^2, adjusted R^2, F, residuals, `predict`, `ci`), `logistic` (IRLS), `polyfit`, `polyval` |
| Resampling      | `bootstrap` (`BootMethod::Percentile`, `Basic`, `Bca`) |

Methods: core's pairwise summation and the corrected two-pass variance
(`Array::sum`, `mean`, `var`, and the `_along` forms behind
`stats::axis`);
introselect for quantiles; Knight's O(n log n) Kendall tau-b. ln Gamma
and the rest of `stats::special` come from the `special` package, which
gives their methods and accuracy.
Draws come from the standard library's `Rng`: normal by the ziggurat
(Marsaglia and Tsang), gamma by Marsaglia and Tsang, Poisson by PTRS
(Hormann), binomial by beta splitting (Devroye). Kolmogorov-Smirnov p-values exact by Marsaglia, Tsang and
Wang's matrix method, Shapiro-Wilk by Royston's AS R94. Each
function's docs give its accuracy; the tests compare with SciPy,
statsmodels and NIST's certified Longley regression.

## Docs

https://docs.nabi-lang.org/packages/stats/latest

## License

Licensed under either of [MIT](LICENSE-MIT) or [Apache-2.0](LICENSE-APACHE), at
your option.
