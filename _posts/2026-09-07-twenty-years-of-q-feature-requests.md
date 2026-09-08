---
title: "Twenty Years of Q Feature Requests"
excerpt: "Presented to Iverson College 2026"
date: <span class="timestamp-wrapper"><span class="timestamp">&lt;2026-09-07 Mon 13:30&gt;</span></span>
categories: 
- Presentation
tags: 
- guid 
- xbar 
- qcon 
- timespan 
- wsum
---


# Welcome

-   Twenty years of emails to KX
-   Many new features were added
-   I highlight a few
-   And then move on to others that have not been added (yet)


# Biases and Caveats

-   My feature requests are typically for language additions
-   Not database performance tweaks
-   I love Q's brevity
-   I am always looking for ways to add new, useful operations
-   I appreciate the need for backward compatibility
-   `'type` errors are places where new functionality can be added
    without breaking existing functionality
-   Efficiency and concision make feature requests easier to justify
-   The obviously good requests were implemented immediately
-   But others were either legitimately denied, left in limbo, or never
    responded to


# Implemented


## Automatic Attributes

-   `select by` queries guarantee that the first column is sorted
-   Why not make that contractual and automatically apply attributes

-   **The Ask:** apply `` `s `` and `` `p `` attributes to the first column of a
    `select by` statement making all subsequent joins faster
    -   Single keyed table gets an `` `s `` attribute
        
        ```q
        q)meta select by a from ([]a:1 2;b:1 2)
        c| t f a
        -| -----
        a| j   s
        b| j    
        ```
    -   Multi-keyed table gets a `` `p `` attribute
        
        ```q
        q)meta select by a,b from ([]a:1 2;b:1 2;c:1 2)
        c| t f a
        -| -----
        a| j   p
        b| j    
        c| j    
        ```

-   **Status:** Implemented - Q now adds `` `s `` for single keyed tables and
    `` `p `` to multi-keyed tables that are a result of `select by`


## `xbar` on Timespans

-   Timestamps and timespans were added to the language, but not all
    operators were updated to support them
-   `xbar` is the most obvious and flexible way to subsample timeseries data
-   **The Ask:** extend `xbar` to handle timespans buckets
-   Adding a check within `xbar` to cast timespans to a long solved it!
    
    ```q
    q)xbar
    k){$[9h=t:abs[@x];x*r+y=x*1+r:(y:9h$y)div x;x*y div x:$[16h=t;"j"$x;x]]}
    ```

-   This allows you to choose any level of timespan granularity without having to cast the time to match
    
    ```q
    q)0D01 xbar 0N!2000.01.01 + 3?0D
    2000.01.01D17:04:05.230387151 2000.01.01D09:52:41.978846043 2000.01.01D11:50:11.052963137
    2000.01.01D17:00:00.000000000 2000.01.01D09:00:00.000000000 2000.01.01D11:00:00.000000000
    ```

-   **Status:** Implemented - xbar handles timespan granularity directly, no cast needed


## Native Joins

-   Writing a high frequency option market making system in q, I
    needed every operation to be as fast as possible
