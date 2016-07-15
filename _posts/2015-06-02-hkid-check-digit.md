---
title: "HKID Check Digit"
excerpt: "Presented at Hong Kong Functional Programming User Group Meetup"
date: <span class="timestamp-wrapper"><span class="timestamp">&lt;2015-06-02 Tue&gt;</span></span>
categories: 
- Competition
tags: 
- hkid 
- checkdigit
---


# Why Q?

-   Performance
-   Expressiveness
-   Fun-ctional


# HKID

> Convert English letters to number, using the mapping A->10, B->11,
> C->12 &#x2026;. Z->35

-   character vectors
    
    ```q
    q)"ABCDEFGHIJKLMNOPQRSTUVWXYZ"
    ```

-   .q.k
    
    ```q
    q).Q.n
    "0123456789"
    q).Q.A
    "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
    q).Q.nA
    "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ"
    q).Q.na
    "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ_0123456789"
    q).Q.b6
    "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
    ```

-   character to integer
    
    ```q
    q)-55+"j"$"AZ123456"
    10 35 -6 -5 -4 -3 -2 -1
    ```

-   find
    
    ```q
    q).Q.nA?"AZ123456"
    10 35 1 2 3 4 5 6
    ```


# The Sum

-   Calculate the check sum
    
    -   For HKID with 2 English letters
    
        check sum = char[0] * 9 + char[1] * 8 + char[2] * 7 + char[3] * 6 + char[4] * 5 + char[5] * 4 + char[6] * 3 + char[7] * 2
    
    -   For HKID with only 1 English letter
    
        check sum = 36 * 9 + char[1] * 8 + char[2] * 7 + char[3] * 6 + char[4] * 5 + char[5] * 4 + char[6] * 3 + char[7] * 2

-   what is 36?
    
    ```q
    q)-55+"j"$"[Z123456"
    36 35 -6 -5 -4 -3 -2 -1
    ```

-   one past the end
    
    ```q
    q).Q.nA?" Z123456"
    36 35 1 2 3 4 5 6
    ```

-   reshaping lists
    
    ```q
    q)-8#"[","AZ123456"
    "AZ123456"
    q)-8#"[","Z123456"
    "[Z123456"
    ```

-   padding strings
    
    ```q
    q)-8$"Z123456"
    " Z123456"
    q)-8$"AZ123456"
    "AZ123456"
    ```

-   vector arithmetic
    
    ```q
    q)9 8 7 6 5 4 3 2 * .Q.nA?-8$" Z123456"
    324 280 7 12 15 16 15 12
    q)sum 9 8 7 6 5 4 3 2 * .Q.nA?-8$" Z123456"
    681
    ```

-   dot product
    
    ```q
    q)9 8 7 6 5 4 3 2f $ "f"$.Q.nA?-8$" Z123456"
    681f
    ```


# The Checksum

-   **mooderate** check sum by 11 and use 11 to minus the value
-   check digit = 11 - checksum mod 11

-   modulus
    
    ```q
    q)11 - mod[;11] sum 9 8 7 6 5 4 3 2*.Q.nA?-8$" Z123456"
    1
    q).Q.nA 11 - mod[;11] sum 9 8 7 6 5 4 3 2*.Q.nA?-8$" Z123456"
    "1"
    ```

-   reorder operations
    
    ```q
    q).Q.nA mod[;11] sum 2 3 4 5 6 7 8 9*.Q.nA?-8$" Z123456"
    "1"
    ```

-   lookup table
    
    ```q
    q)sum 2 3 4 5 6 7 8 9*.Q.nA?-8$" Z999999"
    528
    q)(529#.Q.n,"A")sum 2 3 4 5 6 7 8 9*.Q.nA?-8$" Z123456"
    "1"
    ```


# Solutions

-   the shortest (composition)
    
    ```q
    q)hkidcheck:(529#.Q.n,"A")sum 2 3 4 5 6 7 8 9*.Q.nA?-8$
    q)\ts:100000 hkidcheck each ids
    1487 640
    ```

-   the fastest (projection)
    
    ```q
    q)hkidcheck:{x sum 2 3 4 5 6 7 8 9*y?-8$z}[529#.Q.n,"A";.Q.nA]
    q)\ts:100000 hkidcheck each ids
    1335 672
    ```

-   the longest
    
    ```q
    hkidcheck:{[id]
        $[(count id)=8;[abc1:"i"$id[0];abc1:abc1-55;abc2:"i"$id[1];abc2:abc2-55;adigit1:"i"$id[2];adigit2:"i"$id[3];adigit3:"i"$id[4];adigit4:"i"$id[5];adigit5:"i"$id[6];adigit6:"i"$id[7];p1:(abc1*9)];[abc1:324;p1:abc1;abc2:"i"$id[0];abc2-:55;adigit1:"i"$id[1];adigit2:"i"$id[2];adigit3:"i"$id[3];adigit4:"i"$id[4];adigit5:"i"$id[5];adigit6:"i"$id[6]]];
        p2:abc2*8;
        p3:(adigit1-48)*7;
        p4:(adigit2-48)*6;
        p5:(adigit3-48)*5;
        p6:(adigit4-48)*4;
        p7:(adigit5-48)*3;
        p8:(adigit6-48)*2;
        s1:p1+p2;
        s2:p3+p4;
        s3:p5+p6;
        s4:p7+p8;
        s5:s1+s2;
        s6:s3+s4;
        sumtotal:s5+s6;
        remainder:(sumtotal mod 11);
        checksum:(11-remainder);
        checksum }
    ```

-   the winner
    
    ```q
    q)hkidcheck:eval parse"{\"",((2231#"B"),1000#"0", .Q.nA 10-til 10),"\" sum (9 8 7 6 5 4 3 2i)*6h$ $[7=count x; \"[\",x; x]}"
    q)\ts:100000 hkidcheck each ids
    1350 512
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

-   A HKID is a **vector** of characters
-   The modulus operator is slow (but cyclical)
-   Fixed values can be pre-calculate


<!----- Footnotes ----->

