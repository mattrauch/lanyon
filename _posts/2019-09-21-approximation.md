---
layout: post
title: Approximation
mathjax: true
---

I recently took a Project Management Professional (PMP) course that covered cost and schedule estimation. According to Project Management Insitute's (PMI) Project Management Body of Knowledge (PMBOK), statistically based estimate methods can be either parametric estimation or three-point estimate[^fn1].

Parametric estimation, as you may have guessed, involve using the normal distribution. However, the methodology in PMBOK is not so scientific. Parametric estimation in their case is simply using linear estimation to predict a future cost. An example of this would be if Project X used $$ 10 $$ widgets at a cost of $$ 10 $$ each, and our project uses $$ 20 $$ widgets, we estimate the cost to be $$ 20 $$. Not so scientific, but certainly appropriate in certain situations.

What peaked my interest was the the alternative methodoly, known as a three-point estimate. The three point estimate can be broken into two categories, one using the triangular distribution, with the other using the beta distribution (PERT method).

For the PERT methodology, the text suggest estimating parameters to produce a beta distribution which can be approximated by the normal distirbution. In effect, you estimate a mean and standard deviation from your optimistic, most-likely, and pessimistic estimates and use z scores to draw a tolerance interval. By it's very definition (an every example they gave in the course), the three-points would produce a assymetric distribution—otherwise, why not just estimate with a mean and standard deviation? 

What wasn't clear to me was how to bridge the gap between using parametric estimators to characterize asymetric distributions. After some research, I ran into a book[^fn2], and a corresponding website[^fn3] that explained how the PERT distribution is derived.

As it turns out, the PERT methodology uses a beta distribution function to smooth out this three point estimate into a normal-ish distibution or a lognormal distribution[^fn2]. It does this by taking the three paramaters and converting them into $$ a_1 $$ and $$ a_2 $$, the shape parameters used to create the beta smoothing.

The relationship between the two distributions can be defined as[^fn3]

$$ \text{PERT}(a,b,c)=\text{Beta4}(a_1,a_2,a,c) $$

Where $$ a $$ is the optimistic estimate, $$ b $$ is the most likley estimate, and $$ c $$ is the pessimistic estimate.

$$ a_1 = \frac{(\mu-a)(2b-a-c)}{(b-\mu)(c-a)}  $$

$$ a_2 = \frac{a_1(c-\mu)}{\mu-a}  $$

$$ \mu = \frac{a + \lambda b+c }{\lambda + 2} $$

**Note**: $$ \lambda $$ is usually $$ 4 $$, but from my research seems arbitrarily chosen and serves the sole purpose of weighing the most likely estimate higher than the other two.

So lets plug in some numbers here and start modeling this in `R`. For our model, lets say we have an activity that we estimate to cost $$ a=200 $$, $$ b=600 $$, $$ c=1500 $$.

~~~R
# Define inputs
a = 200 # optimistic
b = 600 # most likely
c = 1500 # pessimistic
lambda = 4 # arbitrary
~~~

In order to convert the paramters we have into a beta-smoothed PERT distribution, we need to define our shape parameters, and generate some values. This can be achieved by calculating the shape parameters per Vose's methodology[^fn3] and then scaling each sample by the range and offsetting by the optimistic estimate (minimum) [^fn2]. We need to generate at least $$ 10000 $$ samples to get a good idea of the shape of the distribution. I also took the liberty of rounding to two decimal places to make everything a bit more readable.

~~~R
pert <- function(a, b, c, lambda){
    mu = (a + lambda * b + c) / (lambda + 2)
    a1 = ((mu - a) * (2 * b - a- c)) / ((b - mu) * (c - a))
    a2 = (a1 * (c - mu)) / (mu - a)
    return(round((rbeta(10000, a1, a2 ) * (c-a) + a ),digits=2))
}
~~~

If you run the function and get the quantiles (I chose $$ .05, .50, .95 $$) you get `5% 328.387 50% 664.69 95% 1096.763`.

![Output]({{ "/assets/pert.png" | absolute_url }})

To get the traingular distribution with our parameters, you can simple use the `rtri` function from `EnvStats`[^fn4].

~~~R
library(EnvStats)
tridf <- round(rtri(n = 10000, a, c, b),digits=2)
quantile(tridf, c(.05, .50, .95))
~~~

We get the following quantiles `5% 365.045 50% 734.795 95% 1259.287`.

![Output]({{ "/assets/tri.png" | absolute_url }})

Using the PMI methodology, our 50% would be[^fn1]

$$ \mu = \frac{a+4b+c}{6} = 683.33 $$

Not so bad, about $$ 20 $$ off. Let's see how the variance holds up.

PMI proposes using z scores to determine where 95% of the values would lie given a normal distribution, using the mean $$ \mu $$ calculated above and a standard deviation defined as[^fn1]

$$ \sigma = \frac{c-a}{6} $$

PMI rounds the z score to $2$ for a 95% interval, making the range $$ [\mu-2\sigma , \mu+2\sigma] $$ or $$ `200,1066.7`. Again, not so bad, but probably too otimistic, which is not what you want in project manangement.

To sum it up, we have the following results:

|     |   PMI   |   PERT   |    Triangular   |
|:---:|:-------:|:--------:|:--------:|
|  5% |  200.00 |  328.387 |  365.045 |
| 50% |  683.33 |   664.69 |  734.795 |
| 95% | 1066.67 | 1096.763 | 1259.287 |

So which method is the most accurate? That's hard to say, but I'm tempted to say the PMI methodoly is probably the most error prone, given that it uses parametric estimation techniques for data that is *not* typically parametric. Unfortunately, all of this is subject to the quality of the estimates, which probably contains enough margin or error that the nuance in the equations are negligable. There is some guidance on the use of these formulas. Vose points to some research from Stanton & Farnum that in order to use the PMI standard deviation methodoly, than the mode cannot lie more than 13% of the range from the distance to the optimistic or pessimistic value[^fn5]. Thus, highly skewed estimates are subject to increased error in estimation.

[^fn1]: Project Management Institute. (2017). A guide to the project management body of knowledge (PMBOK guide). Newtown Square, PA: Project Management Institute.

[^fn2]: riskamp.com. (2019). The beta-PERT Distribution. [online] Available at: https://www.riskamp.com/beta-pert [Accessed 22 Sep. 2019].

[^fn3]: Vose, D. (2008). Risk analysis: a quantitative guide. John Wiley & Sons.

[^fn4]: Millard, S. P. (2014). E nv S tats, an RP ackage for E nvironmental S tatistics. Wiley StatsRef: Statistics Reference Online.

[^fn5]: Farnum, N. R., & Stanton, L. W. (1987). Some results concerning the estimation of beta distribution parameters in PERT. Journal of the Operational Research Society, 38(3), 287-290.