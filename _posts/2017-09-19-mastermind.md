---
title: "Functional Solutions to Mastermind in Q"
excerpt: "Presented at Kx Community NYC Meetup"
date: <span class="timestamp-wrapper"><span class="timestamp">&lt;2017-09-19 Tue&gt;</span></span>
categories: 
- Competition
tags: 
- mastermind 
- entropy
---


# Mastermind

-   A popular code-breaking game sold since 1975
-   The "codemaker" picks a 4-color code
-   The "codebreaker" attempts to discover the code by submitting guesses
-   Responses provide two sets of information
    -   the number correct colored pegs in the correct position
    -   the number of correct colored pegs in the wrong position
-   A winning response would be (4,0)
-   Donald E. Knuth published a 5-move algorithm in 1976
-   Better algorithms have since been published that:
    -   achieve lower average move counts
    -   but require some solution to use more than 5 moves
-   Some editions increase the number of pegs and/or colors


# Permutations (with repeat)

The classic Mastermind game allows the code to have repeated colors.

```q
perm:{$[x>0;(cross/)x#enlist y;{raze x{x,/:y except x}\:y}[;y]/[-1-x;y]]}
,,,,
```

Taking 4 pegs from 6 colors generates 1296 permutations:

```q
q)show C:.mm.perm[4] "123456"
"1111"
"1112"
"1113"
"1114"
"1115"
"1116"
"1121"
..
```


# Permutations (without repeat)

Preventing repeats shrinks the solution space to 360 permutations:

```q
q).mm.perm[-4] "123456"
"1234"
"1235"
"1236"
"1243"
"1245"
"1246"
"1253"
..
```

Increasing the number of colors from 6 to 8 brings the complexity of
the problem close to the classic game:

```q
q)count .mm.perm[-4] "12345678"
1680
```


# Scoring

```q
/ drop the first instance of y in x
drop:{x _ x ? y}
```

```q
/ vectorize an atomic function
veca:{[f;x;y]$[type x;$[type y;f[x;y];x f/: y];type y;x f\: y;x f/:\: y]}
```

```q
/ x,y = score,guess in any order
scr:{(e;count[x]-(e:"j"$sum x=y)+count x drop/ y)}
score:veca scr
```

```q
q).mm.score["1123";("1234";"5432")]
1 2
0 2

```


# Score Distributions

```q
freq:count each group@          / frequency distribution
hist:freq asc@                  / histogram
```

```q
q).mm.hist .mm.score[C] "1234"
0 0| 16
0 1| 152
0 2| 312
0 3| 136
0 4| 9
1 0| 108
1 1| 252
1 2| 132
1 3| 8
2 0| 96
2 1| 48
2 2| 6
3 0| 20
4 0| 1
```


# Peg Distributions


## First Guess

```q
/ compute the frequency distribution of x with (c)olumn names
freqdist:{[c;x]([]x:u)!flip c!freq'[x]@\:u:asc distinct raze x}
```

