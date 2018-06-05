---
layout: post
title: Finally, a game about order statistics
tags: [board games, probability, order statistics]
mathjax: true
excerpt_separator: <!--jump-->
---

The last couple months, I've been hearing buzz about a short co-op card game called [The Mind](https://boardgamegeek.com/boardgame/244992/mind) and toward the end of May I finally sat down to play it with one of my regular groups. After a round, it seemed like there was a cool probability exercise in there. So, like explaining the punch line of a joke or sticking a straw in a jelly doughnut, I want to see if I can suck the fun out of this!
<!--jump-->

## The Mind

The Mind is a co-operative card game where players attempt to lay down cards from their hidden hands in ascending order using no communication except the psychic link between them. Unless you're Uri Geller (is he still culturally relevant enough for that reference?), the real decision is about deciding how likely it is your card is the lowest one left, given what you know about its value, the number of other cards left to be played, and how your partners are playing.

This is the catch of the game, something the rule book found important enough to reference as a secret[^1] and print upside down--that the bond between players is just a synthesis of information about time, facial expressions, and body language.

[^1]: Which I'm not totally giving away. I'll say, though, I was all revved up for an exciting revelation and I was disappointed.

I'll leave the game criticism and reviews to people with more insight and name recognition respectively, but I have to comment on the concentration phase, which I think holds this game together. Most of us are skeptical about psychic powers, so much so that it's between difficult and impossible to be earnest that we're using a shared mental connection to know when to play a card. When I read the rules and first sat down for a match, I thought the supernatural part would feel like either a self-aware wink or an inside joke I wasn't quite in on, but I underestimated the value of the explicit action that lets you stop the game to 'refocus concentration' with the group. Whenever a player started to rise out of the shared delusion (maybe after a close call or a giggle fit), we'd call time, put a palm on the table, and re-calibrate our bond. To me, this makes The Mind what it is. There's a big thematic joke that's (at least for me) only funny if you don't look directly at it. And the game gets you to play the straight man with this mechanically pointless action. I love it; it's like a the theme evolved into the mechanics and left this vestigial organ.

## Probability

But there's a cool probability problem under the theme. You're trying to estimate the probability that your card is a particular *order statistic*[^2] of the sample of initial cards, conditional on the number left, the previous order statistics, and nonverbal cues from other players. The nonverbal information is what takes this game from a cold probability calculation to a rewarding, interactive team experience, but let's start with the problem statement.

[^2]: The $k^{th}$ order statistic of a sample, denoted $X_{(k)}$, is the $k^{th}$ smallest value. For example, the first order statistic of $\{3, 5, 12, 20\}$ is $3$ and the third order statistic is $12$. We usually call the first and last order statistics the minimum and maximum.

## Statement
Let $X_1,...,X_n \in \{1,...,100\}$ be a sample drawn *without replacement*[^3] from that discrete uniform distribution. Assume we know $k$ of these (without loss of generality, call them the last $k$ samples): $X_{n-k+1} = x_{n-k+1}, \cdots, X_n = x_n$ and call the minimum of these known values $x$. 

[^3]: This will be important. Finding the distribution of order statistics is tricky, but when the sample isn't independent (and identically distributed, but we're good there), we can't lean on the intro stat theory approach so much.

When we decide whether to play our lowest card, we want to know the probability that the given order statistic is less than that known minimum, conditioning on previous order statistics, known card values, and cards left in play: 
$$p(X_{(i)} \geq x | X_{(1)},\dots,X_{(i-1)},X_{n-k+1} = x_{n-k+1},\dots X_n = x_n)$$ 
In the context of The Mind, this should tell us the probability we should play our card now.

To make things not run off the page on desktop (sorry mobile users, you can scroll), I'm sometimes going to give the group of things we condition on a shorthand, $C$.

$$
\begin{aligned}
     p(X_{(i)} \geq x | C)
  &= p(X_{(i)} \geq x | X_{(1)},\dots,X_{(i-1)},X_{n-k+1} = x_{n-k+1},\dots X_n = x_n) \\
  &= p(X_1 \geq x, X_2 \geq x, \dots, X_n> x | C) \\
  &= p(X_1 \geq x | C) \cdot p(X_2 \geq x | X_1 \geq x, C) \cdots p(X_n \geq x | X_1 \geq x, \dots, X_{n-1} > x, C) \\
  &= p(X_1 \geq x | C) \cdot p(X_2 \geq x | X_1 \geq x, C) \cdots p(X_{n-k} \geq x | X_1 \geq x, \dots, X_{n-k-1} \geq x, C) \\
  &= \frac{100 - x - (k - 1)}{100 - X_{(i-1)} - k} \cdot \frac{100 - x - (k - 1) - 1}{100 - X_{(i)} - k - 1} \cdots \frac{100 - x - (k - 1) - (j-1)}{100 - X_{(i)} - k - (j - 1)} \cdots \frac{100 - x - (k - 1) - (n - k - 1)}{100 - X_{(i)} - k - (n - k - 1)} \\
  &= \prod_{j=0}^{n - k - 1} \frac{100 - x - (k - 1) - j}{100 - X_{(i)} - k - j}
\end{aligned}
$$

