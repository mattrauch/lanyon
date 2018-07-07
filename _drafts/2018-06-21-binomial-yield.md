layout: post
title: Yield, Sample Size, and the Binomial Distribution
---

![Coin]({{ "/assets/cointoss.jpg" | absolute_url }})

You've just finished off a process improvement and you are ready to see if the new process meets the yield that you need to close out the project. The question is, how many runs are needed in order to get a representative take on the actual process yield? Some may say 30, others may say 100, your boss may only want 10 to get the line back up, but what is the correct answer?

As always, we can use statistics to get a good number on what the actual yield is—with the right sample size, that is. In order to figure out this problem, we will be using the binomial distribution to calculate the the number of events requiqed, $n$, to observe $\text{Yield} \geq X$ given a certain probability. If you're familiar with the binomial distribution, you'll see how yield, a metric with binomial outcomes, can be modeled using this distribution.

Here's a quick 101 on our distribution. The binomial distribution with parameters $n$ and $p$ is the discrete probability distribution of the number of successes in a sequence of $n$ independent experiments, each with its own boolean-valued indepentent outcome[^1]. THis is a fancy way of saying that the binomial distribution models events that can only have two outcomes: A or B. Let's look at an example with a little Python.

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

This is the probability density function of a binomial distribution where $n=50$ and $p=0.5$. It shows, in our case, the likelyhood of observing $x$ number of heads given $50$ coin tosses. On the y-axis is the probability of a given value $x$, where we see $25$ is the most likely. This makes sense, since we believe the coin has a $50/50$ chance of getting heads. You'll notice as we move from the mean of this distribution, the probability recedes in a symetrical fashion.

Let's say we wanted to make sure this coin was fair, and we would consider this only if we flipped exactly $25$ heads. Using the binomial distribution, or more specifically: 

~~~python
binom.pmf(25,50,0.5)
~~~

We would calculate this probability to be $p\sim 0.11$—not very high probability, even with a sample size $n=50$!


[^1] "Binomial Distribution". Wikipedia: The free encyclopedia. (2018, June 21). FL: Wikimedia Foundation, Inc. Retrieved August 10, 2004, from https://www.wikipedia.org