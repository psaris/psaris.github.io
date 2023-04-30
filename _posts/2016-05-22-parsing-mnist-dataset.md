---
title: "Parsing the MNIST Data Set"
excerpt: "Presented at KXCon 2016"
date: <span class="timestamp-wrapper"><span class="timestamp">&lt;2016-05-22 Sun&gt;</span></span>
categories: 
- Competition
tags: 
- mnist
---


# Recap of Competition Rules

-   Convert a self-describing vector of bytes into an n-dimensional
    array of the specified type
-   Points given for:
    -   Fastest submission
    -   Smallest submission
    -   Shortest submission
-   Tie-breaker based on timing of first submission

Complete competition rules can be found [here](/assets/docs/kxcon2016-challenge.pdf). 


# Parsing Data

| | text | binary |
|---|---|---|
| read file | `read0` | `read1` |
| parse file or vector | `0:` | `1:` |
| cast single element | `TYPE $ cvec` | `0x0 sv bvec` |
| fixed width | `(width;type) 0: cvec` | `(width;type) 1: bvec` (big endian) |
| | | `(type;width) 1: bvec` (little endian) |
| delimited | `(type;delim) 0: cvec` | |


# Understanding the Header

-   Bytes 1 and 2 are empty (presumably reserved for versioning)
-   Byte 3 indicates the data type ([un]signed byte, short, int, real,
    float)
-   Byte 4 specifies the dimensionality of th n-array
-   The next n integers (n\*4 bytes) list the sizes of each dimension
-   Remaining bytes contain the data

| 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 |
| 00 | 00 | 09 | 02 | 00 | 00 | 00 | 02 | 00 | 00 | 00 | 10 |


# Parsing the Header

```q
ldidx:{
 d:first (1#4;1#"i") 1: 4_(h:4*1+x 3)#x;
```

-   `h` is the length of the header
-   `d` is the vector of dimensions


# Parsing the Data

```q
ldidx:{
 d:first (1#4;1#"i") 1: 4_(h:4*1+x 3)#x;
 x:first ((1 1 0N 2 4 4 8;"xx hief")@\:(),x[2]-0x08) 1: h _x;
```

-   `x` now has a single vector of data cast to the type specified in
    byte 3
-   There is no parse-time syntax for specifying a single element
    literal list so we need an extra operation: `(),`


# Reshaping the Data

-   For 1 dimension we can just return the data
-   For 2 dimensions we can use `cut` or `#`
    
    ```q
    q)10 cut til 20
    0  1  2  3  4  5  6  7  8  9 
    10 11 12 13 14 15 16 17 18 19
    q)2 10 # til 20
    0  1  2  3  4  5  6  7  8  9 
    10 11 12 13 14 15 16 17 18 19
    ```

-   What about 3+ dimensions?[^fn1]
    
    ```q
    shape:{y {y cut x}/reverse 1_x,()}
    ```
    
    ```q
    q)shape[2 10] til 20
    0  1  2  3  4  5  6  7  8  9 
    10 11 12 13 14 15 16 17 18 19
    q)shape[2 2 5] til 20
    0 1 2 3 4      5 6 7 8 9     
    10 11 12 13 14 15 16 17 18 19
    ```

-   Permit atomic argument by promoting `x` to list
-   But what about the edge conditions? What would q do?
    
    ```q
    q)2 5 # til 20
    0 1 2 3 4
    5 6 7 8 9
    q)shape[2 5] til 20
    0  1  2  3  4 
    5  6  7  8  9 
    10 11 12 13 14
    15 16 17 18 19
    ```
-   We need to extend/truncate the initial vector before reshaping it.

```q
shape:{(prd[x]#y){y cut x}/reverse 1_x,()}
k)shape:{((*/x)#y){y#x}/0N,'|1_x,()}
```

```q
q)0N!shape[2 2 5] til 10;
((0 1 2 3 4;5 6 7 8 9);(0 1 2 3 4;5 6 7 8 9))
```


# Fast Solutions

-   Basic Solution
    
    ```q
    ldidx:{
     d:first (1#4;1#"i") 1: 4_(h:4*1+x 3)#x;
     x:first ((1 1 0N 2 4 4 8;"xx hief")@\:(),x[2]-0x08) 1: h _x;
     x:((prd[d])#x){y cut x}/reverse 1_d;
     x}
    ```
    
    But why call `1:` to convert bytes to bytes?  Just return the vector!

-   Fastest (and Smallest) Solution
    
    ```q
    ldidx:{
     d:first (1#4;1#"i") 1: 4_(h:4*1+x 3)#x;
     x:$[0>i:x[2]-0x0b;::;first ((2 4 4 8;"hief")@\:i,()) 1:] h _x;
     x:((prd[d])#x){y cut x}/reverse 1_d;
     x}
    ```


# Short Solutions

-   Function calls (including lambdas) only add two bytes (no matter
    what is inside the function/lambda)
    
    ```q
    ldidx:{
     {d:first (1#4;1#"i") 1: 4_(h:4*1+x 3)#x;
     x:$[0>i:x[2]-0x0b;::;first ((2 4 4 8;"hief")@\:i,()) 1:] h _x;
     x:((prd[d])#x){y cut x}/reverse 1_d;
     x}x}
    ```
    
    ```q
    q)count first get ldidx
    5
    ```


# Shortest Solution

-   Given a composition, `get` returns the list of composed functions
    
    ```q
    q)f:{}@
    q)get f
    @
    {}
    q)count first get f
    1
    ```
    
    ```q
    ldidx:{
     d:first (1#4;1#"i") 1: 4_(h:4*1+x 3)#x;
     x:$[0>i:x[2]-0x0b;::;first ((2 4 4 8;"hief")@\:i,()) 1:] h _x;
     x:((prd[d])#x){y cut x}/reverse 1_d;
     x}@
    ```


# Runner-Up Solution

-   Skipped calling 1: when dimension was greater than 2
    
    ```q
    k)ldidx:{
     r:,/(2 1 1 4;" xxi")1:8#x;
     f:([f:0x08090B0C0D0E]t:"xxhief";l:1 1 2 4 4 8)@r 0;
     x:8_x;
     $[0i~I:r 2;:(f`t)$();0x1~r 1;
      :((,f`l;,f`t)1:(I*f`l)#x)0;0x2~r 1;
      [M:((,4;,"i")1:4#x)[0;0];R:((,f`l;,f`t)1:(M*I*f`l)#x:4_x)0;:(M,I)#R]];
     O:,/(4 4;"ii")1:8#x;
     (O#)'[I#((+[prd O])\[7h$I;0])_x:8_x]}
    ```


# Winning Solution

-   1 byte-code and least bytes allocated
    
    ```q
    ldidx:{ {(prd[x]#y){y#x}/0Ni,'reverse 1_x}[0x0 sv'0N 4#x 4+til 4*x 3;first(1 2 4 4 8;"xhief")[;enlist 0|-10+x 2]1:(4*1+x 3)_ x]}@
    ```

-   Combining the Best Ideas and kdb+ 3.4's native multi-dimensional `#`
    
    ```q
    ldidx:{
     d:first (1#4;1#"i") 1: 4_(h:4*1+x 3)#x;
     x:d#$[0>i:x[2]-0x0b;::;first ((2 4 4 8;"hief")@\:i,()) 1:] h _x;
     x}@
    ```


<!----- Footnotes ----->

[^fn1]: kdb+ 3.4 was updated to include a multi-dimensional `#` operator (specifically for this competition)