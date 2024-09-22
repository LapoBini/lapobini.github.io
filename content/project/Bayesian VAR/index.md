---
date: "2024-09-01"
#image:
#  caption: Embed rich media such as videos and LaTeX math
math: false
summary: BVAR Estimation and Structural Analysis
title: BayesianVAR.jl
type: docs
---
## Introduction
The repository [BayesianVAR.jl](https://github.com/LapoBini/BayesianVAR.jl) contains the code to estimate a Bayesian VAR with minnesota, sum-of-coefficients and dummy-initial-observations priors, and to perform structural analysis using sign reristrictions or external instruments. 

The repository has an excel file (__Data__ > __FinalData__ > __test.xlsx__) that can be used for testing and running the model using __main_executer.jl__. The settings for the VAR (sample dates, directory of the data file, name of the result folder, required packages etc.) can be changed in __Code__ > __SetUp__ > __SetupBVAR.jl__. Every function contains a brief description of the econometric framework with equations and references. 

### Brief Comment on the Excel File 
The model, using __Code__ > __DataSection__ > readdata_haver.jl__,  could aggregate series at different frequencies by applying interpolation via splines. Every worksheet with data must be created following the "Haver Convention" (have a look at the excel file, first column and first 18 rows). All the worksheets with data must be named following the convention "Data_" + frequency ("d", "w", "m", "q"). Moreover, the function __Code__ > __Toolbox__ > __transformation.jl__, called inside __readdata_haver.jl__, applies standard transformations to the data following the instructions provided in __worksheet "Legend" - columns C-E__ ("LOG"=1 takes log, "DIFF"=1 applies difference with respect to previous realization, "FILT" is used to transform data in YoY growth if the other two cells are set to one). The __worksheet "Legend" - column F__ specifies the minnesota prior, __worksheet "Legend" - column G__ specifies which series we want to include. 

Other instructions about the structure of the excel file are provided in __Code__ > __DataSection__ > __readdata_haver.jl__, and below in the paragraph that explains how to perform structural analysis.


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

Assuming $\beta = [A_1 \;A_2 \;\dots A_p]$ to be a priori independent and normally distributed, the prior beliefs are imposed by setting the following moments for the prior distribution of the coefficients:
{{< math >}}
$$E[(A_k)_{ij}] = \begin{cases}
  \gamma_i \quad \text{if} \;\; j=i \;\; k=1 \\
  0 \quad\; \text{otherwise}
\end{cases}\;.$$
{{< /math >}}
Originally, $\textcolor{orange}{Litterman \; (1986)}$ sets $\gamma_i = 1$ for all $i$, reflecting the belief that all the variables are characterized by high persistence. However, this prior is not appropriate for variables displaying substantial mean reversion. For those we impose the prior belief of white noise by setting $\gamma_i = 0$. The variance of the parameters is:
{{< math >}}
$$Var((A_k)_{ij}) =  \begin{cases} 
\frac{\lambda^2}{\kappa^2} \quad\quad\quad \text{if}\; i=j \\ \vartheta\frac{\lambda^2}{\kappa^2}\frac{\sigma_i^2}{\sigma_j^2} \quad\text{ otherwise}
\end{cases}\;.$$
{{< /math >}}
As for the covariance matrix of the residuals, in the original formulation of the minnesota prior it was originally assumed to be diagonal and known: $\Psi = \Sigma$ where $\Sigma = diag(\sigma_1^2,\;\dots, \sigma_n^2)$. However, in the context of the structural analysis we need to take into account possible correlation among the residual of different variables. To overcome this problem we follow $\textcolor{orange}{Kadiyala}$ $\textcolor{orange}{and}$ $\textcolor{orange}{Karlsson\; (1997)}$ and impose a Normal inverted Wishart prior which retains the principles of the Minnesota prior. This is possible under the condition that $\vartheta = 1$, which will be assumed in what follows. Finally, the prior on the intercept $c$ is diffuse. 

The minnesota prior is implemented in __Code__ > __BVAR__ > __bvar_minnesota_prior.jl__. First, the function shrinks each variable towards the random walk or the white noise as specified in the __worksheet "Legend" - column F__; then, it calculates $\{\sigma^n_1,\dots,\sigma^2_n\}$.

### Sum-of-Coefficients Prior
The literature has suggested that improvement in forecasting performance can be obtained by imposing additional priors that constrain the sum of coefficients (see e.g. $\textcolor{orange}{Sims,\;1992}$; $\textcolor{orange}{Sims\; and\; Zha,\; 1998}$; $\textcolor{orange}{Robertson\; and\; Tallman,\; 1999}$). This is the same as imposing “inexact differencing” and it is a simple modification of the Minnesota prior involving linear combinations of the VAR coefficients.
 
We now manipulation the VAR to rewrite it in the error-correction form: $$Y_t = c + A_1 Y_{t-1}+\dots+A_p Y_{t-p} + u_t$$  $$\Delta Y_t = c - (I_n - A_1 - A_2- \dots- A_p) Y_{t-1} + C_1 \Delta Y_{t-1} + \dots + C_p \Delta Y_{t-p} + \Delta u_t$$ with $C_k = - \sum_{j=k+1}^p A_j$. The second term $- (I_n - A_1 - A_2- \dots- A_p) Y_{t-1} $ is the error correction term, which guaranties stationarity of the model by correcting the current-period change in $Y_t$ with part of the preceding period deviation from the long-run value of the model. The second part, $C_1 \Delta Y_{t-1} + \dots + C_p \Delta Y_{t-p} + \Delta u_t$, is made of stationary terms and hence does not impact the stationarity of the model. 

We know that $\Pi = I_n - A_1 - A_2- \dots- A_p = 0$ is implied by a VAR in first difference. By inexact differencing, and thus imposing the sum-of-coefficients prior, we shrink $\Pi$ toward zero in order to guarantee that the draws obtained from the posterior are characterized by a unit root rather than by explosive behavior. In this case, $Y_t$ is equal to its previous value, plus the sum of stationary terms not affecting stationarity. It follows that in the limit ($\Pi \xrightarrow{} 0$), each variable in $Y_t$ contains a unit root and the model rules out cointegration.

### Dummy-Initial-Observation Prior
As we mentioned before, the sum-of-coefficients prior rules out cointegration in the limit, which may be undesirable. Given $$\Delta Y_t = c - (I_n - A_1 - A_2- \dots- A_p) Y_{t-1} +  C_1 \Delta Y_{t-1} + \dots + C_p \Delta Y_{t-p} + \Delta u_t$$ by applying the sum-of-coefficients prior, in the limit $\Pi \xrightarrow{} 0$ and we get: $$\Delta Y_t= c +  C_1 \Delta Y_{t-1} + \dots + C_p \Delta Y_{t-p} + \Delta u_t$$ which is equivalent to $$Y_t= c + Y_{t-1} + C_1 \Delta Y_{t-1} + \dots + C_p \Delta Y_{t-p} + \Delta u_t\;.$$ 
Note that the absence of error correction term rules out the possibility of cointegration relations. Thus $\textcolor{orange}{Sims\;(1993)}$ proposed the use of an additional prior known as "dummy-initial-observations". 

The prior states that a no-change forecast for all variables is a good forecast at the beginning of the sample. Then, either all the variables are at their unconditional mean, which implies that the model is stationary despite the unit roots in the variables (implying cointegration), or the dynamic of the system is characterized by an unspecified number of unit roots, and the variables share a common stochastic trend.

## Implementation of the Priors
### Prior Distribution 
Following $\textcolor{orange}{Kadiyala\; and\; Karlsson\; (1997)}$ and $\textcolor{orange}{Robertson\; and\; Tallman\; (1999)}$, we impose a Normal Inverted Wishart prior distribution in the form:
$$ 
vec(\beta)|\Psi \sim N(vec(\beta_0), \Psi \otimes \Omega_0 ) \quad \text{and}\quad \Psi \sim iW(S_0, \alpha_0)
$$
where the prior parameters $\beta_0$, $\Omega_0$, $S_0$ and $\alpha_0$ are chosen so that prior expectations and
variances of $\beta$ coincide with those implied by the minnesota prior, and the expectation of $\Psi$ is
equal to the fixed residual covariance matrix $\Sigma$. We implement the prior by adding dummy observations: it can be shown that adding $T_d$ dummy observations $Y_d$ and $X_d$ to the VAR(p) is equivalent to imposing the Normal inverted Wishart prior with:
* $\beta_0 = (X_d'X_d)^{−1}(X_d'Y_d)$;
* $\Omega_0 = (X_d'X_d)^{-1}$;
* $S_0 = (Y_d − X_d \beta_0)'(Y_d − X_d\beta_0)$;
* $α_0 = T_d − k$.

where $S_0$ is a $(k\text{x}k)$ scale matrix and $\alpha_0$ are the degrees of freedom of the Inverted Wishart distribution.

### Dummy Observations
We follow $\textcolor{orange}{Banbura,\; Giannone,\; and\; Reichlin\; (2007)}$ and implement the prior distribution using dummy observations. The dummy observations $Y_d$ and $X_d$ are constructed as follows:
$$
Y_d = \begin{bmatrix}
        diag(\delta_1\sigma_1,\dots,\delta_n\sigma_n)/\lambda\\
        0_{n(p-1)\,\text{x}\,n}\\[0.5em]\hline\\[-0.8em]
        diag(\sigma_1,\dots,\sigma_n)\\[0.5em]\hline\\[-0.8em]
        0_{1\text{x}n}\\[0.5em]\hline\\[-0.8em]
        diag(\mu_1\delta_1,\dots,\mu_n\delta_n)/\tau\\[0.5em]\hline\\[-0.8em]
        diag(\mu_1,\dots,\mu_n)/\gamma
\end{bmatrix}
\quad
X_d = \begin{bmatrix}
        J_p \otimes diag(\sigma_1,\dots,\sigma_n)/\lambda & 0_{np \text{x} 1}\\[0.5em]\hline\\[-0.8em]
        0_{n \text{x} np} & 0_{n \text{x} 1} \\[0.5em]\hline\\[-0.8em]
        0_{1 \text{x} np} & \varepsilon\\[0.5em]\hline\\[-0.8em]
        \mathbf{1}_p\otimes diag(\mu_1\delta_1,\dots,\mu_n\delta_n)/\tau & 0_{n \text{x} 1} \\[0.5em]\hline\\[-0.8em]
        \mathbf{1}_p\otimes diag(\mu_1,\dots,\mu_n)/\gamma & 1/\gamma
\end{bmatrix}
$$
where the the first block of dummies imposes prior beliefs on the autoregressive coefficients, the second block implements the prior for the covariance matrix, the third block reflects the uninformative prior for the intercept ($\varepsilon$), the fourth block implements the sum-of-coefficients prior, the fifth one the dummy initial observation prior. The function that constructs the dummy observations could be found in __Code__ > __BVAR__ > __bvar_dummies.jl__.

Notice that the prior are parametrized using additional hyperparameters, also known as hyperpriors:
1. $\delta_i$, which assumes value 1 or 0, reflecting the belief that the $i^{th}$ series is characterized by high persistence (random-walk) or substantial mean reversion (white noise) respectively.
2. $\sigma_i$, which is the standard deviation of the residual of the random walk or the white noise. 
3. $\lambda$ controls the overall tightness of the prior distribution around the random walk or white noise and governs the relative importance of the prior beliefs with respect to the information contained in the data. $\lambda = 0$ the posterior is equal to the prior, while $\lambda = \infty$ the posterior is the OLS.
4. $\tau$ controls for the degree of shrinkage for the sum-of-coefficient prior: as $\tau$ goes to zero we approach
the case of exact differences and, as $\tau$ goes to infinity, we approach the case of no shrinkage.
5. $\mu_i$ aims at capturing the unconditional mean of variable $y_{it}$. 
6. $\gamma$ controls the tightness of the dummy initial observation prior, when it approaches infinity we have no shrinkage.

How should the hyperpriors be specified? What is the optimal lag number? What are the best shrinkage values for the prior distribution? The goal is to ensure that the model's prior assumptions align well with the observed data, improving the model's overall performance without overfitting. However, the priors are influenced by the hyperpriors that control how strong or loose our prior beliefs are. The procedure to optimize their values is illustrated in the next paragraph.

### Informativeness of the Prior Distribution
From a Bayesian perspective, the choice of the informativeness of the prior distribution is conceptually identical to the inference on any other unknown parameter of the model. Suppose, for instance, that a model is described by a likelihood function $p(y|\theta)$ and a prior distribution $p_\gamma(\theta)$ where $\theta$ is the vector of the VAR(p) parameters and $\gamma$ collects the hyperparameters (hyperpriors), i.e. those coefficients that parameterize the prior distribution, but do not directly affect the likelihood. It is then natural to choose these hyperparameters by interpreting the model as a hierarchical model, i.e. replacing $p_\gamma(\theta)$ with $p(\theta|\gamma)$ and evaluating their posterior ($\textcolor{orange}{Berger,\, 1985}$; $\textcolor{orange}{Koop,\, 2003}$). 

Following $\textcolor{orange}{Giannone,\, Lenza,\, Primicieri\, (2012)}$, we pick the hyperpriors that maximize the marginal likelihood of the data, i.e. the optimal hyperpriors is the set of hyperparameters that makes the data most likely after integrating out the uncertainty in the model parameters. The procedure is implemented in __Code__ > __BVAR__ > __bvar_opt_prior.jl__ 

## Posterior Distribution
We now augment the data with the dummy observations:
* $Y_* = (Y', Y_d')'$;
* $X_* = (X', X_d')'$;
* $U_* = (U', U_d')'$;

and we rewrite the VAR(p) as a general regression model: 
$$Y_* = X_* \,\beta + U_*\,.$$
The posterior distribution has the form: 
$$ 
vec(\beta)|\Psi,Y \sim N(vec(\tilde \beta), \Psi \otimes (X_*' X_*)^{-1} ) \quad \text{and}\quad \Psi|Y \sim iW(\tilde \Sigma, T_d + T + 2 - n)\,.
$$
It can be noticed that the posterior expectations of the coefficients corresponds to the OLS estimates on the regression model based on the appended data, with:
* $\tilde \beta = (X_*'X_*)^{−1}(X_*'Y_*)$;
* $\tilde \Sigma = (Y_* − X_* \tilde \beta)'(Y_* − X_* \tilde \beta)$;

The functions that draw from the posterior distributions giben the optimal informativeness of the prior distribution could be found in __Code__ > __BVAR__ > __bvar_gibbs_helper.jl__ > __bvar_IWRD__ (Inverted Wishart) and __bvar_MNRD__ (Multivariate Normal).

## Structural Identification 
The structural VAR(p) is specified as
$$ B_0Y_t=B_1Y_{t-1}+\dots+B_pY_{t-p}+\varepsilon_t$$
If we multiply both sides of the structural form of the VAR by $B_0^{-1}$, the matrix governing the contemporaneous relations among endogenous variables, also known as structural impact multiplier matrix, we obtain the reduced form representation: 
$$ Y_t=B_0^{-1}B_1Y_{t-1}+\dots+B_0^{-1}B_pY_{t-p}+B_0^{-1}\varepsilon_t $$
$$ Y_t=A_1Y_{t-1}+\dots+A_pY_{t-p}+u_t \;.$$
The key implication is that the vector of structural shocks $\varepsilon_t$ can be seen as the linear combination of the reduced form residuals $u_t$, i.e. $\varepsilon_t=B_0u_t$. Thus, given that our stochastic process $\{Y_t\}_{t\geq0}$ is weakly stationary by construction, the Wold Representation theorem allows us to express $Y_t$ as the infinite sum of the reduced form residuals $u_t$ with weights the dynamic multiplier $\Phi_h$:
$$Y_t=\sum_{h=0}^{\infty}\Phi_hu_{t-h}\;.$$
At this point we derive the structural moving average representation of our vector autoregressive process by multiplying and dividing each element of the summation by the matrix $B_0$: 
$$ Y_t=\sum_{h=0}^{\infty}\Phi_hB_0^{-1}B_0u_{t-h}=\sum_{h=0}^{\infty}\Theta_h\varepsilon_{t-h}\;.$$
Our goal is to estimate the structural impulse response functions of our endogenous variables to the desired exogenous shocks. We define the impulse responses at time $t$ as: 
$$ \frac{\partial Y_{t+h}}{\partial \varepsilon_t}=\Theta_h= \Phi_h B_0^{-1}
$$
where $\Phi_h$ is estimated by OLS. The main goal is to estimate $B_0^{-1}$ or one of its column associated with the desired structural shock $\varepsilon_{jt}$. The repository implements two different identification strategies, using sign restrictions or via external instrument. 


### Sign Restrictions
The Gibbs sampling algorithm to estimate the parameters of the VAR model cycles through two steps, sampling successively from $p(\beta|\Psi,Y)$ and $p(\Psi|\beta,Y)$. Once past the burn-in stage the following steps are used to calculate the structural impact multiplier matrix $B_0$ via sign restrictions à la $\textcolor{orange}{Ramirez}$ $\textcolor{orange}{et}$ $\textcolor{orange}{al.}$ $\textcolor{orange}{(2010)}$:
1. Draw a $(n\text{x}n)$ matrix $K$ from the standard normal distribution. 
2. Calculate the matrix $Q$ from the $QR$ decomposition of $K$, note that $Q$ is orthonormal i.e. $Q'Q = I_n$.
3. Calculate the choleski decomposition of the current draw $\Psi = \tilde B_0' \tilde B_0$.
4. Calculate the candidate $B_0$ matrix as $B_0 = Q \tilde B_0$. Note that because $Q'Q = I_n$ this implies that $B_0'B_0$
will still equal $\Psi$. By calculating the product $B_0 = Q \tilde B_0$ we alter the elements of $\tilde B_0$ but not the property that $\Psi = \tilde B_0' \tilde B_0$.
5. Check the sign restrictions related to the $j^{th}$ column of $B_0$. If they are not satisfied, go to step 1 and try with a new $K$ matrix.
6. For a valid candidate $B_0$, compute the Structural IRF recursively assuming that the system was at the steady state at time $t$ when the shock hitted the system; then, the structural IRFs for each retained Gibbs draw are saved.

The procedure is implemented in __Code__ > __BVAR__ > __bvar_sign_gibbs.jl__.


### IV Identification 

The identification strategy consists of using external instruments which are able to capture a fraction of the true structural shock. This methodology was introduced by $\textcolor{orange}{Stock}$ $\textcolor{orange}{and}$ $\textcolor{orange}{Watson}$ $\textcolor{orange}{(2012)}$ and it has become increasingly popular in the literature seeking to estimate the effects of the monetary policy shock. We apply the same procedure outlined in $\textcolor{orange}{Stock}$ $\textcolor{orange}{and}$ $\textcolor{orange}{Watson}$  $\textcolor{orange}{(2018)}$.

We assume that we are interested in the shock order first ($\varepsilon_{1t}$, without lost of generality. The instrument must satisfy the so called "SVAR-IV" conditions:
1. $E[Z_t\varepsilon_{1t}]=\alpha\neq 0 \quad\quad\quad\quad(relevance)$
2. $E[Z_t\varepsilon_{jt}]=0\quad \forall j\neq 1 \quad\quad\,(exogeneity)$

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
Therefore, if we exploit the relation between structural shock and reduced form residual, $B_0^{-1}\varepsilon_t=u_t$, and the SVAR-IV conditions, we obtain: 
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
If we normalize the coefficient of the instrumented variable $b_{1,1}$ to be equal to one, the coefficients of the first column of $B_0^{-1}$ relative to the exogenous shock can be expressed as:
$$  
b_{j,1}=\big(\mathrm{E}[Z_tu_{1,t}])^{-1}(\mathrm{E}[Z_tu_{j,t}]\big)\quad\textit{with}\quad j=2,...,n\;.
$$

The Gibbs sampling algorithm to estimate the parameters of the VAR model cycles through two steps, sampling successively from $p(\beta|\Psi,Y)$ and $p(\Psi|B,Y)$. Once past the burn-in stage, For each draw of $\beta$ and $\Psi$, we compute the corresponding sequence of residual $u_t = Y_t - X_t \beta$ without using the appended data. Thus, we compute $\hat b_{j,1}$ for $j=2,\dots,n$ by GMM. The function performing these steps is __Code__ > __BVAR__ > __bvar_iv_gibbs.jl__.


### What to do in the Excel File
- Create __worksheet "Sign"__ if you want to perform structural identification via sign restrictions. The first column, __"policy variable"__, assigns to each shock the corresponding column of $B_0^{-1}$. In the excel file, the first shock (1), column __"Global Supply Chain Shock"__, corresponds to the column of $B_0^{-1}$ relative to the variable *GSCPI*. The second shock (2), column __"Other Shock"__, corresponds to the column of $B_0^{-1}$ relative the variable *United States_GDP*. The signs of the response of each variable are created using $\textcolor{red}{(+1)}$ and $\textcolor{blue}{(-1)}$, leave the cell blank if you do not want to impose any restriction.

- Create __worksheet "IV"__ if you want to compute Structural IRFs using an external instrument. The first column of the worksheet must contain the dates, the second one the shock (only one shock for each tun (sigh), going to implement different IVs and multiple IVs for a single shock soon). The column name for the shock MUST BE the tickers of the instrumented variable (we are going to identify the only column of $B_0^{-1}$ corresponding to the instrumented variable). 


