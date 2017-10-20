Best Of 7 Tournament
================

Problem Statement
-----------------

> Two teams are going to play a best-of-7 match (the match will end as soon as either team has won 4 games). Each game ends in a win for one team and a loss for the other team. Assume that each team is equally likely to win each game, and that the games played are independent. Find the mean and variance of the number of games played.

-   Introduction to Probability (J. Blitzstein), Exercises 4.12, Problem 6.

Solution by Simulation
----------------------

We can simulate the problem in a very literal form.

``` r
single_tournament = function() {
  A_wins <- 0
  B_wins <- 0
  num_games <- 0
  
  # Play 7 games
  games <- sample(0:1, 7, replace = TRUE)
  i <- 0
  
  # As soon as either A or B have 4 wins, we don't need the results of the remaining games
  while (A_wins < 4 & B_wins < 4) {
    i <- i + 1
    
    # For this game, did A win or B win?
    game <- games[i]

    A_wins <- A_wins + (1 - game)
    B_wins <- B_wins + game
  }
  
  i
}

mean(replicate(10^5, single_tournament()))
```

    ## [1] 5.81313

Solution by Theory
------------------

We can solve this theoretically by solving for the probabilities of playing X games for X = 0, 1, 2, ... 7, and then using the standard definition of expectation.

``` r
probabilities_of_x_games <- function() {
  px <- c(0, 0, 0, 0, 0, 0, 0)
  
  # The probability of playing 1, 2 or 3 games is 0 because
  # it is not possible to determine the winner in less than 4 games
  for (i in 4:7) {
    
    # Fix the ith win, and pick the number of ways you can win 3 out of i-1 games
    # And divide by the total number of games possible
    px[i] = choose(i - 1, 3) / 2^(i - 1)
  }
  
  px
}

probabilities_of_x_games()
```

    ## [1] 0.0000 0.0000 0.0000 0.1250 0.2500 0.3125 0.3125

``` r
expected_number_of_games <- function() {
  sum(1:7 * probabilities_of_x_games())
}

expected_number_of_games()
```

    ## [1] 5.8125

We can see that the actual result is very close to the one we arrived on using simulation.

We can also solve for variance since it is part of the question:

``` r
variance_of_num_games <- function() {
  x2 = (1:7) ^ 2
  sum(x2 * probabilities_of_x_games()) - expected_number_of_games()^2
}

variance_of_num_games()
```

    ## [1] 1.027344
