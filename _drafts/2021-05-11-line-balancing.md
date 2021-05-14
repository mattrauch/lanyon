---
layout: post
title: Line Balancing
mathjax: true
---

Line Balancing is one of the simplest, yet most impactful tools that an Industrial Engineer has in their tool kit. In a manufacturing environment, line balancing it the science of distributing the work across a set of workstations to minimize the amount of workstations required to meet the takt time. This minimizes WIP, and also increases utilization at each station in the line. Because line balancing, in it's simplist form, takes the process as-is, and simply rearranges the tasks, it can be provide an incredible return on investment for unbalanced systems.

Line balancing is all about maximizing output to the takt time, but why is this important? When we think about a production line, it's a little like an accordion. We know from Little's Law that at steady state, a system produces output at the bottleneck rate of the process, $$ r_b $$. The easy thing to is to make each distinct process step it's own workstation, giving us the most workstations, but the lowest $$ r_b $$. In The process below, the system will produce a part every 4.9 min at the last station. However, you'll notice that there is quite some space between this and the takt time of 9.6. This is where utilization is important. To minimize costs, we want to create product at the speed it is consumed, or the taxt time, no more, no less. If the takt time is below $$ r_b $$, we can look at combining steps to reduce the amount of workstations, and increase utilization. As takt decreases towards $$ r_b $$, we can continue to add process steps to lower the $$ r_b $$ of the system and still meet the takt.

![process bar](assets/chart1.png)

While performing line balancing seems fairly simple at face value, this quickly gets complicated as process dependencies emerge as well as complexity around reducing cycle times. It's not always as simple as adding more labor or stations to the system. After all, in order to minimize WIP, we want to minimize the number of stations $$ W $$, and to reduce costs, we want to maximize line utilization. With this simple objective, an optimization problem is born.

## But First, A Disclaimer...

Line balancing can be done by hand, and frankly is probably quicker in most cases. However, this blog isn't about applying pragmatic solutions to complex problems—we like to indulge ourselves in automation and high-tech tools for even the simplest use cases. For today's exercise in self indulgence, we are exploring the application of Mixed Integer Programming (MIP) to automate our line balancing problems.

## MIP & The Line Balancing Problem

Now, there are a couple options for solving MIP problems, and optimization in general. There's [PuLP](https://github.com/coin-or/pulp), [CPLEX](https://www.ibm.com/docs/en/icos/12.8.0.0?topic=cplex), [Gurobi](https://www.gurobi.com/documentation/9.1/quickstart_mac/cs_python.html), and many more free and costly options. Today, we'll be using Google's [OR-Tools](https://developers.google.com/optimization). 

Now you may be asking, why MIP and not a regular LP? The Line Balancing Problem is a type of assignment problem, where binary variables are assigned. In our case, $$ X_{ij} $$ is 1 if station $$ i $$ is assigned process $$ j $$. 

$$
X_{ij} =
\begin{cases}
1,  & \text{if process $i$ is assigned to station $j$}\\
0,  & \text{otherwise}
\end{cases}
$$

In our use case, the process is either happening at that station, or not, and it can't be anything in between. As we add constraints to the system, in a very simple case, this can produce a optimized result with a linear solver that isn't actually feasible in reality. For example, it may suggest we have 3.26 stations, when we can only have 3 or 4, but not in between. With this out of the way, let's jump into the problem.

## Problem Overview

Let's imagine an eight step process to build a widget that we are balancing. In addition to this, our factory is available for eight hours a day, and we have a demand of 50 widgets per day. Times below are in minutes.

![process flow](assets/processflow.png)

Before we begin entering the data into our program, let's pull in the necessary packages and initialize our solver. We are going to use [SCIP](https://www.scipopt.org/), but their are other options.

~~~python
from ortools.linear_solver import pywraplp
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

solver = pywraplp.Solver.CreateSolver('SCIP')
~~~

Based on the process flow, we can enter our data in an calculate our current line utilization.

~~~python
no = 8 # number of process steps
steps = list(range(0, no))
process = ['Task %s' % s for s in steps]

ct = [2.7, 4.8, 4.5, 3.6, 4.2, 2.3, 1.7, 4.9]

avail = 8 * 60 # 8 hours converted to minutes
demand = 50
takt = avail / demand
util = sum(ct)/(takt*len(process))
rb = max(ct)

print('Utilization:',util)
print('Takt:', takt)
print('r_b:',rb)
~~~

~~~
Utilization: 0.37369791666666674
Takt: 9.6
r_b: 4.9
~~~

Alright, not so great. With the system $$ r_b = 4.9 $$ and the $$ \text{Takt}=9.6 $$, we can see why the utilization is so low. To visualize this, we can put the data into a bar plot with the takt time.

~~~python
fig = plt.figure()
ax = fig.add_axes([0,0,1,1])
ax.set_ylabel('Cycle Time (Min)')
ax.bar(process,ct)
ax.axhline(y=takt)
plt.show()
~~~

![process bar](assets/chart1.png)

Our takt time is what we'd like to set $$ r_b $$ of the system at or below. This will ensure that widgets exit the system at a rate equal to or faster than the demand, ensuring we keep our customers happy. A $$ r_b > \text{Takt}  $$ doesn't make enough parts in the day (too slow), and a $$ r_b < \text{Takt}  $$ produces parts at a rate faster than we need them. Ideally $$ r_b = \text{Takt}  $$, but we try to shoot for a little below to be safe.