-   Each Q operator added interpreter overhead
-   I wanted a native `lj`
-   The 2.8 `lj` did too much &#x2013; including filling nulls!
    
    ```q
    lj:{$[`s=-2!y;
      aj[!+!y;x;0!y];
      .Q.ft[{
        $[&/j:(#y:. y)>i?:(!+i:!y)#x;
         .Q.fl[x]y i;
         +.[+x;(f;j);:;.+.Q.fl[((f:!+y)#x:.Q.ff[x]y)j]y i j:&j]
         ]
        }[;y]]x]}
    ```

-   **The Ask:** make `,` and `,\:` perform a left join (without filling
    nulls)
    -   Dictionaries
        
        ```q
        q)([a:1]) , ([a:1 3]b:2 3)
        a| 1
        b| 2
        ```
    -   Tables
        
        ```q
        q)([]a:1 2) ,\: ([a:1 3]b:2 3)
        a b
        ---
        1 2
        2  
        ```

-   To many users' consternation, `lj` became `,\:`!
    
    ```q
    q)lj
    k){.Q.sx[x[;z]]y}[k){$[$[99h=@y;(98h=@!y)&98h=@. y;()~y];x,\:y;'"type"]}]
    ```

-   And `ljf` (and family) were begrudgingly added back for backward compatibility

-   **KX's response:** "my mistake was to attempt fills in the first place."
-   **Status:** Implemented - `lj` is now a thin wrapper around `,\:`


## [Ephemeral Ports](https://en.wikipedia.org/wiki/Ephemeral_port)

-   Having previously built a discovery service in Java, I wanted to do
    the same in Q (for the market making system)
-   Flexible horizontal scaling requires dynamically picking the next
    available ephemeral port
-   **The Ask:** let `\p` pick a port dynamically when given an infinite
    value
    
    ```q
    q)\p 0W
    q)\p
    57036i
    ```

-   **Status:** Implemented - `0W` and `-0W` open ephemeral ports


## [GUIDs](https://en.wikipedia.org/wiki/Universally_unique_identifier)

-   I needed the ability to generate orderids that were guaranteed to be
    independent across processes (again for the market making system)
-   Every combination of mac/ip/time/pid/etc I came up with always
    exceeded 8 bytes (long integer)
-   All that effort, and then I found that UUIDs were already well
    documented and did indeed take 16 bytes
-   **The Ask:** add a native UUID type to the language
    
    ```q
    q)rand 0Ng 
    cddeceef-9ee9-3847-9172-3e3d7ab39b26
    q)count 0x0 vs rand 0Ng 
    16
    ```

-   **Status:** Implemented - new GUID was added with type "g"


## Random Permutation

-   I wanted an elegant way to randomly permute data when running
    machine learning algorithms.
-   A positive left argument to `?` produces a random selection with
    replacement
    
    ```q
    q)10 ? til 10
    1 3 3 7 8 2 1 4 2 8
    ```

-   A negative argument produces a random selection without replacement
    
    ```q
    q)-10? til 10
    0 6 2 8 7 4 9 1 3 5
    ```

-   But there was no argument that would permute an arbitrary-length list
-   **The Ask:** allow `0N` to mean random permutation
    
    ```q
    q)0N ? til 10
    6 1 7 3 4 8 5 0 2 9
    ```

-   **Status:** Implemented - `0N?` generates a random permutation


## Reshape Beyond 2 Dimensions

-   I needed to initialize a three-dimensional tensor with random values
-   But the reshape operator `#` only supported one and two dimensions
    
    ```q
    x:til 12
    q)3#x
    0 1 2
    q)0N!3 2#x;
    (0 1;2 3;4 5)
    q)0N!3 0N#x;
    (0 1 2 3;4 5 6 7;8 9 10 11)
    ```

-   **The Ask:** extend reshape (`#`) to n dimensions
    
    ```q
    q)0N!3 2 2#x;
    ((0 1;2 3);(4 5;6 7);(8 9;10 11))
    ```

-   **Status:** Implemented - `#` can now generate n-dimensional shapes


# Unimplemented


## GUID Min/Max

-   GUIDs are lexicographically comparable
    
    ```q
    q)x:5?0Ng
    q)x<reverse x
    01001b
    ```

-   And sortable
    
    ```q
    q)enlist each asc x
    0e51cbcc-c939-5269-131b-28c6cfb7d101
    5a23e05b-3cb6-e1c0-7564-1ce09d944918
    9f46482d-c9b1-2919-3598-5c7b643fed66
    a77a5d84-0127-d8a7-0098-42a2b68ac00b
    af710df1-7881-982e-b964-52245f33eb1f
    ```

-   But `min`, `max`, `mins`, and `maxs` are not implemented
    
    ```q
    q)min x
    'type
      [0]  min x
           ^
    q)max x
    'type
      [0]  max x
           ^
    q)mins x
    'type
      [0]  mins x
           ^
    q)maxs x
    'type
      [0]  maxs x
           ^
    ```