```q
q)G:("1111";"1112";"1122";"1123";"1234")
q)show T:`score xcol .mm.freqdist[`$G] .mm.score[G;C]
score| 1111 1112 1122 1123 1234
-----| ------------------------
0 0  | 625  256  256  81   16  
0 1  |      308  256  276  152 
0 2  |      61   96   222  312 
0 3  |           16   44   136 
0 4  |           1    2    9   
1 0  | 500  317  256  182  108 
1 1  |      156  208  230  252 
1 2  |      27   36   84   132 
1 3  |                4    8   
2 0  | 150  123  114  105  96  
2 1  |      24   32   40   48  
2 2  |      3    4    5    6   
3 0  | 20   20   20   20   20  
4 0  | 1    1    1    1    1   
```


## MINIMAX (knuth)

```q
q)T upsert (1 2#0N),value max T
score| 1111 1112 1122 1123 1234
-----| ------------------------
0 0  | 625  256  256  81   16  
0 1  |      308  256  276  152 
0 2  |      61   96   222  312 
0 3  |           16   44   136 
0 4  |           1    2    9   
1 0  | 500  317  256  182  108 
1 1  |      156  208  230  252 
1 2  |      27   36   84   132 
1 3  |                4    8   
2 0  | 150  123  114  105  96  
2 1  |      24   32   40   48  
2 2  |      3    4    5    6   
3 0  | 20   20   20   20   20  
4 0  | 1    1    1    1    1   
     | 625  317  256  276  312 
```


## IRVING (min expected size)

```q
q)show T upsert (1 2#0N),value "j"$T wavg T
score| 1111 1112 1122 1123 1234
-----| ------------------------
0 0  | 625  256  256  81   16  
0 1  |      308  256  276  152 
0 2  |      61   96   222  312 
0 3  |           16   44   136 
0 4  |           1    2    9   
1 0  | 500  317  256  182  108 
1 1  |      156  208  230  252 
1 2  |      27   36   84   132 
1 3  |                4    8   
2 0  | 150  123  114  105  96  
2 1  |      24   32   40   48  
2 2  |      3    4    5    6   
3 0  | 20   20   20   20   20  
4 0  | 1    1    1    1    1   
     | 512  236  205  185  188 
```


## MAXENT (maximum entropy)

```q
q)show T upsert (1 2#0N),value "j"$100*.mm.entropy each flip value T
score| 1111 1112 1122 1123 1234
-----| ------------------------
0 0  | 625  256  256  81   16  
0 1  |      308  256  276  152 
0 2  |      61   96   222  312 
0 3  |           16   44   136 
0 4  |           1    2    9   
1 0  | 500  317  256  182  108 
1 1  |      156  208  230  252 
1 2  |      27   36   84   132 
1 3  |                4    8   
2 0  | 150  123  114  105  96  
2 1  |      24   32   40   48  
2 2  |      3    4    5    6   
3 0  | 20   20   20   20   20  
4 0  | 1    1    1    1    1   
     | 150  269  289  304  306 
```


## MOSTPARTS (most partitions)

```q
q)show T upsert (1 2#0N),value sum 0<T
score| 1111 1112 1122 1123 1234
-----| ------------------------
0 0  | 625  256  256  81   16  
0 1  |      308  256  276  152 
0 2  |      61   96   222  312 
0 3  |           16   44   136 
0 4  |           1    2    9   
1 0  | 500  317  256  182  108 
1 1  |      156  208  230  252 
1 2  |      27   36   84   132 
1 3  |                4    8   
2 0  | 150  123  114  105  96  
2 1  |      24   32   40   48  
2 2  |      3    4    5    6   
3 0  | 20   20   20   20   20  
4 0  | 1    1    1    1    1   
     | 5    11   13   14   14  
```


# Filtering

Filter guess list to only those that would produce returned score

```q
/ unused (C)odes, viable (G)uesses, next (g)uess, (s)core
filt:{[C;G;g;s](drop[C;g];G where (s~score[g]@) each G)}
```

```q
q)last .mm.filt[C;C;"1234";1 1]
"1112"
"1113"
"1121"
"1122"
"1125"
"1126"
"1141"
..
```


# Guess Algorithms

Given the frequency distribution of the remaining guesses, we have
four one-step algorithms to pick the next best guess.

```q
minimax:{x=min x:max each x}       / min max size (knuth)
irving:{x=min x:{x wavg x} each x} / min expected size
maxent:{x=max x:entropy each x}    / max entropy
maxparts:{x=max x:count each x}    / most parts
```

```q
q)C where .mm.maxent .mm.freq each .mm.score[C;C]
"1234"
"1235"
"1236"
"1243"
"1245"
"1246"
"1253"
..
```


# Best Guess

```q
/ use (f)unction to filter all unpicked (C)odes for best split. 
/ pick a solution from viable (G)uesses (if possible)
best:{[f;C;G]first $[3>count G;G;count G:G inter C@:where f freq each score[C;G];G;C]}
```

```q
q)CG:.mm.filt[C;C;"1234";1 1]
q).mm.best[;CG 0;CG 1] each `.mm.minimax`.mm.irving`.mm.maxent`.mm.maxparts
"1135"
"1256"
"1356"
"1125"
```


# A Game

-   A turn calls the algorithm and returns the score of the guess.
    
    ```q
    turn:{[a;c;CGgs] CGg,enlist score[c]last CGg:a CGgs}
    ```

-   A game keeps taking turns until a perfect score is reached.
    
    ```q
    game:{[a;C;g;c](not count[g]=first last@) turn[a;c]\(C;C;g;score[c;g])}
    ```

-   A game summary returns the number of viable guesses, the actual
    guess and the resulting guess.
    
    ```q
    summary:{[CGgs]`n`guess`score!(count CGgs 1),-2#CGgs}
    ```


# An Interactive Game

With these abstractions, we can compose a new game which allows us to
pass each guess in from STDIN.

```q
q).mm.summary each .mm.game[.mm.stdin[.mm.onestep[`.mm.maxent]];C;"1234"] rand C
n    guess  score
-----------------
1296 "1234" 0 2  
guess (HINT 2356): 2356
n   guess  score
----------------
312 "2356" 1 0  
guess (HINT 4164): 4164
n  guess  score
---------------
22 "4164" 3 0  
guess (HINT 4166): 4166
n    guess  score
-----------------
1296 "1234" 0 2  
312  "2356" 1 0  
22   "4164" 3 0  
1    "4166" 4 0  
```


# Faster

Calculating the average guess count for a given algorithm involves
iterating over every possible score.

-   The slowest part of the process is the scoring algorithm.
-   What if we pre-calculated each of the scores?
    
    ```q
    q)score:C!C!/:C .mm.score\:/: C
    q)C score\:/: C
    'rank
      [0]  C score\:/: C
    ```
-   This works for matrices though?!
    
    ```q
    x:(1 2 3;4 5 6;7 8 9)
    q)0 1 2 x/:\: 2 1 0
    3 2 1
    6 5 4
    9 8 7
    ```


# Vector Indexing

The solution is vector indexing.

-   Vector indexing works for matrices,
    
    ```q
    q)x[0 1 2;2 1 0]
    3 2 1
    6 5 4
    9 8 7
    ```

-   and dictionaries - because vector indexing is implemented as 'each
    right' - 'each left'.
    
    ```q
    q) score[C;C]
    4 0 3 0 3 0 3 0 3..
    3 0 4 0 3 0 3 0 3..
    3 0 3 0 4 0 3 0 3..
    3 0 3 0 3 0 4 0 3..
    3 0 3 0 3 0 3 0 4..
    3 0 3 0 3 0 3 0 3..
    3 0 2 2 2 1 2 1 2..
    ..
    ```

== Algorithm Comparison

With the speed improvement gained through caching the scoring
function, we can now traverse all paths and see which algorithm takes
the least turns (on average)footnote:[in 1993, Kenji Koyama and
Tony W. Lai found a method that required an average of 5625/1296 =
4.340 turns to solve, with a worst-case scenario of six turns. The
game theory optimal value is 5600/1296 = 4.321.].

```q
q)D:`turns xcol .mm.freqdist[`simple`minimax`irving`maxent`maxparts] (a;b;c;d;e)
q)show ("f"$D) upsert 0N,value[flip value D] wavg\: key[D]`turns
turns| simple  minimax irving   maxent   maxparts
-----| ------------------------------------------
1    | 1       1       1        1        1       
2    | 4       6       10       4        12      
3    | 25      62      54       71       72      
4    | 108     533     645      612      635     
5    | 305     694     583      596      569     
6    | 602             3        12       7       
7    | 196                                       
8    | 49                                        
9    | 6                                         
     | 5.76466 4.47608 4.395062 4.415123 4.373457
```


# Solutions

```c++
#include"k.h"
#include <string.h>

// x,y: 4-digit char vector representing mastermind code and guess
// returns (# correct value,position;# correct value wrong position)
K2(score) {
  K r;
  I i,j,e,n;
  char X[5], Y[5];

  P(xt != KC || y->t != KC, krr("type"));
  P(xn !=  4 || y->n !=  4, krr("length"));

  memcpy(X,kC(x),4);            // copy data before manipulating
  memcpy(Y,kC(y),4);

  i=0;
  n=xn;
  do {                
    if (X[i] == Y[i]){          // first check for correct positions
      memmove(X+i,X+i+1,--n);   // remove matches from list
      memmove(Y+i,Y+i+1,n);
    } else
      i++;
  } while(i<n);

  e=xn-n;                       // record exact matches

  // now we check for matches but in the wrong position
  if(n>1) // can't have wrong position unless 2 or more values remain
    for(i=n-1;i>=0;--i)
      for(j=0;j<n;++j)
        if(X[i] == Y[j]){
          memmove(Y+j,Y+j+1,--n); // remove match from list
          break;
        }

  r=ktn(KI,2);
  kI(r)[0]=e;
  kI(r)[1]=xn-e-n;
  R r;
}
```

```q
score:{n,4-(n:sum x=y)+count{x _x?y}/[x;y]}
```

```q
score:{(4-c),$[f2~df:distinct f2:f where not (c:count w)=f:(x@:w)?y@:w:where not x=y;
  count df;
  count[df]+sum @[y;f?df;:;"_"] in @[x;df;:;" "]
  ]}
```

```q
score:{n:0 0 0 0 0 0 0 0 0 0 0 0i;n[-49 -49 -49 -49 -43 -43 -43 -43i+"i"$x,y]+:1i;b,(sum(&). 0 6_n)-b:sum x=y}
```

```q
k)score:{[x;y;z;w]y@6/:x w,z}[@[&,55;"123456";:;!6];{(+/m;0+/&/g@\:!*g:(#:'=:)'(y n;x n:&~m:x=y))}.',/(,\:/:)/2#,,:'.q.cross/4#,"123456"]
```


<!----- Footnotes ----->

