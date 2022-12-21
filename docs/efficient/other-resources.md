Don't reinvent the wheel!
=========================

Someone may have already solved your problem. There are many user-written packages and programs that are designed to be efficient and  may already do what you want to do. While custom tools tailored to your individual situation will typically be faster, it's often the case that the general-purpose tools are fast enough to justify avoiding making the time investment.

Here I list just a handful of popular user-written Stata packages that can be very helpful when working with large datasets. I've personally used all of these at some point or another and can vouch they can be really helpful:

- [`reghdfe`](github.com/sergiocorreia/reghdfe) (and [`ivreghdfe`](https://github.com/sergiocorreia/ivreghdfe), [`ppmlhdfe`](https://github.com/sergiocorreia/ppmlhdfe)): High-dimensional fixed effects for regression modes.

- [`parallel`](https://github.com/gvegayon/parallel) Parallelize code execution.

- [`gtools`](https://gtools.readthedocs.io) Fast by-able data management and summary statistics (disclamer: I authored this package).

- [`ftools`](https://github.com/sergiocorreia/ftools): Fast implementation of several Stata commands (e.g. `fegen group`, `fcollapse`, `fmerge`, `fisid`, `flevelsof`, `fsort`). This package was the inspiration for `gtools` and while its functions are slower than their `gtools` counterparts, it retains some benefits: Namely there is no `gtools` counterpart of `fmerge`, and its `mata` API can be very useful.
