---
date: "2024-09-01"
#image:
#  caption: Embed rich media such as videos and LaTeX math
math: false
summary: BVAR Estimation and Structural Analysis
code: ECON220D
repo: "https://github.com/LapoBini/BayesianVAR.jl"
weight: 1
abstract: "Part of the teaching material for ECON220D (Time Series Econometrics and Applied Macroeconomics, Prof. James D. Hamilton), the mandatory second-year PhD course at UC San Diego. The Julia code estimates Bayesian VARs with Minnesota, sum-of-coefficients and dummy-initial-observation priors, selects the lag length and the prior tightness by maximizing the marginal likelihood, and identifies structural shocks through sign restrictions or external instruments."
title: Bayesian Vector Autoregression
type: docs
---
## Introduction
The repository [BayesianVAR.jl](https://github.com/LapoBini/BayesianVAR.jl) contains the code to estimate a Bayesian VAR with Minnesota, sum-of-coefficients and dummy-initial-observation priors, and to perform structural analysis using sign restrictions or external instruments. It is part of the teaching material for ECON220D, Time Series Econometrics and Applied Macroeconomics, the mandatory second-year PhD course taught by Prof. James D. Hamilton at UC San Diego.

The repository includes an Excel file (__Data__ > __FinalData__ > __Data.xlsx__) with the data of the baseline model in [Bini](#ref-bini2025) ([2025](#ref-bini2025)), which can be used to test and run the model with __main_executer.jl__. The settings for the VAR (sample dates, location of the data file, name of the results folder, required packages, etc.) can be changed in __Code__ > __SetUp__ > __SetupBVAR.jl__. The structure of the Excel file is explained in [The Excel File](#the-excel-file), at the end of this page. Every function contains a brief description of the econometric framework, with equations and references.

## Setting the Priors
Let $Y_t = (y_{1t},\dots,y_{nt})'$ be a potentially large vector of random variables. We estimate the following VAR(p) model:
{{< math >}}
$$Y_t = c + A_1 Y_{t-1}+\dots+A_p Y_{t-p} + u_t$$
{{< /math >}}
using the Bayesian approach, which overcomes the curse of dimensionality by imposing prior beliefs on the parameters. In setting the prior distributions, we follow standard practice and use the procedure developed in [Litterman](#ref-litterman1986) ([1986](#ref-litterman1986)), with the modifications proposed by [Kadiyala and Karlsson](#ref-kadiyala1997) ([1997](#ref-kadiyala1997)) and [Sims and Zha](#ref-sims1998) ([1998](#ref-sims1998)). The priors are implemented using dummy observations, and the informativeness of the prior distribution is chosen by maximizing the marginal likelihood of the observed data. In this respect, we closely follow the exposition and the procedure of [Bańbura, Giannone and Reichlin](#ref-banbura2010) ([2010](#ref-banbura2010)) and [Giannone, Lenza and Primiceri](#ref-giannone2015) ([2015](#ref-giannone2015)).



### Minnesota Prior
[Litterman](#ref-litterman1986) ([1986](#ref-litterman1986)) introduced the well-known Minnesota prior. Its basic principle is that all the equations are centered around a random walk with drift:
$$ Y_t = c + Y_{t-1} + u_t\,.$$
This amounts to shrinking the diagonal elements of $A_1$ toward one and the remaining coefficients in $A_1,\dots,A_p$ toward zero. In addition, the prior incorporates the beliefs that more recent lags carry more reliable information than more distant ones, and that own lags explain more of the variation of a given variable than the lags of other variables in the equation.

Assuming the coefficients in $\beta = [A_1 \;A_2 \;\dots \;A_p]$ to be a priori independent and normally distributed, the prior beliefs are imposed by setting the following moments for the prior distribution of the coefficients:
{{< math >}}
$$E[(A_k)_{ij}] = \begin{cases}
  \delta_i \quad \text{if} \;\; j=i,\; k=1 \\
  0 \quad\; \text{otherwise}
\end{cases}\;.$$
{{< /math >}}
[Litterman](#ref-litterman1986) ([1986](#ref-litterman1986)) originally sets $\delta_i = 1$ for all $i$, reflecting the belief that all the variables are highly persistent. However, this prior is not appropriate for variables displaying substantial mean reversion. For those, we impose the prior belief of white noise by setting $\delta_i = 0$. The variance of the coefficients is:
{{< math >}}
$$Var((A_k)_{ij}) =  \begin{cases}
\frac{\lambda^2}{k^2} \quad\quad\quad \text{if}\; i=j \\ \vartheta\frac{\lambda^2}{k^2}\frac{\sigma_i^2}{\sigma_j^2} \quad\text{ otherwise}
\end{cases}\;,$$
{{< /math >}}
where $k$ is the lag, so the prior tightens as the lag increases. In the original formulation of the Minnesota prior, the covariance matrix of the residuals is assumed to be diagonal and known: $\Psi = \Sigma$, where $\Sigma = diag(\sigma_1^2,\;\dots, \sigma_n^2)$. However, structural analysis requires taking into account the possible correlation among the residuals of different variables. To overcome this problem, we follow [Kadiyala and Karlsson](#ref-kadiyala1997) ([1997](#ref-kadiyala1997)) and impose a Normal-inverse-Wishart prior, which retains the principles of the Minnesota prior. This is possible under the condition that $\vartheta = 1$, which we assume in what follows. Finally, the prior on the intercept $c$ is diffuse.

The Minnesota prior is implemented in __Code__ > __BVAR__ > __bvar_minnesota_prior.jl__. First, the function shrinks each variable toward a random walk or white noise, as specified in the column "PRIOR AR(1)" of the [worksheet "Legend"](#legend); then, it calculates $\{\sigma^2_1,\dots,\sigma^2_n\}$.

### Sum-of-Coefficients Prior
The literature has suggested that forecasting performance can be improved by imposing additional priors that constrain the sum of the coefficients (see, e.g., [Sims, 1993](#ref-sims1993); [Sims and Zha, 1998](#ref-sims1998); [Robertson and Tallman, 1999](#ref-robertson1999)). This is the same as imposing "inexact differencing", and it is a simple modification of the Minnesota prior involving linear combinations of the VAR coefficients.

To see this, we rewrite the VAR in its error-correction form: $$Y_t = c + A_1 Y_{t-1}+\dots+A_p Y_{t-p} + u_t$$  $$\begin{aligned}\Delta Y_t = {} & c - (I_n - A_1 - A_2- \dots- A_p) Y_{t-1} \\ & + C_1 \Delta Y_{t-1} + \dots + C_{p-1} \Delta Y_{t-p+1} + u_t\end{aligned}$$ with $C_k = - \sum_{j=k+1}^p A_j$. The term $- (I_n - A_1 - A_2- \dots- A_p) Y_{t-1}$ is the error-correction term: it corrects the current-period change in $Y_t$ with part of the previous period's deviation from the long-run relation among the variables. The remaining terms, $C_1 \Delta Y_{t-1} + \dots + C_{p-1} \Delta Y_{t-p+1} + u_t$, involve only first differences.

A VAR in first differences implies $\Pi = I_n - A_1 - A_2- \dots- A_p = 0$. By inexact differencing, that is, by imposing the sum-of-coefficients prior, we shrink $\Pi$ toward zero, so that the posterior draws are characterized by unit roots rather than by explosive behavior. In this case, $Y_t$ is equal to its previous value plus terms in first differences. It follows that, in the limit ($\Pi \to 0$), each variable in $Y_t$ contains a unit root and the model rules out cointegration.

### Dummy-Initial-Observation Prior
As mentioned above, the sum-of-coefficients prior rules out cointegration in the limit, which may be undesirable. Starting from $$\begin{aligned}\Delta Y_t = {} & c - (I_n - A_1 - A_2- \dots- A_p) Y_{t-1} \\ & + C_1 \Delta Y_{t-1} + \dots + C_{p-1} \Delta Y_{t-p+1} + u_t\,,\end{aligned}$$ the sum-of-coefficients prior implies, in the limit $\Pi \to 0$: $$\Delta Y_t= c +  C_1 \Delta Y_{t-1} + \dots + C_{p-1} \Delta Y_{t-p+1} + u_t\,,$$ which is equivalent to $$\begin{aligned}Y_t = {} & c + Y_{t-1} \\ & + C_1 \Delta Y_{t-1} + \dots + C_{p-1} \Delta Y_{t-p+1} + u_t\;.\end{aligned}$$
Without an error-correction term, there can be no cointegration relations. For this reason, [Sims](#ref-sims1993) ([1993](#ref-sims1993)) proposed an additional prior, known as the "dummy-initial-observation" prior.

This prior states that a no-change forecast for all variables is a good forecast at the beginning of the sample. Then, either all the variables are at their unconditional mean, which implies that the model is stationary despite the unit roots in the variables (implying cointegration), or the dynamics of the system are characterized by an unspecified number of unit roots, and the variables share a common stochastic trend.

## Implementation of the Priors
### Prior Distribution
Following [Kadiyala and Karlsson](#ref-kadiyala1997) ([1997](#ref-kadiyala1997)) and [Robertson and Tallman](#ref-robertson1999) ([1999](#ref-robertson1999)), we impose a Normal-inverse-Wishart prior distribution of the form:
$$
\begin{gathered}
vec(\beta)|\Psi \sim N(vec(\beta_0), \Psi \otimes \Omega_0 ) \\
\text{and}\quad \Psi \sim iW(S_0, \alpha_0)\,,
\end{gathered}
$$
where the prior parameters $\beta_0$, $\Omega_0$, $S_0$ and $\alpha_0$ are chosen so that the prior expectations and variances of $\beta$ coincide with those implied by the Minnesota prior, and the expectation of $\Psi$ is equal to the fixed residual covariance matrix $\Sigma$. We implement the prior by adding dummy observations: it can be shown that adding $T_d$ dummy observations $Y_d$ and $X_d$ to the VAR(p) is equivalent to imposing the Normal-inverse-Wishart prior with:
* $\beta_0 = (X_d'X_d)^{-1}(X_d'Y_d)$;
* $\Omega_0 = (X_d'X_d)^{-1}$;
* $S_0 = (Y_d - X_d \beta_0)'(Y_d - X_d\beta_0)$;
* $\alpha_0 = T_d - k$,

where $k = np+1$ is the number of regressors in each equation, $S_0$ is the $(n\times n)$ scale matrix and $\alpha_0$ the degrees of freedom of the inverse-Wishart distribution.

### Dummy Observations
We follow [Bańbura, Giannone and Reichlin](#ref-banbura2010) ([2010](#ref-banbura2010)) and implement the prior distribution using dummy observations. The dummy observations $Y_d$ and $X_d$ are constructed as follows:
$$
Y_d = \begin{bmatrix}
        diag(\delta_1\sigma_1,\dots,\delta_n\sigma_n)/\lambda\\
        0_{n(p-1)\times n}\\[0.5em]\hline\\[-0.8em]
        diag(\sigma_1,\dots,\sigma_n)\\[0.5em]\hline\\[-0.8em]
        0_{1\times n}\\[0.5em]\hline\\[-0.8em]
        diag(\mu_1\delta_1,\dots,\mu_n\delta_n)/\tau\\[0.5em]\hline\\[-0.8em]
        (\mu_1,\dots,\mu_n)/\gamma
\end{bmatrix}
$$
$$
X_d = \begin{bmatrix}
        J_p \otimes diag(\sigma_1,\dots,\sigma_n)/\lambda & 0_{np \times 1}\\[0.5em]\hline\\[-0.8em]
        0_{n \times np} & 0_{n \times 1} \\[0.5em]\hline\\[-0.8em]
        0_{1 \times np} & \varepsilon\\[0.5em]\hline\\[-0.8em]
        \mathbf{1}_p'\otimes diag(\mu_1\delta_1,\dots,\mu_n\delta_n)/\tau & 0_{n \times 1} \\[0.5em]\hline\\[-0.8em]
        \mathbf{1}_p'\otimes (\mu_1,\dots,\mu_n)/\gamma & 1/\gamma
\end{bmatrix}
$$
where $J_p = diag(1,2,\dots,p)$ and $\mathbf{1}_p$ is a $p$-dimensional vector of ones. The first block of dummies imposes the prior beliefs on the autoregressive coefficients, the second block implements the prior for the covariance matrix, the third block reflects the uninformative prior for the intercept ($\varepsilon$ is a very small number), the fourth block implements the sum-of-coefficients prior, and the fifth one the dummy-initial-observation prior. The function that constructs the dummy observations can be found in __Code__ > __BVAR__ > __bvar_dummies.jl__.

Notice that the priors are parametrized by additional parameters, known as hyperparameters:
1. $\delta_i$, which takes the value 1 or 0, reflecting the belief that the $i^{th}$ series is highly persistent (random walk) or displays substantial mean reversion (white noise).
2. $\sigma_i$, the standard deviation of the residuals of the random walk or the white noise.
3. $\lambda$, which controls the overall tightness of the prior distribution around the random walk or white noise, and governs the relative importance of the prior beliefs with respect to the information contained in the data. When $\lambda = 0$ the posterior equals the prior, while as $\lambda \to \infty$ the posterior mean approaches the OLS estimate.
4. $\tau$, which controls the degree of shrinkage of the sum-of-coefficients prior: as $\tau$ goes to zero we approach the case of exact differences and, as $\tau$ goes to infinity, we approach the case of no shrinkage.
5. $\mu_i$, which aims at capturing the unconditional mean of variable $y_{it}$.
6. $\gamma$, which controls the tightness of the dummy-initial-observation prior: as it approaches infinity, there is no shrinkage.

How should the hyperparameters be specified? What is the optimal number of lags? What are the best shrinkage values for the prior distribution? The goal is to ensure that the model's prior assumptions align well with the observed data, improving the model's overall performance without overfitting. The procedure to choose their values is illustrated in the next section.

### Informativeness of the Prior Distribution
From a Bayesian perspective, choosing the informativeness of the prior distribution is conceptually identical to inference on any other unknown parameter of the model. Suppose, for instance, that a model is described by a likelihood function $p(y|\theta)$ and a prior distribution $p_\eta(\theta)$, where $\theta$ is the vector of the VAR(p) parameters and $\eta$ collects the hyperparameters, i.e. the coefficients that parameterize the prior distribution but do not directly affect the likelihood. It is then natural to choose these hyperparameters by interpreting the model as a hierarchical model, i.e. replacing $p_\eta(\theta)$ with $p(\theta|\eta)$ and evaluating their posterior ([Berger, 1985](#ref-berger1985); [Koop, 2003](#ref-koop2003)).

By Bayes' rule, the posterior of the hyperparameters is
$$ p(\eta|y) \propto p(y|\eta)\, p(\eta)\,, $$
where $p(\eta)$ is a prior on the hyperparameters (a hyperprior) and $p(y|\eta)$ is the marginal likelihood, obtained by integrating out the VAR parameters:
$$ p(y|\eta) = \int p(y|\theta)\, p(\theta|\eta)\, d\theta\,. $$
Following [Giannone, Lenza and Primiceri](#ref-giannone2015) ([2015](#ref-giannone2015)), with a flat hyperprior, maximizing the posterior of $\eta$ amounts to maximizing the marginal likelihood, i.e. picking the hyperparameters that make the data most likely after integrating out the uncertainty about the model parameters.

Why is this a sensible criterion? By the chain rule of probability, the marginal likelihood can be written as
$$ p(y|\eta) = \prod_{t=1}^{T} p(y_t \,|\, y_1,\dots,y_{t-1}, \eta)\,, $$
the product of one-step-ahead predictive densities. It therefore rewards the hyperparameters that would have produced the best out-of-sample forecasts. A prior that is too loose fits the sample well but forecasts poorly (overfitting), while a prior that is too tight ignores the information in the data.

Because the Normal-inverse-Wishart prior is conjugate, the integral has a closed form. Let $Y$ $(T\times n)$ and $X$ $(T\times k)$ collect the observed data, without the dummy observations, and let $\beta_0$, $\Omega_0$, $S_0$ and $\alpha_0$ be the prior parameters implied by the dummies, all of which depend on $\eta$. The log marginal likelihood is
$$
\begin{aligned}
& \log p(y|\eta) = -\frac{Tn}{2}\log\pi + \log\Gamma_n\Big(\frac{T+\alpha_0}{2}\Big) \\
& \quad - \log\Gamma_n\Big(\frac{\alpha_0}{2}\Big) - \frac{n}{2}\log\big|I_T + X\Omega_0X'\big| \\
& \quad + \frac{\alpha_0}{2}\log\big|S_0\big| - \frac{T+\alpha_0}{2}\log\big|S_0 \\
& \quad + (Y - X\beta_0)'(I_T + X\Omega_0 X')^{-1}(Y - X\beta_0)\big|\,,
\end{aligned}
$$
where $\Gamma_n(\cdot)$ is the multivariate gamma function. The last term rewards fit: it is large when the data are close to the prior mean $X\beta_0$, relative to the prior uncertainty. The term $-\frac{n}{2}\log|I_T + X\Omega_0X'|$ penalizes complexity: it falls as the prior becomes looser or as more lags are added. Equivalently, the marginal likelihood is the ratio between the marginal likelihood of the data augmented with the dummies and that of the dummies alone, $p(y|\eta) = p(Y_*|\eta)/p(Y_d|\eta)$.

The lag order and the hyperparameters are then chosen as
$$ (\hat p, \hat\lambda, \hat\tau, \hat\gamma) = \arg\max_{p,\,\lambda,\,\tau,\,\gamma}\; \log p(y\,|\,p, \lambda, \tau, \gamma)\,. $$
The code performs this maximization by grid search: for each lag order $p$ and each set of values $(\lambda, \tau, \gamma)$ in the grid (by default $\tau = \gamma = 10\lambda$), it builds the dummy observations, computes the log marginal likelihood and keeps the combination with the highest value. To make different lag orders comparable, all of them are evaluated on the same sample, which starts after the largest lag in the grid. The procedure is implemented in __Code__ > __BVAR__ > __bvar_opt_hyperprior.jl__, and the marginal likelihood is computed in __Code__ > __BVAR__ > __bvar_marginal_likelihood.jl__.

## Posterior Distribution
We now augment the data with the dummy observations:
* $Y_* = (Y', Y_d')'$;
* $X_* = (X', X_d')'$;
* $U_* = (U', U_d')'$;

and we rewrite the VAR(p) as a general regression model:
$$Y_* = X_* \,\beta + U_*\,.$$
The posterior distribution has the form:
$$
\begin{gathered}
vec(\beta)|\Psi,Y \sim N(vec(\tilde \beta), \Psi \otimes (X_*' X_*)^{-1} ) \\
\text{and}\quad \Psi|Y \sim iW(\tilde \Sigma, T_d + T + 2 - k)\,.
\end{gathered}
$$
Notice that the posterior expectations of the coefficients correspond to the OLS estimates of the regression model based on the augmented data, with:
* $\tilde \beta = (X_*'X_*)^{-1}(X_*'Y_*)$;
* $\tilde \Sigma = (Y_* - X_* \tilde \beta)'(Y_* - X_* \tilde \beta)$.

For structural analysis, the code draws from the posterior with a Gibbs sampler that alternates between $p(\beta|\Psi,Y)$, the normal distribution above, and $p(\Psi|\beta,Y)$, an inverse-Wishart distribution with scale matrix $(Y_* - X_* \beta)'(Y_* - X_* \beta)$. Draws of $\beta$ for which the VAR is not stable are discarded. The functions that perform the draws can be found in __Code__ > __BVAR__ > __bvar_gibbs_helper.jl__ > __bvar_IWRD__ (inverse Wishart) and __bvar_MNRD__ (multivariate normal).

## Structural Identification
The structural VAR(p) is specified as
$$ B_0Y_t=B_1Y_{t-1}+\dots+B_pY_{t-p}+\varepsilon_t\,.$$
If we multiply both sides of the structural form of the VAR by $B_0^{-1}$, the matrix governing the contemporaneous relations among the endogenous variables, also known as the structural impact multiplier matrix, we obtain the reduced-form representation:
$$ Y_t=B_0^{-1}B_1Y_{t-1}+\dots+B_0^{-1}B_pY_{t-p}+B_0^{-1}\varepsilon_t $$
$$ Y_t=A_1Y_{t-1}+\dots+A_pY_{t-p}+u_t \;.$$
The key implication is that the vector of structural shocks $\varepsilon_t$ is a linear combination of the reduced-form residuals $u_t$, i.e. $\varepsilon_t=B_0u_t$. Since the sampler keeps only draws for which the VAR is stable, we can express $Y_t$ as an infinite sum of the reduced-form residuals $u_t$, weighted by the dynamic multipliers $\Phi_h$ (Wold representation):
$$Y_t=\sum_{h=0}^{\infty}\Phi_hu_{t-h}\;.$$
We then derive the structural moving-average representation of the VAR by multiplying and dividing each element of the sum by the matrix $B_0$:
$$ Y_t=\sum_{h=0}^{\infty}\Phi_hB_0^{-1}B_0u_{t-h}=\sum_{h=0}^{\infty}\Theta_h\varepsilon_{t-h}\;.$$
Our goal is to estimate the structural impulse response functions of the endogenous variables to the structural shocks of interest. We define the impulse responses at horizon $h$ as:
$$ \frac{\partial Y_{t+h}}{\partial \varepsilon_t}=\Theta_h= \Phi_h B_0^{-1}\,,
$$
where $\Phi_h$ is computed from each posterior draw of the VAR coefficients. The main goal is to estimate $B_0^{-1}$, or the column of it associated with the structural shock of interest $\varepsilon_{jt}$. The repository implements two identification strategies: sign restrictions and external instruments.


### Sign Restrictions
The Gibbs sampler cycles through two steps, sampling successively from $p(\beta|\Psi,Y)$ and $p(\Psi|\beta,Y)$. After the burn-in stage, the following steps are used to compute the structural impact multiplier matrix $B_0^{-1}$ via sign restrictions, following [Rubio-Ramírez, Waggoner and Zha](#ref-rubio2010) ([2010](#ref-rubio2010)):
1. Draw an $(n\times n)$ matrix $K$ from the standard normal distribution.
2. Compute the matrix $Q$ from the $QR$ decomposition of $K$. Note that $Q$ is orthonormal, i.e. $Q'Q = I_n$.
3. Compute the Cholesky decomposition of the current draw $\Psi = PP'$, where $P$ is lower triangular.
4. Compute the candidate impact matrix $B_0^{-1} = P Q'$. Because $Q'Q = I_n$, the candidate still satisfies $B_0^{-1}(B_0^{-1})' = P Q' Q P' = \Psi$: rotating $P$ changes its elements but not the covariance matrix it implies.
5. Check the sign restrictions on the $j^{th}$ column of $B_0^{-1}$. If all the signs are reversed, flip the sign of the candidate; if the restrictions are not satisfied, go back to step 1 and try a new matrix $K$.
6. For a valid candidate, compute the structural IRFs recursively, assuming that the system was at its steady state at time $t$ when the shock hit; then save the structural IRFs for each retained Gibbs draw.

The procedure is implemented in __Code__ > __BVAR__ > __bvar_sign_gibbs.jl__.


### IV Identification

This identification strategy uses external instruments that capture a fraction of the true structural shock. The methodology was introduced by [Stock and Watson](#ref-stock2012) ([2012](#ref-stock2012)), and it has become increasingly popular in the literature estimating the effects of monetary policy shocks. We apply the procedure outlined in [Stock and Watson](#ref-stock2018) ([2018](#ref-stock2018)).

Without loss of generality, assume that we are interested in the shock ordered first, $\varepsilon_{1t}$. The instrument $Z_t$ must satisfy the so-called "SVAR-IV" conditions:
1. $E[Z_t\varepsilon_{1t}]=\alpha\neq 0 \quad\quad\quad\quad \text{(relevance)}$
2. $E[Z_t\varepsilon_{jt}]=0\quad \forall j\neq 1 \quad\quad\, \text{(exogeneity)}$

Given the construction of $\Theta_h$, the goal is to identify only the first column of $B_0^{-1}$:
$$
\begin{gather*}
\begin{split}
    &\Theta_0=\mathrm{I}B_0^{-1}\quad\\
    &\Theta_1=\Phi_1B_0^{-1}\\
    &\;\vdots\\
    &\Theta_h=\Phi_hB_0^{-1}
    \end{split}\quad\quad
    \begin{split}
        B_0^{-1}=\begin{bmatrix}
        b_{1,1}&*&*&\dots\\
        b_{2,1}&*&*&\dots\\
        \vdots&\vdots&\vdots&\vdots\\
        b_{n,1}&*&*&\dots
        \end{bmatrix}{}
    \end{split}{}
\end{gather*}
$$
Exploiting the relation between structural shocks and reduced-form residuals, $u_t = B_0^{-1}\varepsilon_t$, together with the SVAR-IV conditions, we obtain:
$$
\begin{gather*}
\begin{split}
    &\mathrm{E}[Z_tu_t]=B_0^{-1}\mathrm{E}[Z_t\varepsilon_t]\\[0.3em]
    &\mathrm{E}[Z_tu_t]=B_0^{-1}\begin{bmatrix}
    \alpha\\
    0
    \end{bmatrix}\\[0.3em]
    & \mathrm{E}[Z_tu_t]=\begin{bmatrix}
    b_{1,1}\,\alpha\\
    b_{2:n,1}\,\alpha
    \end{bmatrix}
    \end{split}
\end{gather*}
$$
If we normalize the impact on the instrumented variable, $b_{1,1}$, to one, the remaining elements of the first column of $B_0^{-1}$ can be expressed as:
$$
\begin{gathered}
b_{j,1}=\big(\mathrm{E}[Z_tu_{1,t}]\big)^{-1}\,\mathrm{E}[Z_tu_{j,t}] \\
\textit{with}\quad j=2,\dots,n\;.
\end{gathered}
$$

As for sign restrictions, the Gibbs sampler cycles through two steps, sampling successively from $p(\beta|\Psi,Y)$ and $p(\Psi|\beta,Y)$. After the burn-in stage, for each draw of $\beta$ and $\Psi$ we compute the corresponding sequence of residuals $u_t = Y_t - X_t \beta$ on the actual sample, without the dummy observations. We then estimate $\hat b_{j,1}$ for $j=2,\dots,n$ by replacing the population moments above with their sample counterparts. The function performing these steps is __Code__ > __BVAR__ > __bvar_iv_gibbs.jl__.


## The Excel File

The data are read from an Excel file in __Data__ > __FinalData__, whose name is set in __Code__ > __SetUp__ > __SetupBVAR.jl__. The repository includes __Data.xlsx__, with the data of the baseline model in [Bini](#ref-bini2025) ([2025](#ref-bini2025)): five monthly US series from January 1998 to December 2024 (CPI, the WTI oil price, industrial production, the unemployment rate and the Global Supply Chain Pressure Index of the New York Fed), and the instrument for global supply chain shocks constructed in the paper from the surcharges of the three largest container shipping companies. In the paper, these data are used in a structural VAR identified with an external instrument and estimated with frequentist methods, not in a Bayesian VAR; here they illustrate how to use the code. The file is read by __Code__ > __DataSection__ > __readdata.jl__ and has four worksheets, described below.

### Data Worksheets
The data are stored in one worksheet for each frequency, named "Data_" + frequency: "Data_d" (daily), "Data_w" (weekly), "Data_m" (monthly) and "Data_q" (quarterly). Column A, "Dates", contains the dates, and every other column contains a raw series, with its name in the first row; the data start in the second row. The dates should be stored as Excel dates (for instance in the format DD-MM-YYYY), and for monthly and quarterly data any day of the period can be used: 01-01-1998 and 31-01-1998 both refer to January 1998. __Data.xlsx__ contains only the worksheet "Data_m":

<div class="xl"><div class="xl-scroll"><table>
<thead><tr><th></th><th>A</th><th>B</th><th>C</th><th>D</th><th>E</th><th>F</th></tr></thead><tbody>
<tr><th>1</th><td class="h">Dates</td><td class="h">CPI</td><td class="h">WTI Oil Price</td><td class="h">Industrial Production</td><td class="h">Unemployment Rate</td><td class="h">GSCPI</td></tr>
<tr><th>2</th><td>31-01-1998</td><td>162</td><td>16.71</td><td>84.1564</td><td>4.6</td><td>−1.0791</td></tr>
<tr><th>3</th><td>28-02-1998</td><td>162</td><td>16.06</td><td>84.3066</td><td>4.6</td><td>−0.4501</td></tr>
<tr><th>4</th><td>31-03-1998</td><td>162</td><td>15.02</td><td>84.3420</td><td>4.7</td><td>−0.0743</td></tr>
<tr><th>⋮</th><td class="dots">⋮</td><td class="dots">⋮</td><td class="dots">⋮</td><td class="dots">⋮</td><td class="dots">⋮</td><td class="dots">⋮</td></tr>
<tr><th>325</th><td>31-12-2024</td><td>317.685</td><td>69.7</td><td>103.1654</td><td>4.1</td><td>−0.2224</td></tr>
</tbody></table></div><div class="xl-tabs"><span class="on">Data_m</span><span>Legend</span><span>Sign</span><span>IV</span></div></div>

The model can combine series at different frequencies. To add quarterly series, create a worksheet "Data_q" with the same structure, and add a row for each new series to the [worksheet "Legend"](#legend) (and to the [worksheet "Sign"](#sign), if used). For instance, to add real GDP:

<div class="xl"><div class="xl-scroll"><table>
<thead><tr><th></th><th>A</th><th>B</th></tr></thead><tbody>
<tr><th>1</th><td class="h">Dates</td><td class="h">Real GDP</td></tr>
<tr><th>2</th><td>31-03-1998</td><td>3175.9</td></tr>
<tr><th>3</th><td>30-06-1998</td><td>3205.3</td></tr>
<tr><th>4</th><td>30-09-1998</td><td>3245.7</td></tr>
<tr><th>⋮</th><td class="dots">⋮</td><td class="dots">⋮</td></tr>
</tbody></table></div><div class="xl-tabs"><span>Data_m</span><span class="on">Data_q</span><span>Legend</span><span>Sign</span><span>IV</span></div></div>

The frequency of the VAR is the highest frequency among the included series. Each series is first transformed at its own frequency, as specified in the worksheet "Legend" (for instance, "DIFF" = 1 gives quarter-on-quarter changes for a quarterly series); each observation of a lower-frequency series is then placed on the last month of its period, and the months in between are filled by interpolation with splines.

### Legend
The worksheet "Legend" has one row for each series, with its name as in the data worksheets. The series included in the model enter the VAR in the order of this worksheet. The columns are:
- __"LOG", "DIFF" and "FILT"__: the transformation of the series, applied by __Code__ > __Toolbox__ > __transformation.jl__ (see the table below).
- __"PRIOR AR(1)"__: the prior for the [Minnesota prior](#minnesota-prior), 1 for a random walk ($\delta_i = 1$) and 0 for white noise ($\delta_i = 0$).
- __"INCL"__: 1 to include the series in the VAR, 0 to leave it out, so that a series can remain in the file without being used.
- __"INSTRUMENTED"__: 1 for the variable whose shock is identified with the external instrument in the [worksheet "IV"](#iv). Only one variable can be instrumented in each run.

| LOG | DIFF | FILT | Transformation |
|:---:|:---:|:---:|---|
| 0 | 0 | 0 | Level |
| 1 | 0 | 0 | 100 × log level |
| 0 | 1 | 0 | First difference |
| 1 | 1 | 0 | 100 × log difference (period-on-period growth rate) |
| 0 | 1 | 1 | Year-on-year difference |
| 1 | 1 | 1 | Year-on-year growth rate, in percent |

In __Data.xlsx__, CPI, the oil price and industrial production enter in 100 × log levels with a random walk prior, the unemployment rate and the GSCPI enter in levels with a white noise prior, and the GSCPI is the instrumented variable:

<div class="xl"><div class="xl-scroll"><table>
<thead><tr><th></th><th>A</th><th>B</th><th>C</th><th>D</th><th>E</th><th>F</th><th>G</th></tr></thead><tbody>
<tr><th>1</th><td class="h">Series</td><td class="h">LOG</td><td class="h">DIFF</td><td class="h">FILT</td><td class="h">PRIOR AR(1)</td><td class="h">INCL</td><td class="h">INSTRUMENTED</td></tr>
<tr><th>2</th><td>CPI</td><td>1</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td></tr>
<tr><th>3</th><td>WTI Oil Price</td><td>1</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td></tr>
<tr><th>4</th><td>Industrial Production</td><td>1</td><td>0</td><td>0</td><td>1</td><td>1</td><td>0</td></tr>
<tr><th>5</th><td>Unemployment Rate</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>0</td></tr>
<tr><th>6</th><td>GSCPI</td><td>0</td><td>0</td><td>0</td><td>0</td><td>1</td><td>1</td></tr>
</tbody></table></div><div class="xl-tabs"><span>Data_m</span><span class="on">Legend</span><span>Sign</span><span>IV</span></div></div>

### Sign
The worksheet "Sign" is needed only for [identification via sign restrictions](#sign-restrictions). It has one row for each series included in the model, with its name in column B. Columns C onward contain one shock each, with the name of the shock in the first row: the response of each variable is set to $\textcolor{red}{+1}$ (positive) or $\textcolor{blue}{-1}$ (negative), and the cell is left blank for no restriction. Column A, "policy variable", assigns each shock to a column of $B_0^{-1}$: write 1 on the row of the variable associated with the first shock, 2 on the row of the variable associated with the second shock, and so on.

__Data.xlsx__ identifies two shocks. A global supply chain shock, associated with the GSCPI, raises the GSCPI, CPI and the unemployment rate and lowers industrial production, leaving the oil price unrestricted. A demand shock, associated with industrial production, raises CPI and industrial production and lowers the unemployment rate, leaving the GSCPI and the oil price unrestricted:

<div class="xl"><div class="xl-scroll"><table>
<thead><tr><th></th><th>A</th><th>B</th><th>C</th><th>D</th></tr></thead><tbody>
<tr><th>1</th><td class="h">policy variable</td><td class="h">Series</td><td class="h">Global Supply Chain Shock</td><td class="h">Demand Shock</td></tr>
<tr><th>2</th><td></td><td>CPI</td><td class="pos">+1</td><td class="pos">+1</td></tr>
<tr><th>3</th><td></td><td>WTI Oil Price</td><td></td><td></td></tr>
<tr><th>4</th><td>2</td><td>Industrial Production</td><td class="neg">−1</td><td class="pos">+1</td></tr>
<tr><th>5</th><td></td><td>Unemployment Rate</td><td class="pos">+1</td><td class="neg">−1</td></tr>
<tr><th>6</th><td>1</td><td>GSCPI</td><td class="pos">+1</td><td></td></tr>
</tbody></table></div><div class="xl-tabs"><span>Data_m</span><span>Legend</span><span class="on">Sign</span><span>IV</span></div></div>

### IV
The worksheet "IV" is needed only for [identification with an external instrument](#iv-identification). Column A contains the dates and column B the instrument, with the name of the shock in the first row, which is used in the titles of the plots. The variable whose shock is instrumented is set in the column "INSTRUMENTED" of the [worksheet "Legend"](#legend). In __Data.xlsx__, the instrument is the one constructed in [Bini](#ref-bini2025) ([2025](#ref-bini2025)): it is zero in the months without surcharges and is first different from zero in August 2014:

<div class="xl"><div class="xl-scroll"><table>
<thead><tr><th></th><th>A</th><th>B</th></tr></thead><tbody>
<tr><th>1</th><td class="h">Dates</td><td class="h">Global Supply Chain Shock</td></tr>
<tr><th>2</th><td>31-01-1998</td><td>0</td></tr>
<tr><th>3</th><td>28-02-1998</td><td>0</td></tr>
<tr><th>⋮</th><td class="dots">⋮</td><td class="dots">⋮</td></tr>
<tr><th>201</th><td>31-07-2014</td><td>0</td></tr>
<tr><th>202</th><td>31-08-2014</td><td>200</td></tr>
<tr><th>⋮</th><td class="dots">⋮</td><td class="dots">⋮</td></tr>
<tr><th>325</th><td>31-12-2024</td><td>300</td></tr>
</tbody></table></div><div class="xl-tabs"><span>Data_m</span><span>Legend</span><span>Sign</span><span class="on">IV</span></div></div>

When the file contains both the worksheet "Sign" and the worksheet "IV", the code runs both identification schemes, and saves the results in __Results__ > *results folder* > __IRF_sign__ and __IRF_iv__.

## References

<div class="references">

<span id="ref-banbura2010"></span>**Bańbura, Marta, Domenico Giannone, and Lucrezia Reichlin.** 2010. “Large Bayesian vector auto regressions.” *Journal of Applied Econometrics*, 25(1): 71–92.

<span id="ref-berger1985"></span>**Berger, James O.** 1985. *Statistical Decision Theory and Bayesian Analysis*. 2nd ed. New York: Springer.

<span id="ref-bini2025"></span>**Bini, Lapo.** 2025. “[The Macroeconomic Effects of Global Supply Chain Shocks](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=5375776).” Working paper.

<span id="ref-giannone2015"></span>**Giannone, Domenico, Michele Lenza, and Giorgio E Primiceri.** 2015. “Prior selection for vector autoregressions.” *Review of Economics and Statistics*, 97(2): 436–451.

<span id="ref-kadiyala1997"></span>**Kadiyala, K Rao, and Sune Karlsson.** 1997. “Numerical methods for estimation and inference in Bayesian VAR-models.” *Journal of Applied Econometrics*, 12(2): 99–132.

<span id="ref-koop2003"></span>**Koop, Gary.** 2003. *Bayesian Econometrics*. Chichester: John Wiley & Sons.

<span id="ref-litterman1986"></span>**Litterman, Robert B.** 1986. “Forecasting with Bayesian vector autoregressions: Five years of experience.” *Journal of Business & Economic Statistics*, 4(1): 25–38.

<span id="ref-robertson1999"></span>**Robertson, John C, and Ellis W Tallman.** 1999. “Vector autoregressions: Forecasting and reality.” *Federal Reserve Bank of Atlanta Economic Review*, 84(1): 4–18.

<span id="ref-rubio2010"></span>**Rubio-Ramírez, Juan F, Daniel F Waggoner, and Tao Zha.** 2010. “Structural vector autoregressions: Theory of identification and algorithms for inference.” *Review of Economic Studies*, 77(2): 665–696.

<span id="ref-sims1993"></span>**Sims, Christopher A.** 1993. “A nine-variable probabilistic macroeconomic forecasting model.” In *Business Cycles, Indicators, and Forecasting*, ed. James H Stock and Mark W Watson, 179–212. Chicago: University of Chicago Press.

<span id="ref-sims1998"></span>**Sims, Christopher A, and Tao Zha.** 1998. “Bayesian methods for dynamic multivariate models.” *International Economic Review*, 39(4): 949–968.

<span id="ref-stock2012"></span>**Stock, James H, and Mark W Watson.** 2012. “Disentangling the channels of the 2007–09 recession.” *Brookings Papers on Economic Activity*, 2012(1): 81–135.

<span id="ref-stock2018"></span>**Stock, James H, and Mark W Watson.** 2018. “Identification and estimation of dynamic causal effects in macroeconomics using external instruments.” *Economic Journal*, 128(610): 917–948.

</div>
