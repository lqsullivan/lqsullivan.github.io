---
layout: post
title: Pulling Levers - A Multi-armed bandit approach to board game night
tags: [board games, probability, concentration inequalities]
feature-img: "assets/img/pexels/game_shelf.jpg"
thumbnail: "assets/img/thumbnails/game_shelf.jpg"
mathjax: true
excerpt_separator: <!--jump-->
---

By many measures, I have too many games. The [current count](https://www.boardgamegeek.com/collection/user/Sullysaurus?own=1&subtype=boardgame&ff=1) is about 69. That's pretty pedestrian compared to some people but, considering I have a one room apartment and those games take up two bookcases, the space under my coffee table, and the top shelf of my pantry, I'd say it's too many.

The real issue is that, of those 69 games I currently own, I haven't played about 15% of them. There are so many games worth playing, it's hard to decide whether to stick with something I like or try something that might become a new favorite. To help solve that problem, I'm going to play one of my least favorite games--the slot machine.
<!--jump-->

## Multi-armed Bandit 

Multi-armed bandit problems are a pretty famous set questions in probability theory that relate to abstracted slot machines. Imagine you're standing in front of a line of $N$ slots. When you pull the handle on machine $i$ it pays out according to some distribution $D_i$. Maybe it's a normal distribution, maybe it's a half-Cauchy, you lucky duck[^1]. You don't know anything about these machines other than what you learn by playing. Like any good gambler, you want to maximize the overall payout by deciding which machine is the best and playing that a bunch.

[^1]: Except, now that I think about it, an undefined expecation probably grinds this question to a halt. It's all about maximizing mean payouts and maximizing things you can't evaluate is a bad idea. I'm leaving this in, though, cause I like the joke.

There are all sorts of cool tweaks to the problem like changing the payout structures over time or having an adversary behind the scenes choose the payouts as you choose your play. I don't know much about these extensions yet, but I'll at least mention the first idea later and maybe pocket them for future posts.

If I want to maximize my payouts, I really care about learning the expectation of each $D_i$. (If I didn't feel the need to justify the enormous student loans behind my statistics degree, I might just call it the mean value of a pull.) Using that measure of success, there's at least one 'best' machine out there, in that it has an expectation at least as great as every other machine. It's the one I should be playing all the time. But that's the rub, I don't know what that machine is. And depending on how big $N$ is, it might take a ton of pulls to find it with any confidence. This is a classic exploration vs exploitation problem. Do I exploit the thing I think is optimal right now, or explore in hopes of finding a better optimum?

There's the connection to my premise. Think of every board game in my collection as a slot machine. When I play it, I have some amount of fun. There might be a lot of factors affecting that fun--the group, my mood, whether I won. But for now I'm going to treat it like I'm observing a realization of a random variable with "fun distribution" $D_i$. My long-term goal is to maximize the fun I have playing board games, so I should balance playing new games that may have a high fun return with games I already know I love. Exactly a multi-armed bandit problem.

So we've established the broad strokes of how this'll work. I'll use the information about past payouts to make a decision about which game to play. Then I'll play it, record that payout, and start the process over again. That's hiding a lot of difficulty behind "make a decision"--what does that decision look like?

I'm trying to maximize the payout. Over the long term, that's equivalent to picking the game with the highest expected payout because of ~~common sense~~ *The Law of Large Numbers*. If you take enough samples from the distribution $X$, the mean $\frac{1}{n}\sum_{i = 1}^n X_i$ will be close to $E[X]$.

But how close? That statement is valuable, but it just tells me that I'm doing the right thing asymptotically. At any given point, I still have a bunch of samples and possibly no idea how to use them. I'd really like to know how likely it is that the true mean is some amount higher than my sample mean, $P(\bar{X} - E[\bar{X}] \geq u)$. If I have that value in terms of the upper bound $u$, I can set it equal to some probability and solve for that upper bound.

If I know the distribution, this might not be too bad (example, if $X \sim \mathcal{N}(\mu, \sigma^2))$ we know the sampling distribution of the mean is $\bar{X_n} \sim \mathcal{N}(\mu,\sigma^2/n)$. But I need an approach that doesn't require me to know much about the underlying distribution, because I don't presume to know much about what a 'fun distribution' look like. I might be willing to make some assumptions, but definitely not anything that strong.

That's kind of asking for a miracle. So maybe I should try appealing to one.

The central limit theorem does something close to what I want. If I've got a sequence of random variables normalized in a particular way, the limit of that sequence converge to a normal distribution (we call that the asymptotic distribution, the thing it becomes in the long term). And I know a lot about the normal distribution; I could make one sided confidence intervals for the mean and get my bound that way. The only problem is, asymptotics are asymptotic. The rule of thumb you usually hear is, 30-ish samples, sure, call the mean normally distributed. Anything less than that, you're on your own. I'm not expecting to play most of these games 30 times, maybe not even 5 so, no matter how loudly I yell "look over there!" and how vigorously I point at an imaginary distraction, asymptotics won't take the bait.

### Concentration Inequalities
There's a famous set of inequalities that bound the probability a realization of a random variable deviates from some value. If there's one to know, it's Markov's inequality, which says that you can bound the probability a non-negative random variable is greater than a certain value as a fraction of the expectation.

$$ 
\begin{align}
P(X \geq t) &\leq \frac{E[X]}{t} \text{ or}\\
t           &\leq \frac{E[X]}{\alpha}
\end{align}
$$ 

Note the second line, where I picked a coverage probability $P(X \geq t) = \alpha$ and solved for the bound $t$, giving me an upper bound for a probability rather than a probability for a bound.

Once you get into the other ones, you'll notice Markov's Inequality is kind of the pilgrimage all concentration inequalities have to make. The second most famous one, Chebyshev's Inequality, is based on Markov's inequality and gives a tighter bound that takes the variance into account (notice $t^2$ in the denominator instead of $t$, that's really important to how quickly the probability 'concentrates'. $t$ is not great, $t^2$ is better, but we'd really like something exponential in the bound.).

$$
\begin{align}
P(|X - E[X]| \geq t) &\leq \frac{var(X)}{t^2} \text{ or} \\
t                    &\leq \sqrt{\frac{var(X)}{\alpha}}
\end{align}
$$

The problem is, these (and most other concentration inequalities) require us to know some basic facts about the distribution that we might not want to specify. Markov's requires the mean, Chebyshev's requires the variance, another called the Chernoff bound requires the Moment Generating Function (which is a lot to know, consider that the mean and variance are aliases of the first and second moments, you can see how that knowing the MGF is a big deal. In fact, it's equivalent to knowing the whole probability distribution). That's a non-starter in the multi-armed bandit problem--if I knew the distribution or even just the mean payout of each slot machine, I'd just go play the best one. 

I thought about appealing to the plug-in principle, a well-known tactic[^2] where you replace a quantity by an estimate for that quantity. So, if I didn't know the mean of the distribution but I had a sample from it, I'd just use the sample mean instead. Quick, dirty, probably fine in many situations.

[^2]: I thought a little about how to characterize what this is. It's sure not a theorem. A tool? A gut-feeling? If statistics were lockpicking, the central limit theorem would be a classic pickset, bootstrapping would be a skeleton key, and this would be a shim made out of a beer can.

My first thought was, it runs into the same problem as before. I want to use this in places where I have very small samples, exactly where the plug-in principle is most likely to lead me astray. But there's an even bigger issue--the Markov bound just plain sucks[^3]. It's huuuuuge. So big that any volatility I might add by using the plug-in principle on a small sample is dwarfed by the fact that the bound from Markov's inequality decreases very slowly with decreasing $\alpha$. However, Chebyshev's inequality has that quadratic term, so maybe the bound there is small enough that I can start having meaningful worries about plugging in the sample variance? 

[^3]: Natural question, if it sucks so much, what's the point? That's like saying, bedrock is boring and I don't like it. Well, sure, but that's where all the fancy stuff sits. I didn't have to say anything besides the random variable is positive and I got a nice theorem about the probability a realization is $a$ big in terms of the expectation. But in practice, things build off that inequality, they don't use it.

`<aside>`
How bad is plugging in estimates?

Here's a simulation exploring that. First, I generated some samples of varying sizes from a beta distribution, then constructed the upper bound implied by those two inequalities (by specifying the probability I want and solving for the threshold, plugging in the sample mean or variance where it asks for the population value. Markov's is $t \leq \frac{\bar{X}}{\alpha}$ and Chebyshev's is $\sqrt{\frac{-ln(\alpha)}{2n}}$). The plot shows the empirical coverage probability (how often in my 10,000 samples the upper bound was above the true mean for each sized sample) and the nominal coverage $1 - \alpha = 0.95$ in this example.

``` r
# Sim if the plug-in principle does poorly here with small sample sizes
markov <- function(x, a){
  return(mean(x) + mean(x) / a)
}
chebyshev <- function(x, a){
  return(mean(x) + sqrt(var(x) / a))
}
clt <- function(x, a){
  return(mean(x) + qnorm(1 - a, 0, 1) * sd(x))
}

# Test: beta(2, 5) ----
a         <- 2
b         <- 5
true_mean <- a / (a + b)
alpha     <- 0.05
n_sim     <- 1e4

# Setup sequence of sample sizes, lots toward the low end
n_seq <- unique(floor(exp(seq(0, 5, by = 0.2)))) # from about 1-150
sub_list <- list(coverage = matrix(NA, nrow = length(n_seq), ncol = n_sim),
                 bound    = matrix(NA, nrow = length(n_seq), ncol = n_sim))
out <- list(markov    = sub_list,
            chebyshev = sub_list,
            normal    = sub_list)
# Simulate data and record CIs resulting from markov and chebyshev
for(i in 1:length(n_seq)){
  for(j in 1:n_sim){
    samp <- rbeta(n_seq[i], a, b)
    
    out$markov$bound[i, j] <- markov(samp, alpha)
    out$markov$coverage[i, j] <- out$markov$bound[i, j] >= true_mean
    
    out$chebyshev$bound[i, j] <- chebyshev(samp, alpha)
    out$chebyshev$coverage[i, j] <- out$chebyshev$bound[i, j] >= true_mean
    
    out$normal$bound[i, j] <- clt(samp, alpha)
    out$normal$coverage[i, j] <- out$normal$bound[i, j] >= true_mean
  }
}
```

![Example of play probabilities]({{ site.url }}/assets/img/2018-08-26-pulling-levers/plug_in.png)

I plotted the sample size on the log scale cause things change really quickly over the first few samples and I wanted to make those first 5 values more visible. Terrible for interpretation though, so pay attention to the axis values.

Looks like the problem isn't as big as I thought for this one distribution [^4] The simulated coverage is at least as good as the nominal coverage for the Markov inequality at any sample size (again, not surprising), and it only performs poorly for Chebyshev's inequality when the sample size is 2, the minimum you need to compute a variance. This doesn't seem like a bad thing to do but...

[^4]: It's almost not worth even showing this for just one distribution because the result is so dependent on which one I picked. And, if you think about when those estimates will be poor, it's when we have a small sample and a distribution that's relatively likely to take values far from the mean...which is exactly what we're trying to understand with concentration inequalities. For more details on this circular argument, see footnote 4.

`<\aside>`

...look at average value of that bound for each method

Method    | Mean bound (for sample size 5)
--------- | -------------
Markov    | 6.02
Chebyshev | 0.86
CLT       | 0.50
    
Which is why the previous images of the coverage probability are kind of deceptive. In practice, I care about both the mean being inside my confidence interval and the width of that interval being small. But I probably care about the second one more. At least enough to trade some assumptions to make it smaller.

There's another concentration inequality that gives exponentially decreasing bounds where Chebyshev's are only quadratic. I mentioned Chernoff bounds earlier, but noted that they require even more information about the distribution--the moment generating function. There's another inequality belonging to Wassily Hoeffding that leans on a result called Hoeffding's lemma, which bounds the MGF of a random variable with a bounded distribution (for simplicity say in $[0, 1]$) in a way that only depends on the size of the sample! This is exactly what I want! Once I understand how this works, I can make a pretty mild assumption (the fun distribution only takes values in a given interval) and have the power of coverage probability that decreases exponentially with the upper bound! That exponential also lets us easily use a sum of random variables (and the mean is a scaled sum). This seems like a great fit.

The first half dozen times I read about Hoeffding's Inequality, it didn't click for me why I should care so much. But the lemma is the key, trading away a high-information requirement (MGF) for an assumption that feels kind of mild (bounded random variable). I want to step through this one in more detail

## Hoeffding's Lemma

Let $X$ be a random variable with $E[X] = 0$ and $P(a \leq X \leq b) = 1$. For all $\lambda \in \mathbb{R}$, $E[e^{\lambda X}] \leq e^{\frac{\lambda^2 (b - a)^2}{8}}$.

**Proof**

Recall that if $f$ is convex, for any $x,y$ in the domain of $f$ and $\alpha \in [0, 1]$ $f(\alpha x + (1 - \alpha) y) \leq \alpha f(x) + (1 - \alpha) f(y)$.

Since $e^{\lambda X}$ is convex, rewrite it in that form with $\alpha = \frac{x-a}{b-a}$ (so $1 - \alpha = \frac{b-x}{b-a})$). Since $a \leq X \leq b$ with probability 1, $\alpha \in [0, 1]$. Then take the expectation of both sides, remembering that we defined $E[X] = 0$

$$
\begin{aligned}
e^{\lambda X}    &\leq \frac{X-a}{b-a} e^{\lambda b} + \frac{b-X}{b-a} e^{\lambda a} \\
E[e^{\lambda X}] &\leq \frac{E[X]-a}{b-a} e^{\lambda b} + \frac{b-E[X]}{b-a} e^{\lambda a} \\
                 &\leq \frac{-a}{b-a} e^{\lambda b} + \frac{b}{b-a} e^{\lambda a} \\
\end{aligned}
$$

Now, become divinely inspired to let $h = \lambda (b-a)$ and $p = \frac{-a}{b-a}$. Notice $hp = -\lambda a$ and $h(1-p) = \lambda b$. Rewrite the left hand side as $e^{L(h)}$

$$
\begin{aligned}
\phantom{e^{\lambda X}} &\leq p e^{h(1-p)} + (1-p) e^{-hp} \\
                        &\leq p e^h e^{-hp} + e^{-hp} - p e^{-hp} \\
                        &\leq e^{-hp} (pe^h - p + 1) \\
                        &\leq e^{-hp + \log{(1-p+pe^h)}} \\
                        &\leq e^{L(h)}
\end{aligned}
$$

where $L(h) = -hp + \log{(1-p+pe^h)}$. Now we're going to use Taylor's theorem with a second-order polynomial approximation to bound this. For that we need to evaluate the function and a couple derivatives.

$$
\begin{aligned}
L(0)   &= 0 + log(1) = 0 \\
L'(h)  &= -p + \frac{1}{1-p+pe^h} pe^h \\
L'(0)  &= -p + p = 0 \\
L''(h) &= \frac{(1-p+pe^h)pe^h - {(pe^h)}^2}{(1-p+pe^h)^2} \\
       &= \frac{pe^h}{1-p+pe^h} \frac{(1-p+pe^h)-pe^h}{(1-p+pe^h)} \\
       &= \frac{pe^h}{1-p+pe^h} (1 - \frac{pe^h}{1-p+pe^h}) \\
\end{aligned}
$$

The last line is quadratic in $\frac{pe^h}{1-p+pe^h}$ and that quadratic has maximum value 1/4 when t = 1/2, which happens when $h = \log(\frac{1-p}{p}) = \log(\frac{-b}{a})$ (remember $p = \frac{-a}{b-a}$). And it can actually obtain that because $E[X] = 0$ and $P(a \leq X \leq b) = 1$ so $a \leq 0$ (except if X has a degenerate distribution with all the mass on 0. In that case, the bound we're getting to still holds, except then we don't care cause we already know the MGF)

Sooo...back to Taylor's theorem, remembering that the remainder term can be written in terms of $L''(\eta)$ for some $a \leq \eta \leq x$

$$
\begin{aligned}
L(h) &= L(0) + L'(0) \cdot h + \frac{L''(\eta)}{2!}\cdot h^2\\
     &\leq 0 + 0\cdot h + \frac{1}{2} \cdot \frac{1}{4}h^2 = \frac{h^2}{8} \\
\end{aligned}
$$

And we're done (with the lemma). $E[e^{\lambda X}] \leq \frac{\lambda^2}{8}$.

To get a useful bound out of it, we can go back to Markov's inequality, but this time using $S_n = \sum X_i = n\cdot\bar{X}$ (along with the $X_i$ independence and the lemma we just proved to get rid of that expectation)

$$
\begin{aligned}
P(\bar{X} - E[\bar{X}] \geq t) &=    P(S_n - E[S_n] \geq nt) \\
                               &=    P(e^{s(S_n - E[S_n])} \geq e^{stn}) \\
                               &\leq e^{-stn} E[e^{s(S_n - E[S_n])}] \\
                               &\leq e^{-stn} \prod^n_{i=1}E[e^{s(X_i - E[X_i])}] \\
                               &\leq e^{-stn} e^{\frac{s^2n}{8}} \\
                               &\leq e^{-stn + \frac{s^2n}{8}} \\
\end{aligned}
$$

Now notice that the thing in the exponent is quadratic in $s$, so we can find the minimum of it just like in high school $\frac{-b}{2a}$ (and that minimum will be the smallest thing that's still an upper bound). So the $s$ that gives the smallest result is $\frac{+tn}{2n/8} = 4t$ and that gives a max of $e^{-2nt^2}$.

That's Hoeffding's inequality: $P(\bar{X} - E[\bar{X}] \geq t) \leq e^{-2nt^2}$. Or, solving for $t$ like we did for the others, $t \leq \sqrt{\frac{-\log{(\alpha)}}{2n}}$ Not bad.

## Let's play some games
Here's an example to illustrate what the game selection process looks like. Imagine I have 5 games with different fun distributions. In reality, I just need to be able to sample from them (i.e. play the game), but for this simulation I need to specify what those distributions are. Remember, for Hoeffding's inequality to hold, the distributions have to be bounded.

So, I picked a few name-brand distributions with different properties, one has a heavier upper tail, one has a heavier lower tail, one is bimodal, one takes only values 0 or 1, and one is uniform. This is nowhere near an exhaustive display of this approach, but it's a good illustration of the practical decisions you need to make to implement it.

Like, how do you pick the confidence level? The bound needs $\alpha$ to function. One immediate idea is to pick a low number, maybe 1%. How about instead, we pick $\alpha$ proportional to the total number of times we've played so that as we gain information, we require more confidence in the result? $\alpha = \frac{1}{\text{iter}^4}$ is a commonly used 'decay' for the confidence that goes by the name UCB1 (for Upper Confidence Bound). That leads to an upper bound of $\sqrt{\frac{2\log{t}}{2n}}$, which is what I used for the simulation below.

The process is pretty easy, calculate the upper bound of the mean fun for each game, pick the highest one to play (if there are any that have no plays, randomly play one of those instead), then repeat. Each iteration the confidence of the confidence bound increases for all games and the mean changes for one of them, so the estimates can rise and fall. Since games that don't get sampled have upper bounds that rise relative to the last iteration, we'll eventually play every game an infinite amount of times. So there's no worry about getting stuck in a loop where one game has a few bad plays and we never touch it again.

The plot shows 3 different things. The big one is the real-time plot of payouts from each pull (shown as black dots), the current Hoeffding upper bound (red/green bars, green is the current maximum which will be played on the next iteration), and the mean fun for each (blue bars). Each fun distribution has a label below it and right under that is a barplot of the total proportion of times we've played each game. The bar on the far right shows the mean fun over all the plays in relation to blue lines matching the mean fun for each game. The games are sorted with the lowest on the left to make it easier to parse. It runs for 500 plays at a pretty good clip, so stick around for a couple repeats.

![]({{ site.url }}/assets/img/2018-08-26-pulling-levers/hoeffding_ex.gif)

Notice that the binomial-ly fun game gets played more than you might expect based on its mean fun. The bars of proportion of total plays for each game seems to go in order from left to right, except that. In fact, there's *another* inequality called the Bhatia-Davis inequality that bounds the variance of a bounded distribution and it shows that the highest variance happens when the distribution is concentrated on the boundaries of its support (exactly what binomial does). So, in that sense, it's reasonable that it'd need more disproportionate samples to pin down the mean (and it's only the mean estimate goofing this up, cause remember the upper bound has nothing to do with the values of the realizations, only the count).

So that's it. Feel free to try it out yourself at home. Although, if your collection is anything like mine, gathering that initial data from a play of every game you own might be the barrier.

## Future directions

In the future, I might try to record some subjective 'fun ratings' of games and try to approximate their 'fun distributions'. That might be hard, cause I tend to not play games enough to estimate that empirical distribution function well.

My highest priority for future work is checking out more of the differences between these inequalities for example distributions. I'm particularly interested in what the gain is if I actually have the MGF to use in that exponential twist on Markov's inequality instead of defering to Hoeffding's Lemma to bound it. That'll depend heavily on the distribution, but I'd like to see it in practice for a couple examples and maybe try to make some general statements too.

There are some tweaks to the multi-armed bandit problem that apply to this situation like having the payout structure change over time (maybe it increases as you get deeper into the strategy, maybe you have a large 'novelty effect', maybe you're playing a lot of Diplomacy and your group starts to hate each other IRL). 

The next related piece I'll tackle is what happens when the payouts are binary. That's a very low-information result, but the distributional assumption lets us escape all these fun general inequalities and go straight to a classic confidence interval. I'll explore this more in an upcoming post on Monte Carlo Tree Searches.

### Code

I'll post the relevant code I used to generate this data on my GitHub soon and link it here. Standard disclaimer applies, I don't pretty up most of my code for headhunters. I'm not writing immaculately structured/commented maintainable corporate stuff here, I'm getting analysis out the door and moving on to a new topic :)

## References
https://lilianweng.github.io/lil-log/2018/01/23/the-multi-armed-bandit-problem-and-its-solutions.html#thompson-sampling

https://www.cs.cornell.edu/~sridharan/concentration.pdf

https://en.wikipedia.org/wiki/Hoeffding%27s_inequality

https://crypto.stanford.edu/~blynn/pr/markov.html


### Footnotes
