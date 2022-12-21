Improve your Algorithms
=======================

- A parallelized and extremely efficient loop is slower than the equivalent vector operation.
- The fastest regression with a full set of fixed-effect indicators is slower than `reghdfe`.
- Code tailored to your specific task is faster than general-purpose code.

A simple example I think most people should find intuitive is computing a leave-one-out mean. One definition is

$$
    \bar{x}_{-i} = \dfrac{\sum_{j \ne i} x_j}{\sum_{j \ne i} 1}
$$

And the corresponding code is given by
```stata
clear
set rmsg on
set obs 10000
gen x = runiform()
gen x_loo = .
forvalues i = 1 / `=_N' {
    qui sum x if _n != `i', meanonly
    qui replace x_loo = r(mean) in `i'
}
```

With only 10,000 observations this already takes a few seconds. Another definition, which is one I think most people will have used, is
$$
    \bar{x}_{-i} = \dfrac{(\sum_j x_j) - x_i}{(\sum_j 1) - 1}
$$

```stata
qui sum x, meanonly
gen x_loo2 = (r(sum) - x) / (r(N) - 1)
assert x_loo == x_loo2

clear
set obs 10000000
gen x = runiform()
qui sum x, meanonly
gen x_loo2 = (r(sum) - x) / (r(N) - 1)
```

Even with 10M observations, this method gives the answer in a fraction of a second. In this case, it truly would not have mattered how fast you make your loop: The faster algorithm will win every time.
