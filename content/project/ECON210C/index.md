---
date: "2024-06-24"
#image:
#  caption: Embed rich media such as videos and LaTeX math
math: false
summary: ECON210C
title: Structural Macro
type: docs
---

## Introduction

The repository [ECON210C](https://github.com/LapoBini/210C) contains the codes for assignments and final exam for ECON210C of Prof Johannes Wieland (Spring 2024). The course covers different solution methods for DSGE model, and identification strategies for structural VAR. 

### Sequence Space

The first assignment (__ECON210C__ > __HW1__) require to solve a RBC DSGE model using the Sequence Space methodology ($\textcolor{orange}{Auclert}$ $\textcolor{orange}{et}$ $\textcolor{orange}{al.,}$ $\textcolor{orange}{(2021)}$). 

The basic idea is to organize the model into blocks that represents different behaviors of possible heterogeneous agents and interact in GE via a small set of aggregates. Let's now denote $\mathrm{U} = \{U_1,\dots,U_t\}$

$$Y_t = c + A_1 Y_{t-1}+\dots+A_p Y_{t-p} + u_t$$

### Structural VAR

### Undetermine Coefficients

### New-Keynesian Model


## Setting the Priors

Let $Y_t = (y_{1t},\dots,y_{nt})'$ be a potentially large vector of random variables. We estimate the following VAR(p) model:
{{< math >}}
$$Y_t = c + A_1 Y_{t-1}+\dots+A_p Y_{t-p} + u_t$$
{{< /math >}}
using the Bayesian approach to overcome the curse of dimensionality via the imposition of prior beliefs on the parameters. In setting the prior distributions, we follow standard practice and use the procedure developed in $\textcolor{orange}{Litterman \; (1986)}$ with modifications proposed by $\textcolor{orange}{Kadiyala\; and\; Karlsson\; (1997)}$ and $\textcolor{orange}{Sims\; and\; Zha\; (1998)}$. The priors are implemented using dummy observations, and the choice of the informativeness of the prior distribution is determined by the marginal likelihood of the observed data; in that sense, we follow strictly the exposition and the procedure of $\textcolor{orange}{Banbura,}$ $\textcolor{orange}{Giannone,}$ $\textcolor{orange}{and}$ $\textcolor{orange}{Reichlin}$ $\textcolor{orange}{(2007)}$ and $\textcolor{orange}{Giannone,}$ $\textcolor{orange}{Lenza,}$ $\textcolor{orange}{Primicieri\, (2012)}$.



### Minnesota Prior
$\textcolor{orange}{Litterman \; (1986)}$ introduced the well known Minnesota prior: the basic principle behind it is that all the equations are centered around the random walk with drift:
$$ Y_t = c + Y_{t-1} + u_t\,.$$
This amounts to shrinking the diagonal elements of $A_1$ toward one and the remaining coefficients in $A_1,\dots,A_p$ toward zero. In addition, the prior specification incorporates the belief that the more recent lags should provide more reliable information than the more distant ones and that own lags should explain more of the variation of a given variable than the lags of other variables in the equation.
