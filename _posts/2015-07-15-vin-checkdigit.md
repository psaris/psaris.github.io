---
title: "VIN Check Digit"
excerpt: "Presented at Kx Community NYC Meetup"
date: <span class="timestamp-wrapper"><span class="timestamp">&lt;2015-07-15 Wed&gt;</span></span>
categories: 
- Competition
tags: 
- hkid 
- checkdigit
---


# What is a VIN?

A VIN is a 17 digit vehicle identification number.

| digit(s) | meaning |
|---|---|
| 1 | Country of Manufacture |
| 2 | Manufacturer |
| 3 | Type or Division |
| 4-8 | Brand, Body, Style, Engine Size/Type, Model, Series |
| **9** | **Check Digit** |
| 10 | Model Year |
| 11 | Assembly Plant |
| 12-17 | Production (Serial) Number |


# Mapping the Characters

Convert English letters to number, using the following mapping table:

| **A**:1 | **B**:2 | **C**:3 | **D**:4 | **E**:5 | **F**:6 | **G**:7 | **H**:8 | |
| **J**:1 | **K**:2 | **L**:3 | **M**:4 | **N**:5 | | **P**:7 | | **R**:9 |
| | **S**:2 | **T**:3 | **U**:4 | **V**:5 | **W**:6 | **X**:7 | **Y**:8 | **Z**:9 |

-   Map Key
    
    ```q
    q).Q.nA except "IOQ"
    "0123456789ABCDEFGHJKLMNPRSTUVWXYZ"
    ```

-   Map value
    
    ```q
    q)(40#til 10) _/ 31 30 28 26 20 19 10
    0 1 2 3 4 5 6 7 8 9 1 2 3 4 5 6 7 8 1 2 3 4 5 7 9 2 3 4 5 6 7 8 9
    ```

-   Character to Integer Map
    
    ```q
    q)show m:(.Q.nA except "IOQ")!"f"$(40#til 10) _/ 31 30 28 26 20 19 10
    0| 0
    1| 1
    2| 2
    3| 3
    4| 4
    5| 5
    6| 6
    ..
    ```

-   Randomly Generate a VIN
    
    ```q
    q)show vin:17?.Q.nA except "IOQ"
    "K7HJVYXBG1DSBZTSS"
    q)m vin
    2 7 8 1 5 8 7 2 7 1 4 2 2 9 3 2 2f
    ```


# The Weighted Sum

Using the specified weights, compute the weighted sum of all characters:

-   Weights
    
    ```q
    w:8 7 6 5 4 3 2 10 0 9 8 7 6 5 4 3 2f
    ```
-   Two-operator weighted sum
    
    ```q
    q)sum w * m vin
    330f
    ```

-   One-operator weighted sum
    
    ```q
    q)w wsum m vin
    330f
    ```

-   Dot product weighted sum
    
    ```q
    q)w$m vin
    330f
    ```


# The Check Digit

Compute the check digit by mapping the modulus base 11

-   Modulus
    
    ```q
    q)"0123456789X" "j"$(m[vin]$w) mod 11f
    "0"
    ```

-   Lookup table
    
    ```q
    q)(802#"0123456789X") "j"$m[vin]$w
    "0"
    ```


# Approaches

-   Basic
    
    ```q
    validvin:{
     if[type x;:first .z.s enlist x];
     m:(.Q.nA except "IOQ")!"f"$(40#til 10) _/ 31 30 28 26 20 19 10;
     w:8 7 6 5 4 3 2 10 0 9 8 7 6 5 4 3 2f;
     c:"0123456789X";
     v:x[;8]=c"j"$mod[;11] m[x]$w;
     v}
    ```

-   Using Attributes
    
    ```q
    validvin:{
     if[type x;:first .z.s enlist x];
     m:(`s#.Q.nA except "IOQ")!"f"$(40#til 10) _/ 31 30 28 26 20 19 10;
     w:8 7 6 5 4 3 2 10 0 9 8 7 6 5 4 3 2f;
     c:"0123456789X";
     v:x[;8]=c"j"$mod[;11] m[x]$w;
     v}
    ```

-   Vectorized
    
    ```q
    validvin:{
     if[type x;:first .z.s enlist x];
     m:(`s#.Q.nA except "IOQ")!"f"$(40#til 10) _/ 31 30 28 26 20 19 10;
     w:8 7 6 5 4 3 2 10 0 9 8 7 6 5 4 3 2f;
     c:"0123456789X";
     v:r[8+17*til count x]=c"j"$mod[;11](17 cut m r:raze x)$w;
     v}
    ```

-   Lookup Table
    
    ```q
    validvin:{
     if[type x;:first .z.s enlist x];
     m:(`s#.Q.nA except "IOQ")!"f"$(40#til 10) _/ 31 30 28 26 20 19 10;
     w:8 7 6 5 4 3 2 10 0 9 8 7 6 5 4 3 2f;
     c:"0123456789X";
     v:r[8+17*til count x]=(802#c)"j"$(17 cut m r:raze x)$w;
     v}
    ```

-   Short Circuit
    
    ```q
    validvin:{
     if[type x;:first .z.s enlist x];
     m:(`s#.Q.nA except "IOQ")!"f"$(40#til 10) _/ 31 30 28 26 20 19 10;
     w:8 7 6 5 4 3 2 10 0 9 8 7 6 5 4 3 2f;
     c:"0123456789X";
     v:x[;8] in c;
     if[count k:where v;v[k]:r[8+17*til count x]=(802#c)"j"$(17 cut m r:raze x@:k)$w];
     v}
    ```

-   The Winner
    
    ```q
    validvin:{
     m:(`u#"0123456789ABCDEFGHJKLMNPRSTUVWXYZ")!0 1 2 3 4 5 6 7 8 9 1 2 3 4 5 6 7 8 1 2 3 4 5 7 9 2 3 4 5 6 7 8 9f;
     x:$[t:type x;enlist x;x];
     if[count x@:k:where j:x[;8]in c:"0123456789X";j[k]:x0[8+17*til count x]=c"i"$mod[;11f](0N 17#m x0:raze x)$8 7 6 5 4 3 2 10 0 9 8 7 6 5 4 3 2f];
     $[t;first j;j]}
    ```


# SEDOL

> The check digit for a SEDOL is chosen to make the total weighted sum
> of all seven characters a multiple of 10. The check digit is computed
> using a weighted sum of the first six characters. Letters have the
> value of 9 plus their alphabet position, such that B = 11 and Z
> = 35. While vowels are never used in SEDOLs, they are not ignored when
> computing this weighted sum (e.g. H = 17 and J = 19, even though I is
> not used), simplifying code to compute this sum. The resulting string
> of numbers is then multiplied by the weighting factor as follows:
> 
>     First   1
>     Second  3
>     Third   1
>     Fourth  7
>     Fifth   3
>     Sixth   9
>     Seventh 1 (the check digit)
> 
> &#x2013; [Wikipedia Sedol Page](https://en.wikipedia.org/wiki/SEDOL#Description)


# Insights

-   Add attributes to large dictionaries
-   Matrix multiplication `$/mmu` is faster than `sum *`
-   The modulus operator is slow (but cyclical)
-   Vector operations are very fast (CPU can use SIMD/SSE)
-   Don't use `each`
-   Don't perform unnecessary computations


<!----- Footnotes ----->

