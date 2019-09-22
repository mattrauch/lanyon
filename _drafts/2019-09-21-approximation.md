---
layout: post
title: Approximation
---

I recently took a Project Management Professional (PMP) course that covered cost and schedule estimation. According to Project Management Insitute's (PMI) Project Management Body of Knowledge (PMBOK), statistically based estimate methods can be either parametric estimation or three-point estimate[^fn1].

Parametric estimation, as you may have guessed, involve using the normal distribution. However, the methodology in PMBOK is not so scientific. Parametric estimation in their case is simply using linear estimation to predict a future cost. An example of this would be if Project X used $10$ widgets at a cost of $\$10$ each, and our project uses $20$ widgets, we estimate the cost to be $\$20$. Not so scientific, but certainly appropriate in certain situations.

What peaked my interest was the the alternative methodoly, known as a three-point estimate. The three point estimate can be broken into two categories, one using the triangular distribution, with the other using the beta distribution (PERT method).

### Triangular Distribution

Triangular Distribution: 
Mean: 
$E = \frac{(o + m + p )}{3}$

Average: 
$E = \frac{(o + m + p )}{3}$

### Beta Distribution
For the PERT methodology, the text suggest estimating parameters to produce a beta distribution which can be approximated by the normal distirbution. In effect, you estimate a mean and standard deviation from your optimistic, most-likely, and pessimistic estimates and use z scores to draw a tolerance interval. By it's very definition (an every example they gave in the course), the three-points would produce a assymetric distribution—otherwise, why not just estimate with a mean and standard deviation? 

What wasn't clear to me was how to bridge the gap between using parametric estimators to characterize asymetric distributions. After some research, I ran into a risk-estimating site (Riskamp.com) which dove a little deeper into what was going on here.

As it turns out, the PERT methodology uses a beta distribution function to smooth out this three point estimate into a normal-ish distibution or a lognormal distribution[^fn2]. It does this by taking the three paramaters and using them to define $\nu$ and $w$, the shape parameters used to create the beta smoothing.

$$ \mu = \frac{x_O + \lambda x_L+x_P }{\lambda + 2} $$

Where $x_O$ is the optimistic estimate, $x_L$ is the most likely, and $x_P$ is the pessimistic estimate.

This is then plugged into the beta paramters to build a smooth distibution with a peak around the most likely estimate.

$$ \nu = \frac{x_O + \lambda x_L+x_P }{\lambda + 2} $$
Beta Distribution (PERT): 

Mean: $\mu = \frac{O + 4M + P}{6}$

Standard Deviation: $\sigma = \frac{P-O}{6}$

[^fn1]: Project Management Institute. (2017). A guide to the project management body of knowledge (PMBOK guide). Newtown Square, PA: Project Management Institute.

[^fn2]riskamp.com. (2019). The beta-PERT Distribution. [online] Available at: https://www.riskamp.com/beta-pert [Accessed 22 Sep. 2019].