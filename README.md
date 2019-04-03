
[![Travis-CI Build
Status](https://travis-ci.org/hrbrmstr/tdigest.svg?branch=master)](https://travis-ci.org/hrbrmstr/tdigest)
[![Coverage
Status](https://codecov.io/gh/hrbrmstr/tdigest/branch/master/graph/badge.svg)](https://codecov.io/gh/hrbrmstr/tdigest)
[![CRAN\_Status\_Badge](https://www.r-pkg.org/badges/version/tdigest)](https://cran.r-project.org/package=tdigest)

# tdigest

Wicked Fast, Accurate Quantiles Using ‘t-Digests’

## Description

The t-digest construction algorithm uses a variant of 1-dimensional
k-means clustering to produce a very compact data structure that allows
accurate estimation of quantiles. This t-digest data structure can be
used to estimate quantiles, compute other rank statistics or even to
estimate related measures like trimmed means. The advantage of the
t-digest over previous digests for this purpose is that the t-digest
handles data with full floating point resolution. With small changes,
the t-digest can handle values from any ordered set for which we can
compute something akin to a mean. The accuracy of quantile estimates
produced by t-digests can be orders of magnitude more accurate than
those produced by previous digest algorithms.

See [the original paper by Ted
Dunning](https://raw.githubusercontent.com/tdunning/t-digest/master/docs/t-digest-paper/histo.pdf)
for more details on t-Digests.

## What’s Inside The Tin

The following functions are implemented:

  - `is_tdigest`: Test to see if an object is classed as `tdigest`
  - `tdigest`: Create a new t-digest histogram from a vector
  - `td_add`: Add a value to the t-digest with the specified count
  - `td_create`: Allocate a new histogram
  - `td_merge`: Merge one t-digest into another
  - `td_quantile_of`: Return the quantile of the value
  - `td_total_count`: Total items contained in the t-digest
  - `td_value_at`: Return the value at the specified quantile
  - `tquantile`: Calcuate sample quantiles from a t-digest

## Installation

``` r
install.packages("tdigest", repos="https://cinc.rud.is/")
# or 
devtools::install_git("https://sr.ht.com/~hrbrmstr/tdigest.git")
# or
devtools::install_gitlab("hrbrmstr/tdigest")
# or (if you must)
devtools::install_github("hrbrmstr/tdigest")
```

## Usage

``` r
library(tdigest)

# current version
packageVersion("tdigest")
## [1] '0.2.0'
```

### Basic (Low-level interface)

``` r
td <- td_create(10)

td
## <tdigest; size=0>

td_total_count(td)
## [1] 0

td_add(td, 0, 1) %>% 
  td_add(10, 1)
## <tdigest; size=2>

td_total_count(td)
## [1] 2

td_value_at(td, 0.1) == 0
## [1] TRUE
td_value_at(td, 0.5) == 5
## [1] TRUE

quantile(td)
## [1]  0  0  5 10 10
```

#### Bigger (and Vectorised)

``` r
td <- tdigest(c(0, 10), 10)

is_tdigest(td)
## [1] TRUE

td_value_at(td, 0.1) == 0
## [1] TRUE
td_value_at(td, 0.5) == 5
## [1] TRUE

set.seed(1492)
x <- sample(0:100, 1000000, replace = TRUE)
td <- tdigest(x, 1000)

td_total_count(td)
## [1] 1e+06

tquantile(td, c(0, 0.01, 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 0.99, 1))
##  [1]   0.0000000   0.8632378   9.6763281  19.7028368  29.7718982  39.9706864  50.0032181  60.0859360  70.1951621
## [10]  80.2785864  90.3290326  99.5151872 100.0000000

quantile(td)
## [1]   0.00000  24.81839  50.00322  75.23076 100.00000
```

#### Proof it’s faster

``` r
microbenchmark::microbenchmark(
  tdigest = tquantile(td, c(0, 0.01, 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 0.99, 1)),
  r_quantile = quantile(x, c(0, 0.01, 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 0.99, 1))
)
## Unit: microseconds
##        expr       min        lq        mean     median         uq       max neval
##     tdigest    26.334    31.162    61.02889    67.3605    71.0985   177.618   100
##  r_quantile 61909.704 64146.167 66500.42677 65329.2830 68093.9355 78102.683   100
```

## tdigest Metrics

| Lang         | \# Files |  (%) | LoC |  (%) | Blank lines |  (%) | \# Lines |  (%) |
| :----------- | -------: | ---: | --: | ---: | ----------: | ---: | -------: | ---: |
| C            |        3 | 0.27 | 337 | 0.68 |          45 | 0.38 |       26 | 0.11 |
| R            |        6 | 0.55 | 118 | 0.24 |          25 | 0.21 |      133 | 0.56 |
| Rmd          |        1 | 0.09 |  32 | 0.06 |          37 | 0.32 |       51 | 0.21 |
| C/C++ Header |        1 | 0.09 |  10 | 0.02 |          10 | 0.09 |       28 | 0.12 |

## Code of Conduct

Please note that this project is released with a [Contributor Code of
Conduct](CONDUCT.md). By participating in this project you agree to
abide by its terms.