-   Byte vectors already support min/max
    
    ```q
    q)min "x"$()
    0xff
    q)max "x"$()
    0x00
    ```

-   GUIDs should behave the same way
    
    ```q
    q)min "g"$()
    ffffffff-ffff-ffff-ffff-ffffffffffff
    
    / max
    q)max "g"$()
    00000000-0000-0000-0000-000000000000
    ```
-   **The Ask:** implement `min`, `max`, `mins`, and `maxs` for GUIDs

-   **KX's response:** "yes. Let us know if you need this."
-   **Status:** Unimplemented - agreed in principle, no business need, so never implemented


## Real Precision for Linear Algebra

-   The dot product and matrix multiplication operator `$` work on
    floats
    
    ```q
    q)1 2f$1 2f
    5f
    ```
-   But not reals
    
    ```q
    q)1 2e$1 2e
    'type
      [0]  1 2e$1 2e
    
    ```
-   Some machine learning techniques can be sped up by using reduced the
    precision (and therfore size) of the data
-   **The Ask:** Extend `$` to real-typed data
-   **Status:** No response


## `mod` on Timestamp / Timespan

-   `xbar` wasn't the only functionality missed when timestamp and
    timespans were added
    
    ```q
    q).z.P mod 0D01
    'type
    q).z.N mod 1D
    'type
    ```
-   Casting the timespan to a long fixes this
    
    ```q
    q).z.P mod "j"$0D01
    0D00:21:43.316884000
    ```
-   Note that `div` already returns a value that is not intuitive.
    Might fixing this have downstream benefits for `mod` and `xbar` as
    well?
    
    ```q
    q)0D10:01 div 0D01
    0D00:00:00.000000010
    ```

-   **The Ask:** extend `mod` and `div` to support timestamp and timespans
-   **Status:** No response


## The `+/` Null Bug

-   Operations on null values return null results
    
    ```q
    q)0Wi {x+y}\ 1 -1 -1i
    0N 0N 0Ni
    ```
-   But scan and over with native operators do not
    
    ```q
    q)0Wi +\ 1 -1 -1i
    0N 0W 2147483646i
    q)0Wi +/ 1 -1 -1i
    2147483646i
    ```
-   Unless we use vectors?!
    
    ```q
    q)enlist[0Wi] +/ 1 -1 -1i
    ,0Ni
    ```

-   **The Ask:** make `+/` preserve nullness for atoms the same way it does for vectors
-   **KX's response:** "we should probably fix this&#x2026; But we'll give it another thought."
-   **Status:** Unimplemented


## `wsum` Efficiency

-   `wsum` avoids the intermediate vector that `sum[x*x]` allocates:
    
    ```q
    x:10000000?1f
    q)\ts sum x*x
    93 134217968
    q)\ts x wsum x
    15 704
    ```

-   But it always casts to floats, even for long integer inputs
-   And casting makes it slower
    
    ```q
    q)x:10000000?1000
    q)\ts x wsum x
    380 268435632
    q)\ts sum x*x
    132 134217968
    q)type x wsum x
    -9h
    ```

-   And on a matrix, the optimization is not implemented at all
    
    ```q
    q)x:1000 cut 10000000?1f
    q)\ts x wsum x
    43 82002080
    q)\ts sum x*x
    41 82002128
    ```

-   **KX's explanation:** float promotion is a simple way to handle
    overflow and nulls, and the optimization only applies to plain
    vectors, which covers most cases
-   **The Ask:** let `x wsum x` return the type `sum x*x` would, leaving
    overflow handling to the caller, and extend the optimization to
    matrices
-   **Status:** Unimplemented
-   **Acknowledgment:** not likely to ever be implemented because KX
    places extremely high priority on backward compatibility


## Character Arithmetic

-   Q is beautiful because you increment each type with addition
    
    ```q
    q)2000.01.01+1
    2000.01.02
    q)00:00:00+1
    00:00:01
    ```
-   But not for characters
    
    ```q
    q)"a"+1
    'type
    ```
