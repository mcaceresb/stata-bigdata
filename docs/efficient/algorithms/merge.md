Algorithm Showdown: Merge
=========================

Often it is possible to merge on single, integer IDs (vs arbitrary sets of variables).

```stata
merge 1:1 id using data.dta
// vs
merge 1:1 var_double var_str8 var_long // ...
```

Sorting and matching on a single variable is much faster than sorting and matching many variables of different types, regardless of how efficient you make the sort or the merge itself! Even merging on multiple integers is faster than merging on multiple mixed-type variables.

!!! tip "Anecdote: Recasting Data Types"
    I was working with insurance data on 80M individuals, and their ID was a 21-character variable. The first thing I did was map them into an integer ID numbered 1 through 80M and then I worked with that 4-byte identifier for the rest of the analysis. I knew every merge and group operation would be much faster! In that project I did the same type of re-mapping for basically every variable I could (e.g. categorical demographic data).
    
    Recently someone asked me if I could help speed up their merge. All I suggested was that they recast each of their variables as their smallest type (all their numbers were doubles, but I suggested they recast dates as 2-byte integers, some categories as 1-byte integers, IDs as 4 or 8-byte integers, etc.) Their merge was suddenly 2-3 times faster just with that change.
