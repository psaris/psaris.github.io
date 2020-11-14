---
title: "Fun Q: The Origin Story"
excerpt: "Presented at AquaQ's AquaQuarantine kdb+ Tech Talks"
date: <span class="timestamp-wrapper"><span class="timestamp">&lt;2020-08-13 Thu 11:00&gt;</span></span>
categories: 
- Presentation
tags: 
- ml 
- decisiontree 
- funq 
- mnist
---


# Origin Story

![Fun Q Cover](/assets/images/funq_cover.jpg)


## Timeline

-   2015: Reimplemented the Coursera [machine-learning course](https://www.coursera.org/learn/machine-learning) in q
-   2016: Presented results at [kxcon2016](https://kx.com/kxcon2016)
-   2017: Presented recommender systems at Jan [kx nyc meetup](https://www.meetup.com/kx-nyc/events/236400086)
-   2017: Presented master mind challenge at Nov  [kx nyc meetup](https://kx.com/blog/kdb-mastermind-challenge)
-   2018: Presented decision trees at [kx25](https://kx.com/kx25/) and began writing book
-   2019: Sent preliminary chapters to machine learning PhD
-   2020: Published


## Algorithms

-   K-Nearest Neighbors (KNN)
-   K-Means/Medians/Mediods Clustering
-   Hierarchical Agglomerative Clustering (HAC)
-   Expectation Maximization (EM)
-   Naive Bayes
-   Decision Tree (ID3,C4.5,CART)
-   Discrete Adaptive Boosting (AdaBoost)
-   Random Forest and Boosted Aggregating (BAG)
-   Linear Regression
-   Logistic Regression and One-vs.-All
-   Neural Network Classification/Regression
-   Content-Based/Collaborative Filtering (Recommender Systems)
-   Google PageRank


## Algorithms Left Out

-   Dimensionality Reduction (PCA) &#x2013; eigen vectors
-   Support Vector Machines (SVM) &#x2013; maximize margin between classes
-   Apriori &#x2013; market basket analysis (MBA)
-   Extreme Gradient Boosting (XGBoost) &#x2013; parallel boosting


## Hurdles Along the Way

-   Implementing text-based decision tree graph
-   Passing NN activation functions as parameters
-   Passing regularization as a list of functions
-   Generalizing KNN to work with null values


## Last Minute Additions

-   Silhouette &#x2013; cluster quality
-   Heckbert Algorithm &#x2013; nice numbers
-   Receiver Operating Characteristic (ROC)


# Publishing Toolchain

![Emacs Image](/assets/images/emacs.png)


## Latex?

-   Professional layout and formatting
-   Automatic hyphenation
-   Automatic table of contents
-   Automatic internal reference numbering
-   Native math equation support
-   Auctex Emacs mode!


## Markdown?

-   Emacs [Markdown](https://daringfireball.net/projects/markdown/syntax) mode
-   Plain text with comfortable syntax
-   Prior Art: [Discover object-oriented programming using WordPress](https://carlalexander.ca/write-book-markdown/)


## Org mode?

-   Emacs [Org mode](https://orgmode.org/)
-   "Organize Your Life In Plain Text!"
-   Exports to html, latex, markdown, text, slidy, reveal
-   Prior Art: [Ending Malnutrition: from commitment to action](https://www.wisdomandwonder.com/link/10021/a-book-produced-using-org)


## AsciiDoc!

-   Markdown++
-   First discovered in Git project documentation
-   Designed to generate [DocBook](https://docbook.org/)
-   Uses DocBook .xml with XSLT and [dblatex](http://dblatex.sourceforge.net/) to generate epub and latex
-   Supports latex tables, figures, equations, examples and listings
-   Supported by Github and O'Reilly
-   Prior Art: [The Hacker's Guide to Python](https://julien.danjou.info/asciidoc-book-toolchain-released/)


## AsciiDoctor?

-   Development of AsciiDoc stalled at Python 2.7
-   [AsciidDoctor](https://asciidoctor.org/) rebuilt (and extended) with Ruby
-   But AsciiDoc was [ported to Python 3](https://github.com/asciidoc/asciidoc-py3) just as Fun Q went to print!
-   Added 'floating' tables, figures, equations and examples!


## Makefile

-   Added targets: 'pdf', 'epub', 'mobi', 'lang', 'spell', 'clean'
-   Regenerates the book on any change of input: code, image, etc
-   Calls 'xetex' instead of 'latexpdf' for unicode support
-   Calls 'epubcheck' to validate ePUB format
-   Calls 'kindlegen' to build .mobi format for Amazon


## GIMP

-   [GNU Image Manipulation Program](https://www.gimp.org/)
-   Dedicated to using open source tools for the whole project
-   Refused to pay Amazon to generate the cover
-   Generated the original Q Tips 'q)'


## Publishing

-   [CreateSpace](https://www.createspace.com/)
-   [Amazon Kindle Direct Publishing](https://www.kdp.com/)
-   [IngramSpark](https://www.ingramspark.com/) (LighteningSource)
-   [Lulu](https://www.lulu.com/)


# Zoo Data Set

-   Richard Forsyth's [Zoo Data Set](https://archive.ics.uci.edu/ml/datasets/Zoo)

```q
q)zoo.t
typ          animal   hair feathers eggs milk airborne aquatic predator toothed backbone breathes venomous fins legs tail domestic catsize
------------------------------------------------------------------------------------------------------------------------------------------
mamal        aardvark 1    0        0    1    0        0       1        1       1        1        0        0    4    0    0        1      
mamal        antelope 1    0        0    1    0        0       0        1       1        1        0        0    4    1    0        1      
fish         bass     0    0        1    0    0        1       1        1       1        0        0        1    0    1    0        0      
mamal        bear     1    0        0    1    0        0       1        1       1        1        0        0    4    0    0        1      
mamal        boar     1    0        0    1    0        0       1        1       1        1        0        0    4    1    0        1      
mamal        buffalo  1    0        0    1    0        0       0        1       1        1        0        0    4    1    0        1      
mamal        calf     1    0        0    1    0        0       0        1       1        1        0        0    4    1    1        1      
fish         carp     0    0        1    0    0        1       0        1       1        0        0        1    0    1    1        0      
fish         catfish  0    0        1    0    0        1       1        1       1        0        0        1    0    1    0        0      
..
```

```q
q)\l zoo.q
q)select[>n] n:count i by typ from zoo.t
typ         | n 
------------| --
mamal       | 41
bird        | 20
fish        | 13
invertebrate| 10
insect      | 8 
reptile     | 5 
amphibian   | 4 

```


## Partition

```q
q).ut.part
{[w;sf;x]
 if[99h=type w;:key[w]!.z.s[value w;sf;x]];
 if[99h<type sf;:x (floor sums n*prev[0f;w%sum w]) _ sf n:count x];
 x@:raze each flip value .z.s[w;0N?] each group sf; / stratify
 x}
```

-   parameterize algorithms with functions
-   overload algorithm on parameter type
-   recurse to handle different parameter types

```q
q)d:`train`test!.ut.part[3 1;0N?] zoo.t       / random
q)d:.ut.part[`train`test!3 1;til] zoo.t       / time-series
q)d:.ut.part[`train`test!3 1;zoo.t.typ] zoo.t / stratified
```


## Decision Tree

```q
q)-1 .ml.ptree[0] tr:.ml.ct[();::] delete animal from d`train;
mamal (n = 73, err = 58.9%)
|  milk >[;0.5] 0: bird (n = 43, err = 65.1%)
|  |  feathers >[;0.5] 0: fish (n = 28, err = 67.9%)
|  |  |  fins >[;0.5] 0: invertebrate (n = 19, err = 63.2%)
|  |  |  |  backbone >[;0.5] 0: invertebrate (n = 13, err = 46.2%)
|  |  |  |  |  legs >[;5f] 0: invertebrate (n = 5, err = 0%)
|  |  |  |  |  legs >[;5f] 1: insect (n = 8, err = 25%)
|  |  |  |  |  |  aquatic >[;0.5] 0: insect (n = 6, err = 0%)
|  |  |  |  |  |  aquatic >[;0.5] 1: invertebrate (n = 2, err = 0%)
|  |  |  |  backbone >[;0.5] 1: amphibian (n = 6, err = 50%)
|  |  |  |  |  aquatic >[;0.5] 0: reptile (n = 3, err = 0%)
|  |  |  |  |  aquatic >[;0.5] 1: amphibian (n = 3, err = 0%)
|  |  |  fins >[;0.5] 1: fish (n = 9, err = 0%)
|  |  feathers >[;0.5] 1: bird (n = 15, err = 0%)
|  milk >[;0.5] 1: mamal (n = 30, err = 0%)
```


## Confusion Matrix

```q
q).ut.totals[`TOTAL] .ml.cm[yt] p
y           | amphibian bird fish insect invertebrate mamal reptile TOTAL
------------| -----------------------------------------------------------
amphibian   | 1         0    0    0      0            0     0       1    
bird        | 0         5    0    0      0            0     0       5    
fish        | 0         0    4    0      0            0     0       4    
insect      | 0         0    0    2      0            0     0       2    
invertebrate| 0         0    0    1      2            0     0       3    
mamal       | 0         0    0    0      0            11    0       11   
reptile     | 1         0    0    0      0            0     1       2    
            | 2         5    4    3      2            11    1       28   
```


## Classification Errors

```q
q)([]d.test.animal;yt;p) where not p=yt
animal   yt           p        
-------------------------------
starfish invertebrate insect   
tortoise reptile      amphibian
```


## Silhouette

```q
q).ml.silhouette
{[df;X;I]
 if[type I;s:.z.s[df;X]I:value group I;:raze[s] iasc raze I];
 if[1=n:count I;:count[I 0]#0f]; / special case a single cluster
 a:{[df;X](1f%-1+count X 0)*sum f2nd[df X] X}[df] peach G:X@\:/:I;
 b:{[df;G;i]min{f2nd[avg x[z]::]y}[df;G i]'[G _ i]}[df;G] peach til n;
 s:0f^(b-a)%a|b;                / 0 fill to handle single point clusters
 s}
```

```q
q)select[>silhouette] avg silhouette by typ from s
typ         | silhouette
------------| ----------
fish        | 0.6814285 
insect      | 0.5795206 
amphibian   | 0.5675568 
bird        | 0.5446474 
mamal       | 0.2539767 
reptile     | -0.4007103
invertebrate| -0.4366271
```


## Poor Clusters

```q
q)select[-10;>silhouette] from s where typ=`mamal
typ   animal   silhouette 
--------------------------
mamal girl     0.1842328  
mamal sealion  0.155154   
mamal cavy     0.1376027  
mamal squirrel 0.1306259  
mamal fruitbat 0.02075501 
mamal vampire  0.02075501 
mamal platypus -0.02258724
mamal seal     -0.3825486 
mamal dolphin  -0.4940877 
mamal porpoise -0.4940877 
```


## K-Means Clustering

-   Check elbow curve for optimal number of clusters

```q
q)C:{[X;k].ml.kmeans[X] over last k .ml.kmeanspp[X]// 2#()}[zoo.X] each 1+til 10
q).ut.plt .ml.distortion[zoo.X] each C
800| "                   "
   | "+                  "
600| "                   "
   | "                   "
400| "                   "
   | "  +                "
   | "    +              "
200| "      + +          "
   | "          + + + + +"
0  | "                   "
```


# MNIST Data Set

-   Yann LeCun's [MNIST Data Set](http://yann.lecun.com/exdb/mnist)

```q
q)\l mnist.q
q)`X`Xt`y`yt set' mnist`X`Xt`y`yt;
q)X:1000#'X;y:1000#y;
q)X%:255f;Xt%:255f
q)plt:value .ut.plot[28;14;.ut.c10;avg] .ut.hmap flip 28 cut
q)-1 (,'/) plt each X@\:/: -3?count X 0;


                                        .:#+-                                       
                                          :=%@:                                     
                -+##@@@@@x                 -%@x                          :=.        
               x@%##x++:--               -%@@x.                      =+ @%-    :    
          +#x                        -+%@x=-                      -+%x:+@.     -    
         @@%:-.                    .+@+-                         .%@%x#@%--=+x.     
         -+##@@#.                 -@x                              .-+@#====:- .    
             .x@%                 x#    =.. ::..                    -@x             
       -:.  .x%@x                 .#@x#+@#x###%%@%+.               -%x              
      #@@@@@@@#:                     ::--       .:=-              :%=               
      -++-:-                                                     :%:                
                                                                -=                  
```


## Neural Network Configuration

```q
q)Y:.ml.diag[(1+max y)#1f]@\:y
q)n:0N!"j"$.ut.nseq[2;count X;count Y]
784 397 10
q)theta:2 raze/ THETA:.ml.heu'[1+-1_n;1_n]
q)rf:.ml.l2[1f]
q)hgolf:`h`g`o`l!`.ml.sigmoid`.ml.dsigmoid`.ml.softmax`.ml.celoss
```


## Neural Network Confusion Matrix

```q
q)theta:first r:.fmincg.fmincg[50;.ml.nncostgrad[rf;n;hgolf;Y;X];theta]
Iteration 50 | cost: 0.3687174
q)avg yt=p
0.8601
q)show .ut.totals[`TOTAL] .ml.cm[yt;"i"$p]
y| 0    1    2    3   4   5   6   7   8   9    TOTAL
-| -------------------------------------------------
0| 945  0    13   1   0   4   8   5   2   2    980  
1| 0    1085 4    2   0   1   3   1   37  2    1135 
2| 15   14   894  12  10  3   25  13  36  10   1032 
3| 6    2    44   763 1   128 4   15  36  11   1010 
4| 3    3    10   9   814 1   21  1   18  102  982  
5| 22   7    15   31  15  694 19  11  44  34   892  
6| 16   2    25   0   11  40  855 1   8   0    958  
7| 5    16   37   18  5   4   0   891 2   50   1028 
8| 22   15   8    14  12  35  24  13  790 41   974  
9| 14   4    15   23  26  9   0   39  9   870  1009 
 | 1048 1148 1065 873 894 919 959 990 982 1122 10000
```


## Neural Network Misclassification

```q
q)(,'/) plt each flip Xt[;i:3?where not yt=p]
"                                                                                    "
"                                                                                    "
"                                                                                    "
"             .++-                                                                   "
"          -x#@=:##                                                                  "
"           x%:++:.                      .%%#xx+#%%:                                 "
"          :@::+==++:                       .-=x#x=-               :====++:.-:=:=:::-"
"          -@@=     x%                 .xxx=:-                    x%--:x#x=:.        "
"           --      .@=                   .x+                     =%#@+              "
"                   =%                     :@#                   ##  .%#             "
"       -         .+%-          :x=:--:=x%@x=                    +###xx-             "
"      :x+:.     x%-             --:=:.-.                                            "
"         :x###+x:                                                                   "
"                                                                                    "
q)([]yt;p) i
yt p
----
5  3
3  2
8  6
```


## Expectation-Maximization Prototypes

```q
q)X>:.5                           / convert to binary
q)k:15                            / prototypes
q)phi:k#1f%k                      / equal prior probability
q)mu:flip last k .ml.kpp[.ml.hdist;X]// 2#() / pick k distant proto
q)mu:.5*mu+.15+count[X]?/:k#.7    / randomly perturb around .5
q)-1 (,'/) plt each 3#mu;

- -- .: - .:.  .:.:..  - -::-..------:.:. : ..--..-.:-.---..--.-..---.-:-=-.:. --   
----.-.:-. ...:--- .... -.-.--.-..-- ..-....-. :.----...----:---=::.:.-.:=#%:: -....
.: -.-..--.: .=.-.-...-- . -.--:.-.--.--.:-..--.-::- -..-.: ----:-.::..:%+:  ::--- .
--:.--...-..=+#%+x#x:- --. :..: .:  .. .----+=.:+=++..:.:-..---.. ---#%:.- :..:-.-..
..-.-  .. x%%+- :-.@.:....:.-:-.:.-:.-...=%#%%x#%@+=-...-.- .=-- -.:%x: - .----:-.--
.-...-..:@x- .-.-:-:...-.- .--...----..#@#:=--:.: .-..:-.--:- .:---+@ .-.. -.... ---
.--....-x+-.-.- .-::----. --.:-----.-.-+=#%%%+: -. ..-.- .:=::-:-.-@--.++++...:.-::-
:-.  :. #=--.-. +#=..-- .--:.--:=.-...--:--- %%@:.-- . .-:.....--:.#%@#+x+@@.-.:-.:-
-..-.. -=##@#%+:@=--=.-. :.:.-::#@+x-:. -:.: %@x-..:-.::-.....-:-+#%@--- :x%=.-:..-.
..-.--.. --...-#x.. .- -.--- . ..:+x%=x+:+x+%#=..:-. ---..=:-.-.-+#-+@%:x%@+-..:.:--
 .-.--.-.- .:.#@-.-::.-- .-. ---:-:.x+#%@%+=+--.--.-  ..-.-.- --:-.:..=++:.-:---- --
-  -  .- ..--.%. .- -..-..-.--:::..:...- .-...: --: : -:-..-.-.:--.-.::-:- .-.:--:-.
- ..-..:--.---=+ -:. -.. .-.:-- ...-.--.-.-.--.--.- : .-..- ..--:-----:.:-..---- .-.
```


## Expectation-Maximization Probabilites

```q
q)lf:.ml.bmml[1]                  / binomial mixture model likelihood
q)mf:.ml.wbmmmle[1;1e-8] / binomial mixture model maximization likelihood estimator
q)pT:(phi;flip enlist mu)
q)pT:10 .ml.em[1b;lf;mf;X]// pT
q)-1 (,'/) (plt first::) each 3#pT 1;



         .-=+xx++:-.                      .-=+++=:.               .-=+#x=-.         
      .-=#@@@@@@@@%x:.                  .=#@@@@%#x-.           ..-==x@@#+:.         
      -=x%@#+=:=+x##=:.               -+#@@%x+=#%#=-            ..:=+xx+=:.         
     .-=x#x=:::-::==+=:.            -+#@%x:...-=##+-             .-=+##x=:.         
     .:=++=:==::::==++=-.         .+%@%+-     :x%#=.             .-:+%@@+:-.        
     .:=+=:------::+++=:.        -=%@#=.     -+%%x-              .:=x%%#=::-..      
     .:+x+=-....--=+#x+:.       .=#%#:    .-=x%%+-               .==+#x+::=:-..     
      -+###+:--::+#%%x:-        .+%@x=::=+x#%%x:.                -:=+x##xxx+-.      
       .=#@@@%%@@@%#=-.         .:+%@@@@@%#x=-                  ..:=+#%@%x=-        
         .:=xx#x+:-               .-:::::..                        ..-::-.          

```


## Expectation-Maximization Clusters

```q
q)p:.ml.imax .ml.likelihood[0b;.ml.bmml[1];X] . pT
q)m:asc .ml.mode each y g:group p
q)-1 (,'/) plt each flip X[;3#value[g]1];



               +@@@+                         +@@+                      ++++         
             +@@@@+@+                     ++@@@@@@                   +@@@@@@@       
           @@@@+@@ +@@                 +@@@@@@  @@                  +@@@@+@@@       
        :x@xx:      @@x             :x@@x::@x:  @@                :@@@@:  :@@       
       +@@          @@@            @@+          @@                @@@@   +@@@       
       @@          +@@+          +@@@           @@               @@@@    @@@+       
       @@        +@@+            @@@          +@@+               @@@@  +@@@         
       @@+   +++@@+              +@@@++  +++@@++                 @@@ +@@@@+         
       @@@@@@@@++                  +@@@@@@@@+                    @@@@@@@+           
        +++++                         +++                         +@@++             


```


## Clusters Confusion Matrix

```q
q)avg y=m p
0.617
q)show .ut.totals[`TOTAL] .ml.cm[y;m p]
y| 0  1  2  3   4   5   6   7   8  9 TOTAL
-| ---------------------------------------
0| 80 0  0  0   0   12  5   0   0  0 97   
1| 0  88 0  6   0   1   3   0   18 0 116  
2| 0  4  73 9   7   0   5   0   1  0 99   
3| 1  0  1  66  2   19  0   2   2  0 93   
4| 0  0  0  4   51  13  3   33  1  0 105  
5| 9  0  1  8   5   52  10  0   7  0 92   
6| 1  1  2  0   3   4   83  0   0  0 94   
7| 0  4  0  11  30  5   0   67  0  0 117  
8| 0  1  0  15  2   10  2   0   57 0 87   
9| 1  0  0  10  37  8   1   42  1  0 100  
 | 92 98 77 129 137 124 112 144 87 0 1000 
```


## Confused Digits

```q
q)(,'/) plt each flip X[;i:3?where not y=m p]
"                                                                                    "
"                                                                                    "
"                                                                                    "
"                                             ++@@@@+                                "
"            +++@@@@@+                     +@@+++ +@@+               ++@@++@@        "
"          +@@@@@+++@@@                  @@++   +@++               +@@+     @        "
"       :x@@x::   :x@x                  x@: :x@x:                :@x:       :        "
"      @@@@   ++@@@@@+                   @@@@@                   @+        ++        "
"      @@@@@@@@@@@@@@                   @@+ ++@                  @+      +@+         "
"         +++   @@@@                ++@@+   +@+                  +@@@@@++@+          "
"              @@@@               +@@@@+++@@@                           @+           "
"             @@@@                 @@@@@@@++                           @@            "
"            @@@@+                   ++                                @             "
"            :::                                                       ::            "
q)([]y;m p) i
y p
---
9 4
8 5
9 4
```


# Sparklines

-   Michael Brown's [Dow Jones Index Data Set](https://archive.ics.uci.edu/ml/datasets/Dow+Jones+Index)
-   Use Unicode to render [sparklines](https://en.wikipedia.org/wiki/Sparkline)

```q
/ allocate x into n bins
nbin:{[n;x](n-1)&floor n*.5^x%max x-:min x}
/ generate unicode sparkline
spark:raze("c"$226 150,/:129+til 8)nbin[8]::
```

```q
q)\l dji.q
[down]loading dji data set
"unzip -n dow_jones_index.zip"
q)-1@'10#exec ((4$string stock 0),": ",.ut.spark close) by stock from dji.t;
AA  : ▅▄▃▄▇▇▇▅▅▄▄▆▇█▅▆▆▇▆▄▅▃▂▁▂
AXP : ▁▃▃▁▁▄▃▁▁▁▁▃▂▃▃▄▆▇▇██▇▅▆▆
BA  : ▁▁▂▁▂▃▃▃▃▂▁▄▄▄▃▅███▇▆▅▃▄▂
BAC : ▇█▇▆▇██▇▇▇▆▅▅▆▄▄▃▄▃▂▂▂▁▁▁
CAT : ▁▁▁▂▃▄▅▄▄▃▅▆█▇▆▆█▇▅▅▅▃▂▂▃
CSCO: ▇█▇▇█▅▅▅▄▄▃▃▃▄▃▃▃▃▃▂▂▂▁▁▁
CVX : ▁▁▂▁▃▃▄▅▆▄▆▇██▇██▆▅▅▆▅▄▄▃
DD  : ▂▂▁▂▄▆█▆▆▅▅▆▇▇▇██▆▅▅▄▂▂▂▄
DIS : ▃▃▃▂▅█████▅██▆▆▇██▆▆▆▃▂▁▁
GE  : ▂▂▅▆▆██▇▆▆▃▅▆▆▅▅▆▅▅▄▄▂▁▂▁
```


# Thank You

[Fun Q Site](https://fun-q.net)

> Never, ever underestimate the importance of having fun &#x2013; Randy Pausch


<!----- Footnotes ----->

