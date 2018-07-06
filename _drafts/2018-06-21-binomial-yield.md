---
layout: post
title: Yield, Sample Size, and the Binomial Distribution
---

![Coin]({{ "/assets/cointoss.jpg" | absolute_url }})


Sample size tends to be a hot topic—we never seem to have enough samples to provide confidence our results, yet the appropriate amount of samples is often seen as not practical. While there are many ways to calculate sample size, today we will look at a overlooked yet important application of sample size calculation when looking at binary event predicition. Where this is applied is often at the end of an improvement project, when the improved state performance is being assessed. WHen not using methods like capability studies ($Cpk$ and $Ppk$), we may be assessing the overall yield of a process. This is where the binary event prediction bit comes into play. When calculating yield, we look at whether a product passes or fails, and normalize this to a ratio of 

$$\text{Yield}=\frac{\text{Passing Units}}{\text{Total Units Tested}}$$

Seems simple enough, yet we often don't think to calculate sample size in this scenario. Statistically, if we don't calculate the correct sample size, there could be a chance that we could miss the true population yield and falsely conclude that the population doesn't meet our expectation. This is a little complicated in theory, but the following example should clear up any confusion.

Before jumping into the example, yield can be modeled using the binomial distribution. The binomial distribution with parameters $n$ and $p$ is the discrete probability distribution of the number of successes in a sequence of $n$ independent experiments, each asking a yes–no question, and each with its own boolean-valued outcome[^1]. The classic example of the binomial distribution is flipping a coin 50 times, with the chance of getting heads modeled. In this situation, $n$ is 50, since it is the number of events, and $p=0.5$ since there is a $50%$ chance of getting heads. We can generate the distribution using the following code in python

~~~Python
%matplotlib inline
import numpy as np
from scipy.stats import binom
import matplotlib.pyplot as plt

n = 50
p = 0.5
mean, var, skew, kurt = binom.stats(n, p, moments='mvsk')

fig, ax = plt.subplots(1, 1)
x = np.arange(binom.ppf(0.0001, n, p),
              binom.ppf(0.9999, n, p))
ax.vlines(x, 0, binom.pmf(x, n, p), colors='b', lw=4, alpha=0.5)
~~~

![Binomial Distribution]({{ "/assets/binomialp0.5.png" | absolute_url }})

This is the probability density function of a binomial distribution where $n=50$ and $p=0.5$. It shows, in our case, the likelyhood of observing $x$ number of heads given $50$ coin tosses. On the y-axis is the probability of a given value $x$, where we see 25 is the most likely. This makes sense, since we believe the coin has a 50/50 chance of getting heads. You'll notice as we move from the mean of this distribution, the probability recedes in a symetrical fashion.

Let's say we wanted to make sure this coin was fair, and we would consider this only if we flipped exactly 25 heads. Using the binomial distribution, or more specifically: 

~~~python
binom.pmf(25,50,0.5)
~~~

We would calculate this probability to be $p\sim 0.11$—not very high probability, even with a sample size ($n$) of 50!


[^1] "Binomial Distribution". Wikipedia: The free encyclopedia. (2018, June 21). FL: Wikimedia Foundation, Inc. Retrieved August 10, 2004, from https://www.wikipedia.org