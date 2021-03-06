The iPod Song Problem
================

Problem
-------

> Joe’s iPod has 500 different songs, consisting of 50 albums of 10 songs each. He listens to 11 random songs on his iPod, with all songs equally likely and chosen independently (so repetitions may occur).
>
> 1.  What is the PMF of how many of the 11 songs are from his favorite album?
> 2.  What is the probability that there are 2 (or more) songs from the same album among the 11 songs he listens to?
> 3.  A pair of songs is a match if they are from the same album. If, say, the 1st, 3rd, and 7th songs are all from the same album, this counts as 3 matches. Among the 11 songs he listens to, how many matches are there on average?

-   Introduction to Probability (J. Blitzstein), Exercises 4.12, Problem 78.

Solution
--------

### The Probability Mass Function

For the PMF, we can use simulations to visually analyze the problem:

``` r
all_songs <- c(rep(1, 10), rep(0, 490))

num_iterations <- 10^5

single_simulation <- function() {
  sum(sample(all_songs, 11, replace = TRUE))
}

experimental_distribution <- tabulate(as.factor(replicate(num_iterations, single_simulation())), nbins=12) / num_iterations

barplot(
  experimental_distribution, 
  xlab = 'Number of songs from favorite album', 
  names.arg = 0:11,
  col = 'white',
  main = 'Distribution of # Songs from Favorite Album'
)
```

![](iPodSongProblem_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-1-1.png)

Theoretically, this can be thought of as a binomial distribution. We can imagine the 10 favorite songs as successful cases, and the rest as failure. Probability of a song from the favorite album is 10/500 = 0.02. Therefore, P(X = x) = bin(11, 0.02). We can verify this in R:

``` r
theoretical_distribution <- dbinom(x = 0:11, size = 11, prob = 0.02)
barplot(
  theoretical_distribution,
  xlab = 'Number of Songs from Favorite Album', 
  names.arg = 0:11,
  col = 'white',
  main = 'Binomial Distribution of # Favorite Songs'
)
```

![](iPodSongProblem_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-2-1.png)

Clearly, the experimental and theoretical distributions are identical.

### Probability of 2 or More Songs from the Same Album

The first option is to simply simulate the scenario, and use the empirical result:

``` r
all_songs <- rep(1:50, 10)

has_two_or_more_songs_from_same_album <- function() {
  length(unique(sample(all_songs, 11, replace = TRUE))) != 11
}

num_iterations <- 10^5

simulations <- replicate(num_iterations, has_two_or_more_songs_from_same_album())

probability_of_2_or_more_from_same_album <- sum(simulations) / num_iterations

probability_of_2_or_more_from_same_album
```

    ## [1] 0.69583

We can verify this by using the basic definition of probability, and calculate the probability of no two songs being from the same album. The answer to this question is the complement:

``` r
prob_of_no_two_songs_from_same_album <- 1

# The second song has only 490 options
# Then the third song has 480, and so on...
for (i in seq(490, 400, -10)) {
  prob_of_no_two_songs_from_same_album <- prob_of_no_two_songs_from_same_album * (i/500)
}

probability_of_2_or_more_from_same_album <- 1 - prob_of_no_two_songs_from_same_album
probability_of_2_or_more_from_same_album
```

    ## [1] 0.6946347

### Expected Number of Matches

Again, we can take a quick look at the answer with simulation:

``` r
all_songs <- rep(1:50, 10)

num_matches_with_k_songs_from_same_album <- function(k) {
  choose(k, 2)
}

number_of_matches <- function() {
  random_songs <- sample(all_songs, 11, replace=TRUE)
  # tabulate will group by each album
  # And sapply will count the number of matches possible for x songs from each album
  num_matches_per_album <- sapply(tabulate(random_songs), num_matches_with_k_songs_from_same_album)
  
  # Finally we add up all the matches
  sum(num_matches_per_album)
}

num_iterations <- 10^5

# We take expected (mean) value of the number of matches
mean(replicate(num_iterations, number_of_matches()))
```

    ## [1] 1.10064

We can verify this by using indicator variables and the linearity of expectation. If I(ij) is the indicator variable that the ith and jth song are from the same album, and X the number of matches, then we have:

![NumberOfMatches](images/iPodSongEquation_01.png)

Taking expectation, and applying the fundamental bridge of probability and expectation we have:

![Expectation](images/iPodSongEquation_02.png)

``` r
e_X = choose(11, 2) * 0.02
e_X
```

    ## [1] 1.1

This is the same answer as the one we got using simulations, verifying that our analysis is correct.
