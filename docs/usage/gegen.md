gegen
=====

Efficient implementation of by-able egen functions using C.

_Note for Windows users:_ It may be necessary to run `gtools, dependencies` at
the start of your Stata session.

Syntax
------

<p>
<span style="font-family:monospace">gegen [type] newvar = fcn(arguments) [if] [in] [, ///</spen>
</br>
<span style="font-family:monospace">&emsp;&emsp;&emsp; replace <a href="#compiled-functions">fcn_options</a> <a href="#gtools-options">gtools_options</a> ]
</p>

### Gtools options

- `verbose` prints some useful debugging info to the console.

- `benchmark` prints how long in seconds various parts of the program take to execute.

- `hashlib(str)` Custom path to spookyhash.dll

- `gtools_capture(str)`  The above 3 options are captured and not passed to
                                 egen in case the requested function is not
                                 internally supported by gtools. You can pass
                                 extra arguments here if their names conflict
                                 with captured gtools options.

Compiled functions
------------------

The functions listed here have been compiled and hence will run very quickly.
Functions not listed here hash the data and then call egen with by(varlist)
set to the hash, which is often faster than calling egen directly, but not
always. The functions here _should_ always be faster, however.

### Generate IDs

    group(varlist) [, missing counts(newvarname) fill(real)]
        may not be combined with by.  It creates one variable taking on
        values 1, 2, ... for the groups formed by varlist.  varlist may
        contain numeric variables, string variables, or a combination of
        the two.  The order of the groups is the order in which varlist
        appears in the data.  However, the user can specify:

            [+|-] varname [[+|-] varname ...]

        And the order will be inverted for variables that have -
        prepended.  missing indicates that missing values in varlist
        (either . or "") are to be treated like any other value when
        assigning groups, instead of as missing values being assigned to
        the group missing.

        You can also specify counts() to generate a new variable with the
        number of observations per group; by default all observations
        within a group are filled with the count, but via fill() the user
        can specify the value the variable will take after the first
        observation that appears within a group. The user can also
        specify fill(data) to fill the first Jth observations with the
        count per group (in the sorted group order) or fill(group) to
        keep the default behavior.

### Tag groups

    tag(varlist) [, missing]
        may not be combined with by.  It tags just 1 observation in each
        distinct group defined by varlist.  When all observations in a
        group have the same value for a summary variable calculated for
        the group, it will be sufficient to use just one value for many
        purposes.  The result will be 1 if the observation is tagged and
        never missing, and 0 otherwise.

        Note values for any observations excluded by either if or in are
        set to 0 (not missing).  Hence, if tag is the variable produced
        by egen tag = tag(varlist), the idiom if tag is always safe.
        missing specifies that missing values of varlist may be included.

### Summary stats

All the functions listed here allow `by(varlist)`. If this is not specified,
then operations are performed by row. `exp` must be a valid Stata espression
or a list of variables.

    first|last|firstnm|lastnm(exp)
        creates a constant (within varlist) containing the first, last,
        first non-missing, and last non-missing observation. The
        functions are analogous to those in collapse and not to those in
        egenmore.

    count(exp)
        creates a constant (within varlist) containing the number of
        nonmissing observations of exp.

    iqr(exp)
        creates a constant (within varlist) containing the interquartile
        range of exp.  Also see pctile().

    max(exp)
        creates a constant (within varlist) containing the maximum value
        of exp.

    mean(exp)
        creates a constant (within varlist) containing the mean of exp.

    median(exp)
        creates a constant (within varlist) containing the median of exp.
        Also see pctile().

    min(exp)
        creates a constant (within varlist) containing the minimum value
        of exp.

    pctile(exp) [, p(#)]
        creates a constant (within varlist) containing the #th percentile
        of exp.  If p(#) is not specified, 50 is assumed, meaning
        medians.  Also see median().

    sd(exp)
        creates a constant (within varlist) containing the standard
        deviation of exp.  Also see mean().

    total(exp) [, missing]
        creates a constant (within varlist) containing the sum of exp
        treating missing as 0.  If missing is specified and all values in
        exp are missing, newvar is set to missing.  Also see mean().

Description
-----------

gegen creates newvar of the optionally specified storage type equal to
fcn(arguments). Here fcn() is either one of the internally supported
commands above or a by-able function written for egen, as documented
above. Only egen functions or internally supported functions may be used
with egen.  If you want to generate multiple summary statistics from a
single variable it may be faster to use gcollapse with the merge option.

Depending on fcn(), arguments, if present, refers to an expression,
varlist, or a numlist, and the options are similarly fcn dependent.

Out of memory
-------------

(See also Stata's own discussion: help memory.)

There are many reasons for why an OS may run out of memory. The best-case
scenario is that your system is running some other memory-intensive
program.  This is specially likely if you are running your program on a
server, where memory is shared across all users. In this case, you should
attempt to re-run gegen once other memory-intensive programs finish.

If no memory-intensive programs were running concurrently, the second
best-case scenario is that your user has a memory cap that your programs
can use. Again, this is specially likely on a server, and even more
likely on a computing grid.  If you are on a grid, see if you can
increase the amount of memory your programs can use (there is typically a
setting for this). If your cap was set by a system administrator,
consider contacting them and asking for a higher memory cap.

If you have no memory cap imposed on your user, the likely scenario is
that your system cannot allocate enough memory for gegen. At this point
you have two options: One option is to try fegen or egen, which are
slower but using either should require a trivial one-letter change to the
code; another option is to re-write egen the data in segments (the
easiest way to do this would be to egen a portion of all rows at a time
and perform a series of append statements at the end.)

Replacing gegen with fegen or plain egen is not guaranteed to work. I
have not benchmarked memory use very extensively, but it is possible that
the latter use less memory. If all fail, you will have to perform the
task on segments of the data.

Examples
--------

You can download the raw code for the examples below
[here  <img src="https://upload.wikimedia.org/wikipedia/commons/6/64/Icon_External_Link.png" width="13px"/>](https://raw.githubusercontent.com/mcaceresb/stata-gtools/master/docs/examples/gegen.do)

```stata
. sysuse auto, clear
. gegen id   = group(foreign)
. gegen tag  = group(foreign)
. gegen sum  = sum(mpg), by(foreign)
. gegen sum2 = sum(mpg rep78), by(foreign)
. gegen p5   = pctile(mpg rep78), p(5) by(foreign)
```

The function can be any of the supported functions above.
It can also be any function supported by egen:

```stata
. webuse egenxmpl4, clear

. gegen hsum = rowtotal(a b c)
rowtotal() is not a gtools function and no by(); falling back on egen

. sysuse auto, clear
(1978 Automobile Data)

. gegen seq = seq(), by(foreign)
seq() is not a gtools function; will hash and use egen
```