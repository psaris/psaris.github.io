---
title: "Profiling Q Functions"
excerpt: "Presented at Kx Community NYC Meetup"
date: <span class="timestamp-wrapper"><span class="timestamp">&lt;2015-07-15 Wed&gt;</span></span>
categories: 
- Presentation
tags: 
- qtips 
- profile
---


# Why Q Tips?

-   Unique perspective
-   Biggest impact
-   Show. Don't tell


# Q Tips CEP

-   Running the CEP

```sh
$ q cep.q -p 5001 -w 200 -t 1000 -eod 16:00 -db qdb
```

-   Parameters

```q
q)p
     | ()
ref  | `:ref.csv
eod  | 0D16:00:00.000000000
db   | `:qdb
debug| 0b
log  | 2
```


# Tables

```q
q)price
id| px       time
--| --------------------------------------
0 | 110.1492 2015.07.15D10:06:04.631217000
1 | 80.12466 2015.07.15D10:06:04.631217000
2 | 10.02082 2015.07.15D10:06:04.631217000
3 | 4.986975 2015.07.15D10:06:04.631217000
4 | 1.957649 2015.07.15D10:06:04.631217000
```

```q
q)quote
id| bs  bp    ap     as  time
--| --------------------------------------------------
0 | 6   110.1 110.15 23  2015.07.15D10:06:04.631217000
1 | 7   80.1  80.15  12  2015.07.15D10:06:04.631217000
2 | 65  10.02 10.03  73  2015.07.15D10:06:04.631217000
3 | 51  4.98  4.99   225 2015.07.15D10:06:04.631217000
4 | 896 1.955 1.96   351 2015.07.15D10:06:04.631217000
```

```q
q)trade
id| ts  tp     time
--| ----------------------------------------
1 | 7   80.15  2015.07.15D10:06:04.631217000
2 | 14  10.03  2015.07.15D10:06:04.631217000
0 | 8   110.15 2015.07.15D10:06:04.631217000
4 | 322 1.96   2015.07.15D10:06:04.631217000
3 | 41  4.98   2015.07.15D10:06:03.631217000
```


# Functions

```q
q).sim.path
{[s;r;t]
 z:.stat.norminv count[t]?1f;
 p:prds .stat.gbm[s;r;deltas[first t;t]] z;
 p}
```

```q
q).sim.genp
{[id;S;s;r;dtm]
 t:abs type dtm;
 tm:("np" t in 12 14 15h)$dtm;
 p:S*path[s;r;tm%365D06];
 c:`id,`time`date[t=14h],`price;
 p:flip c!(id;dtm;p);
 p}
```

```q
q).sim.genq
{[ts;qs;p]
 q:p,'flip `bp`ap!tickrnd[ts] p `price;
 q:q,'flip `bs`as!1+count[p]?/:2#qs;
 q:`id`time`bs`bp`ap`as#q;
 q}
```

```q
q).sim.gent
{[pct;q]
 q:filter[pct] raze (-1_@[;`time;delay] q@) each group q `id;
 t:q,' flip `ts`tp!trd[n?0b;(n:count q)?1f] . q `bs`bp`ap`as;
 t:`id`time`ts`tp#t;
 t}
```


# Profiling

-   Line Level - Scripts
-   Function Level - CEP
-   Instrumentation


# Timing a Function

-   Create table to record name, start and end time
    
    ```q
    q)prof.events:flip `id`pid`func`time!"jjsn"$\:()
    q)prof.events
    id pid func time
    ----------------
    ```

-   Keep track of call id and parent call id
    
    ```q
    pid:id:0
    ```

-   Record the start time
-   Set call id and parent call id
-   Call function (which may call other functions)
-   Reset parent call id
-   Record the end time and save the results

```q
.prof.time:{[n;f;a]
 s:.z.p;
 id:.prof.id+:1;
 pid:.prof.pid;
 .prof.pid:id;
 r:f . a;
 .prof.pid:pid;
 `prof.events upsert (id;pid;n;.z.p-s);
 r}
```

```q
q).prof.time[`foo;{x+y};1 2]
3
q)prof.events
id pid func time
--------------------------------
1  0   foo  0D00:00:00.000028000
```


# Instrumenting a Function

-   Obtain the \*\*m\*\*eta information from a \*\*f\*\*unction \*\*n\*\*ame
-   Change to the function's directory
-   Redefine the function as a composition of <del>.prof.time</del> and <del>enlist</del>
-   Change back to root directory

```q
.prof.instr:{[n]
 m:get f:get n;
 system "d .",string first m 3;
 n set (')[.prof.time[n;f];enlist];
 system "d .";
 n}
```

```q
q)f:{x+y}
q).prof.instr `f
`f
```

```q
q)f
{[n;f;a]
 s:.z.p;
 id:.prof.id+:1;
 pid:.prof.pid;
 .prof.pid:id;..enlist
q)type f
105h
```

```q
q)count get f
2
q)first get f
{[n;f;a]
 s:.z.p;
 id:.prof.id+:1;
 pid:.prof.pid;
 .prof.pid:id;
 r:f . a;
 .prof.pid:pid;
 `prof.events upsert (id;pid;n;.z.p-s);
 r}[`f;{x+y}]
q)last get f
enlist
```

```q
q)enlist[1;2;3;4;5;6;7;8;9;10]
1 2 3 4 5 6 7 8 9 10
```


# Profiling the CEP Engine

```q
q).prof.instrall`
`..genu`..main`.util.use`.util.wday`.util.rng`.util.rnd`.util.ran..
```

```q
q)prof.rpt
func          | time  n   nc timepc      pct
--------------| ----------------------------------
.timer.run    | 9.751 118 1  0.08263559  34.30914
.md.updq      | 3.714 40  1  0.09285     13.0678
.md.updp      | 3.241 50  2  0.06482     11.40354
.md.updt      | 2.73  28  1  0.0975      9.605573
.timer.until  | 2.44  118 1  0.02067797  8.585201
.stat.tnorminv| 1.764 50  1  0.03528     6.206678
.stat.cnorminv| 1.576 50  2  0.03152     5.545195
.stat.norminv | 1.411 50  2  0.02822     4.964639
.stat.horner  | 0.95  150 0  0.006333333 3.342599
.stat.gbm     | 0.496 50  0  0.00992     1.745188
.sim.tickrnd  | 0.201 40  0  0.005025    0.7072235
.sim.trd      | 0.147 28  0  0.00525     0.5172232
```


# Thank You

-   [Q Tips](http://q-tips.net)

-   [Q Tips Review](http://archive.vector.org.uk/art10501500)


<!----- Footnotes ----->

