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
The first assignment (__ECON210C__ > __HW1__) require to solve a RBC DSGE model with a composite consumption basket of goods and money, using the Sequence Space methodology ($\textcolor{orange}{Auclert}$ $\textcolor{orange}{et}$ $\textcolor{orange}{al.,}$ $\textcolor{orange}{2021}$). The code in __ECON210C__ > __HW1__ > __RBC.jl__ solves a specific RBC model with a consumption basked made of consumption good and money. In this paragraph we provide a general overview of how the sequence space procedure works.  

The basic idea is to organize the GE model into blocks that represents different behaviors of possible heterogeneous agents and interact in GE via a small set of aggregates. If we consider a basic RBC model, an equilibrium is an allocation $\{C_{t+s},N_{t+s},Y_{t+s},B_{t+s}\}_{s=0}^{\infty}$, a set of prices $\{W_{t+s},P_{t+s},Q_{t+s}\}_{s=0}^{\infty}$, an exogenous processes $\{A_{t+s},Tr_{t+s},M_{t+s}\}_{s=0}^{\infty}$ and initial conditions for bonds and capital $B_{t-1}$ such that:
1. Households maximize utility subject to budget constraints.
2. Firms maximize profits given their technology.
3. The government satisfies its budget constraint.
4. Markets clear.

The set of log-linearized equilibrium conditions, with  $\hat x \approx \ln X_t - \ln X^{ss}$, is given by:
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
    \quad\quad \xrightarrow{} \quad\quad
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

Let's now denote $\mathrm{X} = \{X_1,\dots,X_t\}$ as the perfect-foresight time sequence of aggregate macroeconomic outcomes, and $\mathrm{Z} = \{Z_1,\dots,Z_t\}$ exogenous shock sequence. A perfect-foresight equilibrium given shock sequence $\mathrm{Z}$ is the set of paths $\mathrm{X}$ such that:
$$ F(\mathrm{X}, \mathrm{Z}) = 0 $$
where $F(\cdot)$ embeds the model's equilibrium relationship (the ones highlighted before). Then, we can define a mapping from $\mathrm{Z}$ to $\mathrm{X}$ by taking total differentiation:
$$ \text{d}F(\mathrm{X}, \mathrm{Z}) = \text{d}0 $$
$$ F_x \, \text{d}\mathrm{X} + F_z \, \text{d}\mathrm{Z} = 0 $$
However, we can we often times can arrive at a lower-dimensional representation of equilibria that does not involve all variables: given $\mathrm{Z}$, may be able to characterize equilibria via
$$ H(\mathrm{U}, \mathrm{Z}) = 0 $$
where $\mathrm{U} = \{U_1,\dots,U_t\}$ is the time path of some unknown macroeconomic outcomes and $ n_u < n_x$. Thus,h $\mathrm{X}$ is determined residually via $ \mathrm{X}= M(\mathrm{U}, \mathrm{Z})$. The number of unknown endogenous variables in $\mathrm{U}$ must be the same as the number of equilibrium condition in $H(\cdot)$ (in the code __ECON210C__ > __HW1__ > __RBC.jl__ only one equilibrium condition, bond demand equation, and one endogenous variable, labor supply). To first order we can write
$$ \text{d}H(\mathrm{U}, \mathrm{Z}) = \text{d}0 $$
$$ \text{d}H(\mathrm{U}, \mathrm{Z}) = H_u \, \text{d}\mathrm{U} + H_z \, \text{d}\mathrm{Z} = 0 $$
$$ \text{d}\mathrm{U} = - H_u^{-1} H_z \, \text{d}\mathrm{Z} $$
We thus recover the solution of the model as:
$$ \text{d}\mathrm{X} = M_u \, \text{d}\mathrm{U} + M_z \, \text{d}\mathrm{Z} $$
$$ \text{d}\mathrm{X} = M_u \big( - H_u^{-1} H_z \, \text{d}\mathrm{Z} \, \big) + M_z \, \text{d}\mathrm{Z} \, .$$
Remember that $H_u$ and $H_z$ are sequence-space Jacobians, which are computed as:
1. $ H_u = \frac{\partial H}{\partial \mathrm{U}} = \frac{\partial H}{\partial Y} \frac{\partial Y}{\partial \mathrm{U}}$
2. $ H_z = \frac{\partial H}{\partial \mathrm{Z}} = \frac{\partial H}{\partial Y} \frac{\partial Y}{\partial \mathrm{Z}}$

The solution of the model gives us immediately the structural IRF to the $j^{th}$ shock of interest:
$$ \Theta_{\cdot,j,\cdot} = M_u \big( - H_u^{-1} H_{z_j} e_1 \, \big) + M_{z_j} \, e_1 \, .$$
where the indicatior vector $e_1 = (1, 0, \dots, 0)'$ specifies a transitory shock happening at $t=0$ and then the system converges back to steady state (stability). For $t=1,2,\dots$ the linearized optimality conditions hold in expectation. The sequence space procedure has given us the $SVMA(\infty)$ representation:
$$ \mathrm{X}_t=\sum_{h=0}^{\infty}\Theta_h\varepsilon_{t-h} \, . $$


### Structural VAR

### Undetermine Coefficients

### New-Keynesian Model

