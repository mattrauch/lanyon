layout: post
title: Modeling with Monte Carlo
---

In the business and manufacturing world, we are confronted with problems that require us to make estimations. Generally, these are planning related---such as how many people will be needed, how much time will something take, ecetera. More often than not, these questions can be answered with sufficient precision through paramateric estimation, that is to say simply multipling the average time the expected frequency. Today, we will be rejecting such simplicity to utilize a toolset from the Monte Carlo methods.

The easiest way to think about Monte Carlo is perhaps from the engineering domain. During product design, analysis of tolerance stack up must occur to make sure the product, given it's defined tolerances, can still achieve it's desired function. Take for example, a metal piece that has multiple dimensions. Each of these dimensions can vary to their specified degree, but what happens when different combinaitons of variation are observed? What if each dimension randomly was at the high end of it's dimensional tolerance, would it still fit with the other pieces in the assembly? In order to 