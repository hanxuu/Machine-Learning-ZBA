# Bayesian Theory  

## Introduction of Bayesian

### Frequency


* A set of random samples, the frequency school believes that the overall parameters are constant, and the samples are obtained randomly;
* The Bayesian school believes that the **overall parameters** are random, and the sample obtained is constant. The Bayesian school does not care much about the correct parameters, but needs to obtain the **posterior** by adding the **acquired data to the prior knowledge**


### Posterior distribution

The posterior distribution summarises our uncertainty over the value of a parameter. If the distribution is narrower, then this indicates that we have greater confidence in our estimates of the parameter’s value. More narrow posterior distributions can be obtained by collecting more data.

* The posterior probability is the probability of the parameters $\theta$ given the evidence $X: p(\theta \mid X)$
* It contrasts with the likelihood function, which is the probability of the evidence given the parameters: $p(X \mid \theta)$

The two are related as follows: Given a prior belief that a probability distribution function is $p(\theta)$ and that the observations $x$ have a likelihood $p(x \mid \theta)$, then the posterior probability is defined as
$$
p(\theta \mid x)=\frac{p(x \mid \theta)}{p(x)} p(\theta)
$$


























