---
layout: post
title: About Different Correlation Methods
tags: correlations methodology pandas blog draft
---

There are three different correlations: "Pearson", "Kendall" and "Spearman" based on `pandas.DataFrame.corr` [method](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.corr.html)

What are those? What does each do? What are the differences between them? What should I use? In this short article, we will walk you through all three and explore a little deeper to answer what we should pick under different scenarios.

## Pearson Correlation

One of the most commonly used method is Pearson correlation. It is defined as:

$$
\rho(X, Y) = \frac{Cov(X,Y)}{\sigma_{X}\sigma_{Y}}
$$

The covariance is defined as $Cov(X, Y) = E[(X -(\bar{X}))(Y-(\bar{Y}))]$. 

Covariance itself already gives some measures to the association between two variables. However, the issue of covariance alone is if you scale the variable (e.g.: multiply every Y value with 100), the covariance changes as well. This does not sound like a good idea as we are measuring the intrinsic relationship between X and Y. And scaling Y by 100 times should not change that intrinsic relationship at all. To solve this issue, we normalized the covariance by the product of standard deviations for both $X$ and $Y$. After this, the quantity is not scaled with X and Y. And the value is perfectly between -1 and 1.


You may ask that if the only reason that we divide the product of standard deviation is to scale down the covariance appropriately. Why not we use other quantities like mean to scale it?

The first thought is mean is the first order moment while covariance/standard deviation and covariance are both second order moments. so it is more natural to use standard deviation than mean to normalize the covariance. Second, using mean or other metrics leads to a non bounded to upper limit of 1. And the resulting quantity actually depends on the size of the sample. This would causes some inconvience as I don't want my correlation metrics have anything to do with how many samples I use.

Assuming that we use mean as denominators for the Pearson correlations:

$$
\begin{eqnarray}
Corr_{mean} & = & \frac{Cov(X,Y)}{E(X)E(Y)} \\
            & = & \frac{E(XY) - E(X)E(Y)}{E(X)E(Y)} \\
            & = & \frac{E(XY)}{E(X)E(Y)} - 1
\end{eqnarray}
$$

The first part of the above equation $\frac{E(XY)}{E(X)E(Y)}$ is dependent on the sample size (numerator is $\frac{1}{N}$ and the demoninator has $\frac{1}/{N^2}$. This does not sound like a good idea to have.

Pearson correlation can also be considered as degree of linear relationship between X and Y. This is because if we do a least square fit. Then the slope of the line that fit X,Y with least square error is:

$$
\beta_{X~Y} = \frac{Cov(X,Y)}{Var(X)}
$$

The $\beta$ calculated in this way has one problem. The scale of it can change dramatically when the scale of y changed. Therefore, we use $Var(Y)$ to scale it down. And we reached the Pearson correlation formula again.

## Kendall's $\tau$ correlation

There is another category of correlation called rank correlation. Instead of looking at the value of the variables, we use the rank of the variables to calculate the correlation instead.

Kendall's tau is one of the rank correlation measures. It is defined by counting concordant and discordant pairs formed between $X$ and $Y$s.

Every $x_i$ from $X$ can pair with another $y_i$ from $Y$. This pair can compare against another pair $(x_j, y_j)$. If the order between $x_i$ and $x_j$ is the same as the order between $y_i$ and $y_j$. This is called concordant. The vice versa is called discordant. If there is a tie then it is neither concordant nor discordant.

The Kendall's tau is defined as 

$$
\tau = \frac{concordant - discordant}{n \choose 2}
$$

In extreme cases, if all pairs are concordant, we will have a 1. If all pairs are discordant, the correlation is -1.

## Spearman correlation

The Spearman correlation is an interesting one. It actually uses the Person correlation equation but replace the variable value with the rank of the value.


$$
r(X, Y) = \frac{Cov(R_X,R_Y)}{\sigma_{R_X}\sigma_{R_Y}}
$$

## Which one should I use under what circumstance?

After knowing the definition of all three, let's find out which one should we use. To answer this question, let's make some simulation and calculate different correlations.


```julia
using StatsBase
using Statistics
using Plots
using LinearAlgebra

x1 = 1:1.:100
y1 = exp.(x1) + sin.(x1)
(cov(x1, y1)) / (std(x1) * std(y1))


# This group of example to show linear Pearson correlations to others like Kendall and Spearman
cor(x1, y1)
corkendall(collect(x1), y1)
corspearman(collect(x1), y1)


plot(x1, y1)
```

This is a typical non-linear but monotonic relationship between X and Y:
![monotonic increase curve](https://i.imgur.com/1R7uaPg.png)


| Metrics  | Values |
| -------- | ------ |
| Pearson  | 0.252  |
| Kendall  | 1.0    |
| Spearman | 1.0    |

From the above table, you can see that if the relationship is non-linear but monotonic. Pearson correlation gives a falsified low value. But rank based correlations like Kendall and Spearman give the correct values.

In the second example, let's see if any of them can tell when the dependent is non monotonic.

```julia
y2 = 100*sin.(x1)
cor(x1, y2)
corkendall(collect(x1), y2)
corspearman(collect(x1), y2)
```

This is a typical dependent while non-monotonical example. The relationship looks like:
![sin curve](https://i.imgur.com/gi85b51.png)

And we have the correlation coefficients as:

| Metrics  | Values |
| -------- | ------ |
| Pearson  | -0.048 |
| Kendall  | -0.031 |
| Spearman | -0.047 |

You can see that none of them reveals the non monotonical relationship between $X$ and $Y$.

## Conclusions

Pearson correlation is widely used to meaure correlations between two variables. However, Pearson correlation heavily affected by the linearity between these two variables. If these two variables are strongly dependent with each other monotonically. Then rank based correlation, like Kendall or Spearman is a better metrics to measure the dependency between variables. However, none of them gives you any clue if the dependence is not monotonic.
