Caution! Swapping
=================

### What is swapping?

RAM (memory) is both limited and essential for an operating system. By and large an OS will take one of two measures to avoid running out of memory and ensure it can continue to function:

1. Free up memory by Killing/stopping programs.

2. Move programs' data out of memory.

Swap is a dedicated space in a computer's storage that acts as a backup for situations where the OS needs to move data out of memory. From an OS' point of view, this option is much preferred: Programs can continue to run if their data is moved instead of deleted, and the memory they were using can be reallocated. So what's the catch?

Well, there's a reason programs use memory: It's orders of magnitude faster than using storage (even compared to more modern solutions like SSDs). In many cases, of course, you want program to keep running, even if they are doing do much more slowly. For analysis on big data, however, unexpectedly swapping can grind execution to a halt. There are certainly programs optimized to analyze data on disk, but in my experience if a program was written to analyze data in memory and it suddenly finds portions of its data in swap, it might simply not finish in any reasonable time frame.

### How to avoid it?

- Minimize your program's memory footprint.

- Get more memory! But easier said than done.

- Keep only the essential variables in memory.

- Chunk your program execution.

Sometimes high memory use is inevitable, but before looking for a solution that processes data on disk, there is one more thing to consider: Can you process your data separately for groups of observations (i.e. by row)? Stata stores data in row order, so this can be efficient both in terms of memory and speed.

!!! tip "Anecdote: Process data in chunks"
    I once had health insurance enrollment data on 80M individuals.
    For each person I had an ID and five variables for each month
    over the span of 10 years. The data had hundreds of variables and
    I wanted to reshape it long and collapse to annual enrollment.

    If you do some quick math, you'll realize the reshape would have to
    allocate billions of observations, which presented an issue from the
    POV of memory use. At the time I had to run this on a shared server,
    and I was concerned that even if this fit in memory, when anyone else
    ran a program with even modest memory requirements, my reshape would
    start to swap and then might never finish.

    The solution I cape up with was to process the data in chunks:
    At each iteration of the loop, I read in 8M individuals, reshaped
    the data, and collapsed to annual enrollment. This was possible
    each individual's enrollment was independent from the other, and
    suddenly memory requirements dropped by an order of magnitude.
