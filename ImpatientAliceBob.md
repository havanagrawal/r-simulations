Impatient Alice and Bob
================

Problem Statement
-----------------

> Alice and Bob arrange to meet for lunch on a certain day at noon. However, neither is known for punctuality. They both arrive independently at uniformly distributed times between noon and 1 pm on that day. Each is willing to wait up to 15 minutes for the other to show up. What is the probability they will meet for lunch that day?

-   Introduction to Probability (J. Blitzstein), Exercises 7.8, Problem 1.

Solution By Simulation
----------------------

This is a fairly easy problem to solve by simulation:

``` r
num_obs <- 10^7

X <- runif(num_obs, min = 0, max = 60)
Y <- runif(num_obs, min = 0, max = 60)

sum(abs(X - Y) <= 15) / num_obs
```

    ## [1] 0.4377645
