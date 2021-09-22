---
title: "Aggregating r values with metafor"
---

# Premable


```
## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.0 ──
```

```
## ✔ ggplot2 3.3.1     ✔ purrr   0.3.4
## ✔ tibble  3.0.1     ✔ dplyr   1.0.0
## ✔ tidyr   1.1.0     ✔ stringr 1.4.0
## ✔ readr   1.3.1     ✔ forcats 0.5.0
```

```
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## ✖ dplyr::filter() masks stats::filter()
## ✖ dplyr::lag()    masks stats::lag()
```

```
## Loading required package: Matrix
```

```
## 
## Attaching package: 'Matrix'
```

```
## The following objects are masked from 'package:tidyr':
## 
##     expand, pack, unpack
```

```
## Loading 'metafor' package (version 2.4-0). For an overview 
## and introduction to the package please type: help(metafor).
```

## Generate random estimates


```r
true_R <- 0.6
nmodels <- 4

means <- rnorm(nmodels, true_R, 0.2)
stds <- rgamma(nmodels, 1, 5)

means
```

```
## [1] 1.0057706 0.4945920 0.6983826 0.6722164
```

```r
stds
```

```
## [1] 0.008631012 0.472291411 0.068520085 0.072039456
```

## Produce an estimate


```r
r <- rma(yi=test_means, sei=test_stds, weighted=FALSE, level=90)
```

```
## Error in eval(mf.yi, data, enclos = sys.frame(sys.parent())): object 'test_means' not found
```

```r
r
```

```
## Error in eval(expr, envir, enclos): object 'r' not found
```

```r
forest(r)
```

```
## Error in forest(r): object 'r' not found
```

# Experiment 1: move the mean of one estimate up by 1


```r
exp_1 <- list()
for (i in 1:nmodels) {
  test_means <- means
  test_stds <- stds
  test_means[i] <- test_means[i] + 1
  r <- rma(yi = test_means, sei = test_stds, weighted = FALSE, level = 90)
  exp_1[[i]] <- tibble(mean = r$b, lower = r$ci.lb, upper = r$ci.ub)
}

exp_1 <- bind_rows(exp_1)

exp_1
```

```
## # A tibble: 4 x 3
##   mean[,1] lower upper
##      <dbl> <dbl> <dbl>
## 1    0.968 0.362  1.57
## 2    0.968 0.715  1.22
## 3    0.968 0.514  1.42
## 4    0.968 0.530  1.41
```

# Experiment 2: quintuplicate one estimate


```r
exp_2 <- list()
for (i in 1:nmodels) {
  test_means <- c(means, rep(means[i], 4))
  test_stds <- c(stds, rep(stds[i], 4))
  r <- rma(yi = test_means, sei = test_stds, weighted = FALSE, level = 90)
  exp_2[[i]] <- tibble(mean = r$b, lower = r$ci.lb, upper = r$ci.ub)
}

exp_2 <- bind_rows(exp_2)

exp_2
```

```
## # A tibble: 4 x 3
##   mean[,1] lower upper
##      <dbl> <dbl> <dbl>
## 1    0.862 0.735 0.989
## 2    0.606 0.366 0.847
## 3    0.708 0.582 0.834
## 4    0.695 0.566 0.824
```
