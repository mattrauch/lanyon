---
title: The Devil's Staircase
layout: post
---

![Devil's Staircase]({{ "/assets/cantor.png" | absolute_url }})

In the spirit of Pi Day and my amateur interest in mathematical oddities, it seemed to suit to make the first post on Lean Focus about a mathematical function. With that said, let’s begin with a short trivia question: name a continuous function that has a zero derivative almost everywhere and rises from 0 to 1 when plotted.

Stumped? So was I. To be honest, the only reason I knew the answer to this was due to a post on stack exchange. The name of this function is the Cantor function, otherwise known as the Devil’s Staircase. The function itself is a cantor ternary set, defined by[^fn1]:

$${\displaystyle {\mathcal {C}}=[0,1]\smallsetminus \bigcup _{n=1}^{\infty }\bigcup _{k=0}^{3^{n-1}-1}\left({\frac {3k+1}{3^{n}}},{\frac {3k+2}{3^{n}}}\right)}$$

To plot the function, iteratively remove the middle thirds of each interval for an infinite set of intervals. This creates a sort of fractal effect as illustrated by the header photo. The Cantor function on its own is interesting, but what really caught my attention was the shape it took when plotted on the unit interval as pictured below.

![Cantor Function]({{ "/assets/cantorfunction.png" | absolute_url }})

As implied by the name, the function resembles an ascending staircase with the largest plateau at the function’s origin. I couldn’t help but see the parallel between the reality-defying characteristics of this function and the continuous improvement process that happens on the shop floor.

The Cantor function has two distinct aspects that make it relevant to the continuous improvement process. The first aspect is that the Cantor function has a zero derivative almost everywhere, but also rises from 0 to 1. Think about that last improvement your team made to some process. At any distinct point in time, was it obvious or measurable that improvement was occurring? Most of the time, we can only find the true improvement over a series of PDCA cycles that culminate into a lasting impact to the process — this is simply the nature of continuous improvement. When we map the series of PDCA cycles over time, distinct changes emerge, and we see the change from the current state to the target state.

Math aside, the shape and progression of the Cantor function also resemble the distinct inconsistency that we face in continuous improvement. In essence, the blind nature of the continuous improvement process leads us to times of plateau as well as rapid improvement. Teams that take the time to understand the process will find these plateau periods often, as some issues have more elusive solutions than others. Those who have read Mike Rother’s Toyota Kata will be familiar with this parallel between a staircase and the continuous improvement process. If you haven’t read it already, it provides great insight into the subtleties of this process of “moving up the staircase” and the mindset and approach to avoid moving backward.

In short, remember that the continuous improvement process can be foggy, but patience and observance yields improvement. If you’re feeling unmotivated, see if you can map out the same staircase of improvement from where the process started. If the staircase doesn’t work for your team, look out for other elements of nature that resemble the process. Most people probably don’t need a math equation to see it, but sometimes these metaphors can offer the right perspective for the right mind. Happy Pi Day.

[^fn1]: Soltanifar, Mohsen (2006). "A Different Description of A Family of Middle-a Cantor Sets". American Journal of Undergraduate Research. 5 (2): 9–12.