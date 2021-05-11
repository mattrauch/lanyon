---
layout: post
title: Line Balancing
mathjax: true
---

Line Balancing is one of the simplest, yet most impactful tools that an Industrial Engineer has in their tool kit. In a manufacturing environment, line balancing it the science of distributing the work across a set of distinct workstations to minimize the $$ r_b $$, or bottleneck rate of the line. This in turn, allows maximization of system output, and minimization of WIP in a system.

## The Chipotle Example

The effect of this manifests itself in many processes we interact with on a daily basis. Take for example, the last time you ordered at Chipotle. When you arrive at Chipotle, you wait in a short line, typically do the first half, or the "filling" station, of your order with one person, then you are passed onto another person for the "toppings" station, and then another person to pay. Let's put some hypothetical numbers to this process to see a real life application  of line balancing.

| Station  |   $$ \text{CT} $$  |   Operators   |    $$ \text{TH} $$    |
|:--------:|:------------------:|:-------------:|:---------------------:|
|  Filling |  1.5 min |  1 | 0.67 |
| Toppings |  1.0 min |   1 |  1.00 |
| Payment  |  2.0 min | 1 | 0.50 |

We understand that the maximum rate of this system is equal to $$ r_b $$, which in this case, is our Payment station, with a $$ \text{CT}=2.0 $$ and $$ \text{TH}=0.5 $$. Now, in a system like this, simply adding a second payment station will equalize the throughput for payments with the toppings station, so that filling becomes the new bottleneck. The innovation then follows by slicing and dicing the processes into smaller and smaller intervals, allowing incremental balancing as you continue to minimize $$ r_b $$. 

## Optimization

This however, quickly gets complicated as process dependencies emerge as well as complexity around reducing cycle times. It's not always as simple as adding more labor or stations to the system. After all, in order to minimize WIP (and lead time), we want to minimize the number of stations $$ W $$, as well as minimize $$ r_b $$. And with the simple objective, an optimization problem is born.

Line balancing can be done by hand, and frankly is probably quicker in most cases. However, this blog isn't about applying pragmatic solutions to complex problems—we like to indulge ourselves in automation and high-tech tools for even the simplest use cases. For today's exercise in self indulgence, we are exploring the application of Mixed Integer Programming (MIP) to automate our line balancing problems.

Now, there are a couple options for solving MIP problems, and optimization in general. There's [PuLP](https://github.com/coin-or/pulp), [CPLEX](https://www.ibm.com/docs/en/icos/12.8.0.0?topic=cplex), [Gurobi](https://www.gurobi.com/documentation/9.1/quickstart_mac/cs_python.html), and many more free and costly options. Today, we'll be using Google's [OR-Tools](https://developers.google.com/optimization). Let's get started.

## Line Balancing Problem

Let's imagine an eight step process to build a widget that we are balancing. In addition to this, our factory is available for eight hours a day, and we have a demand of 50 widgets per day. Times below are in minutes.

| Process | $$ \text{CT} $$ | Dependency |
|:-------:|:---------------:|:----------:|
| A | 6.7 | - |
| B | 4.8 | A |
| C | 4.5 | B |
| D | 7.6 | B |
| E | 4.2 | C |
| F | 7.3 | D, E |
| G | 1.7 | F, G |
| H | 8.9 | G |

## Takt Time & Minimum Stations

Our takt time is what we'd like to set $$ r_b $$ of the system at or below. This will ensure that widgets exit the system at a rate equal to or faster than the demand, ensuring we keep our customers happy. A $$ r_b > \text{Takt}  $$ doesn't make enough parts in the day (too slow), and a $$ r_b < \text{Takt}  $$ produces parts at a rate faster than we need them. Ideally $$ r_b = \text{Takt}  $$, but we try to shoot for a little below to be safe.

$$ \text{Takt} = \frac{\text{Available Time}}{\text{Demand}} = \frac{480}{50} = 9.6 $$

Now that we know our $$ \text{Takt Time}=9.6 $$, we can solve for the minimum amount of stations in the system. This is a quick way to figure out how close we are to minimizing our stations, and in effect, minimizing our lead time.

$$ W_m = \frac{T_0}{\text{Takt Time}} = \frac{45.7}{9.6} = 5 $$

The absolute minimum amount of stations is 5 based on a $$ \text{Takt}=9.6 $$. Now, you may have solved for $$ W_m $$ and got something like $$ 4.760417 $$. We rounded that up to $$ 5 $$ in this case because, well, good luck with your $$ 4 + 0.760417 $$ stations. Herein lies the reason why we are using MIP to solve this problem. Given our constraints of, well, reality, we need to use whole numbers, or integers when solving this equation. In a normal LP, we can in fact, optimize to $$ 4.760417 $$ stations, but that's not a practical solution we can employ in reality. In our problem, we deal with whole stations, and entire process steps, no more and no less. MIP allows us to solve with integers, which will give us nice whole numbers that we can use in real life.

## Objective Equation