-   The workaround is a round trip through int:
    
    ```q
    q)"c"$1+"i"$"a"
    "b"
    ```
-   **The Ask:** add support for character arithmetic
-   **KX's response:** "you're right!"
-   **Status:** Unimplemented


## Dyadic Run Until Convergence

-   There are three types of iteration control flow
    -   Run n times
        
        ```q
        n f/ x
        ```
    -   Run until f returns 0b
        
        ```q
        g f/ x
        ```
    -   Run until convergence
        
        ```q
        f over x
        ```
-   Two dyadic and one monadic
-   There should be a single argument that switches between them
-   Can we define a dyadic argument to mean run until convergence?
-   KX suggested a workaround:
    
    ```q
    g:{(f/). x,y}
    ```
-   Called as `g[();x]` to converge
-   **The Ask:** add syntax for dyadic run until convergence  `() f/ x`
-   **Status:** Unimplemented


## Deep `where`

-   Having `where` return the coordinates for true values is useful for
    sparse matrices and [advent of code](https://adventofcode.com/)
-   But `where` does not support this
    
    ```q
    q)show x:(1 0N 0N;0N 2 0N;0N 0N 3)
    1
      2
        3
    q)where not null x
    'type
    ```
-   Proposed implementation
    
    ```q
    k)mwhere:{$[@x;&x;,'/(!#x){(,(#*y)#x),y:$[@y;,y;y]}'.z.s'x]}
    q)mwhere not null x
    0 1 2
    0 1 2
    ```

-   **The Ask:** extend `&` itself to matrices and tensors, the way ngn/k
    already does with [deepwhere](https://xpqz.github.io/kbook/search.html?q=deepwhere)
-   **KX's response:** "just thinking about it now&#x2026;"
-   **Status:** Unimplemented


## Random Sample from a Dictionary

-   Random forests require random sub-samples of features
-   Lists and tables both support random selection
    
    ```q
    q)0N?til 5
    2 4 1 3 0
    q)0N?([]til 5)
    x
    -
    1
    4
    0
    2
    3
    ```

-   But dictionaries do not
    
    ```q
    q)0N?x!x:til 5
    'type
    ```

-   **The Ask:** let `0N?dict` and `n?dict` generate random sub-selections
-   **KX's response:** `n?dict` with positive `n` would produce dictionaries with duplicate keys
-   **Status:** Unimplemented


## The Q Prompt Inside Emacs on Windows

-   A J enthusiast looking to learn Q contacted me about using the emacs
    [q-mode](https://github.com/psaris/q-mode)
-   He reported that the `q)` prompt was not printing
-   J solved this by [flushing STDOUT on every write](https://github.com/jsoftware/jsource/blob/07a9c6b97ea43199441ecb7d31bc027e2dfe7eed/jsrc/jconsole.c#L149)
-   **The Ask:** flush STDOUT on Windows (just like J)
-   **Status:** No response


## Syntax-Check-Only Mode

-   Some languages support a "check syntax without running" flag
-   Perl's `-c` is one example
    
    ```sh
    % perl --help | grep -- -c
      -c                    check syntax only (runs BEGIN and CHECK blocks)
    ```
-   **The Ask:**
    -   A command-line flag that allows Q code to be scanned for syntax
        errors without actually executing it
    -   Provide a language server (supporting LSP) along the lines of
        python's pylsp or pyright
-   **Status:** No response


## `qcon` Password

-   `qcon` passes connection parameters on the command line:
    `host:port:user:password`
-   That makes the password visible to anyone running `ps` on the
    machine
-   **The Ask:** allow `qcon` read the password from an environment variable
-   **Status:** No response


# Closing

-   Having a direct line to the Q development team is amazing
-   I take pride in seeing some of my ideas make it into the language -
    even those that were merely for language consistency
-   Recent functionality has increasingly focused on kdb+/kdb-x (the
    database) rather than Q (the language)
-   Q developers are still one of KX's greatest assets
-   My wish is to see KX continue investing in the language and the
    developers who use it every day to build systems


<!----- Footnotes ----->

