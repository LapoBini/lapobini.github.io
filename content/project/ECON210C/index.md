---
date: "2024-06-24"
#image:
#  caption: Embed rich media such as videos and LaTeX math
math: false
summary: ECON210C
title: Monetary Economics
type: docs
---

## Introduction

The repository [ECON210C](https://github.com/LapoBini/210C) contains the solutions to assignments and final exam for ECON210C of Prof Johannes Wieland (Spring 2024). The course covers different solution methods for RBC and NK DSGE models, and identification strategies for structural VAR. 

### Sequence Space
The first assignment (__ECON210C__ > __HW1__) require to solve a RBC DSGE model using the Sequence Space methodology ($\textcolor{orange}{Auclert}$ $\textcolor{orange}{et}$ $\textcolor{orange}{al.,}$ $\textcolor{orange}{(2021)}$). The basic idea is to organize the model into blocks that represents different behaviors of possible heterogeneous agents and interact in GE via a small set of aggregates.

If we consider the basic RBC model, an equilibrium is an allocation $\{C_{t+s},N_{t+s},Y_{t+s},B_{t+s}\}_{s=0}^{\infty}$, a set of prices $\{W_{t+s},P_{t+s},Q_{t+s}\}_{s=0}^{\infty}$, an exogenous processes $\{A_{t+s},Tr_{t+s},M_{t+s}\}_{s=0}^{\infty}$ and initial conditions for bonds and capital $B_{t-1}$ such that:
1. Households maximize utility subject to budget constraints.
2. Firms maximize profits given their technology.
3. The government satisfies its budget constraint.
4. Markets clear.

The set of log-linearized equilibrium conditions, with  $\hat x \approx \ln X_t - \ln X^{ss}$, are:
{{< math >}}
$$
\begin{gather*}
\begin{split}
    Y_t&=A_t N_{t} \\
	\frac{W_t}{P_t}&= A_t  \\
	\frac{W_t}{P_t}&=\frac{\chi N_t^\varphi}{C_t^{-\gamma}} \\
	Y_t&=C_t \\
	C_{t}^{-\gamma}&=\beta E_t\big\{R_{t+1} C_{t+1}^{-\gamma} \big\} 
    \end{split}
    \xrightarrow{}
    \begin{split}
    \hat{y}_t&=\hat{a}_t + \hat{n}_{t}  \\
	\hat{w}_t - \hat{p}_t &= \hat{a}_t  \\
	\hat{w}_t - \hat{p}_t &=\varphi \hat{n}_t + \gamma \hat{c}_t \\
	\hat{y}_t& =\hat{c}_t \\
	-\hat{c}_{t}&= E_t\left\{\hat{r}_{t+1} - \gamma \hat{c}_{t+1} \right\} 
    \end{split}
\end{gather*}
$$
{{< /math >}}
Let's now log-linearized the equilibrium conditions around the steady state, with $\hat x \approx \ln X_t - \ln X^{ss}$
Let $Y_t = (y_{1t},\dots,y_{nt})'$ be a potentially large vector of random variables. We estimate the following VAR(p) model:
{{< math >}}
$$Y_t = c + A_1 Y_{t-1}+\dots+A_p Y_{t-p} + u_t$$
{{< /math >}}
The first assignment (__ECON210C__ > __HW1__) require to solve a RBC DSGE model using the Sequence Space methodology ($\textcolor{orange}{Auclert}$ $\textcolor{orange}{et}$ $\textcolor{orange}{al.,}$ $\textcolor{orange}{(2021)}$). 

The basic idea is to organize the model into blocks that represents different behaviors of possible heterogeneous agents and interact in GE via a small set of aggregates. Let's now denote $\mathrm{U} = \{U_1,\dots,U_t\}$

$$Y_t = c + A_1 Y_{t-1}+\dots+A_p Y_{t-p} + u_t$$

### Structural VAR

### Undetermine Coefficients

### New-Keynesian Model

