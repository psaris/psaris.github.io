---
title: "Building Histograms"
excerpt: "Presented at Hong Kong Functional User Group Meetup"
date: <span class="timestamp-wrapper"><span class="timestamp">&lt;2015-06-02 Tue&gt;</span></span>
categories: 
- Presentation
tags: 
- ascii 
- histogram 
- qtips
---


# Goals

-   Write in a functional style
-   Use higher order functions
-   Include math, stats, and finance
-   Have fun


# Overview

-   Histograms
-   Factoring the Algorithm
-   Histogram Implementation
-   Sample Plots
-   Further Reading


# Histograms

-   Graphical presentation of a data set's distribution
-   More descriptive than summary statistics
-   Chart granularity critically depends on the number of bins

There have been many attempts over the past 80 years to compute the
optimal number of bins given a specific dataset. There are also many
different ways to plot the histogram data.  If we factor the code
properly, we can compose a custom histogram function with exactly the
properties we want.

```q
\l qtips.q
q).util.use `.hist
`.
q)myhist:chart[bar"*";30] count each bgroup[sturges]@
```

-   Uniform Distribution
    
    ```q
    q)myhist 100?1f
    0.01976295| 11 "***********                   "
    0.1414961 | 10 "**********                    "
    0.2632293 | 14 "**************                "
    0.3849625 | 13 "*************                 "
    0.5066957 | 14 "**************                "
    0.6284289 | 15 "***************               "
    0.7501621 | 16 "****************              "
    0.8718953 | 6  "******                        "
    0.9936284 | 1  "*                             "
    ```

-   Normal Distribution
    
    ```q
    q)myhist .stat.bm 100?1f
    -2.013341 | 4  "****                          "
    -1.443895 | 11 "***********                   "
    -0.8744492| 22 "**********************        "
    -0.3050036| 26 "**************************    "
    0.2644421 | 17 "*****************             "
    0.8338878 | 12 "************                  "
    1.403333  | 6  "******                        "
    1.972779  | 1  "*                             "
    2.542225  | 1  "*                             "
    ```


# Factoring the Algorithm

-   Histogram Generator
    
    ```q
    / create range of n buckets between (s)tart and (e)nd
    nrng:{[n;s;e]s+til[1+n]*(e-s)%n}
    
    / group data by a (b)inning (f)unction
    bgroup:{[bf;x]
     b:nrng[bf x;min x;max x];
     g:group b bin x;
     g:b!x g til count b;
     g}
    
    / use (p)lotting (f)unction to chart (d)ata with max (w)idth
    chart:{[pf;w;d]
     n:"j"$(m&w)*n%m:max n:value d;
     d:d,'enlist each pf[w] each n;
     d}
    ```


# Histogram Implementation

-   Binning Algorithms
    
    ```q
    / square root bucket algorithm
    sqrtn:{ceiling sqrt count x}
    
    / sturges' bucket algorithm
    sturges:{ceiling 1f+2f xlog count x}
    
    / doane's bucket algorithm
    doane:{ceiling 1f+(2f xlog count x)+2f xlog 1f+abs nskew x}
    
    / scott's windowing algorithm
    scott:{nw[;x] 3.4908*sdev[x]*count[x] xexp -1f%3f}
    
    /freedman-diaconis windowing algorithm
    fd:{nw[;x] 2f*.stat.iqr[x]*count[x] xexp -1f%3f}
    ```

-   `sqrtn` - simplest algorithm (used by Excel)
-   `sturges` - (1926) assumes data is a normally distributed
-   `doane` - (1976) modified `sturges` for skewed data - `skew`
-   `scott` - (1979) mathematically rigorous because it uses `stdev`
-   `fd` - (1981) modified `scott` for skewed data - `iqr`

-   Plotting Algorithms
    
    ```q
    / bar-chart plotting function
    / (c)haracter, (w)indow size, (n)umber of points
    bar:{[c;w;n]w$n#c}
    
    / dot-chart plotting function
    / (c)haracter, (w)indow size, (n)umber of points
    dot:{[c;w;n]w$neg[n]$1#c}
    
    / use (p)lotting (f)unction to chart (d)ata with max (w)idth
    chart:{[pf;w;d]
     n:"j"$(m&w)*n%m:max n:value d;
     d:d,'enlist each pf[w] each n;
     d}
    ```

-   `bar` - generates a line of characters
-   `dot` - generates a single character
-   `chart` - generates a line of text for each bin


# Sample Plots

-   Basic Strurges Bar Chart
    
    ```q
    q)chart[bar"*";30] count each bgroup[sturges] x:exp .stat.bm 100?1f
    0.08791095| 66 "******************************"
    1.448485  | 19 "*********                     "
    2.80906   | 10 "*****                         "
    4.169634  | 1  "                              "
    5.530209  | 2  "*                             "
    6.890783  | 1  "                              "
    8.251358  | 0  "                              "
    9.611932  | 0  "                              "
    10.97251  | 1  "                              "
    ```

-   Robust Freedman-Diaconis Dot Chart
    
    ```q
    q)chart[bar"*";30] count each bgroup[fd] x
    0.08791095| 32 "******************************"
    0.7281813 | 29 "***************************   "
    1.368452  | 11 "**********                    "
    2.008722  | 13 "************                  "
    2.648992  | 7  "*******                       "
    3.289263  | 2  "**                            "
    3.929533  | 1  "*                             "
    4.569803  | 0  "                              "
    5.210073  | 3  "***                           "
    5.850344  | 0  "                              "
    6.490614  | 0  "                              "
    7.130884  | 1  "*                             "
    7.771155  | 0  "                              "
    8.411425  | 0  "                              "
    9.051695  | 0  "                              "
    9.691966  | 0  "                              "
    10.33224  | 0  "                              "
    10.97251  | 1  "*                             "
    ```

-   Scott Dot Chart with "@"
    
    ```q
    q)chart[dot"@";30] count each bgroup[scott] x
    0.08791095| 59 "                             @"
    1.29731   | 26 "            @                 "
    2.50671   | 9  "    @                         "
    3.716109  | 1  "@                             "
    4.925509  | 3  " @                            "
    6.134908  | 0  "                              "
    7.344308  | 1  "@                             "
    8.553707  | 0  "                              "
    9.763107  | 0  "                              "
    10.97251  | 1  "@                             "
    ```


# Further Reading

-   Hyndman, R.J. (1995) The problem with Sturges’ rule for constructing
    histograms, Technical report, Monash University.

-   Wand, M.P. (1995) Data-based choice of histogram
    bin-width. Technical report, Australian Graduate School of
    Management, University of NSW.

Hyndman dissects each of the binning functions and explains the
deficiencies of each.

Wand proposes what seems like the 'grand unified theory' of optimal
histogram bin-width computation.  The algorithm is much more complex
than what we've encountered so far (includes the use `fft`), but
matches Scott's rule under first order approximations.


<!----- Footnotes ----->

