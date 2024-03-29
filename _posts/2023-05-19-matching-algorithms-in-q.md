---
title: "Matching Algorithms in Q"
excerpt: "Presented at KxCon 2023"
date: <span class="timestamp-wrapper"><span class="timestamp">&lt;2023-05-19 Fri&gt;</span></span>
categories: 
- Presentation
tags: 
- matching 
- funq 
- marriage
---

# Table of Contents

-   [Recorded Presentation](#org3278aa4)
-   [Motivation](#org66ff9d0)
-   [Demonstration](#orgd9c05c5)
-   [Stable Marriage (SM) Problem](#orgb0e606a)
-   [Stable Roommates (SR) Problem](#orga71a201)
-   [Hospital-Resident (HR) Problem](#org43987f1)
-   [Student-Allocation (SA) Problem](#org863ddf5)
-   [Performance](#orgdb33b42)
-   [Q Enhancements](#org5f0e1e8)
-   [Summary](#orgf30bed0)


<a id="org3278aa4"></a>

# Recorded Presentation

[Matching Algorithms in q – An Interactive Presentation](https://kx.com/videos/kx-con-23-matching-algorithms-in-q-an-interactive-presentation/)


<a id="org66ff9d0"></a>

# Motivation

-   Intern-desk pairing
-   Two distinct populations
-   Interns rank desks
-   Desks rank interns
-   Prevent back-room deals
-   Is there a single optimal pairing?


<a id="orgd9c05c5"></a>

# Demonstration

-   Two sets of index cards
-   Qgods: 1-9
-   Qbies: A-J
-   Pair-off so everyone is 'happy'
-   How did we do?


## Qgods and Qbies

```q
q)show g:.Q.n                     / define qgods
"0123456789"
q)show b:10#.Q.A                  / define qbies
"ABCDEFGHIJ"
```


## Qgod and Qbie Preferences

```q
q)\S -314159i                   / reset seed
q)show G:g!-10?/:10#enlist b    / define qgod prefs
0| "ICEBGAFDHJ"
1| "GHDCJBAIEF"
2| "HBDJFAEGIC"
3| "BIFHADGECJ"
4| "EDICJFHABG"
5| "BEHGACFDIJ"
6| "AGCIHEJBDF"
7| "GBHDEIFACJ"
8| "JGEBHADICF"
9| "FIGJBACHED"
q)show B:b!-10?/:10#enlist g    / define qbie prefs
A| "9378126054"
B| "8014967352"
C| "7284539016"
D| "3217965084"
E| "7051684329"
F| "6943720851"
G| "4501672893"
H| "7681405392"
I| "7458106239"
J| "9154860327"
```


## Qgod-Optimal Matches

-   The suitor obtains the best possible match among all stable
    matches

```q
q)eSR:.matching.sm[G;B]           / perform qgod-optimal match
q)eSR 0                           / engagements
0| C
1| G
2| D
3| H
4| I
5| E
6| A
7| B
8| J
9| F
q)eSR 1                           / remaining qgod prefs
0| "CEBGH"
1| "GHJBA"
2| "DAC"
3| "HADC"
4| "ICJHBG"
5| "EHGCJ"
6| "AHBF"
7| "BHEIAC"
8| "JBHAC"
9| "FJBAC"
```


## Qbie Matches

-   The reviewer obtains the worst possible match among all stable
    matches

```q
q)eSR 0                           / engagements
0| C
1| G
2| D
3| H
4| I
5| E
6| A
7| B
8| J
9| F
q)eSR 2                           / remaining qbie prefs
A| "9378126"
B| "8014967"
C| "72845390"
D| "32"
E| "705"
F| "69"
G| "4501"
H| "76814053"
I| "74"
J| "91548"
```


## Qbie-Optimal Matches

-   Allowing Qbies to propose first improves their matches
-   Matches are still stable

```q
q)eSR:.matching.sm[B;G]           / perform qbie-optimal match
q)eSR 0                           / engagements
A| 3
B| 8
C| 0
D| 2
E| 5
F| 6
G| 1
H| 7
I| 4
J| 9
```


<a id="orgb0e606a"></a>

# Stable Marriage (SM) Problem

Given two distinct populations how do you create matches such that no
pair prefers each other over their current matching?


## Stable Marriage Algorithm

The [Gale-Shapley](https://en.wikipedia.org/wiki/Gale%E2%80%93Shapley_algorithm) 1962 (Deferred Acceptance) algorithm:

-   All participants rank partners
-   Iteratively engage each suitor:
    -   Return early if every suitor is engaged
    -   Find preferred reviewer for next single suitor
    -   If reviewer is single, they accepted suitor
    -   Else, allow reviewer to renege and upgrade &#x2013; old suitor gets
        to try again
    -   Return updated engagement vector, and suitor and reviewer
        preference vectors


## Stable Marriage Theorems

-   The algorithm completes in a finite number of steps
-   The algorithm terminates in at most \\(n^2 - n + 1\\) iterations
-   The algorithm always produces stable matches
-   The matches are always suitor-optimal (and reviewer-*pessimal*)
-   The matches are unique if suitor-optimal and reviewer-optimal
    results are identical


## Enumerating Preference Maps

-   Humans prefer names
-   Algorithms prefer indices
-   We convert ranking dictionaries to 0-based lists by enumerating
    each value with the `?` find operator

```q
q)G                               / qgod prefs
0| "ICEBGAFDHJ"
1| "GHDCJBAIEF"
2| "HBDJFAEGIC"
3| "BIFHADGECJ"
4| "EDICJFHABG"
5| "BEHGACFDIJ"
6| "AGCIHEJBDF"
7| "GBHDEIFACJ"
8| "JGEBHADICF"
9| "FIGJBACHED"
q)key B                           / qbie enumeration vector
"ABCDEFGHIJ"
q)show S:key[B]?value G           / qgod enumerations
8 2 4 1 6 0 5 3 7 9
6 7 3 2 9 1 0 8 4 5
7 1 3 9 5 0 4 6 8 2
1 8 5 7 0 3 6 4 2 9
4 3 8 2 9 5 7 0 1 6
1 4 7 6 0 2 5 3 8 9
0 6 2 8 7 4 9 1 3 5
6 1 7 3 4 8 5 0 2 9
9 6 4 1 7 0 3 8 2 5
5 8 6 9 1 0 2 7 4 3
q)R:key[G]?value B                / gbie enumerations
```


## Stable Marriage Wrapper

-   Enumerate the suitor and reviewer dictionaries
-   Build all-null engagement vector
-   Iterator with [`.matching.sma`](#org4fa18bd) until convergence
-   Convert enumerations back to dictionaries

```q
/ given (S)uitor and (R)eviewer preferences, return the (e)ngagement
/ dictionary and remaining (S)uitor and (R)eviewer preferences for inspection
sm:{[S;R]
 us:key S; ur:key R;                      / unique suitors and reviewers
 eSR:(count[S]#0N;ur?value S;us?value R); / initial state/enumerated values
 eSR:sma over eSR;                / iteratively apply Gale-Shapley algorithm
 eSR:(us;us;ur)!'(ur;ur;us)@'eSR; / map enumerations back to original values
 eSR}
```


## Stable Marriage Implementation

-   The first line of every algorithm unpacks its arguments
-   The suitor and reviewer indices &#x2013; `Si` and `Ri` respectively &#x2013;
    are defined as variables so that the same function can be used
    for the [Stable Roommates (SR) Problem](#orga71a201)

```q
/ given (e)ngagement vector and (S)uitor and (R)eviewer preferences, find
/ next engagement, remove undesirable suitors and unavailable reviewers.
/ roommate preferences are assumed if (R)eviewer preferences are not
/ provided.
sma:{[eSR]
 n:count e:eSR 0;S:eSR Si:1;R:eSR Ri:-1+count eSR;
 mi:?[;1b] 0<count each S w:where null e;    / first unmatched with prefs
 if[mi=count w;:eSR];                        / no unmatched suitor
 rp:R ri:first s:S si:w mi;                  / preferred reviewer's prefs
 if[count[rp]=sir:rp?si;:.[eSR;(Si;si);1_]]; / not on reviewer's list
 / renege if already engaged and this suitor is better
 if[not n=ei:e?ri;if[sir<rp?ei;eSR:.[eSR;(Si;ei);1_];e[ei]:0N]];
 e[si]:ri; eSR[0]:e;                      / get engaged
 eSR[Si]:last rpS:prune[rp;eSR Si;ri;si]; / first replace suitor prefers
 eSR[Ri;ri]:first rpS;                    / order matters when used for SR
 eSR}
```


## Pruning Preference Vectors

-   Once a suitor and reviewer are engaged, we can make two optimizations:
    1.  Remove all reviewer preferences that are worse than the current suitor
    2.  Remove the reviewer from all worse suitors' preferences

```q
/ given (r)eviewer (p)refs, (S)uiter preferences and (s)uitor (i)ndice(s) and
/ (r)eviewer (i)ndice(s), return the pruned reviewer and Suitor prefs
prune:{[rp;S;ris;sis]
 if[count[rp]=i:1+max rp?sis;:(rp;S)]; / return early if nothing to do
 rp:first c:(0;i) cut rp;              / drop worse suitors from preferences
 S:@[;last c;drop;]/[S;ris];           / drop reviewers from worse suitors
 (rp;S)}
```


## Pruning Example

-   Assume suitor 0 proposes to reviewer 4
-   All suitors past 0 are removed from reviewer prefs
-   Reviewer 4 is removed from the suitors that were cut

```q
q)rpS:.matching.prune[rp:R ri;S;ri:4;si:0]; / prune
q)show rp                                   / initial reviewer prefs
7 0 5 1 6 8 4 3 2 9
q)show first rpS                            / everything past 0 is cut
7 0
q)show last rpS                   / 4 is dropped from cut reviewers
8 2 4 1 6 0 5 3 7 9
6 7 3 2 9 1 0 8 5
7 1 3 9 5 0 6 8 2
1 8 5 7 0 3 6 2 9
3 8 2 9 5 7 0 1 6
1 7 6 0 2 5 3 8 9
0 6 2 8 7 9 1 3 5
6 1 7 3 4 8 5 0 2 9
9 6 1 7 0 3 8 2 5
5 8 6 9 1 0 2 7 3
```


## Pruning Logistics

[`.matching.prune`](#orgc5e7d14) handles lists of suitors and reviewers

-   The [Stable Roommates (SR) Problem](#orga71a201) requires the Suitor and
    Reviewer preferences to be the same data structure
-   The [Hospital-Resident (HR) Problem](#org43987f1) requires the function to prune
    the **worst** of multiple residents (acting as suitor) when the
    hospital reaches capacity
-   The [Student-Allocation (SA) Problem](#org863ddf5) requires the function to
    prune multiple students (acting as suitor) **and** the **worst** of
    multiple projects (acting as reviewer)

```q
/ given (r)eviewer (p)refs, (S)uiter preferences and (s)uitor (i)ndice(s) and
/ (r)eviewer (i)ndice(s), return the pruned reviewer and Suitor prefs
prune:{[rp;S;ris;sis]
 if[count[rp]=i:1+max rp?sis;:(rp;S)]; / return early if nothing to do
 rp:first c:(0;i) cut rp;              / drop worse suitors from preferences
 S:@[;last c;drop;]/[S;ris];           / drop reviewers from worse suitors
 (rp;S)}
```


## Stable Marriage Execution

-   The implementation returns the engagements as well as the
    remaining unpruned suitor and reviewer preferences as
    dictionaries
-   Strictly speaking, we only need to return the engagement dictionary,
    but having access to the remaining preferences adds intuition

```q
q).matching.sm[B;G]
"ABCDEFGHIJ"!"3802561749"
"ABCDEFGHIJ"!("36";"867352";"06";"264";"5684";"693";"16789";"7632";"40639";"986")
"0123456789"!("IC";,"G";"HBD";"BIFHA";"EDI";"BE";"AGCIHEJBDF";"GBH";"JGEB";"FIGJ")
```


## Stable Marriage Vector Observations

-   The `?` find operator is used in 5 different ways:
    1.  Enumerate dictionary values
    2.  Search engagement vector for the next single suitor
    3.  Search engagement vector to see if reviewer is engaged
    4.  Compare ranking between suitor and existing suitor
    5.  Search reviewer preferences when pruning worse suitors
-   The engagement vector remains the same length across iterations,
    but the preference vectors shrink as suitors are pruned
-   Each iteration needs to unpack the single argument into distinct
    variables and then pack them back up for the next iteration


## Strategy

-   Suitors can not improve their results by changing their rankings
-   Reviewers **can** (sometimes) improve their results by [truncating
    their rankings](https://doi.org/10.1016/j.geb.2014.01.005) &#x2013; but risk of not getting matched at all
-   Reviewer "H" originally gets 8th suitor on their list
-   By not permitting this matching, they (and "A" as well) improve
    their match

```q
q)1+(,'/)(B?'{value[x]!key x} first .matching.sm[G]::) each 1 @[;"H";7#]\ B
A| 7 2
B| 7 7
C| 8 8
D| 2 2
E| 3 3
F| 2 2
G| 4 4
H| 8 2
I| 2 2
J| 5 5
```


<a id="orga71a201"></a>

# Stable Roommates (SR) Problem

-   What if we only had a single population?
-   Each participant is required to rank every other participant
-   It is possible that no stable solution exists


## Stable Roommates Algorithm

-   Robert W. Irving published a 2-phase solution in 1985
-   Phase 1 passes the roommate preferences to the [Gale-Shapley](#org41ff197)
    algorithm as both the suitor and reviewer
-   Since `q` does not allow passing by pointer, the [`.matching.sma`](#org4fa18bd)
    function was conditioned on how many preference lists were passed
-   Phase 2 removes 'cycles' which are rotations that produce equally
    stable solutions


## Stable Roommates Wrapper

-   Once again, the preference dictionary is enumerated
-   And the results are unenumerated before being passed back

```q
/ given (R)oomate preference dictionary, return the (a)ssignment dictionary
/ and (R)oommate preference dictionaries from each decycle stage
sr:{[R]
 ur:key R;                      / unique roommates
 aR:(count[R]#0N;ur?value R);   / initial assignment/enumerated values
 aR:sra aR;                     / apply stable roommate (SR) algorithm
 aR:ur!/:ur aR;                 / map enumerations back to original values
 aR}
```


## Stable Roommates Algorithm

-   Phase 1 applies the stable marriage ([Gale-Shapley](#org41ff197)) algorithm
-   The results of phase 1 are then passed to [`.matching.decycle`](#org1ca29c9) to
    remove unstable cycles
-   A final assignment vector is prepended to the intermediate 'decycle'
    states before being returned

```q
/ given (a)ssignment vector and (R)oomate preferences, return the completed
/ (a)ssignment vector (R)oommate preferences from each decycle stage
sra:{[aR]
 R:last sma over aR;            / apply phase 1 and throw away assignments
 R:decycle scan R;              / apply phase 2
 aR:enlist[last[R][;0]],R;      / prepend assignment vector
 aR}
```


## Decycling Roommate Assignments

-   The algorithm has no solution if any participant goes unmatched
-   The algorithm terminates when all participants are uniquely matched
-   Cycles are discovered with the `.matching.cycle` function and
    removed with the `.matching.pruner` roommate prune function &#x2013;
    neither of which will be discussed

```q
/ phase 2 of the stable roommates (SR) problem removes all cycles within the
/ remaining candidates leaving the one true stable solution
decycle:{[R]
 if[any 0=c:count each R;'`unstable]; / unable to match a roommate
 if[count[c]=i:?[;1b] c>1;:R];        / first roommate with multiple prefs
 c:cycle[R] enlist (i;R[i;0]);        / build the cycle starting here
 R:pruner/[R;c[;1];-1 rotate c[;0]];  / prune prefs based on dropped cycle
 R}
```


## Stable Roommates Setup

-   A worked example (including decycling) can be found on the Stable
    Roommates Problem [Wikipedia page](https://en.wikipedia.org/wiki/Stable_roommates_problem)
-   Each participant ranks all **other** participants
-   Even though these are integers, the algorithm requires 0-index
    enumerations so we create a dictionary and supply it to the
    algorithm wrapper

```q
q)show R:(1+til count R)!R:get each read0 `wmate.txt
1| 3 4 2 6 5
2| 6 5 4 1 3
3| 2 4 5 1 6
4| 5 2 3 6 1
5| 3 1 2 4 6
6| 5 1 3 4 2
```


## Stable Roommates Execution

-   The [`.matching.sr`](#org2676b5f) function produces:
    -   the assignment dictionary
    -   the results of the [Gale-Shapley](#org41ff197) algorithm
    -   each step of the decycling process
-   Notice how the assignment dictionary is symmetric. 1 is assigned
    6 and 6 is assigned 1

```q
q).matching.sr R
1 2 3 4 5 6!6 4 5 2 3 1
1 2 3 4 5 6!(4 2 6;6 5 4 1 3;2 4 5;5 2 3 6 1;3 2 4;1 4 2)
1 2 3 4 5 6!(2 6;6 5 4 1;4 5;5 2 3;3 2 4;1 2)
1 2 3 4 5 6!(,6;5 4;4 5;2 3;3 2;,1)
1 2 3 4 5 6!(,6;,4;,5;,2;,3;,1)
```


## Stable Roommates Vector Observations

-   The `?` find operator is used two more times:
    1.  Search roommate preference counts for decycle opportunities
    2.  Search chain for 'tail' location so the non-repeating section
        can be excluded from the cycle


<a id="org43987f1"></a>

# Hospital-Resident (HR) Problem

-   What if there was capacity for more than a single match?
-   Conceptually the same as SM but algorithm needs to be generalized
    for multiple matches
-   The hospitals, in this case, may have capacity greater than one


## National Residency Matching Program

-   1940s &#x2013; Newly graduating MDs were being given earlier and
    earlier offers resulting in poor matches and/or *exploding*
    offers
-   1950s &#x2013; The [National Residency Matching Program](https://www.nrmp.org/) was created to
    match residents to hospitals in a hospital-optimal stable
    allocation
-   1998 &#x2013; Matching updated to the student-optimal [Roth-Peranson
    algorithm](https://doi.org/10.1257/aer.89.4.748) that also permits couples to submit ranked pairs of
    position
-   2003 &#x2013; Alvin Roth published a summary of the NRMP in his paper
    [The Origins, History, and Design of the Resident Match](https://jamanetwork.com/journals/jama/fullarticle/195998)
-   2012 &#x2013; Nobel prize in Economics was given to Alvin Roth and
    Lloyd Shapley (David Gale had passed away in 2008).


## Hospital-Resident Algorithm

-   Initialize all residents to be unmatched and hospitals to have
    an empty match list

-   Hospital-Optimal

    -   Fill each hospital to capacity with most-preferred residents
    -   Allow resident to upgrade for improved offers &#x2013; forcing
        hospital to make next-best offer

-   Resident-Optimal

    -   Match each resident to most-preferred below-capacity hospital
    -   Allow hospital to upgrade for improved offers &#x2013; forcing
        resident to make next-best offer


## Hospital-Resident Wrapper

-   The interface for both the hospital-optimal and resident-optimal
    algorithms are the same and they both require the mapping from
    dictionaries to enumerated lists (and back again)

```q
/ hospital resident (HR) problem wrapper function that enumerates the inputs,
/ calls the hr function and unenumerates the results
hrw:{[hrf;C;H;R]
 uh:key H; ur:key R;
 hrHR:((count[H];0)#0N;count[R]#0N;ur?value H;uh?value R);
 hrHR:hrf[C uh] over hrHR;
 hrHR:(uh;ur;uh;ur)!'(ur;uh;ur;uh)@'hrHR;
 hrHR}

hrr:hrw[hrra]                  / hospital resident (resident-optimal)
hrh:hrw[hrha]                  / hospital resident (hospital-optimal)
```


## Hospital-Resident Resident-Optimal Implementation

-   To find next available resident we limit our search to unmatched
    residents with viable preferences
-   The `?` find operator is used again to find the first such
    resident
-   Drop student when over capacity and prune when at capacity

```q
/ given hospital (c)apacity and (h)ospital matches, (r)esident matches,
/ (H)ospital and (R)esident preferences, find next resident-optimal match
hrra:{[c;hrHR]
 h:hrHR 0;r:hrHR 1;H:hrHR 2;R:hrHR 3;
 mi:?[;1b] 0<count each R w:where null r; / first unmatched with prefs
 if[mi=count w;:hrHR];                    / nothing to match
 hp:H hi:first R ri:w mi;                 / preferred hospital
 if[not ri in hp;:.[hrHR;(3;ri);1_]];     / hospital rejects
 ch:count ris:h[hi],:ri; r[ri]:hi;        / match
 if[ch>c hi;                              / over capacity
  wri:hp max hp?ris;                      / worst resident
  ch:count ris:h[hi]:drop[ris;wri]; / drop resident from hospital match
  r[wri]:0N;                        / drop resident match
  ];
 if[ch=c hi; H[hi]:first hpR:prune[hp;R;hi;ris]; R:last hpR]; / prune
 (h;r;H;R)}
```


## Hospital-Resident Hospital-Optimal Implementation

-   To find the next available hospital we ignore hospitals at
    capacity, then drop existing matches from hospital preferences
-   The `?` find operator is used again to find the first such
    hospital
-   Prune on every match

```q
/ given hospital (c)apacity and (h)ospital matches, (r)esident matches,
/ (H)ospital and (R)esident preferences, find next hospital-optimal match
hrha:{[c;hrHR]
 h:hrHR 0;r:hrHR 1;H:hrHR 2;R:hrHR 3;
 w:where c>count each h;        / limit to hospitals with capacity
 mi:?[;1b] 0<count each m:H[w] except' h w; / first with unmatched prefs
 if[mi=count w;:hrHR];                      / nothing to match
 rp:R ri:first m mi; hi:w mi;               / preferred resident
 if[not hi in rp;:.[hrHR;(2;hi);1_]];       / resident preferences
 if[not null ehi:r ri; h:@[h;ehi;drop;ri]]; / drop existing match
 h[hi],:ri; r[ri]:hi;                           / match
 R[ri]:first rpH:prune[rp;H;ri;hi]; H:last rpH; / prune
 (h;r;H;R)}
```


## Hospital-Resident Setup

-   The Python [matching](https://matching.readthedocs.io/en/latest/index.html) package provides links to [hospital capacity](https://zenodo.org/record/3688091/files/capacities.yml)
    and [hospital](https://zenodo.org/record/3688091/files/hospitals.yml) and [resident](https://zenodo.org/record/3688091/files/residents.yml) preference data in YAML format
-   Convert and store YAML files in JSON format

```q
q)2#C:.j.k raze read0 `:capacities.json
Dewi Sant     | 30
Prince Charles| 30
q)2#H:`$.j.k raze read0 `:hospitals.json
Dewi Sant     | `093`067`136`177`060`196`197`184`156`075`092`034`111`174`171`064`022`..
Prince Charles| `124`146`027`017`174`133`001`106`097`179`018`006`172`057`163`103`081`..
q)2#R:`$.j.k raze read0 `:residents.json
000| `Royal Glamorgan`Prince of Wales`Dewi Sant`Royal Gwent`Prince Charles
001| `Prince of Wales`Royal Gwent`Royal Glamorgan`University`Prince Charles`St. David
```


## Hospital-Resident Execution

-   Both approaches return a hospital -> residents dictionary,
    resident -> hospital dictionary as well as the pruned hospital
    and resident preference dictionaries

```q
q)first hrHR:.matching.hrr[C;H;R]
Dewi Sant      | `010`011`013`019`022`023`037`039`040`045`046`065`067`072`079`083`086..
Prince Charles | `007`008`009`026`027`031`034`041`044`051`059`061`069`070`087`107`110..
Prince of Wales| `001`004`017`030`035`048`064`078`088`097`111`112`124`128`132`138`140..
Royal Glamorgan| `000`014`015`016`018`021`024`029`033`042`053`058`073`075`076`089`096..
Royal Gwent    | `002`006`028`036`054`068`071`090`091`105`120`121`141`145`155`161`163..
St. David      | `005`012`020`032`043`049`056`060`063`077`084`085`092`093`094`099`101..
University     | `038`047`050`052`055`057`062`074`080`082`098`100`102`103`109`122`148..
q)5#hrHR 1
000| Royal Glamorgan
001| Prince of Wales
002| Royal Gwent
003| University
004| Prince of Wales
```


<a id="org863ddf5"></a>

# Student-Allocation (SA) Problem

-   Let's relax the constraints once more and insert an intermediary
    between the suitor and reviewer
-   Supervisors have projects
-   Students rank projects
-   Supervisors rank all students that have ranked their projects


## Student-Allocation Algorithm

-   Initialize all students to be unmatched and supervisors and
    projects to have empty match lists

-   Supervisor-Optimal

    -   Fill each project to capacity with most-preferred students
    -   Allow student to upgrade for improved offers &#x2013; forcing
        supervisor to make next-best offer

-   Student-Optimal

    -   Match each student to most-preferred below-capacity project
    -   Allow supervisor to upgrade for improved offers &#x2013; forcing
        student to make next-best offer


## Student-Allocation Wrapper

-   The interface for both the supervisor-optimal and student-optimal
    algorithms are, once again, the same and they both require the
    mapping from dictionaries to enumerated lists (and back again)

```q
/ student-allocation (SA) problem wrapper function that enumerates the
/ inputs, calls the sa function and unenumerates the results
saw:{[saf;PC;UC;PU;U;S]
 up:key PU; uu:key U; us:key S; / unique project, supervisors and students
 pusUS:((count[PU];0)#0N;(count[U];0)#0N;count[S]#0N;us?value U;up?value S);
 pusUS:saf[PC up;UC uu;uu?PU up] over pusUS;
 pusUS:(up;uu;us;uu;us)!'(us;us;up;us;up)@'pusUS;
 pusUS}

sas:saw[sasa]                   / student-allocation (student-optimal)
sau:saw[saua]                   / student-allocation (supervisor-optimal)
```


## Student-Allocation Student-Optimal Implementation

-   Limit search to unmatched students with viable preferences
-   The `?` find operator is used again to find the first such
    student
-   Drop student when over capacity and prune when at capacity

```q
/ given (p)roject (c)apacity, s(u)pervisor (c)apacity, (p)roject to
/ s(u)pervisor map and (p)roject matches, s(u)pervisor matches, (s)tudent
/ matches, s(U)pervisor preferences and (S)tudent preferences, find next
/ student-optimal match
sasa:{[pc;uc;pu;pusUS]
 p:pusUS 0;u:pusUS 1;s:pusUS 2;U:pusUS 3;S:pusUS 4;
 mi:?[;1b] 0<count each S w:where null s; / first unmatched student
 if[mi=count w;:pusUS];                   / nothing to match
 up:U ui:pu pi:first S si:w mi; / preferred project's supervisors preferences
 cu:count usis:u[ui],:si;cp:count psis:p[pi],:si;s[si]:pi; / match
 if[cp>pc pi;                         / project over capacity
  wsi:up max up?psis; s[wsi]:0N;      / worst student
  cp:count psis:p[pi]:drop[psis;wsi]; / drop from project
  cu:count usis:u[ui]:drop[usis;wsi]; / drop from supervisor
  ];
 if[cu>uc ui;                         / supervisor over capacity
  wsi:up max up?usis;                 / worst student
  p:@[p;s wsi;drop;wsi]; s[wsi]:0N;   / drop from other project
  cu:count usis:u[ui]:drop[usis;wsi]; / drop from supervisor
  ];
 if[cp=pc pi;S:last prune[up;S;pi;psis]]; / prune
 if[cu=uc ui;U[ui]:first upS:prune[up;S;where pu=ui;usis]; S:last upS];
 (p;u;s;U;S)}
```


## Student-Allocation Supervisor-Optimal Implementation

-   The [`.matching.nextusp`](#orgbdb2bf6) function is used to find the next
    available supervisor, student and project to match
-   Iterate until either a match is found, or no matches available
-   Iteration passes the supervisor index and increments each time

```q
/ given (p)roject (c)apacity, s(u)pervisor (c)apacity, (p)roject to
/ s(u)pervisor map and (p)roject matches, s(u)pervisor matches, (s)tudent
/ matches, s(U)pervisor preferences and (S)tudent preferences, find next
/ supervisor-optimal match
saua:{[pc;uc;pu;pusUS]
 p:pusUS 0;u:pusUS 1;s:pusUS 2;U:pusUS 3;S:pusUS 4;
 ubc:uc>count each u;                          / supervisors below capacity
 pbc:pc>count each p;                          / projects below capacity
 usp:(1=count::) nextusp[pbc;ubc;pu;p;S;U]/ 0; / iterate across supervisors
 if[not count usp;:pusUS];                     / no further matches found
 ui:usp 0; sp:S si:usp 1; pi: usp 2;           / unpack
 if[not null epi:s si; u:@[u;pu epi;drop;si]; p:@[p;epi;drop;si]]; / drop
 u[ui],:si; p[pi],:si; s[si]:pi;                                   / match
 S[si]:first prune[sp;U;();pi];                                    / prune
 (p;u;s;U;S)}
```


## Student-Allocation Supervisor Search

-   Finding the next supervisor's favorite student's favorite
    supervisor's project is the slowest function
-   The python implementation has a triple-nested `for` loop and
    breaks out immediately upon success

```q
/ given (p)rojects (b)elow (c)apacity boolean vector, s(u)pervisors (b)elow
/ (c)apacity vector, (p)roject to s(u)pervisor map, (p)roject matches,
/ (S)tudent preferences, s(U)pervisor preferences and a single s(u)pervisor
/ (i)ndex, return the s(u)pervisor's preferred (s)tudent and their preferred
/ (p)roject (that is mapped to the supervisor) as a triplet (u;s;p). if no
/ match is found, return the next supervisor index ui.  return an empty list
/ if all supervisors have been exhausted.
nextusp:{[pbc;ubc;pu;p;S;U;ui]
 if[ui=count U;:()];                  / no more supervisors
 if[not ubc ui;:ui+1];                / supervisor at capacity
 pis:S sis:U ui;                      / unpack students and their projects
 pis:pis@'where each (pbc&ui=pu) pis; / supervisor's projects with capacity
 pis:pis@'where each not sis (in/:)' p pis;  / not already matched
 if[not count sp:raze sis (,/:)' pis;:ui+1]; / (student;project)
 usp:ui,first sp;                            / (supervisor;student;project)
 usp}
```


## Student-Allocation Setup

-   The Python [matching](https://matching.readthedocs.io/en/latest/index.html) package provides links to [student](https://zenodo.org/record/3514287/files/students.csv), [project](https://zenodo.org/record/3514287/files/projects.csv) and
    [supervisor](https://zenodo.org/record/3514287/files/supervisors.csv) capacity and preference data in CSV format

```q
q)2#s:2!("JJ",(-2+count first x)#"S";1#",") 0: x:read0 `:students.csv
name   rank| 0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18 19 20 21 22 23 ..
-----------| ------------------------------------------------------------------------..
190000 3   | G2 P1 V2 S0 A0 O0 L0 D2 K1 V1 R2 G2 Y2 G2 W0 K0 X0 O1                   ..
190001 56  | Q0 P1 P0 M1 N0 P1 T2 N1 I1 K0 P3 X1 F0 P1 S0 C0 Z0 L0 H2                ..
q)2#p:("SJS";1#",") 0: `:projects.csv
code capacity supervisor
------------------------
A0   2        A         
A1   3        A         
q)2#u:("SJ";1#",") 0: `:supervisors.csv
name capacity
-------------
A    3       
B    1       
```


## Student-Allocation Execution

-   Both approaches return supervisor -> student, project -> student
    and student -> project dictionaries as well as the pruned
    supervisor and student preference dictionaries

```q
q)d:preprocess[u;p;s]
q)5#first pusUS:.matching.sas . d`PC`UC`PU`U`S
A0| `long$()
A1| 190019 190034
A2| ,190017
B0| `long$()
B1| ,190091
q)5#pusUS 1
A| 190019 190034 190017
B| ,190091
C| 190003 190062 190068 190079 190070
D| 190008 190009 190015 190039 190056
E| 190022 190063
q)5#pusUS 2
190000| G2
190001| Q0
190002| U0
190003| C0
190004| I0
```


<a id="orgdb33b42"></a>

# Performance

> The key to performance is elegance, not battalions of special cases
> &#x2013; Jon Bentley and Doug McIlroy


## Code Profiling

-   When the whole implementation is ~15 lines of code, we need a line-profiler
-   [Leslie Goldsmith](https://www.arraycast.com/episodes/episode47-leslie-goldsmith) created the [`qprof`](https://github.com/LeslieGoldsmith/qprof) line-profiler in 2015
    
    ```q
    q)\l prof.q
    q).prof.prof `.matching
    q)\ts:100 eSR:.matching.sm[G;B]
    130 8720
    q).prof.report`
    Name            Line Stmt                           Count Total     Own       Pct   
    ------------------------------------------------------------------------------------
    .matching.prune 3    S:@[;last c;drop;]/[S;ris];    1400  00:00.036 00:00.028 22.07%
    .matching.sma   9    eSR[Si]:last rpS:prune[rp;eSR  1600  00:00.063 00:00.013 10.25%
    .matching.sm    3    eSR:smpa over eSR;             100   00:00.128 00:00.008 6.59% 
    .matching.sma   2    mi:?[;1b]0<count each S w:wher 1700  00:00.008 00:00.008 6.77% 
    .matching.drop  0    x _ x?y                        5200  00:00.007 00:00.007 5.95% 
    .matching.prune 2    rp:first c:(0;i)cut rp;        1400  00:00.006 00:00.006 4.84% 
    .matching.sma   3    if[mi=count w;:eSR];           1700  00:00.006 00:00.006 4.78% 
    .matching.sma   4    rp:R ri:first s:S si:w mi;     1600  00:00.006 00:00.006 4.61% 
    .matching.sma   6    if[not n=ei:e?ri;if[sir<rp?ei; 1600  00:00.006 00:00.006 5.29% 
    .matching.sma   8    e[si]:ri;eSR[0]:e;             1600  00:00.006 00:00.006 4.98%
    ..
    q)
    ```


## Timing Setup

-   Using PyKX we can access both implementations from python
-   Using `np.array` prevents q from using mixed lists
-   Need to [increase recursion limit](https://github.com/daffidwilde/matching/issues/139) due to `copy.deepcopy` call
    
    ```python
    sys.setrecursionlimit(10000) # overcome call to copy.deepcopy
    ```


## Timing Result

-   Using the `timeit` package we can compare the execution times
-   Python implementation is ~10x slower than q (and worsens with
    increased dimensions)
    
    ```python
    >>> assert smq(sd, rd) == smp(sd, rd)  # assert equality
    >>> timeit.timeit('smq(sd,rd)', number=1, globals=globals())
    0.05793206300000975
    >>> timeit.timeit('smp(sd,rd)', number=1, globals=globals())
    0.5015301169999589
    ```


<a id="org5f0e1e8"></a>

# Q Enhancements

> Nothing happens unless first we dream  &#x2013; Carl Sandburg

> Well, the the J mentality is that only J is needed and they're
> right &#x2013; Marshall Lochbaum "Naming is Hard" [The Array Cast](https://www.arraycast.com/)


## PyKX Type Handling

-   Uniform lists are promoted to vectors in `q`
    
    ```python
    >>> _ = kx.q("0N!(1;2)")
    1 2
    ```
-   But not when passed from python
    
    ```python
    >>> _ = kx.q("0N!",[1,2])
    (1;2)
    ```
-   For that you need `numpy`
    
    ```python
    >>> _ = kx.q("0N!",np.array([1,2]))
    1 2
    ```
-   But what about dictionary keys?
    
    ```python
    >>> _ = kx.q("0N!",{1:np.array([1,2]),2:np.array([2,3])})
    (1;2)!(1 2;2 3)
    ```


## Multiple Assignment

-   Using `over` and `scan` requires packing and unpacking complex
    state for each iteration
    
    ```q
    p:pusUS 0;u:pusUS 1;s:pusUS 2;U:pusUS 3;S:pusUS 4;
    ```
-   Multiple assignment would make this much more elegant
    
    ```q
    q)(p;u;s;U;S):pusUS;
    'assign
    [0]  (p;u;s;U;S):pusUS;
                    ^
    ```


## YAML Support

-   The hospital-resident problem [capacities](https://zenodo.org/record/3688091/files/capacities.yml), [hospitals](https://zenodo.org/record/3688091/files/hospitals.yml) and [residents](https://zenodo.org/record/3688091/files/residents.yml)
    inputs are stored in YAML files
-   `KDB Insights` [configuration](https://code.kx.com/insights/1.4/enterprise/assemblies/building-assemblies.html) is stored in YAML files
-   YAML supports comments
-   YAML supports more types than JSON (booleans, dates, timestamps, null)

```q
q).y.k "\n" sv read0 `:hospitals.yml
```


## Native `assert`

-   Every project I create requires the definition of a `.util.assert`
    function to build unit tests.
    
    ```q
    / throw verbose exception if x <> y
    assert:{if[not x~y;'`$"expecting '",(-3!x),"' but found '",(-3!y),"'"]}
    ```

-   Don't need a complex `qunit` framework, just an `assert`
    
    ```q
    q).util.assert[`foo] `bar
    'expecting '`foo' but found '`bar'
    [0]  .util.assert[`foo] `bar
         ^
    ```


<a id="orgf30bed0"></a>

# Summary

-   Which of these algorithms *matches* my intern problem?
-   Did I get assigned more appropriate interns?
-   Deferred acceptance algorithms appear in real life
    -   [Cognitive Radio Networks](https://doi.org/10.1016/j.icte.2018.01.008)
    -   [National Residency Matching Program](https://www.nrmp.org/)
    -   [The Boston Public School Match](https://blueprintcdn.com/wp-content/uploads/2005/12/Boston-Public-High-School-Math.pdf)
    -   [The New York City High School Match](https://economics.mit.edu/research/publications/new-york-city-high-school-match)
    -   [The Kidney Donor Problem](https://nap.nationalacademies.org/read/23508/)
-   Vector implementations are faster than object-oriented ones
-   The algorithms are heavily reliant on the `?` find operator
-   The `q` `matching` library can be found on github:
    [https://github.com/psaris/matching](https://github.com/psaris/matching)
-   This presentation can be found at [https://nick.psaris.com](https://nick.psaris.com)


<!----- Footnotes ----->

