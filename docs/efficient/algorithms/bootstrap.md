Algorithm Showdown: Bootstrap
=============================

The typical bootstrap involves drawing a random sample of the data and re-computing your summary statistic. This can be a slow process (even Stata's built-in `bootstrap` can be slow). While in many situations this is the only way to bootstrap your data, the _**Bayesian bootstrap**_ (Rubin, 1981) is much more efficient (computationally) and works in most common scenarios:

- Draw iid Dirichlet weights (one for each observation).

- Recompute statistic, weighted.

That's it! (Added bonus: Since all observations are kept, just re-weighted, it's also more stable for e.g. regressions, as no variables can unexpectedly drop due to collinearity.) Let's look at a concrete example; the built-in command takes about a minute here:

```stata
clear
set seed 1
set rmsg on
set obs 1000000
gen x = rnormal() * 2
gen y = x + rnormal() * 10
bootstrap, reps(100): reg y x

    -------------------------------------
                 |   Observed   Bootstrap
               y | coefficient  std. err.
    -------------+----------------------- ...
               x |   .9925311   .0050175
           _cons |  -.0151513   .0087632
    -------------------------------------
```

The Bayesian bootstrap runs in 20s (3x as fast)!

```stata
mata b = J(100, 2, .)
tempvar weight
qui gen `weight' = .
qui forvalues b = 1 / 100 {
    replace `weight' = rgamma(1, 1)
    sum `weight', meanonly
    replace `weight' = `weight' / r(sum)
    reg y x [pw = `weight']
    mata b[`b', .] = st_matrix("e(b)")
}
mata diagonal(sqrt(variance(b)))'

                  1             2
     +-----------------------------+
   1 |  .0048730429   .0093159813  |
     +-----------------------------+
```
