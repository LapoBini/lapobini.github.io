---
date: "2024-06-24"
#image:
#  caption: Embed rich media such as videos and LaTeX math
math: false
summary: ECON210C
code: ECON210C
repo: "https://github.com/LapoBini/210C"
weight: 2
abstract: "This page explains the tools used in ECON210C (Monetary Economics, Prof. Johannes Wieland, UC San Diego, Spring 2024), and the GitHub repository contains the code with the solutions to each assignment and to the final exam. The code implements modern solution methods for RBC and New Keynesian DSGE models, including the method of undetermined coefficients and the sequence-space approach of Auclert et al. (2021), together with identification strategies for structural VARs based on recursive ordering and the external instrument approach."
title: Monetary Economics
type: docs
---

## Introduction

This page explains the tools used in ECON210C, taught by Prof. Johannes Wieland at UC San Diego (Spring 2024), and the repository [ECON210C](https://github.com/LapoBini/210C) contains the code with the solutions to each assignment and to the final exam. The course covered different solution methods for RBC and New Keynesian DSGE models, and identification strategies for structural VARs.

## Sequence Space
The first assignment (__ECON210C__ > __HW1__) required solving an RBC DSGE model with the sequence-space methodology of [Auclert et al.](#ref-auclert2021) ([2021](#ref-auclert2021)). The code in __ECON210C__ > __HW1__ > __RBC.jl__ solves a specific RBC model with a composite basket of a consumption good and money. In this section we give a general overview of how the sequence-space procedure works. The basic idea is to organize the general equilibrium model into blocks, which represent the behavior of different (possibly heterogeneous) agents and interact in general equilibrium through a small set of aggregates.

Let's start from the definition of equilibrium for a basic RBC model. The equilibrium is defined as an allocation $\{C_{t+s},N_{t+s},Y_{t+s},B_{t+s}\}_{s=0}^{\infty}$, a set of prices $\{W_{t+s},P_{t+s},Q_{t+s}\}_{s=0}^{\infty}$, exogenous processes $\{A_{t+s},Tr_{t+s},M_{t+s}\}_{s=0}^{\infty}$ and initial conditions for bonds such that:
1. Households maximize utility subject to their budget constraints.
2. Firms maximize profits given their technology.
3. The government satisfies its budget constraint.
4. Markets clear.

The set of log-linearized equilibrium conditions, with $\hat x_t \approx \ln X_t - \ln X^{ss}$, is given by:
{{< math >}}
$$
\begin{gather*}
\begin{split}
    Y_t&=A_t N_{t} \\
	\frac{W_t}{P_t}&= A_t  \\
	\frac{W_t}{P_t}&=\frac{\chi N_t^\varphi}{C_t^{-\gamma}} \\
	Y_t&=C_t \\
	C_{t}^{-\gamma}&=\beta E_t\big\{R_{t+1} C_{t+1}^{-\gamma} \big\}
    \end{split} \\[0.5em]
    \big\downarrow \\[0.5em]
    \begin{split}
    \hat{y}_t&=\hat{a}_t + \hat{n}_{t}  \\
	\hat{w}_t - \hat{p}_t &= \hat{a}_t  \\
	\hat{w}_t - \hat{p}_t &=\varphi \hat{n}_t + \gamma \hat{c}_t \\
	\hat{y}_t& =\hat{c}_t \\
	-\gamma\hat{c}_{t}&= E_t\left\{\hat{r}_{t+1} - \gamma \hat{c}_{t+1} \right\}
    \end{split}
\end{gather*}
$$
{{< /math >}}

Let's now denote by $\mathrm{X} = \{X_1,\dots,X_T\}$ the perfect-foresight time path of the aggregate macroeconomic outcomes, and by $\mathrm{Z} = \{Z_1,\dots,Z_T\}$ the path of the exogenous shocks. A perfect-foresight equilibrium, given the shock path $\mathrm{Z}$, is a set of paths $\mathrm{X}$ such that:
$$ F(\mathrm{X}, \mathrm{Z}) = 0\,, $$
where $F(\cdot)$ embeds the model's equilibrium relationships listed above. We can then define a mapping from $\mathrm{Z}$ to $\mathrm{X}$ by taking the total derivative:
$$ \text{d}F(\mathrm{X}, \mathrm{Z}) = \text{d}0 $$
$$ F_x \, \text{d}\mathrm{X} + F_z \, \text{d}\mathrm{Z} = 0\,. $$
However, we can often arrive at a lower-dimensional representation of the equilibrium that does not involve all variables: given $\mathrm{Z}$, we may be able to characterize the equilibrium via
$$ H(\mathrm{U}, \mathrm{Z}) = 0\,, $$
where $\mathrm{U} = \{U_1,\dots,U_T\}$ is the time path of a smaller set of unknown macroeconomic outcomes, with $n_u < n_x$ (the number of unknowns and of variables, respectively). Then, $\mathrm{X}$ is determined residually via
$$ \mathrm{X}= M(\mathrm{U}, \mathrm{Z}) \; .$$
The number of unknown endogenous variables ($n_u$) in $\mathrm{U}$ must equal the number of equilibrium conditions in $H(\cdot)$. In the code (__ECON210C__ > __HW1__ > __RBC.jl__) there is only one equilibrium condition, the bond demand equation, and one unknown endogenous variable, labor supply. To first order, we can write:
$$ \text{d}H(\mathrm{U}, \mathrm{Z}) = \text{d}0 $$
$$ \text{d}H(\mathrm{U}, \mathrm{Z}) = H_u \, \text{d}\mathrm{U} + H_z \, \text{d}\mathrm{Z} = 0 $$
$$ \text{d}\mathrm{U} = - H_u^{-1} H_z \, \text{d}\mathrm{Z} \, .$$
Thus, we derive the solution of the model as follows:
$$ \text{d}\mathrm{X} = M_u \, \text{d}\mathrm{U} + M_z \, \text{d}\mathrm{Z} $$
$$ \text{d}\mathrm{X} = M_u \big( - H_u^{-1} H_z \, \text{d}\mathrm{Z} \, \big) + M_z \, \text{d}\mathrm{Z} \, .$$
Recall that $H_u$ and $H_z$ are sequence-space Jacobians, computed as:
1. $ H_u = \frac{\partial H}{\partial \mathrm{U}} = \frac{\partial H}{\partial X} \frac{\partial X}{\partial \mathrm{U}}$
2. $ H_z = \frac{\partial H}{\partial \mathrm{Z}} = \frac{\partial H}{\partial X} \frac{\partial X}{\partial \mathrm{Z}}$

The solution of the model immediately gives us the structural IRFs to the $j^{th}$ shock of interest:
$$ \Theta_{\cdot,j,\cdot} = M_u \big( - H_u^{-1} H_{z_j} e_1 \, \big) + M_{z_j} \, e_1 \, ,$$
where the indicator vector $e_1 = (1, 0, \dots, 0)'$ specifies a transitory shock occurring at $t=0$, after which the system converges back to its steady state (stability). For $t=1,2,\dots$ the linearized optimality conditions hold in expectation. The sequence-space procedure gives us the $SVMA(\infty)$ representation:
$$ \mathrm{X}_t=\sum_{h=0}^{\infty}\Theta_h\varepsilon_{t-h} \, . $$

## Structural VAR
The second assignment (__ECON210C__ > __HW2__) required an empirical analysis of the assumption of money neutrality. Students were asked to use structural vector autoregressive (SVAR) models to study the response of real activity and inflation to an exogenous monetary policy shock. The identification strategies are the Cholesky decomposition of [Sims](#ref-sims1980) ([1980](#ref-sims1980)) and the approach of [Romer and Romer](#ref-romer2004) ([2004](#ref-romer2004)).

### Cholesky Decomposition
Let $Y_t = (y_{1t},\dots,y_{kt})'$ be a vector of macroeconomic variables. The structural VAR of order p is given by:
$$
B_0Y_t=B_1Y_{t-1}+\dots+B_pY_{t-p}+\varepsilon_t\,,
$$
where $\varepsilon_t$ is the $k$-dimensional vector of structural shocks and $B_0$ is the $(k\times k)$ matrix governing the contemporaneous relations among the endogenous variables. If we multiply both sides of the SVAR by $B_0^{-1}$, we obtain the reduced-form VAR representation:
$$ Y_t=B_0^{-1}B_1Y_{t-1}+\dots+B_0^{-1}B_pY_{t-p}+B_0^{-1}\varepsilon_t $$
$$ Y_t=A_1Y_{t-1}+\dots+A_pY_{t-p}+u_t \;.$$
Therefore, the vector of structural shocks $\varepsilon_t$ is a linear combination of the reduced-form residuals $u_t$: $$\varepsilon_t=B_0u_t \quad\quad \text{with} \quad\quad \varepsilon_t \sim N(0, I_k) \,.$$
To derive the structural impulse response functions $\partial Y_{t+h}/\partial \varepsilon_t = \Theta_h$, we cast the VAR in its infinite-order structural vector moving average (SVMA) representation. First, let's rewrite the VAR(p) in companion form:
$$
\mathbb{\xi}_t= \mathbb{A \xi}_{t-1} + U_t
$$
$$
\xi_t=
\begin{bmatrix}
Y_t \\
Y_{t-1} \\
Y_{t-2} \\
\vdots \\
Y_{t-p+1}
\end{bmatrix}\,,
\quad
U_t=
\begin{bmatrix}
u_t \\
0 \\
0 \\
\vdots \\
0
\end{bmatrix}\,,
$$
$$
\mathbb{A}=
\begin{bmatrix}
A_1 & A_2 & \dots & A_{p-1} & A_p\\
I_k& 0_k &  \dots & 0 &0\\
0 & I_k &  \dots & 0 & 0\\
\vdots&  &  & & \vdots\\
0 & 0 & \dots & I_k & 0
\end{bmatrix}\,.
$$
The coefficients $A_1,\dots,A_p$ in the first block row of $\mathbb{A}$ are estimated by OLS, while the remaining blocks are fixed:
$$
\hat{\mathbb{A}}=\Big(\sum_t \xi_t \xi_{t-1}^{'}\Big)\Big(\sum_t \xi_{t-1} \xi_{t-1}^{'}\Big)^{-1} \; .
$$
Assuming the VAR is stable (all the eigenvalues of $\mathbb{A}$ lie inside the unit circle), we can apply the Wold decomposition theorem and express the companion-form VAR as an infinite-order moving average:
$$
\xi_t=\sum_{h=0}^{\infty}\mathbb{A}^h U_{t-h} \, .
$$
If we pre-multiply both sides by $J=[I_k \quad 0_{k \times k(p-1)}]$, and multiply and divide the right-hand side by $B_0$, we get the $SVMA(\infty)$ representation:
$$
\begin{gather*}
\begin{split}
J\xi_t &=\sum_{h=0}^{\infty}J\mathbb{A}^h J^{'} J U_{t-h} \\
Y_t &=\sum_{h=0}^{\infty}\Phi_h u_{t-h} \\
Y_t &=\sum_{h=0}^{\infty}\Phi_h B_0^{-1} B_0 u_{t-h} \\
Y_t &=\sum_{h=0}^{\infty} \Theta_h \varepsilon_{t-h} \, .
\end{split}
\end{gather*}
$$
Thus, the sequence of structural IRFs is:
$$
\{ \; \Theta_0=\mathrm{I} B_0^{-1}, \Theta_1=\Phi_1 B_0^{-1},\, \dots, \, \Theta_h = \Phi_h B_0^{-1} \; \} \, .
$$
As mentioned above, $B_0^{-1}$ is estimated with the most commonly used identification method in macroeconomics, the Cholesky decomposition. Introduced by [Sims](#ref-sims1980) ([1980](#ref-sims1980)), it imposes a recursive set of zero restrictions on the contemporaneous coefficients. The recursive structure we impose follows the spirit of [Christiano, Eichenbaum and Evans](#ref-christiano1999) ([1999](#ref-christiano1999)): the slow-moving variables are ordered first and the policy instrument last. In particular, the VAR is estimated on quarterly data from 1960 to 2007 with four lags, and the variables in $Y_t$ are ordered as follows:
$$
\mathrm{Y_t}=\begin{bmatrix}slow\;moving\\[0.5em]\hline\\[-0.8em]
slow\;moving\\[0.5em]\hline\\[-0.8em]
policy\;instrument
\end{bmatrix}
=\begin{bmatrix} \pi_t\\[0.5em]\hline\\[-0.8em]
u_t \\[0.5em]\hline\\[-0.8em]
i_t
\end{bmatrix}\, ,
$$
where $\pi_t$ is inflation (the annual growth rate of the GDP deflator), $u_t$ is the unemployment rate, and $i_t$ is the federal funds rate. This recursive structure implies that inflation and unemployment do not react to a monetary policy shock within the quarter, while the federal funds rate responds contemporaneously to all the shocks hitting the system. This identification strategy imposes a lower-triangular structure on the matrix $B_0$ and, consequently, $B_0^{-1}$ is also lower triangular. The monetary policy shock corresponds to its last column:
$$
B_0^{-1} = \begin{bmatrix}
b_{11} & 0 & \textcolor{orange}{0} \\
b_{21} & b_{22} & \textcolor{orange}{0} \\
b_{31} & b_{32} & \textcolor{orange}{b_{33}} \\
\end{bmatrix} \; .
$$
The matrix is derived by applying the Cholesky decomposition to the variance-covariance matrix of the residuals:
$$ \mathrm{E}[\varepsilon_t \varepsilon_t'] = \mathrm{E}[B_0 u_t u_t' B_0'] $$
$$\mathrm{I}_{k} = B_0 \Sigma_u B_0' $$
$$ \Sigma_u = B_0^{-1} (B_0^{-1})' $$

The function __ECON210C__ > __HW2__ > __SVAR.jl__ > __IRF_CH__ performs the identification via the Cholesky decomposition, while __ECON210C__ > __HW2__ > __SVAR.jl__ > __WILD_CH__ implements the wild bootstrap used to compute the confidence intervals.


### Romer and Romer Shock
The function __ECON210C__ > __HW2__ > __SVAR.jl__ > __IRF_RR__ estimates the structural IRFs using the monetary policy shock and the procedure of [Romer and Romer](#ref-romer2004) ([2004](#ref-romer2004)).

[Romer and Romer](#ref-romer2004) ([2004](#ref-romer2004)) use narrative records of FOMC meetings to derive a series of intended changes in the federal funds rate. They then regress the intended change on the Federal Reserve's Greenbook forecasts of real output growth, inflation and unemployment, and take the residual of this regression, which "shows changes in the intended funds rate not taken in response to information about future economic developments. The resulting series for monetary policy shocks should be relatively free of both endogenous and anticipatory actions" ([Romer and Romer, 2004](#ref-romer2004)).

The procedure for the estimation of the structural IRFs is the following:
1. Given the external monetary policy instrument $Z_t$, we estimate the model in log differences ($\Delta Y_t$):
$$ \Delta Y_t = C + \sum_{i=1}^{p} A_i \Delta Y_{t-i} + \sum_{j=1}^{s} \gamma_j Z_{t-j} + e_t \, ;$$
2. Assuming that the system was at its steady state at time $t$, we compute: $$ \theta_{t+h} = \gamma_h + A_1 \theta_{t+h-1} + \dots + A_p \theta_{t+h-p} \, ;$$
3. Lastly, the structural IRFs of the variables in logs are obtained as $$\Theta_h = \sum_{j=1}^{h} \theta_{t+j} \; .$$


## Undetermined Coefficients
The fourth assignment (__ECON210C__ > __HW4__) required solving the canonical log-linearized New Keynesian DSGE model for a TFP shock using the method of undetermined coefficients. In this section, we walk through the entire solution process.

The core of the benchmark three-equation New Keynesian model is:
$$
\begin{gather*}
\begin{split}
(AD) \quad \hat y_t &=  E_t[\,\hat y_{t+1}\,] - \sigma \big(\, \hat i_t - E_t[\,\hat \pi_{t+1}\,]\, \big) \\
(AS) \quad \hat \pi_t &= \kappa \big(\, \hat y_t - \hat y_t^f \,\big) + \beta E_t[\,\hat \pi_{t+1}\,] \\
(MP) \quad \hat i_t & = \phi_{\pi} \hat \pi_t + \nu
\end{split}
\end{gather*}
$$
where $\sigma = 1/\gamma$ is the elasticity of intertemporal substitution, the flexible-price level of output is $$\hat y^f_t = \frac{1+\varphi}{\varphi + \gamma}\hat a_t = \Omega \hat a_t \, ,$$ the TFP shock follows an AR(1) process $$\hat a_t = \rho_a \hat a_{t-1} + \varepsilon^a_t\,,$$ and the monetary policy shock is set to zero, $\nu = 0$. The other equations of interest are:
$$
\begin{gather*}
\begin{split}
(LD)& \quad \hat n_t =  \hat y_t - \hat a_t\\
(OG)& \quad \tilde y_t =  \hat y_t - \hat y_t^f\\
(RR)& \quad \hat r_t = \hat i_t - E_t[\,\hat \pi_{t+1}\,]\\
(NR)& \quad \hat r^n_t = \frac{1}{\sigma} \frac{1+\varphi}{\varphi + \gamma} \big(\, E_t[\,\hat a_{t+1}\,] - \hat a_t \,\big) \, .
\end{split}
\end{gather*}
$$
The method of undetermined coefficients can be described in five steps:
1. __Guess__. We conjecture the form of the stationary solution of the dynamic system. In our case, we guess that the policy functions are linear in the TFP shock:
$$
\begin{gather*}
\begin{split}
\hat y_t &=  \psi_{ya} \hat a_t \\
\hat \pi_t &=  \psi_{\pi a} \hat a_t \\
E_t[\,\hat y_{t+1}\,] &= \psi_{ya} E_t[\,\hat a_{t+1}\,] = \psi_{ya} \, \rho_a \, \hat a_t \\
E_t[\,\hat \pi_{t+1}\,] &= \psi_{\pi a} E_t[\,\hat a_{t+1}\,] = \psi_{\pi a} \, \rho_a \, \hat a_t
\end{split}
\end{gather*}
$$
2. __Plug into the system__. We replace the variables in the aggregate demand and aggregate supply equations with the linear policy functions. Make sure to plug the monetary policy rule into the aggregate demand equation to eliminate the interest rate $\textcolor{orange}{\hat i_t}$, and the flexible-price level of output $\textcolor{red}{\hat y^f_t}$ into the aggregate supply equation. We get a system of two equations in two unknowns ($ \psi_{ya}$ and $ \psi_{ \pi a}$):
$$
\begin{gather*}
\begin{split}
(AD) \quad \psi_{ya} \hat a_t &=  \psi_{ya} \, \rho_a \, \hat a_t \\
&\quad - \sigma \big(\, \textcolor{orange}{\phi_{\pi} \psi_{\pi a} \hat a_t} - \psi_{\pi a} \, \rho_a \, \hat a_t \, \big) \\
(AS) \quad \psi_{\pi a} \hat a_t &= \kappa \big(\, \psi_{ya} \hat a_t - \textcolor{red}{\Omega \hat a_t } \,\big) \\
&\quad + \beta \psi_{\pi a} \, \rho_a \, \hat a_t
\end{split}
\end{gather*}
$$
3. __Solve__. Solve the system for a unit shock to TFP ($\hat a_t = \varepsilon^a_t = 1$):
$$
\begin{cases}
\psi_{ya} = \Lambda_a \, \Omega \, \sigma \, \kappa \big(\,\phi_{\pi} - \rho_a  \,\big)\\
\psi_{\pi a} = \frac{\kappa^2}{1- \beta \rho_a}\, \Omega \, \Lambda_a \, \sigma \,\big(\, \phi_{\pi} - \rho_a \big) - \frac{\kappa}{1- \beta \rho_a}\,\Omega
\end{cases}
$$
$$\Lambda_a = [ (\, 1 - \rho_a \,) (1 - \beta \rho_a) + \sigma \, \kappa \, (\phi_{\pi} - \rho_a) ]^{-1}$$
4. __Calibration__. We assign numerical values to the model's structural parameters
$$ \{\beta,\kappa,\rho_a, \varphi, \sigma, \gamma, \phi_{\pi}\} $$
based on external information or by matching moments of the data.
5. __Impulse responses__. Given the log-linear approximation around the steady state, we can express the responses of the endogenous variables as linear functions of the exogenous TFP shock $\varepsilon_0^a = 1$:
$$
\begin{bmatrix}
\hat y_t \\
\hat y^f_t \\
\tilde y_t \\
\hat \pi_t \\
\hat i_t \\
\hat r_t \\
\hat r^n_t \\
\hat n_t \\
\hat a_t
\end{bmatrix} =
\begin{bmatrix}
\psi_{ya} \\
(1 + \varphi)(\varphi + \gamma)^{-1}\\
\psi_{ya} - (1 + \varphi)(\varphi + \gamma)^{-1}\\
\psi_{\pi a}\\
\psi_{\pi a} \phi_{\pi} \\
\psi_{\pi a} \phi_{\pi} - \psi_{\pi a} \rho_{a} \\
\sigma^{-1} (1 + \varphi)(\varphi + \gamma)^{-1} (\rho_a - 1) \\
\psi_{ya} - 1 \\
1
\end{bmatrix} \, \big(\, \rho_a^t \, \varepsilon_0^a \,\big)
$$

The function __ECON210C__ > __HW4__ > __RANK_undetermined.jl__ solves the New Keynesian model for TFP and monetary policy shocks. The same procedure is applied in __ECON210C__ > __Exam__ > __RANK_finalexam.jl__ by the function __IID_shock__ for an i.i.d. monetary policy shock.

## References

<div class="references">

<span id="ref-auclert2021"></span>**Auclert, Adrien, Bence Bardóczy, Matthew Rognlie, and Ludwig Straub.** 2021. “Using the sequence-space Jacobian to solve and estimate heterogeneous-agent models.” *Econometrica*, 89(5): 2375–2408.

<span id="ref-christiano1999"></span>**Christiano, Lawrence J, Martin Eichenbaum, and Charles L Evans.** 1999. “Monetary policy shocks: What have we learned and to what end?” In *Handbook of Macroeconomics*. Vol. 1A, ed. John B Taylor and Michael Woodford, 65–148. Amsterdam: Elsevier.

<span id="ref-romer2004"></span>**Romer, Christina D, and David H Romer.** 2004. “A new measure of monetary shocks: Derivation and implications.” *American Economic Review*, 94(4): 1055–1084.

<span id="ref-sims1980"></span>**Sims, Christopher A.** 1980. “Macroeconomics and reality.” *Econometrica*, 48(1): 1–48.

</div>
