---
title: Multi-armed Bandit non stationary rewards
author: Jonny Law
date: '2019-04-16'
slug: multi-armed-bandit-non-stationary-rewards
categories: []
tags: []
draft: true
---

# Exercise 2.5: Non-stationary Rewards

Assume that $q_t(a) \sim N(q_{t-1}(a), 0.1)$.

```{r}
bandit_non_constant <- function(epsilon, actions, reward, steps) {
  q <- rep(0, times = actions)
  N <- rep(0, times = actions)
  R <- numeric(steps)
  qa <- rnorm(10)
  
  for (i in seq_len(steps)) {
    u <- runif(1)
    next_action <- if (u < epsilon) {
      sample.int(n = actions, size = 1)
    } else {
      which.max(q)
    }
    qr <- reward(qa, next_action)
    qa <- qr[1:actions]
    R[i] <- qr[actions+1]
    N[next_action] <- N[next_action] + 1
    q[next_action] <- q[next_action] + (R[i] - q[next_action]) / N[next_action]
  }
  list(q, N, R)
}

# Random walk for the true expected-reward
step <- function(q) 
  q + rnorm(1, sd = 0.01)

# The reward is selected from a N(q(A_t), 1)
r <- function(qa, action) {
  new_q <- step(qa)
  r <- rnorm(1, mean = new_q[action], sd = 1)
  return(c(new_q, r))
}

average_reward <- function(n, steps, eps) {
  tibble(
    epsilon = rep(eps, times = steps),
    step = seq_len(steps),  
    average_reward = replicate(n = n, expr = bandit_non_constant(eps, actions = 10, reward = r, steps))[3,] %>% 
      reduce(`+`) / n
  )
}

c(0.0, 0.1, 0.01) %>% 
  map_df(~ average_reward(2000, 1000, .)) %>% 
  ggplot(aes(x = step, y = average_reward, colour = as.factor(epsilon))) +
  geom_line()
```

# Constant step-size parameter

Use epsilon = 0.1, alpha = 0.1

```{r}
bandit_alpha <- function(epsilon, actions, reward, steps, alpha) {
  q <- rep(0, times = actions)
  R <- numeric(steps)
  qa <- rnorm(10)
  
  for (i in seq_len(steps)) {
    u <- runif(1)
    next_action <- if (u < epsilon) {
      sample.int(n = actions, size = 1)
    } else {
      which.max(q)
    }
    qr <- reward(qa, next_action)
    qa <- qr[1:actions]
    R[i] <- qr[actions+1]
    q[next_action] <- q[next_action] + alpha * (R[i] - q[next_action])
  }
  list(q, R)
}

# Random walk for the true expected-reward
step <- function(q) 
  q + rnorm(1, sd = 0.01)

# The reward is selected from a N(q(A_t), 1)
r <- function(qa, action) {
  new_q <- step(qa)
  r <- rnorm(1, mean = new_q[action], sd = 1)
  return(c(new_q, r))
}

average_reward <- function(n, steps, al, eps) {
  tibble(
    alpha = rep(al, times = steps),
    epsilon = rep(eps, times = steps),
    step = seq_len(steps),  
    average_reward = replicate(n = n, expr = bandit_alpha(eps, actions = 10, reward = r, steps, al))[2,] %>% 
      reduce(`+`) / n
  )
}

epsilon <- c(0.0, 0.1, 0.01)
alpha <- c(0.1, 0.2, 0.5)
params <- expand.grid(epsilon, alpha)
map2_df(params[,1], params[,2], function(eps, al) average_reward(2000, 1000, al, eps)) %>% 
  ggplot(aes(x = step, y = average_reward, colour = as.factor(epsilon))) +
  geom_line() +
  facet_wrap(~alpha) +
  theme(legend.position = "bottom") +
  labs(title = "Average expected reward for a non-constant actual reward using various values of alpha")
```