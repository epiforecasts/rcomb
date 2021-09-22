Aggregating r values with metafor
================

# Initialise

``` r
library("dplyr")
library("metafor")
set.seed(329012409)
```

# Generate random estimates

``` r
true_R <- 0.6
nmodels <- 4

means <- rnorm(nmodels, true_R, 0.2)
stds <- rgamma(nmodels, 1, 5)
```

``` r
means
```

    ## [1] 1.0057706 0.4945920 0.6983826 0.6722164

``` r
stds
```

    ## [1] 0.008631012 0.472291411 0.068520085 0.072039456

# Produce a meta-estimate

``` r
r <- rma(yi=means, sei=stds, weighted=FALSE, level=90)
r
```

    ## 
    ## Random-Effects Model (k = 4; tau^2 estimator: REML)
    ## 
    ## tau^2 (estimated amount of total heterogeneity): 0.0326 (SE = 0.0356)
    ## tau (square root of estimated tau^2 value):      0.1806
    ## I^2 (total heterogeneity / total variability):   89.72%
    ## H^2 (total variability / sampling variability):  9.73
    ## 
    ## Test for Heterogeneity:
    ## Q(df = 3) = 41.4747, p-val < .0001
    ## 
    ## Model Results:
    ## 
    ## estimate      se    zval    pval   ci.lb   ci.ub 
    ##   0.7177  0.1507  4.7616  <.0001  0.4698  0.9657  *** 
    ## 
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

``` r
forest(r)
```

![](rcomb_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

# Experiment 1: shift an estimate out of the range of the others

``` r
## increase each estimate by 1, in turn
exp_1 <- list()
for (i in 1:nmodels) {
  test_means <- means
  test_stds <- stds
  test_means[i] <- test_means[i] + 1
  r <- rma(yi = test_means, sei = test_stds, weighted = FALSE, level = 90)
  exp_1[[i]] <- tibble(mean = r$b, lower = r$ci.lb, upper = r$ci.ub)
}

exp_1 <- bind_rows(exp_1)

## each row corresponds to one estimate having been increased by one
exp_1
```

    ## # A tibble: 4 × 3
    ##   mean[,1] lower upper
    ##      <dbl> <dbl> <dbl>
    ## 1    0.968 0.362  1.57
    ## 2    0.968 0.715  1.22
    ## 3    0.968 0.514  1.42
    ## 4    0.968 0.530  1.41

Conclusions:

1.  mean estimate unaffected by which estimate is moved up;
2.  resulting uncertainty is greater if a more certain estimate is
    moved;

# Experiment 2: multiply one estimate

``` r
## duplicate each estimate, in turn
nmult <- 1
exp_2 <- list()
for (i in 1:nmodels) {
  test_means <- c(means, rep(means[i], nmult))
  test_stds <- c(stds, rep(stds[i], nmult))
  r <- rma(yi = test_means, sei = test_stds, weighted = FALSE, level = 90)
  exp_2[[i]] <- tibble(mean = r$b, lower = r$ci.lb, upper = r$ci.ub)
}

exp_2 <- bind_rows(exp_2)

exp_2
```

    ## # A tibble: 4 × 3
    ##   mean[,1] lower upper
    ##      <dbl> <dbl> <dbl>
    ## 1    0.775 0.570 0.980
    ## 2    0.673 0.415 0.931
    ## 3    0.714 0.515 0.912
    ## 4    0.709 0.508 0.909

Conclusions: this affects the estimate
