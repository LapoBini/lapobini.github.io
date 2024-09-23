---
date: "2024-06-24"
#image:
#  caption: Embed rich media such as videos and LaTeX math
math: false
summary: Code for Homework and Exam
title: ECON210C
type: docs
---

The repository [ECON210C](https://github.com/LapoBini/210C) contains the codes for assignments and final exam for ECON210C of Prof Johannes Wieland (Spring 2024).

### HW1 - Sequence Space

The first assignment require to solve a RBC DSGE model using the Sequence Space methodology ($\textcolor{orange}{Auclert}$ $\textcolor{orange}{et}$ $\textcolor{orange}{al.,}$ $\textcolor{orange}{(2021)}$). 

$$y_t$$ 
The basic idea is to organize the model into blocks that represents different behaviors of possible heterogeneous agents and interact in GE via a small set of aggregates. Let's now denote $\mathrm{U} = \{U_1,\dots,U_t\}$

### HW2 - Structural VAR

### HW4 - Undetermine Coefficient 

### Exam - RANK Model

## Introduction
The repository [BayesianVAR.jl](https://github.com/LapoBini/BayesianVAR.jl) contains the code to estimate a Bayesian VAR with minnesota, sum-of-coefficients and dummy-initial-observations priors, and to perform structural analysis using sign reristrictions or external instruments. 

The repository has an excel file (__Data__ > __FinalData__ > __test.xlsx__) that can be used for testing and running the model using __main_executer.jl__. The settings for the VAR (sample dates, directory of the data file, name of the result folder, required packages etc.) can be changed in __Code__ > __SetUp__ > __SetupBVAR.jl__. Every function contains a brief description of the econometric framework with equations and references. 

### Brief Comment on the Excel File 
The model  could aggregate series at different frequencies by applying interpolation via splines (__Code__ > __DataSection__ > __readdata_haver.jl__). Every worksheet with data must be created following the "Haver Convention" (have a look at the excel file, first column and first 18 rows). All the worksheets with data must be named following the convention "Data_" + frequency ("d", "w", "m", "q"). Moreover, the function __Code__ > __Toolbox__ > __transformation.jl__, called inside __readdata_haver.jl__, applies standard transformations to the data following the instructions provided in __worksheet "Legend" - columns C-E__ ("LOG"=1 takes log, "DIFF"=1 applies difference with respect to previous realization, "FILT" is used to transform data in YoY growth if the other two cells are set to one). The __worksheet "Legend" - column F__ specifies the minnesota prior, __worksheet "Legend" - column G__ specifies which series we want to include. 

Other instructions about the structure of the excel file are provided in __Code__ > __DataSection__ > __readdata_haver.jl__, and below in the paragraph that explains how to perform structural analysis.


## Setting the Priors

Let $Y_t = (y_{1t},\dots,y_{nt})'$ be a potentially large vector of random variables. We estimate the following VAR(p) model:
{{< math >}}
$$Y_t = c + A_1 Y_{t-1}+\dots+A_p Y_{t-p} + u_t$$
{{< /math >}}
using the Bayesian approach to overcome the curse of dimensionality via the imposition of prior beliefs on the parameters. In setting the prior distributions, we follow standard practice and use the procedure developed in $\textcolor{orange}{Litterman \; (1986)}$ with modifications proposed by $\textcolor{orange}{Kadiyala\; and\; Karlsson\; (1997)}$ and $\textcolor{orange}{Sims\; and\; Zha\; (1998)}$. The priors are implemented using dummy observations, and the choice of the informativeness of the prior distribution is determined by the marginal likelihood of the observed data; in that sense, we follow strictly the exposition and the procedure of $\textcolor{orange}{Banbura,}$ $\textcolor{orange}{Giannone,}$ $\textcolor{orange}{and}$ $\textcolor{orange}{Reichlin}$ $\textcolor{orange}{(2007)}$ and $\textcolor{orange}{Giannone,}$ $\textcolor{orange}{Lenza,}$ $\textcolor{orange}{Primicieri\, (2012)}$.