$$
\text{Takt} = \frac{\text{Available Time}}{\text{Demand}} = \frac{480}{50} = 9.6 $$

## Defining Constraints

We've talked about minimizing a couple things now, specifically utilization through $$ W $$. In order to use MIP to solve this problem, we need to translate our line balancing issue into and equation that can be minimized. To start, we need to initialize a decision variable for each task so we can assign a sequence.

~~~python
infinity = solver.infinity()
y = {}

# create a integer variable y[i] for each process step
for i in range(len(process)):
    y[i] = solver.IntVar(0, infinity, 'y%i' % i)
~~~

Next up, we want to put in the sequence of tasks as a constraint. Since we are using integers, we can assign each unique process step a sequential value to ensure that we don't get things out of order. By using the sum of these values, we can, in turn, create a simple way to ensure tasks don't get assigned to stations in the wrong order. For example, we can make sure Task 1 is before Task 2 with the following:

$$ Y_0 - Y_1 \leq 0 $$

$$ Y_0 = 1x_{00} + 2x_{01} + 3x_{02} + 4x_{03} + 5x_{04} + 6x_{05} + 7x_{06} + 8x_{07} $$
$$ Y_1 = 1x_{10} + 2x_{11} + 3x_{12} + 4x_{13} + 5x_{14} + 6x_{15} + 7x_{16} + 8x_{17} $$

If $$ Y_0 $$ is assigned $$ X_{0j} $$ tasks where $$ X_{0j} > X_{1j} $$, it will result in $$ Y_0 > Y_1 $$, breaking the constraint. I admit I couldn't find a clever way to automate these constraints, so I just hard-coded them. Let me know if you have a quicker way to do it.

~~~python
solver.Add(y[0] - y[1] <= 0)
solver.Add(y[0] - y[2] <= 0) 
solver.Add(y[1] - y[3] <= 0) 
solver.Add(y[1] - y[4] <= 0)
solver.Add(y[2] - y[4] <= 0)
solver.Add(y[3] - y[5] <= 0) 
solver.Add(y[4] - y[6] <= 0) 
solver.Add(y[5] - y[6] <= 0) 
solver.Add(y[6] - y[7] <= 0) 
~~~

Next up, we need to create a binary variable for each $$ X_{ij} $$ where $$X_{ij}= 1 $$ if station $$ i $$ is assigned process $$ j $$. We have eight process steps that can be performed at eight possible stations, resulting in 64 variables.

$$
X_{ij} =
\begin{cases}
1,  & \text{if process $i$ is assigned to station $j$}\\
0,  & \text{otherwise}
\end{cases}
$$

~~~python
a = 0
x= {}

for i in range(len(process)):
    for j in range(len(process)):
        x[a] = solver.IntVar(0, 1, 'x%i%i' % (i, j))
        a += 1
~~~

For our next constraint, we want to make sure each station doesn't have more than the takt time worth of work at the station. The equation is the sum of each task $$ j $$ iterated across station $$ i $$. This was another one that I wasn't clever enough to loop through, so it's about half hard-coded.

$$
\sum_{i} y_{ij}= 1,\space \forall j 
$$

~~~python
for i in range(len(process)):
        solver.Add(x[((i*7)+0+(i*1))] + 
                   x[((i*7)+1+(i*1))] + 
                   x[((i*7)+2+(i*1))] + 
                   x[((i*7)+3+(i*1))] + 
                   x[((i*7)+4+(i*1))] + 
                   x[((i*7)+5+(i*1))] + 
                   x[((i*7)+6+(i*1))] + 
                   x[((i*7)+7+(i*1))] == 1)
~~~

For our next constraint, we want to make sure each station doesn't have more than the takt time worth of work at the station. The equation is the sum of each task $$ j $$ iterated across station $$ i $$. This was another one that I wasn't clever enough to loop through, so it's about half hard-coded.

$$
\sum_{ij} \text{Cycle Time}_{j}x_{ij} \leq \text{Takt}, \forall i
$$

~~~python
for i in range(len(process)):
        solver.Add(ct[0]*x[i] + 
                   ct[1]*x[i+(1*8)] + 
                   ct[2]*x[i+(2*8)] + 
                   ct[3]*x[i+(3*8)] + 
                   ct[4]*x[i+(4*8)] + 
                   ct[5]*x[i+(5*8)] + 
                   ct[6]*x[i+(6*8)] + 
                   ct[7]*x[i+(7*8)] <= takt)
~~~


$$

\begin{aligned}
& \underset{z}{\text{minimize}}
&& z \\
&\text{subject to}
&& \sum_{i} y_{ij} = 1,\space \forall j  \\
&& \sum_{ij} \text{Cycle Time}_{j}x_{ij} \leq \text{Takt}, \forall i \\
&& y_0 - y_1 \leq 0 \\
&& y_0 - y_2 \leq 0 \\
&& y_1 - y_3 \leq 0 \\
&& y_1 - y_4 \leq 0 \\
&& y_2 - y_4 \leq 0 \\
&& y_3 - y_5 \leq 0 \\
&& y_4 - y_6 \leq 0 \\
&& y_5 - y_6 \leq 0 \\
&& y_6 - y_7 \leq 0 \\
&& x_{ij}\in \{0,1\} \\
&& y_{i},z\in \{0,\infty\} \\
\end{aligned}

$$