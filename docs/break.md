 What can break with big data?
==============================

### Generating IDs

Consider a relatively common operation: Generating a unique identifier for each row of your data. Nothing too complicated:

```stata
. clear
. qui set obs `=2^24+5'
. gen id = _n
. format %21.0gc id
. list in -10/l

          +------------+
          |         id |
          |------------|
16777212. | 16,777,212 |
16777213. | 16,777,213 |
16777214. | 16,777,214 |
16777215. | 16,777,215 |
16777216. | 16,777,216 |
          |------------|
16777217. | 16,777,216 |
16777218. | 16,777,218 |
16777219. | 16,777,220 |
16777220. | 16,777,220 |
16777221. | 16,777,220 |
          +------------+

. disp "`:type id'"
float
```

What went wrong?

- Stata's default type is `float`, a 4-byte floating point number.
- The largest integer it can accurately represent is 2<sup>24</sup>
- If we force it to represent larger integers, the variable overflows and we get repeated nubmers, as we can see.


What can we do?

- You can can set the default type to `double`, an 8-byte floating point number which can represent integers up to 2<sup>53</sup>. Downside is increased memory use.
- The efficient solution is to use the `c(obs_t)` macro:

```stata
gen `c(obs_t)' id = _n
```

This uses the smallest _integer_ type that can store the number of observations correctly. In general for any variable bounded above by the number of observations, this is a memory-efficient and safe solution (counts, IDs, etc.) Here you can see it in action:

```stata
clear
qui set obs 1
disp "`c(obs_t)'" // byte
qui set obs `=maxbyte()+1'
disp "`c(obs_t)'" // int
qui set obs `=maxint()+1'
disp "`c(obs_t)'" // long
qui set obs `=maxlong()+1'
disp "`c(obs_t)'" // double
```

### Numerical Precision

This is a general issue when working with floating point numbers (i.e. real numbers, which are uncountably infinite, represented by finite data). However, big data can compound this problem. Large sums, for instance, can give incorrect results by some margin:

```stata
. clear
. set obs 1000000
. gen double x    = sum(_n^2)
. gen double xsum = _n * (_n + 1) * (2 * _n + 1) / 6
. gen double xdif = x - xsum
. sum xdif, d

                            xdif
-------------------------------------------------------------
      Percentiles      Smallest
 1%      -481312        -490752
 5%      -443808        -490752
10%      -396928        -490752       Obs           1,000,000
25%      -256400        -490720       Sum of Wgt.   1,000,000

50%      -123600                      Mean          -154691.7
                        Largest       Std. Dev.      151885.8
75%            0             .5
90%            0             .5       Variance       2.31e+10
95%            0             .5       Skewness      -.5467313
99%            0             .5       Kurtosis       2.055727
```

You can see that for half the observations, the sum is off by more than 10<sup>6</sup>. If course, in relative terms this difference is negligible:

```stata
. gen double xrel = reldif(x, xsum)
. sum xrel, d

                            xrel
-------------------------------------------------------------
      Percentiles      Smallest
 1%            0              0
 5%            0              0
10%            0              0       Obs           1,000,000
25%            0              0       Sum of Wgt.   1,000,000

50%     1.63e-12                      Mean           1.51e-12
                        Largest       Std. Dev.      1.20e-12
75%     2.42e-12       3.80e-12
90%     3.29e-12       3.80e-12       Variance       1.43e-24
95%     3.63e-12       3.80e-12       Skewness       .1221362
99%     3.78e-12       3.80e-12       Kurtosis       1.956908
```

This example merely underlines that care is required when working with large sums. (As a side-note, Stata's internal sums are often quadruple precision, meaning 16-byte floating point number, to avoid these types of issues.)

### A note on long long integers (64-bit, 8-bytes)

Stata's largest integer type is long, 4 bytes as noted above. Sometimes, however, programs will use long long integers, which are 8 bytes. The longest integer represented by a double is 2<sup>53</sup>, but a long long integer can go up to 2<sup>63</sup>-1.

Working around this limitation within Stata is _possible_ since the addition of the Python interface (there's an interesting [statalist thread about this <img src="https://upload.wikimedia.org/wikipedia/commons/6/64/Icon_External_Link.png" width="13px"/>](https://www.statalist.org/forums/forum/general-stata-discussion/general/1669326-missing-data-type-64-bit-integer-variable) where I pointed out such a solution). However, I will also say that sometimes Stata's not the best tool for the job, and stretching it to its limits is risky. I think it's only advisable to do so if using  other software is prohibitive for some reason.
