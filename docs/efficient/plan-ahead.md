Planning ahead
==============

### Sorting and `by`

For example, if you will be doing many operations by group, sorting the data and working on it by group all at once will make each operation much faster. (NB: While `gtools` functions are also faster on sorted data, of course part of the point of `gtools` is to obviate the need to do a sort, and the speed gain is often larger without sorting.)

```stata
set seed 1729
clear
set obs 10000000
gen x = rnormal()
gen y = rnormal()
gen g = mod(_n, 100)
gen r = runiform()
sort r
set rmsg on
```

If should be clear that this is inefficient:
```stata
qui {
    bys g: gen a = sum(x)
    bys g: replace a = a[_N] / _N
    sort r
    gen b = .
    bys g: replace b = max(y, b[_n - 1])
    bys g: replace b = b[_N]
    sort r
    gen c = .
    bys g: replace c = min(y, c[_n - 1])
    bys g: replace c = c[_N]
    sort r
}
```

However, we might accidentally end up doing a version of this if we're
not deliberate about doing similar operations together. A better way to
do it would be

```stata
drop a b c
qui {
    sort g
    by g: gen a = sum(x)
    by g: replace a = a[_N] / _N
    gen b = .
    by g: replace b = max(y, b[_n - 1])
    by g: replace b = b[_N]
    gen c = .
    by g: replace c = min(y, c[_n - 1])
    by g: replace c = c[_N]
    sort r
}
```

Of course, `gegen` is faster in this case if you care about leaving the data in its original state:

```stata
drop a b c
{
    gegen a = sum(x), by(g)
    gegen b = max(y), by(g)
    gegen c = min(y), by(g)
}
```

But you should know two things. First, this is an inefficient `gtools` solution, and we should be using the `merge` option from `gcollapse`:
```stata
drop a b c
gcollapse (sum) a=x (max) b=y (min) c=y, by(g) merge
```

Second, if we don't benchmark the `sorts`, the individual series of `by` operations in Stata are faster than even this `gcollapse` statement. The reason is that `gcollapse` has to group the data internally, whereas the `by` statement relies on the sort statement for that; so it's not a one-to-one comparison, but if your data will be sorted anyway, you should know that sometimes relying on `by` can be the fastest solution!

### Pre-computing variables

You should pre-compute variables that will be re-used instead of creating them on the fly. For example:

```stata
gen byte ind = ...
program1 if ind == 1
program2 if ind == 1

gen var = ...
for i = 3 / 7 {
    program`i' var
}
```

### Very long operations

Sometimes a program that takes a long time to run is inevitable:

- Run overnight or over a break. (So program does not compete for computing time or your own time.)

- Include checkpoints:

    - Do not write a single function to do all your work.

    - Group tasks into programs, and save your data along the way.

    - Print messages along your program to tell you where you are (can check log while program executes).

This program groups execution and has checkpoints and log messages throughout:
```stata
program part1
    display "part 1, task 1"
    * ...
    display "part 1, task 2"
    * ...
end

program part2
    display "part 2, task 1"
    * ...
    display "part 2, task 2"
    * ...
end

part1
save checkpoint1.dta
display "finished part 1"

part2
save checkpoint2.dta
display "finished part 2"
```

We can even be fairly sophisticated about it. The snippet below can scale to multiple parts, for instance:

```stata
program main
    syntax, [cached]

    local nparts  = 2
    local startat = 1
    if "`cached'" == "cached" {
        forvalues part = 1 / `nparts' {
            cap confirm file checkpoint`part'.dta
            local startat = cond(`startat' == `part' & _rc == 0, `part'+1, `startat')
        }
    }

    forvalues part = 1 / `nparts' {
        if `startat' <= `part' {
            part`part'
            save checkpoint`part'.dta, replace
            display "finished part 1"
        }
        else if `startat' == (`part' + 1) {
            display "loading cached part `part' results"
            use checkpoint`part'.dta, clear
        }
    }

    if `startat' > `nparts' {
        disp "Nothing to do with option -cached-; all checkpoints already exist"
    }
end

program part1
    display "part 1, task 1"
    * ...
    display "part 1, task 2"
    * ...
end

program part2
    display "part 2, task 1"
    * ...
    display "part 2, task 2"
    * ...
end

main, cached
```
