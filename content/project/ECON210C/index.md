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

The repository [ECON210C](https://github.com/LapoBini/210C) contains the solutions to assignments and final exam for ECON210C of Prof Johannes Wieland (Spring 2024). The course covered different solution methods for RBC and NK DSGE models, and identification strategies for structural VAR. 

## Sequence Space
The first assignment (__ECON210C__ > __HW1__) required to solve a RBC DSGE model using the Sequence Space methodology ($\textcolor{orange}{Auclert}$ $\textcolor{orange}{et}$ $\textcolor{orange}{al.,}$ $\textcolor{orange}{2021}$). The code in __ECON210C__ > __HW1__ > __RBC.jl__ solves a specific RBC model with a composite basket made of consumption good and money. In this paragraph we provide a general overview of how the sequence space procedure works. The basic idea is to organize the GE model into blocks that represents different behaviors of possible heterogeneous agents and interact in GE via a small set of aggregates. 

Let's start by considering the definition of equilibrium for a basic RBC model. The equilibium is defined as an allocation $\{C_{t+s},N_{t+s},Y_{t+s},B_{t+s}\}_{s=0}^{\infty}$, a set of prices $\{W_{t+s},P_{t+s},Q_{t+s}\}_{s=0}^{\infty}$, an exogenous processes $\{A_{t+s},Tr_{t+s},M_{t+s}\}_{s=0}^{\infty}$ and initial conditions for bonds and capital such that:
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

Let's now denote with $\mathrm{X} = \{X_1,\dots,X_t\}$ the perfect-foresight time sequence of aggregate macroeconomic outcomes, and with $\mathrm{Z} = \{Z_1,\dots,Z_t\}$ the sequence of exogenous shocks. A perfect-foresight equilibrium given shock sequence $\mathrm{Z}$ is the set of paths $\mathrm{X}$ such that:
$$ F(\mathrm{X}, \mathrm{Z}) = 0 $$
where $F(\cdot)$ embeds the model's equilibrium relationships highlighted before. Then, we can define a mapping from $\mathrm{Z}$ to $\mathrm{X}$ by taking total derivative:
$$ \text{d}F(\mathrm{X}, \mathrm{Z}) = \text{d}0 $$
$$ F_x \, \text{d}\mathrm{X} + F_z \, \text{d}\mathrm{Z} = 0 $$
However, we often times can arrive at a lower-dimensional representation of equilibria that does not involve all variables: given $\mathrm{Z}$, we may be able to characterize equilibria via
$$ H(\mathrm{U}, \mathrm{Z}) = 0 $$
where $\mathrm{U} = \{U_1,\dots,U_t\}$ is the time path of some unknown macroeconomic outcomes with $ n_u < n_x$. Thus, $\mathrm{X}$ is determined residually via 
$$ \mathrm{X}= M(\mathrm{U}, \mathrm{Z}) \; .$$
The number of unknown endogenous variables ($n_u$) in $\mathrm{U}$ must be the same as the number of equilibrium condition in $H(\cdot)$ (in the code __ECON210C__ > __HW1__ > __RBC.jl__ only one equilibrium condition, bond demand equation, and one endogenous variable, labor supply). To first order we can write:
$$ \text{d}H(\mathrm{U}, \mathrm{Z}) = \text{d}0 $$
$$ \text{d}H(\mathrm{U}, \mathrm{Z}) = H_u \, \text{d}\mathrm{U} + H_z \, \text{d}\mathrm{Z} = 0 $$
$$ \text{d}\mathrm{U} = - H_u^{-1} H_z \, \text{d}\mathrm{Z} \, .$$
Thus, we derive the solution of the model as follows:
$$ \text{d}\mathrm{X} = M_u \, \text{d}\mathrm{U} + M_z \, \text{d}\mathrm{Z} $$
$$ \text{d}\mathrm{X} = M_u \big( - H_u^{-1} H_z \, \text{d}\mathrm{Z} \, \big) + M_z \, \text{d}\mathrm{Z} \, .$$
Remember that $H_u$ and $H_z$ are sequence-space Jacobians, which are computed as:
1. $ H_u = \frac{\partial H}{\partial \mathrm{U}} = \frac{\partial H}{\partial X} \frac{\partial X}{\partial \mathrm{U}}$
2. $ H_z = \frac{\partial H}{\partial \mathrm{Z}} = \frac{\partial H}{\partial X} \frac{\partial X}{\partial \mathrm{Z}}$

The solution of the model gives us immediately the structural IRFs to the $j^{th}$ shock of interest:
$$ \Theta_{\cdot,j,\cdot} = M_u \big( - H_u^{-1} H_{z_j} e_1 \, \big) + M_{z_j} \, e_1 \, .$$
where the indicatior vector $e_1 = (1, 0, \dots, 0)'$ specifies a transitory shock happening at $t=0$ and then the system converges back to steady state (stability). For $t=1,2,\dots$ the linearized optimality conditions hold in expectation. The sequence space procedure has given us the $SVMA(\infty)$ representation:
$$ \mathrm{X}_t=\sum_{h=0}^{\infty}\Theta_h\varepsilon_{t-h} \, . $$

## Structural VAR
The second assignment (__ECON210C__ > __HW2__) asked to perform an empirical analysis to study the assumption of money neutrality. The students were asked to use Structural Vector Autoregressive (SVAR) models to study the response of real activities and inflation to an exogenous monetary policy shock. The identification strategies used are the Cholesky decomposition of $\textcolor{orange}{Sims}$ $\textcolor{orange}{(1980)}$ and the approach of $\textcolor{orange}{Romer}$ $\textcolor{orange}{and}$ $\textcolor{orange}{Romer}$ $\textcolor{orange}{(2004)}$.

### Cholesky Decomposition 
Let $Y_t = (y_{1t},\dots,y_{kt})'$ be a vector of macroeconomic variables, the structural VAR of order p is given by:
$$
B_0Y_t=B_1Y_{t-1}+\dots+B_pY_{t-p}+\varepsilon_t.
$$
where $\varepsilon_t$ is the $k$-dimensional vector of structural shocks and $B_0$ is the $(k\text{x}k)$ matrix governing the contemporaneous relations among endogenous variables. If we multiply both sides of the SVAR by $B_0^{-1}$ we obtain the reduced form VAR representation: 
$$ Y_t=B_0^{-1}B_1Y_{t-1}+\dots+B_0^{-1}B_pY_{t-p}+B_0^{-1}\varepsilon_t $$
$$ Y_t=A_1Y_{t-1}+\dots+A_pY_{t-p}+u_t \;.$$
Theerefore, the vector of structural shocks $\varepsilon_t$ can be seen as the linear combination of the reduced form residuals $u_t$: $$\varepsilon_t=B_0u_t \quad\quad \text{with} \quad\quad \varepsilon_t \sim N(0, I_k) \,.$$
To derive the formulation of the structural impulse response functions $\partial Y_{t+h}/\partial \varepsilon_t = \Theta_j$, we are going to cast the VAR in its infinite-order structural vector moving average (SVMA) representation. First, let's rewrite the VAR(p) in companion form: 
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
\end{bmatrix}\,
\mathbb{A}=
\begin{bmatrix}
A_1 & A_2 & \dots & A_{p-1} & A_p\\
I_k& 0_k &  \dots & 0 &0\\
0 & I_k &  \dots & 0 & 0\\
\vdots&  &  & & \vdots\\
0 & 0 & \dots & I_k & 0
\end{bmatrix}\,
U_t=
\begin{bmatrix}
u_t \\
0 \\
0 \\
\vdots \\
0
\end{bmatrix}\,.
$$
The matrix $\mathbb{A}$ is estimated by OLS:
$$
\mathbb{A}=(\xi_t \xi_{t-1}^{'})(\xi_{t-1} \xi_{t-1}^{'})^{-1} \; .
$$
Since our stochastic process $\{\xi_t\}_{t\geq0}$ is weakly stationary by construction, we apply the Wold Decomposition Theorem to express the companion form VAR as an infinite-order moving average:
$$
\mathbb{Y}_t=\sum_{h=0}^{\infty}\mathbb{A}^h U_{t-h} \, .
$$
If we pre-multiply both sides by $J=[I_k \quad 0_{k \text{x} k(p-1)}]$ and we multiply and divide the right-hand side by $B_0$ we get the $SVMA(\infty)$ representation:
$$
\begin{gather*}
\begin{split}
J\mathbb{Y}_t &=\sum_{h=0}^{\infty}J\mathbb{A}^h J^{'} J U_{t-h} \\
Y_t &=\sum_{h=0}^{\infty}\Phi_h u_{t-h} \\
Y_t &=\sum_{h=0}^{\infty}\Phi_h B_0^{-1} B_0 u_{t-h} \\
Y_t &=\sum_{h=0}^{\infty} \Theta_h \varepsilon_{t-h} \, .
\end{split}
\end{gather*}
$$
Thus, the sequence of structural IRFs is defined as:
$$
\{ \; \Theta_0=\mathrm{I} B_0^{-1}, \Theta_1=\Phi_1 B_0^{-1},\, \dots, \, \Theta_h = \Phi_h B_0^{-1} \; \} \, .
$$
As mentioned before, the estimation of $B_0^{-1}$ is performed using the most commonly used identification method in macroeconomics, the Cholesky Decocomposition. It was introduced by $\textcolor{orange}{Sims}$ $\textcolor{orange}{(1980)}$, and it imposes alternative sets of recursive zero restrictions on the contemporaneous coefficients. The recursive structure that we impose follows the spirit of $\textcolor{orange}{Christiano}$ $\textcolor{orange}{et}$ $\textcolor{orange}{al.}$ $\textcolor{orange}{(1999)}$. In particular, the variables in $Y_t$ are ordered as follows:
$$
\mathrm{Y_t}=\begin{bmatrix}slow\;moving\\[0.5em]\hline\\[-0.8em]
policy\;instrument\\[0.5em]\hline\\[-0.8em]
fast\;moving
\end{bmatrix}
=\begin{bmatrix} \ln GDP_t\\[0.5em]\hline\\[-0.8em]
ffr_t \\[0.5em]\hline\\[-0.8em]
\ln CPI_t
\end{bmatrix}\, .
$$
The recursive structure implies that the slow moving variable, log of real GDP, does not react to a monetary policy shock within the period (it only reacts to a shock to GDP itself), while the fast moving one does react to all the different shocks hitting the system. This identification strategy enforces a lower triangular structure on the matrix $B_0$, and, consequently, $B_0^{-1}$ will also be lower triangular:
$$
B_0^{-1} = \begin{bmatrix}
b_{11} & \textcolor{orange}{0} & 0 \\
b_{21} & \textcolor{orange}{b_{22}} & 0 \\
b_{31} & \textcolor{orange}{b_{32}} & b_{33} \\
\end{bmatrix} \; .
$$
The matrix is derived by applying the Cholesky Decomposition to the variance/covariance matrix of the residuals:
$$ \mathrm{E}[\varepsilon_t \varepsilon_t'] = \mathrm{E}[B_0 u_t u_t' B_0'] $$
$$\mathrm{I}_{k} = B_0 \Sigma_u B_0' $$
$$ \Sigma_u = B_0^{-1} B_0^{-1} $$

The function __ECON210C__ > __HW2__ > __SVAR.jl__ > __IRF_CH__ performs the identification via Cholesky Decomposition, while __ECON210C__ > __HW2__ > __SVAR.jl__ > __WILD_CH__ implements the wild bootstrap routine to compute the confidence intervals. 


### Romer \& Romer Shock
The function __ECON210C__ > __HW2__ > __SVAR.jl__ > __IRF_RR__ estimates the structural IRFs using the monetary policy shock and the procedure of $\textcolor{orange}{Romer}$ $\textcolor{orange}{and}$ $\textcolor{orange}{Romer}$ $\textcolor{orange}{(2004)}$. 

Romer and Romer use narrative evidence to derive a series of intended changes in the federal funds rate (using the FOMC announcments from the 1990s, before that, the two had to trawl through meeting minutes to figure it out). They then regress the change in the intended FFR on the Greenbook forecast of RGDP growth, inflation and unemployment and take the residual from this regression, which "shows changes in the intended funds rate not taken in response to information about future economic developments. The resulting series for monetary policy shocks should be relatively free of both endogenous and anticipatory actions."

The procedure for the estimation of the structural IRF is the following:
1. Given the external monetary policy instrument $Z_t$, we estimate the model in log difference ($\Delta Y_t$):
$$ \Delta Y_t = C + \sum_{i=1}^{p} A_i \Delta Y_{t-i} + \sum_{j=1}^{s} \gamma_j Z_{t-j} + e_t \, ;$$
2. Assuming that the system was at steady state at time $t$, we compute: $$ \theta_{t+h} = \gamma_h + A_1 \theta_{t+h-1} + \dots + A_p \theta_{t+h-p} \, .$$
3. Lastly, the structural IRFs of the log of the variables are obtained as $$\Theta_h = \sum_{j=1}^{h} \theta_{t+j} \; .$$


## Undetermined Coefficients
The third assignment (__ECON210C__ > __HW3__) asked to solve the canonical log-linearized New-Keynesian DSGE model as a function of a TFP shock using the undetermined coefficient method. In this In this section, we will walk through the entire solution process.

The core of the benchmark three-equation New-Keynesian model is:
$$
\begin{gather*}
\begin{split}
(AD) \quad \hat y_t &=  E_t[\,\hat y_{t+1}\,] - \sigma \big(\, \hat i_t - E_t[\,\hat \pi_{t+1}\,]\, \big) \\
(AS) \quad \hat \pi_t &= \kappa \big(\, \hat y_t - \hat y_t^f \,\big) + \beta E_t[\,\hat \pi_{t+1}\,] \\
(MP) \quad \hat i_t & = \phi_{\pi} \hat \pi_t + \nu
\end{split}
\end{gather*}
$$
where the output level of flexible prices is $$\hat y^f_t = \frac{1+\varphi}{\varphi + \gamma}\hat a_t = \Omega \hat a_t \, ,$$ the TFP shock follows an AR(1) process $$\hat a_t = \rho_a \hat a_{t-1} + \varepsilon^a_t$$ and the monetary policy shock is assumed to be zero $\nu = 0$. The other equations we are interested in are: 
$$
\begin{gather*}
\begin{split}
(LD)& \quad \hat n_t =  \hat y_t - \hat a_t\\
(OG)& \quad \tilde y_t =  \hat y_t - \hat y_t^f\\
(RR)& \quad \hat r_t = \hat i_t - E_t[\,\hat \pi_{t+1}\,]\\
(NR)& \quad \hat r^n_t = \sigma \frac{1+\varphi}{\varphi + \gamma} \big(\, \beta E_t[\,\hat a_{t+1}\,] - \hat a_t \,\big) \, .
\end{split}
\end{gather*}
$$
The undetermined coefficient procedure can be described in five steps:
1. __Guess__. We conjecture about the stationary solution of the dynamical system. In our case, we guess the policy function being linear in the TFP shock:
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
2. __Plug into the system__: We replace the variables in the aggregate demand and aggregate supply equation with the linear policy functions. Make sure to plug the monetary policy rule in the aggregate demand to get rid of the interest rate $\textcolor{orange}{\hat i_t}$, and the output level of flexible prices $\textcolor{red}{\hat y^f_t}$ in the aggregate supply. We get a system of two equations in two unknowns ($ \psi_{ya}$ and $ \psi_{ \pi a}$):
$$
\begin{gather*}
\begin{split}
(AD) \quad \psi_{ya} \hat a_t &=  \psi_{ya} \, \rho_a \, \hat a_t - \sigma \big(\, \textcolor{orange}{\phi_{\pi} \psi_{\pi a} \hat a_t} - \psi_{\pi a} \, \rho_a \, \hat a_t \, \big) \\
(AS) \quad \psi_{\pi a} \hat a_t &= \kappa \big(\, \psi_{ya} \hat a_t - \textcolor{red}{\Omega \hat a_t } \,\big) + \beta \psi_{\pi a} \, \rho_a \, \hat a_t
\end{split}
\end{gather*}
$$
3. __Solve__. Solve the system relative to a unit shock to TFP ($\hat a_t = \varepsilon^a_t = 1$): 
$$
\begin{cases}
\psi_{ya} = \Lambda_a \, \Omega \, \sigma \, \kappa \big(\,\phi_{\pi} - \rho_a  \,\big)\\
\psi_{\pi a} = \frac{\kappa^2}{1- \beta \rho_a}\, \Omega \, \Lambda_a \, \sigma \,\big(\, \phi_{\pi} - \rho_a \big) - \frac{\kappa}{1- \beta \rho_a}\,\Omega
\end{cases}
$$
$$\Lambda_a = [ (\, 1 - \rho_a \,) (1 - \beta \rho_a) + \sigma \, \kappa \, (\phi_{\pi} - \rho_a) ]^{-1}$$
4. __Calibration__. We assign numerical values to the model’s structural parameters
$$ \{\beta,\kappa,\rho_a, \varphi, \sigma, \gamma, \phi_{\pi}\} $$
based on external information or matching moments of the data.
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
\sigma (1 + \varphi)(\varphi + \gamma)^{-1} (\beta \rho_a - 1) \\
\psi_{ya} - 1 \\
1
\end{bmatrix} \, \big(\, \rho_a^t \, \varepsilon_0^a \,\big) 
$$

The function __ECON210C__ > __HW4__ > __RANK_undetermined.jl__ solves the New-Keynesian model as a function of TFP and monetary policy shocks. The same procedure is applied in __ECON210C__ > __Exam__ > __RANK_finalexam.jl__ by the function __IID_shock__ with respect to an iid monetary policy shock. 