We've talked about minimizing a couple things now, specifically $$ r_b $$ and $$ W $$. In order to use MIP to solve this problem, we need to translate our line balancing issue into and equation that can be minimized. We know there can be a minimum of 5 stations, and a maximum of 8 in this system (because in a worse case each process step is at it's own station). This leads us to create a set of binary variables that indicate which processes are at which station.

$$ S_{ij} &= \begin{cases} 1,  & \text{if plant $i$ is running on day $j$ and $R_{ij-1}=0$}\\ 0,  & \text{otherwise} \end{cases} $$



$$     \begin{mini}|l|
	  {w,u}{f(w)+ R(w+6x)}{}{}
	  \addConstraint{g(w_k)+h(w_k)}{=0,}{k=0,\ldots,N-1}
	  \addConstraint{l(w_k)}{=5u,\quad}{k=0,\ldots,N-1}
     \end{mini}

$$

~~~R
set.seed(1234)
a = 200 # optimistic
b = 600 # most likely
c = 1500 # pessimistic
lambda = 4 # arbitrary
~~~

To convert the parameters we have into a beta-smoothed PERT distribution, we need to define our shape parameters and generate some values. This can be achieved by calculating the shape parameters per Vose's methodology[^fn3] and then scaling each sample by the range and offsetting by the optimistic estimate (see footnote link for their `R` code)[^fn2]. We need to generate at least $$ 10000 $$ samples to get a good idea of the shape of the distribution. I also took the liberty of rounding to two decimal places to make everything a bit more readable.

~~~R
pert <- function(a, b, c, lambda) {
  mu = (a + lambda * b + c) / (lambda + 2)
  a1 = ((mu - a) * (2 * b - a - c)) / ((b - mu) * (c - a))
  a2 = (a1 * (c - mu)) / (mu - a)
  data = rbeta(10000, a1, a2)
  return(round(data * (c - a) + a, digits = 2))
}
~~~

If you run the function and get the quantiles (I chose $$ .05, .50, .95 $$) you get `5% 323.94 50% 663.6 95% 1100.46`.

![Output]({{ "/assets/pert.png" | absolute_url }})

To get the triangular distribution with our parameters, you can simply use the `rtri` function from `EnvStats`[^fn4].

~~~R
library(EnvStats)
tridf <- round(rtri(n = 10000, a, c, b), digits = 2)
quantile(tridf, c(.05, .50, .95)) 
~~~

We get the following quantiles `5% 361.325 50% 735.87 95% 1259.053`.

![Output]({{ "/assets/tri.png" | absolute_url }})

Using the PMI methodology, we are given the following formulas[^fn1]:

$$ \text{mean} = \frac{a+4b+c}{6} $$

$$ \text{sd} = \frac{c-a}{6} $$

PMI proposes using z scores to determine where 95% of the values would lie given a normal distribution, using the mean calculated and standard deviation defined above. To keep things simple, we will use the same quantile methodology as before.

~~~R
mean = (a + (4 * b) + c) / 6
sd = ((c - a) / 6)
pmi <- round(rnorm(10000, mean, sd), digits = 2)
quantile(pmi, c(.05, .50, .95))  
~~~

This gets us quantiles of `5% 331.7395 50% 684.34 95% 1035.309`, as well as a nice parametric distribution.

![Output]({{ "/assets/pmi.png" | absolute_url }})


To sum it up, we have the following results:

|     |   PMI   |   PERT   |    Triangular   |
|:---:|:-------:|:--------:|:--------:|
|  5% |  331.7395 |  323.94 |  361.325 |
| 50% |  684.34 |   663.6 |  735.87 |
| 95% | 1035.309 | 1100.46 | 1259.053 |

So which method is the most accurate? That's hard to say. I was surprised the PMI methodology got similar results to the PERT beta smoothed methodology, but I guess that is [central limit theorem](https://en.wikipedia.org/wiki/Central_limit_theorem) in action. All methods ranked probability of the extremes very low, but all three methodologies showed their bias towards the most likely scenario. Given that expert input would place some probability at both the optimistic and pessimistic, it would be interesting to compare the modeled probability with that of the expert. I would imagine its probably not $$ .001 $$ ([see this chart](https://flowingdata.com/2018/07/06/how-people-interpret-probability-through-words/ "How people interpret probability through words") on how people interpret probabilistic words). Given the results, I would probably recommend padding the expert estimates to capture them within the model to the probabilistic interval they would attribute them to.

Having said that, all of this is subject to the quality of the estimates, which probably contains enough margin or error that the nuance in the equations is negligible. There is some guidance on the use of these formulas. Vose points to some research from Stanton & Farnum that to use the PMI standard deviation methodology, then the mode cannot lie more than 13% of the range from the distance to the optimistic or pessimistic value[^fn5]. Thus, highly skewed estimates are subject to increased error in estimation.

In conclusion, I was surprised by the results here—the math seems to work out in the end, contingent on the accuracy of the estimates. The only area of improvement is possibly developing a function to pad the upper and lower estimates to get them at a probability in the model that is more aligned with the expert's intent. Until then, happy project management activity forecasting!

