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

Now, there are a couple options for solving MIP problems, and optimization in general. There's [PuLP](https://github.com/coin-or/pulp), [CPLEX](https://www.ibm.com/docs/en/icos/12.8.0.0?topic=cplex), [Gurobi](https://www.gurobi.com/documentation/9.1/quickstart_mac/cs_python.html), and many more free and costly options. Today, we'll be using Google's [OR-Tools](https://developers.google.com/optimization). Along with this, we also need pandas and numpy. Let's get started.

~~~python
!pip install ortools
from ortools.linear_solver import pywraplp
import numpy as np
import pandas as pd
~~~

## Line Balancing Problem

Let's imagine an eight step process to build a widget that we are balancing. In addition to this, our factory is available for eight hours a day, and we have a demand of 50 widgets per day. Times below are in minutes.

| Process | $$ \text{CT} $$ | Dependency |
|:-------:|:---------------:|:----------:|
| A | 2.7 | - |
| B | 4.8 | A |
| C | 4.5 | B |
| D | 3.6 | B |
| E | 4.2 | C |
| F | 2.3 | D, E |
| G | 1.7 | F, G |
| H | 4.9 | G |

~~~python
process = ['A','B','C','D','E','F','G','H']
ct = [2.7, 4.8, 4.5, 3.6, 4.2, 2.3, 1.7, 4.9]
~~~

## Takt Time & Minimum Stations

Our takt time is what we'd like to set $$ r_b $$ of the system at or below. This will ensure that widgets exit the system at a rate equal to or faster than the demand, ensuring we keep our customers happy. A $$ r_b > \text{Takt}  $$ doesn't make enough parts in the day (too slow), and a $$ r_b < \text{Takt}  $$ produces parts at a rate faster than we need them. Ideally $$ r_b = \text{Takt}  $$, but we try to shoot for a little below to be safe.

$$
\text{Takt} = \frac{\text{Available Time}}{\text{Demand}} = \frac{480}{50} = 9.6 $$

~~~python
avail = 8 * 60 # 8 hours converted to minutes
demand = 50
takt = avail / demand
~~~

Now that we know our $$ \text{Takt Time}=9.6 $$, we can solve for the minimum amount of stations in the system. This is a quick way to figure out how close we are to minimizing our stations, and in effect, minimizing our lead time.

$$
W_m = \frac{T_0}{\text{Takt Time}} = \frac{28.7}{9.6} = 3
$$

~~~python
Wm = round(sum(ct)/takt, 0)
~~~

The absolute minimum amount of stations is 5 based on a $$ \text{Takt}=9.6 $$. Now, you may have solved for $$ W_m $$ and got something like $$ 2.989583 $$. We rounded that up to $$ 3 $$ in this case because, well, good luck with your $$ 2 + 0.989583 $$ stations. Herein lies the reason why we are using MIP to solve this problem. Given our constraints of, well, reality, we need to use whole numbers, or integers when solving this equation. In a normal LP, we can in fact, optimize to $$ 2.989583 $$ stations, but that's not a practical solution we can employ in reality. In our problem, we deal with whole stations, and entire process steps, no more and no less. MIP allows us to solve with integers, which will give us nice whole numbers that we can use in real life.

## Objective Equation

We've talked about minimizing a couple things now, specifically $$ r_b $$ and $$ W $$. In order to use MIP to solve this problem, we need to translate our line balancing issue into and equation that can be minimized. We know there can be a minimum of 5 stations, and a maximum of 8 in this system (because in a worse case each process step is at it's own station). This leads us to create a set of binary variables that indicate which processes are at which station.

$$
X_{ij} =
\begin{cases}
1,  & \text{if process $i$ is assigned to station $j$}\\
0,  & \text{otherwise}
\end{cases}
$$
