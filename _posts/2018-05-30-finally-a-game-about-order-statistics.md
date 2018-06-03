---
layout: post
title: Finally, a game about order statistics
tags: [board games, probability, order statistics]
mathjax: true
excerpt_separator: <!--more-->
---

The last couple months, I've seen buzz about a small, quick German game
(if you're on the 'its a game' side of the argument).
<!--more-->

The Mind is a co-operative card game where players attempt to lay down
cards from their hidden hands in ascending order using no communication
except the psychic link between them. Unless you're Uri Geller (is he
still culturally relevant enough for that reference?), the real decision
is about deciding how likely it is your card is the lowest one left,
given what you know about its value, the number of other cards left to
be played, and how your partners are playing.

This is the catch of the game, something the rule book found important
enough to reference as a secret[1] and print upside down--that the bond
between players is just a synthesis of information about time, facial
expressions, and body language. [2]: Which I'm not totally giving away.
I'll say, though, I was all revved up for an exciting revelation and I
was disappointed.

I'll leave the game criticism and reviews to people with more insight
and name recognition respectively, but I have to comment on the
concentration phase, which I think is the center of gravity for this
game some people might overlook. I'm skeptical about psychic powers, so
much so that it's between difficult and impossible to be earnest that
I'm using a shared mental connection to suss out when it's my turn to
play a card. When I read the rules and first sat down for a match, I
thought the supernatural theme would feel like either a self-aware wink
or an inside joke I wasn't quite in on. But, I underestimated the value
of the explicit action that lets you 'refocus concentration' with the
group. Whenever a player started to rise out of the shared delusion,
we'd call time, put a palm on the table, and re-calibrate our bond. To
me, this is one of the big successes of the game--it's a big thematic
joke that wants the players to be the straight man.

But there's a cool statistics problem under the theme. You're trying to
estimate successive **order statistics**[3] of the sample of initial
cards, conditional on the number left, the previous order statistics,
and nonverbal cues from other players. The nonverbal information is what
takes this game from a cold (albeit, difficult) probability calculation
to a rewarding, interactive team experience. But let's ignore that for a
bit and use statistics to try to suck the jelly out of this doughnut.

Statement
---------

Let *X*<sub>1</sub>, ..., *X*<sub>*n*</sub> ∈ {1, ..., 100} be a sample
drawn **without replacement**[4] from that discrete uniform
distribution. Assume we know *k* of these (without loss of generality,
call them the last *k* samples):
*X*<sub>*n* − *k* + 1</sub> = *x*<sub>*n* − *k* + 1</sub>, ⋯, *X*<sub>*n*</sub> = *x*<sub>*n*</sub>
and call the minimum of these known values *m*.

We want to know the probability that a given order statistic is less
than the minimum of the known values, conditioning on previous order
statistics and the known values:
*p*(*X*<sub>(*i*)</sub> &lt; *m*|*X*<sub>(1)</sub>, …, *X*<sub>(*i* − 1)</sub>, *X*<sub>*n* − *k* + 1</sub> = *x*<sub>*n* − *k* + 1</sub>, …*X*<sub>*n*</sub> = *x*<sub>*n*</sub>)
 In the context of The Mind, this should tell us the probability we
should play our card now.

To make things simpler, I'm sometimes going to give the group of things
we condition on a shorthand, *C*

$$
\\begin{align}
p(X\\_i | Y) 
 = 1 \\\\
&= \\sum\\_{i=1}\^10 x\^2
\\end{align}
$$

$$
\\begin{align}
     p(X\_{(i)} &lt; x | X\_{(1)},\\dots,X\_{(j-1)},X\_{n-k+1} = x\_{n-k+1},\\dots X\_n = x\_n)
  &= 1 - p(X\_{(i)} \\geq x | X\_{(1)},\\dots,X\_{(i-1)},X\_{n-k+1} = x\_{n-k+1},\\dots X\_n = x\_n) \\
  &= 1 - p(X\_1 \\geq x, X\_2 \\geq x, \\dots, X\_n\\geq x | C) \\
  &= 1 - p(X\_1 \\geq x | C) \\cdot p(X\_2 \\geq x | X\_1 \\geq x, C) \\cdots p(X\_n \\geq x | X\_1 \\geq x, \\dots, X\_{n-1} \\geq x, C) \\
  &= 1 - p(X\_1 \\geq x | C) \\cdot p(X\_2 \\geq x | X\_1 \\geq x, C) \\cdots p(X\_{n-k} \\geq x | X\_1 \\geq x, \\dots, X\_{n-k-1} \\geq x, C) \\
  &= 1 - \\frac{100 - x - (k - 1)}{100 - X\_{(i-1)} - k} \\cdot \\frac{100 - x - (k - 1) - 1}{100 - X\_{(i)} - k - 1} \\cdots \\frac{100 - x - (k - 1) - (j-1)}{100 - X\_{(i)} - k - (j - 1)} \\cdots \\frac{100 - x - (k - 1) - (n - k - 1)}{100 - X\_{(i)} - k - (n - k - 1)} \\
  &= 1 - \\prod\_{j=0}^{n - k - 1} \\frac{100 - x - (k - 1) - j}{100 - X\_{(i)} - k - j}
\\end{align}
$$

 From line 2 to 3, the *X*<sub>*j*</sub> are drawn without replacement,
so there's dependence From line 3 to 4, use the fact we know
*X*<sub>*n* − *k* + 1</sub>, …, *X*<sub>*n* − 1</sub> &gt; *X*<sub>*n*</sub> = *x*,
so those probabilities are all 1 Line 4 could (should?) go directly to
the summation, but writing it out helped me From line 4 to 5, think of
each probability as possible 'outs' over all draws (considering known
order statistics and your hand limit the draws and outs). Outs are total
cards left that are not in your hand and are greater than your smallest
card

Now we want the probabilities of each unknown card being at least *x*.
We can find these by counting the possible draws and 'outs.'

-   Why is it so satasfying to complete a run of close values? Because
    you can see the cliff so obviously?
-   Why ninja stars?

[1] The *k*<sup>*t**h*</sup> order statistic of a sample, denoted
*X*<sub>(*k*)</sub>, is the *k*<sup>*t**h*</sup> smallest value. For
example, the first order statistic of {3, 5, 12, 20} is 3 and the third
order statistic is 12. We usually call the first and last order
statistics the minimum and maximum.

[2] The *k*<sup>*t**h*</sup> order statistic of a sample, denoted
*X*<sub>(*k*)</sub>, is the *k*<sup>*t**h*</sup> smallest value. For
example, the first order statistic of {3, 5, 12, 20} is 3 and the third
order statistic is 12. We usually call the first and last order
statistics the minimum and maximum.

[3] The *k*<sup>*t**h*</sup> order statistic of a sample, denoted
*X*<sub>(*k*)</sub>, is the *k*<sup>*t**h*</sup> smallest value. For
example, the first order statistic of {3, 5, 12, 20} is 3 and the third
order statistic is 12. We usually call the first and last order
statistics the minimum and maximum.

[4] This will be important. Finding the distribution of order statistics
is tricky, but when the sample isn't independent (and identically
distributed, but we're good there), we can't lean on the textbook
approach so much.