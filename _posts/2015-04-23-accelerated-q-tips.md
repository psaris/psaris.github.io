---
title: "Accelerated Q Tips"
excerpt: "Presented at Kx Hong Kong User Group Meeting"
date: <span class="timestamp-wrapper"><span class="timestamp">&lt;2015-04-23 Thu&gt;</span></span>
categories: 
- Presentation
tags: 
- qtips 
- profile 
- ascii
---


# Why Q Tips?

-   Unique perspective
-   Biggest impact
-   Show. Don't tell


# What is Q Tips?

-   Learn by example
-   Q-SQL starts with q
-   Q is fun

```sh
$ $QHOME/$QARCH/q cep.q -help
KDB+ 3.2 2015.03.04 Copyright (C) 1993-2015 Kx Systems
m32/ 4()core 4096MB nick nick-macbook-pro.local 172.30.0.169 NONEXPIRE  

usage: q cep.q [option]...
       -ref   <file with reference data>  (`:ref.csv)           
       -eod   <time for end of day event> (0D23:59:00.000000000)
       -db    <end of day dump location>  (`:db)                
       -debug <don't start engine>        (0b)                  
       -log   <log level>                 (2)                   
```


# Normal Random Variables

-   Uniforms
    
    ```q
    q).hist.fdhist 100?1f
    0.001110398| 21 "*********************         "
    0.1991757  | 21 "*********************         "
    0.397241   | 21 "*********************         "
    0.5953062  | 17 "*****************             "
    0.7933715  | 19 "*******************           "
    0.9914368  | 1  "*                             "
    ```

-   12 Uniforms
    
    ```q
    u12:{-6f+sum x cut (12*x)?1f}
    ```
    
    ```q
    q).hist.fdhist .stat.u12 100
    -2.633577 | 1  "*                             "
    -2.141205 | 3  "***                           "
    -1.648833 | 8  "********                      "
    -1.156461 | 6  "******                        "
    -0.664089 | 20 "********************          "
    -0.1717169| 12 "************                  "
    0.3206552 | 29 "***************************** "
    0.8130273 | 8  "********                      "
    1.305399  | 10 "**********                    "
    1.797772  | 2  "**                            "
    2.290144  | 1  "*                             "
    ```

-   Box-Muller
    
    ```q
    bm:{
     if[count[x] mod 2;'`length];
     x:2 0N#x;
     r:sqrt -2f*log x 0;
     theta:2f*acos[-1]*x 1;
     x: r*cos theta;
     x,:r*sin theta;
     x}
    ```

-   Use `if` statements to exit functions early
-   Use compound assignment for increased efficiency
-   Use the closing brace to document a function's return value


# Simulated Security Paths

-   Geometric Brownian Motion
    
    ```q
    gbm:{[s;r;t;z]exp(t*r-.5*s*s)+z*s*sqrt t}
    ```

-   Chain functions
    
    ```q
    q)s:.3;r:.05;dt:.util.wday 2001.01.01+til 365
    q)tm:deltas[first dt;dt]%365.25
    q)100*prds .stat.gbm[s;r;tm] .stat.norminv (count dt)?1f
    100 100.1525 98.91791 97.59687 96.99619 94.84253 93.80587 92.75921 91.29922..
    ```

-   Use the dyadic form of <del>deltas</del>, <del>ratios</del>, <del>differ</del> and <del>prev</del>
-   Factor algorithms into small reusable pieces
-   Reserve the last function parameter for data


# Simulations

-   Tables
    
    ```q
    q).sim.genp[0;100;.3;.03] .util.rng[1;2000.01.01;2001.01.01]
    id date       price   
    ----------------------
    0  2000.01.01 100     
    0  2000.01.02 98.69405
    0  2000.01.03 98.27505
    0  2000.01.04 98.52507
    0  2000.01.05 99.09263
    ..
    ```
    
    ```q
    q).sim.genp[0;100;.3;.03] .util.rng[1;09:00;16:00]
    id time  price
    -----------------
    0  09:00 100
    0  09:01 100.0323
    0  09:02 99.98794
    0  09:03 99.95417
    0  09:04 99.99175
    ..
    ```

-   Eaching
    
    ```q
    q)n:10
    q)dts:.util.rng[1;2000.01.01;2001.01.01]
    q)2!raze sim.genp[;;;;dts]'[til n;n?100;n?.3;n?.03]
    id date      | price
    -------------| --------
    0  2000.01.01| 89
    0  2000.01.02| 88.66867
    0  2000.01.03| 88.52082
    0  2000.01.04| 87.29726
    0  2000.01.05| 85.9732
    ..
    ```

-   Return unkeyed tables by default


# Attributes

-   Sorted
    
    ```q
    q)select avg px by id from prices
    id| px      
    --| --------
    0 | 109.9929
    1 | 80.01755
    2 | 10.00047
    3 | 5.000112
    4 | 1.999113
    ```
    
    ```q
    q)meta select avg px by id from prices
    c | t f a
    --| -----
    id| j   s
    px| f
    ```

-   Unique
    
    ```q
    q)n:10000
    q)d1:(k:til n)!v:n?1f
    q)show d2:(`u#k)!v
    0| 0.3425698
    1| 0.4950533
    2| 0.360046
    3| 0.2700259
    ..
    ```
    
    ```q
    q)\t:10000 d1 5000
    90
    q)\t:10000 d2 5000
    7
    ```

-   Use the `` `u `` attribute on dictionary keys to increase performance
-   Use `\t:n` and `\ts:n` to time multiple runs of a single command

-   Partition
    
    ```q
    q) select avg px by id,time.minute from prices
    id minute| px      
    ---------| --------
    0  22:54 | 110.0092
    0  22:55 | 110.0051
    0  22:56 | 109.9962
    1  22:54 | 80.00557
    1  22:55 | 80.02086
    ..
    ```
    
    ```q
    q)meta select avg px by id,time.minute from prices
    c     | t f a
    ------| -----
    id    | j   p
    minute| u    
    px    | f    
    ```

-   Group
    
    ```q
    q)show trade:flip`id`time`price`size!"jpfi"$\:()
    id time price size
    ------------------
    ```
    
    ```q
    sattr:{[t]
     c:first cols t;
     a:`g`u 1=n:count keys t;
     t:n!@[;c;a#]0!t;
     t}
    ```
    
    ```q
    q)meta .util.sattr trade
    c    | t f a
    -----| -----
    id   | j   g
    time | p    
    price| f    
    size | i    
    ```


# Compression

-   Tree
    
    ```q
    tree:{$[x~k:key x;x;11h=type k;raze (.z.s ` sv x,) each k;()]}
    ```
    
    ```q
    q).util.tree `:qdb
    `:qdb/2015.03.03/price/.d`:qdb/2015.03.03/price/id`:qdb/2015.03.03/price/px..
    q).util.tree `
    `.q.`.q.neg`.q.not`.q.null`.q.string`.q.reciprocal`.q.floor`.q.ceiling`.q.s..
    ```

-   Compress
    
    ```q
    q).z.zd:20 2 9
    q){x set get x} each .util.tree `:qdb
    `:qdb/2015.03.06/prices/.d`:qdb/2015.03.06/prices/id`:qdb/2015.03.06/prices..
    ```

-   Use `.z.s` to make recursive function calls


# q-SQL

-   Verbose use of `select`
    
    ```q
    q)s:select id,date.week,p:price from t
    q)select o:first p,h:max p,l:min p,c:last p by id,week from s
    id week      | o        h        l        c
    -------------| -----------------------------------
    0  1999.12.27| 100      100.0632 100      100.0632
    0  2000.01.03| 100.122  100.122  97.507   99.71328
    0  2000.01.10| 99.36008 99.63464 95.70985 96.14122
    0  2000.01.17| 96.24896 99.87957 95.87985 99.87957
    0  2000.01.24| 98.72707 103.3749 98.72707 102.9145
    ..
    ```

-   Elegant use of <del>exec</del>
    
    ```q
    ohlc:{`o`h`l`c!(first;max;min;last)@\:x}
    ```
    
    ```q
    q)exec .stat.ohlc price by id,date.week from t
    id week      | o        h        l        c
    -------------| -----------------------------------
    0  1999.12.27| 100      100.0632 100      100.0632
    0  2000.01.03| 100.122  100.122  97.507   99.71328
    0  2000.01.10| 99.36008 99.63464 95.70985 96.14122
    0  2000.01.17| 96.24896 99.87957 95.87985 99.87957
    0  2000.01.24| 98.72707 103.3749 98.72707 102.9145
    ..
    ```

-   Factor complex q-SQL statements into functions
-   Simplify queries by using `exec` `by`


# Pivot

-   Simple interface
    
    ```q
    pivot:{[t]
     u:`$string asc distinct last f:flip key t;
     pf:{x#(`$string y)!z};
     p:?[t;();g!g:-1_ k;(pf;`u;last k:key f;last key flip value t)];
     p}
    ```

-   Flexible use
    
    ```q
    q)"i"$.util.pivot select by id,date.year from t
    id| 2000 2001 2002 2003 2004
    --| ------------------------
    0 | 95   130  233  224  237
    1 | 138  165  229  205  239
    2 | 121  121  88   105  84
    3 | 97   62   117  146  237
    4 | 67   84   93   147  156
    ..
    ```

-   Use functional <del>select+/+update</del> to parameterize <del>by</del> clauses

-   Use it everywhere
    
    ```q
    q)"i"$.util.pivot select by date.year,id from t
    year| 0   1   2   3   4   5  6  7  8   9
    ----| ------------------------------------
    2000| 95  138 121 97  67  70 57 87 102 100
    2001| 130 165 121 62  84  64 44 72 161 103
    2002| 233 229 88  117 93  65 32 53 159 112
    2003| 224 205 105 146 147 57 26 56 291 130
    2004| 237 239 84  237 156 63 30 55 256 85
    ```

-   Only transform tables into pivot tables for presentation


# Grid Computing

-   Multiple Slaves
    
    ```q
    $ q qdb -p 5000 -s -4
    KDB+ 3.2 2015.03.04 Copyright (C) 1993-2015 Kx Systems
    m32/ 4()core 2048MB nick nicks-macbook.local 192.168.1.103 NONEXPIRE
    q)(system "q . -p ",) each string p:system["p"]+1+til neg system"s"
    q).z.pd:`u#hopen each p
    ```

-   Multi-process <del>peach</del>
    
    ```q
    q)select pid:.z.i by date from trades
    date      | pid  
    ----------| -----
    2015.03.03| 52436
    2015.03.04| 52438
    2015.03.05| 52440
    2015.03.06| 52436
    ```


# Profiling

-   Timing
    
    ```q
    time:{[n;f;a]
     s:.z.p;
     id:.prof.id+:1;
     pid:.prof.pid;
     .prof.pid:id;
     r:f . a;
     .prof.pid:pid;
     `prof.events upsert (id;pid;n;.z.p-s);
     r}
    ```

-   Instrumenting
    
    ```q
    instr:{[n]
     m:get f:get n;
     system "d .",string first m 3;
     n set (')[.prof.time[n;f];enlist];
     system "d .";
     n}
    ```

-   Recording
    
    ```q
    q)prof.events
    id pid func         time                
    ----------------------------------------
    4  3   .sim.tickrnd 0D00:00:00.000010000
    3  2   .md.updq     0D00:00:00.000235000
    2  1   .timer.until 0D00:00:00.000276000
    1  0   .timer.run   0D00:00:00.000442000
    8  7   .sim.tickrnd 0D00:00:00.000007000
    ..
    ```

-   Reporting
    
    ```q
    q)prof.rpt
    func        | time     n     nc timepc     pct
    ------------| -------------------------------------
    .timer.merge| 1590.278 25553 0  0.06223449 19.18796
    .timer.run  | 1378.854 25553 2  0.05396055 16.63696
    .md.updp    | 1246.257 16400 2  0.07599128 15.03708
    .md.updq    | 841.354  6178  1  0.1361855  10.1516
    .stat.horner| 690.185  49200 0  0.01402815 8.327628
    ..
    ```


# Derivative Pricing

-   Monte Carlo
    
    ```q
    mc:{[S;s;r;t;pf;n]
     z:.stat.norminv n?/:count[t]#1f;
     f:S*.stat.gbm[s;r;deltas[first t;t]] z;
     v:pf[f]*exp neg r*last t;
     v}
    ```

-   Payoffs
    
    ```q
    eu:{[c;k;f]0f|$[c;last[f]-k;k-last f]}
    ```
    
    ```q
    q).util.use `.deriv;
    q)c:1b;S:100;k:90;s:.2;r:.03
    q)t:til[252]%251;n:10000
    q)mcstat raze mc[S;s;r;t;eu[c;k]] peach 20#n
    ev | 15.41932
    err| 0.0741624
    n  | 200000
    ```

-   Use scalar conditional <del>$[;;]</del> to implement lazily evaluated blocks
-   Use `peach` to run computations in parallel

-   Up and Out Barrier Option
    
    ```q
    bo:{[bf;pf;f]bf[f]*pf f}
    ```
    
    ```q
    q)c:1b;S:100;k:90;s:.2;r:.03
    q)mcstat raze mc[S;s;r;t;bo[all 120>]eu[c;k]] peach 20#n
    ev | 3.990454
    err| 0.02974377
    n  | 200000
    ```

-   Use mathematical operations instead of conditionals


# Histograms

-   Reusable Components
    
    ```q
    q).util.use `.hist
    q)chart[bar"-";30] count each bgroup[sqrtn] .stat.bm 100?1f
    -2.498147 | 1  "-                             "
    -1.974753 | 8  "--------                      "
    -1.451358 | 8  "--------                      "
    -0.9279628| 14 "--------------                "
    -0.4045679| 16 "----------------              "
    0.118827  | 22 "----------------------        "
    0.6422219 | 16 "----------------              "
    1.165617  | 10 "----------                    "
    1.689012  | 3  "---                           "
    2.212407  | 2  "--                            "
    2.735802  | 0  "                              "
    ```

-   Encourages Reuse
    
    ```q
    q)chart[dot"*";30] count each bgroup[doane] .stat.bm 100?1f
    -2.45497  | 2  " *                            "
    -1.871575 | 5  "    *                         "
    -1.28818  | 14 "             *                "
    -0.7047846| 22 "                     *        "
    -0.1213896| 25 "                        *     "
    0.4620055 | 14 "             *                "
    1.045401  | 9  "        *                     "
    1.628796  | 7  "      *                       "
    2.212191  | 2  " *                            "
    2.795586  | 0  "                              "
    ```

-   Pursue the functional-vector solution


# Thank You

-   [Q Tips](http://q-tips.net)

-   [Q Tips Review](http://archive.vector.org.uk/art10501500)


<!----- Footnotes ----->

