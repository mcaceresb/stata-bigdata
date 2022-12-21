Gtools
======

There was a section in my presentation about `gtools`, which is the suite of commands I coded for working with big data more efficiently. While `gtools` still has to work within the contraints of Stata, it's underlying code is in C and is much faster than Stata in many places, to the point where some big data [tasks](tasks) are much less of a bottleneck.

- The original impetus for `gtools` was writing a faster collapse. While in Stata 17/MP, `collapse` has caught up to `gcollapse`, the package has expanded well beyond this original idea, and thare is a lot of functionality that remains much faster than any other Stata programs available. (Even in the case of `collapse`, `gcollapse` offers the `merge` and `merge replace` options, for example.)

- Please visit [gtools.readthedocs.io <img src="https://upload.wikimedia.org/wikipedia/commons/6/64/Icon_External_Link.png" width="13px"/>](https://gtools.readthedocs.io/en/latest) for detailed documentation and examples. Below I reproduce some of the tables with an overview of how `gtools` compares to other Stata commands, which you can also find on the official site.

__*Gtools commands with a Stata equivalent*__

| Function     | Replaces    | Speedup (IC / MP)              | Unsupported             | Extras                                  |
| ------------ | ----------- | ------------------------------ | ----------------------- | --------------------------------------- |
| gcollapse    | collapse    | -0.5 to 2 (Stata 17+); 4 to 100 (Stata 16 and earlier)  || Quantiles, merge, labels, nunique, etc. |
| greshape     | reshape     |  4 to 20  / 4 to 15            | "advanced syntax"       | `fast`, spread/gather (tidyr equiv)     |
| gegen        | egen        |  9 to 26  / 4 to 9 (+,.)       | labels                  | Weights, quantiles, nunique, etc.       |
| gcontract    | contract    |  5 to 7   / 2.5 to 4           |                         |                                         |
| gisid        | isid        |  8 to 30  / 4 to 14            | `using`, `sort`         | `if`, `in`                              |
| glevelsof    | levelsof    |  3 to 13  / 2 to 7             |                         | Multiple variables, arbitrary levels    |
| gduplicates  | duplicates  |  8 to 16 / 3 to 10             |                         |                                         |
| gquantiles   | xtile       |  10 to 30 / 13 to 25 (-)       |                         | `by()`, various (see [usage](https://gtools.readthedocs.io/en/latest/usage/gquantiles)) |
|              | pctile      |  13 to 38 / 3 to 5 (-)         |                         | Ibid.                                   |
|              | \_pctile    |  25 to 40 / 3 to 5             |                         | Ibid.                                   |
| gstats tab   | tabstat     |  10 to 50 / 5 to 30 (-)        | See [remarks](#remarks) | various (see [usage](https://gtools.readthedocs.io/en/latest/usage/gstats_summarize)) |
| gstats sum   | sum, detail |  10 to 20 / 5 to 10            | See [remarks](#remarks) | various (see [usage](https://gtools.readthedocs.io/en/latest/usage/gstats_summarize)) |

<small>(+) The upper end of the speed improvements are for quantiles
(e.g. median, iqr, p90) and few groups. Weights have not been
benchmarked.</small>

<small>(.) Only gegen group was benchmarked rigorously.</small>

<small>(-) Benchmarks computed 10 quantiles. When computing a large
number of quantiles (e.g. thousands) `pctile` and `xtile` are prohibitively
slow due to the way they are written; in that case gquantiles is hundreds
or thousands of times faster, but this is an edge case.</small>

__*Extra commands*__

| Function            | Similar (SSC/SJ)         | Speedup (IC / MP)       | Notes                         |
| ------------------- | ------------------------ | ----------------------- | ----------------------------- |
| fasterxtile         | fastxtile                | 20 to 30 / 2.5 to 3.5   | Allows `by()`                 |
|                     | egenmisc (SSC) (-)       | 8 to 25 / 2.5 to 6      |                               |
|                     | astile (SSC) (-)         | 8 to 12 / 3.5 to 6      |                               |
| gstats hdfe         |                          | (.)                     | Allows weights, `by()`        |
| gstats winsor       | winsor2                  | 10 to 40 / 10 to 20     | Allows weights                |
| gunique             | unique                   | 4 to 26 / 4 to 12       |                               |
| gdistinct           | distinct                 | 4 to 26 / 4 to 12       | Also saves results in matrix  |
| gtop (gtoplevelsof) | groups, select()         | (+)                     | See table notes (+)           |
| gstats range        | rangestat                | 10 to 20 / 10 to 20     | Allows weights; no flex stats |
| gstats transform    |                          |                         | Various statistical functions |

<small>(-) `fastxtile` from egenmisc and `astile` were benchmarked against
`gquantiles, xtile` (`fasterxtile`) using `by()`.</small>

<small>(+) While similar to the user command 'groups' with the 'select'
option, gtoplevelsof does not really have an equivalent. It is several
dozen times faster than 'groups, select', but that command was not written
with the goal of gleaning the most common levels of a varlist. Rather, it
has a plethora of features and that one is somewhat incidental. As such, the
benchmark is not equivalent and `gtoplevelsof` does not attempt to implement
the features of 'groups'</small>

<small>(.) Other than the dated 'hdfe' command, I do not know of a stata
command that residualizes variables from a set of fixed effects. The
'hdfe' command, as far as I can tell, morphed into the 'reghdfe'
package; the latter, however, is a fully-functioning regression command,
while 'gstats hdfe' only residualizes a set of variables.</small>
