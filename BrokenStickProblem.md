Broken Stick Problem
================

Problem
-------

> A stick of length 1 is broken at a uniformly random point, yielding two pieces. Let X and Y be the lengths of the shorter and longer pieces, respectively, and let R = X=Y be the ratio of the lengths of X and Y.
>
> 1.  Find the CDF and PDF of R.
> 2.  Find the expected value of R (if it exists).
> 3.  Find the expected value of 1/R (if it exists).

-   Introduction to Probability (J. Blitzstein), Exercises 5.10, Problem 13.

Solution
--------

### Setup

We can set up a function that can simulate the behaviour of R:

``` r
short_by_long <- function(len) {
  # Given the length of one piece of the stick (originally of length 1), 
  # Return the ratio of the shorter piece to the longer piece
  
  shorter <- min(len, 1 - len)
  longer <- 1 - shorter
  
  shorter/longer
}
```

### CDF of R

Calculating the CDF and PDF is tricky for continuous variables. We can visualize the CDF by generating multiple values of R, and then plotting the CDF:

``` r
num_iterations <- 10^5

# Generate 10^5 uniform numbers, 
# which are interpreted as an arbitrary piece of a unit stick broken into 2
x <- runif(num_iterations)

# Generate values of R
r_sample <- sapply(x, short_by_long)

# How many segments to divide the range of R into
divisions <- 100

# cdf[i] actually contains the value of cdf[i/100]
# So cdf[5] is the value of F(0.05)
cdf <- rep(0, 1 + divisions)

x_vals <- seq(0, 1, 1/divisions)

k <- 1
for (i in x_vals) {
  cdf[k] <- mean(r_sample < i)
  k <- k + 1
}

plot(x = x_vals, y = cdf, xlab = "R", ylab = "CDF")
```

![](BrokenStickProblem_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-2-1.png)

The theoretical formula for the CDF of R turns out to be 2x/(1 + x), after solving the problem statement. We can plot this on top of the experimental plot to determine how correct the simulation was:

``` r
expected_cdf <- function(x) {
  2*x/(1 + x)
}

y_vals <- sapply(seq(0, 1, 0.01), expected_cdf)

plot(x = x_vals, y = cdf, xlab = "R", ylab = "CDF")
lines(x = seq(0, 1, 0.01), y=y_vals, col="red", lwd = 2)

legend("topleft", legend = c("Experimental", "Theoretical"), pch = c(1, 45), col=c("black", "red"))
```

![](BrokenStickProblem_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-3-1.png)

Our simulation appears to have given the correct results!

### Expected Value of R

This is fairly easy to calculate at this point:

``` r
mean(r_sample)
```

    ## [1] 0.3856789

To verify this, we can solve the expression for E(X) by integrating the PDF under appropriate limits, and finally we arrive to the value of ln(4) - 1

``` r
log(4) - 1
```

    ## [1] 0.3862944

Again, our empirical value seems to be extremely close to the actual value.

### Expected Value of 1/R

Before we begin to calculate this value, I want to take a moment to reiterate the danger of mistaking E(g(X)) with g(E(X)) when g(X) is non-linear. My initial thought was that if the expected value of R is 0.386, then the expected value of 1/R should be 1/0.386, but this turns out to be incorrect. Using simulations, we can verify this:

``` r
num_iterations <- 10^4
expected_1_by_r <- function() {
  x <- runif(num_iterations)
  
  # Generate values of R
  r_sample <- sapply(x, short_by_long)
  
  mean(1/r_sample)
}

expected_r <- function() {
  x <- runif(num_iterations)
  
  # Generate values of R
  r_sample <- sapply(x, short_by_long)
  
  mean(r_sample)
}

means_of_1_by_r <- replicate(100, expected_1_by_r())
means_r <- replicate(100, expected_r())

par(mfrow=c(2,1))
hist(means_r, breaks = seq(0, 1, 0.01))
hist(means_of_1_by_r[means_of_1_by_r <= 100], breaks = seq(0, 100, 1))
```

![](BrokenStickProblem_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-6-1.png)

We can compare the plots of the means of R and 1/R. For R, the frequency of values closer to the actual mean is extremely high, as opposed to the histogram of mean of 1/R, which appears to be all over the place. Increasing the number of iterations has no visible effect on the histogram.

At this stage, we can assume that 1/R does not converge. While I cannot think of an exact reason, my intuition says that the ratio of the longer to the shorter piece can range from 1 to infinity. This is a very long tail of higher "payoff" with lower and lower probabilities, approaching 0 asymptotically:

``` r
pdf_of_1_by_r <- function(x){
  2/(1+x)^2
}

curve(expr = pdf_of_1_by_r, from=1, to=20, n = 10000)
abline(h=0, lty=2)
```

![](BrokenStickProblem_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-7-1.png)

This is similar to the St. Petersburg paradox from the textbook.
