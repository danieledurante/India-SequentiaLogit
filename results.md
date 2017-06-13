# Results
Tommaso Rigon  
09 giugno 2017  

## Results

This part of the tutorial will reproduce the main results of the paper, including the computation of the DIC and WAIC indexes, the graphs and tables.

The starting point are the file containing the results of the MCMC chain available in the [`workspaces`](https://github.com/tommasorigon/India-SequentiaLogit/blob/master/workspaces) folder, as explained in the [`estimation.md`](https://github.com/tommasorigon/India-SequentiaLogit/blob/master/estimation.md) document. We load all these files in memory, as well as the [`dataset`](https://github.com/tommasorigon/India-SequentiaLogit/blob/master/data-cleaning.md) and the [`R core functions`](https://github.com/tommasorigon/India-SequentiaLogit/blob/master/core_functions.R).



```r
library(dplyr)     # Required to manipulate the dataset
library(splines)   # Required for computing the B-spline basis
library(reshape2)  # Manipulating data
library(ggplot2)   # Graphical library
library(gridExtra) # Graphical library

# Clean everything
rm(list=ls())

# Load the clean dataset
load("dataset.RData")

# Load the results of the MCMC chain
rm(list=ls())

# Load the clean dataset
load("dataset.RData")

# Load the results of the MCMC chain
load("workspaces/ranef.RData")
load("workspaces/ranef_s_part1.RData")
load("workspaces/ranef_s_part2.RData")
load("workspaces/ranef_s_part3.RData")
load("workspaces/dp_ranef.RData")
load("workspaces/dp_ranef_s.RData")

# Core functions. We will only need the function "IC"
source("core_functions.R")
```

## Information criteria (DIC and WAIC)

Following [Gelman et al. (2014)](https://link.springer.com/article/10.1007/s11222-013-9416-2) we compute the DIC (Deviance Information Criterion) and WAIC (Watanabe-Akaike information criterion) indexes. They can be evaluated using the `IC` function, included in the [`core_functions.R`](https://github.com/tommasorigon/India-SequentiaLogit/blob/master/core_functions.R) file. For each model, the IC function return both the DIC and the WAIC but also the effective degree of freedom for each model (not reported in the paper).

Notice that the factorization of our likelihood allows to decompose the DIC and WAIC indexes, that is

> DIC = DIC_Usage + DIC_Reversibility + DIC_Method

and similarly

> WAIC = WAIC_Usage + WAIC_Reversibility + WAIC_Method

The construction of the table is as follows:


```r
# Construction of the table. The part "c(1,2)" select the row of the DIC and the WAIC.
tab <- cbind(c(IC(fit1_ranef)[c(1,2)],
        IC(fit1_ranef_s)[c(1,2)],
        IC(fit1_dp_ranef)[c(1,2)],
        IC(fit1_dp_ranef_s)[c(1,2)]),
      c(IC(fit2_ranef)[c(1,2)],
        IC(fit2_ranef_s)[c(1,2)],
        IC(fit2_dp_ranef)[c(1,2)],
        IC(fit2_dp_ranef_s)[c(1,2)]),
      c(IC(fit3_ranef)[c(1,2)],
        IC(fit3_ranef_s)[c(1,2)],
        IC(fit3_dp_ranef)[c(1,2)],
        IC(fit3_dp_ranef_s)[c(1,2)]))

tab <- rbind(tab[2*(1:4)-1,],-2*tab[2*(1:4),]) # Rearrange for graphical reason
tab <- data.frame(rep(c("baseline","splines","DP", "DP + splines"),2),
                  round(tab,digits=2), round(rowSums(tab),digits=2))
colnames(tab) <- c("DIC","Usage choice","Reversibility choice","Method choice","Total")
knitr::kable(tab[1:4,],format="markdown",row.names = FALSE)
```



|DIC          | Usage choice| Reversibility choice| Method choice|    Total|
|:------------|------------:|--------------------:|-------------:|--------:|
|baseline     |     27565.16|             18238.23|       7710.42| 53513.81|
|splines      |     27271.13|             18166.32|       7658.76| 53096.22|
|DP           |     27557.82|             18241.65|       7719.04| 53518.52|
|DP + splines |     27260.91|             18177.30|       7665.90| 53104.11|


```r
colnames(tab) <- c("WAIC","Usage choice","Reversibility choice","Method choice","Total")
knitr::kable(tab[5:8,],format="markdown",row.names = FALSE)
```



|WAIC         | Usage choice| Reversibility choice| Method choice|    Total|
|:------------|------------:|--------------------:|-------------:|--------:|
|baseline     |     27565.31|             18236.33|       7707.03| 53508.67|
|splines      |     27271.28|             18163.70|       7655.40| 53090.38|
|DP           |     27558.86|             18239.77|       7716.57| 53515.20|
|DP + splines |     27261.94|             18174.20|       7663.52| 53099.66|


Here, we report also the **effective degrees of freedom** for each model, according to both the DIC and the WAIC.


```r
# Construction of the table. The part "c(4,5)" select the row of the p_DIC and the p_WAIC.
tab <- cbind(c(IC(fit1_ranef)[c(4,5)],
        IC(fit1_ranef_s)[c(4,5)],
        IC(fit1_dp_ranef)[c(4,5)],
        IC(fit1_dp_ranef_s)[c(4,5)]),
      c(IC(fit2_ranef)[c(4,5)],
        IC(fit2_ranef_s)[c(4,5)],
        IC(fit2_dp_ranef)[c(4,5)],
        IC(fit2_dp_ranef_s)[c(4,5)]),
      c(IC(fit3_ranef)[c(4,5)],
        IC(fit3_ranef_s)[c(4,5)],
        IC(fit3_dp_ranef)[c(4,5)],
        IC(fit3_dp_ranef_s)[c(4,5)]))

tab <- rbind(tab[2*(1:4)-1,],tab[2*(1:4),]) # Rearrange for graphical reason.
tab <- data.frame(rep(c("baseline","splines","DP", "DP + splines"),2),
                  round(tab,digits=2),round(rowSums(tab),digits=2))
colnames(tab) <- c("Effective degree of freedom - DIC","Usage choice",
                   "Reversibility choice","Method choice","Total")
knitr::kable(tab[1:4,],format="markdown",row.names = FALSE)
```



|Effective degree of freedom - DIC | Usage choice| Reversibility choice| Method choice|  Total|
|:---------------------------------|------------:|--------------------:|-------------:|------:|
|baseline                          |        42.23|                41.24|         39.71| 123.19|
|splines                           |        48.59|                46.05|         43.59| 138.23|
|DP                                |        36.74|                40.32|         41.13| 118.20|
|DP + splines                      |        42.06|                48.58|         43.78| 134.41|



```r
colnames(tab) <- c("Effective degree of freedom - WAIC","Usage choice",
                   "Reversibility choice","Method choice","Total")
knitr::kable(tab[5:8,],format="markdown",row.names = FALSE)
```



|Effective degree of freedom - WAIC | Usage choice| Reversibility choice| Method choice|  Total|
|:----------------------------------|------------:|--------------------:|-------------:|------:|
|baseline                           |        42.39|                39.34|         36.32| 118.04|
|splines                            |        48.75|                43.42|         40.23| 132.39|
|DP                                 |        37.78|                38.45|         38.66| 114.88|
|DP + splines                       |        43.09|                45.48|         41.40| 129.96|

## Fixed effects

In the following, we compute the posterior mean of the fixed effect coefficients, together with 0.95% credible interval for each coefficient. 


```r
alpha <- 0.05 # alpha = 1 - Coverage level

tab <-cbind(
  # Usage choice
  cbind(apply(fit1_dp_ranef_s$beta_Fix,2,mean),apply(fit1_dp_ranef_s$beta_Fix,2,function(x) quantile(x,alpha/2)),apply(fit1_dp_ranef_s$beta_Fix,2,function(x) quantile(x,1-alpha/2))),
  # Reversibility choice
  cbind(apply(fit2_dp_ranef_s$beta_Fix,2,mean),apply(fit2_dp_ranef_s$beta_Fix,2,function(x) quantile(x,alpha/2)),apply(fit2_dp_ranef_s$beta_Fix,2,function(x) quantile(x,1-alpha/2))),
  # Method choice
  cbind(apply(fit3_dp_ranef_s$beta_Fix,2,mean),apply(fit3_dp_ranef_s$beta_Fix,2,function(x) quantile(x,alpha/2)),apply(fit3_dp_ranef_s$beta_Fix,2,function(x) quantile(x,1-alpha/2)))
)

colnames(tab) <- c("Usage (mean)","Usage (lower)","Usage (upper)",
                   "Reversibility (mean)","Reversibility (lower)","Reversibility (upper)", 
                   "Method (mean)","Method (lower)", "Method (upper)")
rownames(tab) <- c("no child","one child","urban","muslim","christian","other","low","intermediate","high")

knitr::kable(round(tab,digits=2),format="markdown")
```



|             | Usage (mean)| Usage (lower)| Usage (upper)| Reversibility (mean)| Reversibility (lower)| Reversibility (upper)| Method (mean)| Method (lower)| Method (upper)|
|:------------|------------:|-------------:|-------------:|--------------------:|---------------------:|---------------------:|-------------:|--------------:|--------------:|
|no child     |        -3.71|         -3.88|         -3.55|                 2.19|                  1.67|                  2.72|          0.18|          -0.24|           0.59|
|one child    |        -1.37|         -1.45|         -1.29|                 2.19|                  2.06|                  2.33|          0.18|           0.03|           0.33|
|urban        |         0.24|          0.16|          0.31|                 0.29|                  0.20|                  0.38|         -0.45|          -0.57|          -0.32|
|muslim       |        -0.43|         -0.52|         -0.34|                 1.24|                  1.12|                  1.36|         -0.13|          -0.28|           0.04|
|christian    |        -0.27|         -0.49|         -0.05|                 0.01|                 -0.29|                  0.30|         -0.38|          -0.89|           0.12|
|other        |         0.08|         -0.14|          0.29|                 0.46|                  0.25|                  0.68|         -0.26|          -0.53|           0.02|
|low          |         0.14|          0.04|          0.23|                 0.09|                 -0.02|                  0.21|         -0.43|          -0.61|          -0.24|
|intermediate |         0.19|          0.11|          0.27|                 0.51|                  0.41|                  0.60|         -0.71|          -0.86|          -0.57|
|high         |         0.26|          0.15|          0.37|                 1.29|                  1.15|                  1.42|         -1.16|          -1.34|          -0.99|


## Age effect

The `age` effect is obtained by computing the posterior mean of each function f_k(). The first step consists in constructing the B-spline basis function evaluated among the grid of values 15,...,49. This is done by using the `spline.des` function from the `splines` R package.


```r
# B-spline basis construction

# Knots placement
inner_knots <- 40; degree <- 3
xl    <- min(dataset$age); xr <- max(dataset$age); dx <- (xr - xl) / (inner_knots-1)
knots <- seq(xl - degree * dx, xr + degree * dx, by = dx)

# Fixed quantities
B        <- spline.des(knots, 15:49, degree + 1, 0 * 15:49, outer.ok=TRUE)$design

# Posterior sample for each f_k()
eta1_spline <- t(B%*%t(fit1_dp_ranef_s$beta_spline))
eta2_spline <- t(B%*%t(fit2_dp_ranef_s$beta_spline))
eta3_spline <- t(B%*%t(fit3_dp_ranef_s$beta_spline))

# Plots
data.plot <- data.frame(x=15:49, y=colMeans(eta1_spline), 
                        ymax=apply(eta1_spline,2,function(x) quantile(x,0.975)),
                        ymin=apply(eta1_spline,2,function(x) quantile(x,1-0.975)),
                        k="Usage choice")
data.plot <- rbind(data.plot,data.frame(x=15:49,y=apply(eta2_spline,2,mean),
                                        ymax=apply(eta2_spline,2,function(x) quantile(x,0.975)),
                                        ymin=apply(eta2_spline,2,function(x) quantile(x,1-0.975)),
                                        k="Reversibility choice"))
data.plot <- rbind(data.plot,data.frame(x=15:49,y=apply(eta3_spline,2,mean),
                                        ymax=apply(eta3_spline,2,function(x) quantile(x,0.975)),
                                        ymin=apply(eta3_spline,2,function(x) quantile(x,1-0.975)),
                                        k="Method choice"))

plot <- ggplot(data = data.plot, aes(x = x, y = y,ymin=ymin,ymax=ymax)) + geom_line()  +theme_bw()+ xlab("") + ylab("") + geom_ribbon(alpha=0.25) + facet_grid(.~k)
```

![Age effect](https://github.com/tommasorigon/India-SequentiaLogit/blob/master/Age_effect.png)


#### Random effects: graphs



```r
# Relabel the names of the random effects appropriately
colnames(fit1_dp_ranef_s$beta_RF) <- colnames(fit2_dp_ranef_s$beta_RF) <- colnames(fit3_dp_ranef_s$beta_RF) <- levels(dataset$state)[-1]

# Usage choice
data.plot <- melt(as.matrix(fit1_dp_ranef_s$beta_RF))
data.plot$k <- "Usage choice"
# Ordering the levels according to the median
data.plot$Var2    <- factor(data.plot$Var2, levels = levels(data.plot$Var2)[order(apply(fit1_dp_ranef_s$beta_RF,2, median))])
p1 <- ggplot(data = data.plot, aes(x = Var2, y = value)) + geom_boxplot(outlier.size = 0.6) + theme_bw() + theme(axis.text.x = element_text(angle = 50,hjust = 1)) + theme(legend.position = "none")+ xlab("") + ylab("State effect") + facet_grid(~k) 


# Reversibility choice
data.plot <- melt(as.matrix(fit2_dp_ranef_s$beta_RF))
data.plot$k <- "Reversibility choice"
# Ordering the levels according to the median
data.plot$Var2    <- factor(data.plot$Var2, levels = levels(data.plot$Var2)[order(apply(fit2_dp_ranef_s$beta_RF,2, median))])
p2 <- ggplot(data = data.plot, aes(x = Var2, y = value)) + geom_boxplot(outlier.size = 0.6) + theme_bw() + theme(axis.text.x = element_text(angle = 50,hjust = 1)) + theme(legend.position = "none")+ xlab("") + ylab("State effect") + facet_grid(~k) 

# Method choice
data.plot <- melt(as.matrix(fit3_dp_ranef_s$beta_RF))
data.plot$k <- "Method choice"
# Ordering the levels according to the median
data.plot$Var2    <- factor(data.plot$Var2, levels = levels(data.plot$Var2)[order(apply(fit3_dp_ranef_s$beta_RF,2, median))])
p3 <- ggplot(data = data.plot, aes(x = Var2, y = value)) + geom_boxplot(outlier.size = 0.6) + theme_bw() + theme(axis.text.x = element_text(angle = 50,hjust = 1)) + theme(legend.position = "none")+ xlab("") + ylab("State effect") + facet_grid(~k)
```

![State effect](https://github.com/tommasorigon/India-SequentiaLogit/blob/master/State_1.png)
![State effect](https://github.com/tommasorigon/India-SequentiaLogit/blob/master/State_2.png)
![State effect](https://github.com/tommasorigon/India-SequentiaLogit/blob/master/State_3.png)

#### Random effects: clustering States