From line 2 to 3, the $X_j$ are drawn without replacement, so there's dependence
From line 3 to 4, use the fact we know $X_{n-k+1},\dots, X_{n-1} > X_n = x$, so those probabilities are all 1
Line 4 could (should?) go directly to the summation, but writing it out helped keep track of everything

### In English, please

The insight here that kind of undercuts the 'it's tricky' claim in footnote 3 is that this (like so many probability problems) is just fancy counting and division. Think of it like calculating outs in poker. The order statistic $X_{(i)} \geq x$ when each card $X_i \geq x$. The probability card $X_i \geq x$ is the number of ways you could draw a card with value $\geq x$ divided by the number of possible draws. Then we use the information we condition on to figure out how many cards are left and how many $\geq x$. Remember that cards have to be played sequentially, so there are no possible draws left less than the previously revealed card (order statistic $X_{(i-1)}$, and there are some cards we're holding that are above our minimum, which further limits the outs.

## Confirmation by simulation

Ok, not *confirming*, just making myself more confident. But that doesn't rhyme.

We've got the explicit expression now. It's got a product, it doesn't depend on the values of any card but our minimum (the rest just change the count of remaining cards), and a few test cases pass sanity checks. But it's easy for mistakes and oversights to sneak in, so, when it's easy to simulate the result, I like to check it that way too.

Here's a function that takes the relevant information and returns the probability your minimum value card is *the* minimum value card. If you supply the optional n\_sim, it'll shoot back a list that also contains the simulated probability.

``` r
mind <- function(cards_left, my_cards, my_min, prev_min, n_sim = NULL){
  # probability from derived expression
  out <- prod((100 - my_min - (my_cards - 1) - (0:(cards_left - my_cards - 1)))
              / (100 - prev_min - my_cards - (0:(cards_left - my_cards - 1))))
  # if asked, probability from simulation
  if(!is.null(n_sim)){
    # Sample other players' cards from the previous played card value to the 
    #   maximum value remaining 
    #   (WLOG, other cards in your hand are the highest possible)
    sim <- replicate(n_sim, 
                     min(sample(setdiff((prev_min + 1):(100 - (my_cards - 1)), my_min), 
                                cards_left - my_cards, 
                                replace = FALSE)))
    sim_out <- 1 - mean(sim < my_min)
    
    return(list(out, sim_out))
  } else{
    return(out)
  }
}
```

Now let's give it a test case. I didn't bother making the arguments of `mind()` a function of the test set, but that shouldn't be hard if I want to later.

``` r
set.seed(1)
# Example round 5 with 4 players
test <- sample(1:100, 4*5, replace = FALSE)
# Your cards
test[1:5]
```

    ## [1] 27 37 57 89 20

``` r
# True play order
sort(test)
```

    ##  [1]  6 16 19 20 27 32 34 37 43 57 58 61 62 63 67 83 86 88 89 97

``` r
# Say we're 8 cards in
mind(cards_left = 12,
     my_cards   = 3,
     my_min     = 57,
     prev_min   = 37,
     n_sim      = 1e5)
```

    ## [[1]]
    ## [1] 0.02369886
    ## 
    ## [[2]]
    ## [1] 0.02308

Hey! It worked!

## What do I do with that info?
So, there's an expression for the probability you should play a card. Neat! But what can we do with it? 

Well...hard to say, but I have some ideas about what it is and isn't useful for. Holding everything else constant, it's a monotonically increasing function of your minimum value card, so knowing every player's play probability is equivalent to knowing the lowest card. There's not much fun about that. 

### Play guide

My first reaction was to take an example game and plot the play probability of a player at each card play. I'm not sure if this helps me understand the game better, because it's more of an 'oracle view' where you have to know the entire game state. But here's an example. Each bar in the plot is a card play, the height is the probability your lowest card was the overall lowest, and a red border around a bar indicates it was your play.

![](assets/img/2018-05-30-finally-a-game-about-order-statistics/ex_play_prob.png)

### Play thresholds

We could calculate a minimum 'play threshold' that gives the group a certain probability of winning the overall game. But that grinds itself to a halt pretty fast. It's cool to think about this like the multiple testing problem; if you set the threshold to require 95% probability every time and play 20 cards, you've only got a $1 - 0.95^(20-1)$ 64% chance of actually making it out of the round alive (the last card is always 100%). But in practice you'll often stall out with all players having a lower confidence than they need to act. This might make a fun extension, though. What's the average maximum play probability at any point in the game? That is, how confident is the most confident player?

### Play difficulty

What if we calculate the play probability for each player, then use how close they are to measure the difficulty of a play? Because of what I mentioned earlier about monotonicity, this will be a very similar measure to using the card values themselves. But, it's certainly a TTD[^4] . (Actual work coming soon)

[^4]: Thing To Do. One of my favorite TLAs[^5] thanks to Charlie Geyer

[^5]: Three Letter Acronyms

## Future directions
* Figure out the distribution of order statistics (maybe not much harder, since for discrete $p(X = x) = p(X \leq x) - p(X < x)$)
* Make this a Shiny app so you can put this into practice at the table
* Maybe there's a way to use this and the empirical win rates of teams to quantify the additional information players gain from the nonverbal communication?
* Figure out why the game uses ninja stars
* Write up my feelings on why its so satasfying to complete a run of close values. Because you can see the cliff so obviously?
* Why ninja stars?

### Footnotes
