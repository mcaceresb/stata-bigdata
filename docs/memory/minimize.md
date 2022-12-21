Minimize Memory Use
===================

### Use Small Variable Types

Stata has multiple variable types. The default is `float`, and you can see [here](break.md) why storing everything as a float can lead to problems. However, storing everything as a double is excessive in most situations. In general, store variables in the smallest sensible type for the data they have:

- double: Largest number type (8 bytes). Integers in -/+ 2<sup>53</sup>, numbers in -/+ 8.988e307.
- float: Stata default type (4 bytes). Integers in -/+ 2<sup>24</sup>, numbers in -/+ 1.701e38.
- long: Largest integer type (4 bytes). Integers from -2<sup>31</sup>+1 to 2<sup>31</sup>-28.
- int: Small integer type (2 bytes). Integers from -2<sup>15</sup>+1 to 2<sup>15</sup>-28.
- byte: Smallest integer type (1 bytes). Integers from -2<sup>7</sup>+1 to 2<sup>7</sup>-28.
- str#: String data from str1 to str2045 (one byte per character; e.g. str10 is 10 bytes).
- strL: Arbitrary data (in general, binary data, but includes strings of variable width) up to 2e9 bytes per row (has a 80 byte overhead, so storage is 80 bytes per row plus the space taken up by the data it stores).

In general you'll use `float` for reals (by default), `long` for integers, and str# for strings. However, there are plenty of variables that are bounded and you can save memory by using `byte` (e.g. indicators, sparse categorical variables) or `int` (e.g. dates, day of the year).

- It's often hard to know the best type to use for all your variables (and sometimes, when coding quickly, you can simply forger to use anything other than the default for numbers and `long` for integers).
- `compress` is a Stata command that recasts all your variables as the smallest safe type.

!!! tip "Always compress data"
    It is _**always**_ (almost) a good idea, and always safe, to use `compress`
    on your data. The only scenario you wouldn't do that is if you
    cannot spare the computing time (compress can be very cpu-intensive in
    some cases).

In this case, compression refers specifically to variable recasting to save memory and is a completely lossless operation: The exact same data is retained, but wastefully allocated space is freed. (In many other computing settings, compression refers to encoding information in fewer bytes to save space; in other words, the data is altered so the encoded version takes up less space, and the original copy can be recovered by uncompressing the data. Stata's `compress` does __*not*__ refer to this.)

!!! tip "Anecdote: Poorly typed data"
    I once received a Stata file over 150GiB in size. While it had close
    to 160M rows, it only had 6 variables. I noticed 4 were string
    variables of type str256, but some quick exploration revealed they
    were really str8. I looped through the data in chunks, using compress
    on each chunk, and then appended the chunks at the end. (By chunk I mean
    a range of rows, which is efficient in Stata as data is stored by row; I
    did it this way to avoid reading too much data into memory at a time.) 
    The final data only took up around 5GiB.

### Make sure numbers are numbers

It almost always takes less memory to store numbers as numbers than as strings, but very often data that should be numeric is stored as string (this can happen often when, for example, importing external data).

- `destring varlist, replace` is a safe way of making sure you don't have any numbers hidden in strings. As `compress`, there are situations where it can be cpu-intensive, but if your strings contain non-numeric data it will not force them to become numbers. The only (default) caveat as far as I know is that missing values are interpreted, even though they're not numbers per se (so `.`, `.a`, `.b` and so on are read as a missing values).

- Sometimes you need to be a bit more proactive: Say you have hourly wages, rounded, for a group of individuals (€20, €30, etc.) truncated above by €1,000. If you import this data into Stata, it would be stored as str6, but it can be stored as a 2-byte integer. In straightforward cases like this, you can just use the `ignore` option in `destring`, but in general you may need to clean up your strings before converting them to numbers:

```stata
. clear
. qui set obs 3
. qui gen str6 wages = "€20" in 1
. qui replace  wages = "€30" in 2
. qui replace  wages = "€1,000" in 3
. destring wages, replace
wages: contains nonnumeric characters; no replace
. destring wages, replace ignore("€,")
wages: characters € , removed; replaced as int
```

### Why can memory use blow up anyway?

Most ancillary objects take little memory (with exceptions), but many programs make copies of the data internally.

- It's often not possible to modify the data in place.

- Even if possible, it's often more efficient to make copies.

Examples:

- `gtools` has to make a copy of all the variables that will be used, in addition to all internal objects. It can be a memory hog!

- `parallel` makes copies of all ancillary objects and starts sepparate instances of Stata, each with its own resource use. For "normal" use cases, each instance would only have a cut of the data, so the memory overhead should be minimal. However, depending on your operation, memory use can increase by up to the number of cores requested.

- `mata`, `frame`, and `python` can all make large objects independent of what is stored in the current dataset.

### Short demo with mata, frames, and Python

```stata
log using memtest.log, replace
* Define function to check this Stata instance's current memory
python:
import os, psutil

def checK_memory():
    print(f"{psutil.Process(os.getpid()).memory_info().rss / 1024**2:,.1f} MiB in use")
end

* Check space taken by integer ID
python: checK_memory()
set obs `:disp %12.0f 1e8'
gen `c(obs_t)' id = _n
python: checK_memory()

* Total space after copying to mata
mata id = st_data(., "id")
python: checK_memory()

* Total space after copying to second frame
frame put id, into(id)
python: checK_memory()

* Total space after copying to Python
python:
from sfi import Data
id = Data.get(var="id")
checK_memory()
end

* Check space after droping each object
drop id
python: checK_memory()
mata mata drop id
python: checK_memory()
frame drop id
python: checK_memory()
python: id=None
python: checK_memory()

* Check objects were dropped
frame dir
mata mata desc
python: [k for k in globals().keys() if not k.startswith('_')]
log close _all
```

These are the statements printed by each `checK_memory` invocation, in Stata 17/MP:

- 38.7 MiB: Baseline memory use.

- 610.8 MiB: Minimum space taken up by 4-byte variable with 100M rows is 381.5 MiB; you can see Stata has an additional 190MiB overhead; this overhead will vary depending on your dataset.

- 1,374.3 MiB: Mata stores numbers as 8-byte doubles; minimum space taken up with 100M rows is 762.9 MiB; you can see there was virtually no overhead in this case.

- 1,946.8 MiB: This effectively makes a copy of the current data frame; it's not too surprising that the additional memory is almost identical here than vs the first step.

- 5,773.5 MiB: Python should store a copy as a double similar to mata; however, we can see python's overhead is nearly 3GiB in this case. I am not entirely sure why this happens, but you should certainly be quite mindful of this when interfacing with Python from Stata. It seems there's intermediate objects that are created in a way that is not always memory-efficient.

- 5,773.5 MiB: Here I dropped the `id` variable but  memory requirements stayed the same. I'm not entirely sure why this is, but this doesn't happen if you don't make all the copies of the ID variables we made (e.g. try dropping `id` right after creation and then run `python: check_memory()`). My guess is that Stata is trying to be smart about your  memory requirements and is concluding that you might need it in the future.

- 5,012.7 MiB:  Dropped mata object `id`.

- 4,438.5 MiB: Dropped frame `id`.

- 613.2 MiB: Dropped Python object; again, the drop is disproportionate to what we might expect, shoring `python` has a hefty memory overhead from within Stata.
