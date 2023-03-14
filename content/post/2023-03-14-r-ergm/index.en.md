---
title: R语言ERGM教程
author: Keping Wang
date: '2023-03-14'
slug: r-ergm
categories:
  - R
  - ERGM
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2023-03-14T19:36:13+08:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---



##  ERG modeling

This is an R Markdown document about how to model ERGM.

You can embed an R code chunk like this:



```r
setwd("E:\\Office Account\\2021\\指数随机图模型2")
# 加载包
library(network)
library(sna)
library(ergm)
library(readxl)
## Step1:导入数据
data1 <- read_xlsx("data1.xlsx")
data1 <- as.matrix(data1)
# 利用network包将原始数据转化为network对象
net1 <- as.network(data1,directed = F)
class(net1)
## [1] "network"
# 简单的描述统计：度数分布
summary(net1 ~ degree(1) + degree(2) + degree(3))
## degree1 degree2 degree3 
##       6       2       1
# 利用sna包的gplot函数进行可视化，默认的可视化方式是fruchtermanreingold
# 默认的可视化方式是fruchtermanreingold,可以调整mode参数进行更改
# 另外：gmode参数指定--单向、无向,具体可以参照sna包gplot函数
gplot(net1,gmode = "graph")
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-1-1.png" width="672" />

```r
# Step2:建模
# (1)基准模型
model1 <- ergm(net1 ~ edges,
               control=control.ergm(MCMC.samplesize=100000, 
                                    MCMC.burnin=1000000, 
                                    MCMC.interval=1000, 
                                    seed = 567))
summary(model1)
## Call:
## ergm(formula = net1 ~ edges, control = control.ergm(MCMC.samplesize = 1e+05, 
##     MCMC.burnin = 1e+06, MCMC.interval = 1000, seed = 567))
## 
## Maximum Likelihood Results:
## 
##       Estimate Std. Error MCMC % z value Pr(>|z|)    
## edges   -1.395      0.215      0  -6.492   <1e-04 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
##      Null Deviance: 188.5  on 136  degrees of freedom
##  Residual Deviance: 135.6  on 135  degrees of freedom
##  
## AIC: 137.6  BIC: 140.5  (Smaller is better. MC Std. Err. = 0)

# (2)加入内生结构项
# 注：网络内生项可以根据模型调试加入，也可以通过模体分析再加入
model2 <- ergm(net1 ~ edges + gwesp(0.1,T),
               control=control.ergm(MCMC.samplesize=100, 
                                    MCMC.burnin=100, 
                                    MCMC.interval=100, 
                                    seed = 567))

# (3)加入节点属性
node_attr <- read_xlsx("patent.xlsx")
net2 <- network(net1,vertex.attr = node_attr,
                vertex.attrnames = "patent",
                directed = F)
summary(net2)
## Network attributes:
##   vertices = 17
##   directed = FALSE
##   hyper = FALSE
##   loops = FALSE
##   multiple = FALSE
##   bipartite = FALSE
##  total edges = 27 
##    missing edges = 0 
##    non-missing edges = 27 
##  density = 0.1985294 
## 
## Vertex attributes:
## 
##  patent:
##    numeric valued attribute
##    attribute summary:
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##    5.00   15.00   28.00   72.29   88.00  289.00 
##   vertex.names:
##    character valued attribute
##    17 valid vertex names
## 
## No edge attributes
## 
## Network edgelist matrix:
##       [,1] [,2]
##  [1,]    2    1
##  [2,]    3    1
##  [3,]    5    1
##  [4,]    7    1
##  [5,]    8    1
##  [6,]    9    1
##  [7,]    3    2
##  [8,]    5    2
##  [9,]    6    2
## [10,]    8    2
## [11,]    5    3
## [12,]    6    3
## [13,]    8    3
## [14,]    7    4
## [15,]    8    5
## [16,]    9    5
## [17,]   16    5
## [18,]    7    6
## [19,]   12    6
## [20,]   13    6
## [21,]   14    6
## [22,]   13    7
## [23,]   14    7
## [24,]   15   10
## [25,]   17   11
## [26,]   13   12
## [27,]   14   13
model3 <- ergm(net2 ~ edges + gwesp(0.1,T) + nodecov("patent"),
               control=control.ergm(MCMC.samplesize=100, 
                                    MCMC.burnin=100,  
                                    MCMC.interval=100,
                                    seed = 567))
summary(model3)
## Call:
## ergm(formula = net2 ~ edges + gwesp(0.1, T) + nodecov("patent"), 
##     control = control.ergm(MCMC.samplesize = 100, MCMC.burnin = 100, 
##         MCMC.interval = 100, seed = 567))
## 
## Monte Carlo Maximum Likelihood Results:
## 
##                  Estimate Std. Error MCMC % z value Pr(>|z|)    
## edges           -3.319307   0.587807      0  -5.647  < 1e-04 ***
## gwesp.fixed.0.1  0.855363   0.462981      0   1.848  0.06467 .  
## nodecov.patent   0.005110   0.001807      0   2.828  0.00468 ** 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
##      Null Deviance: 188.5  on 136  degrees of freedom
##  Residual Deviance: 116.5  on 133  degrees of freedom
##  
## AIC: 122.5  BIC: 131.2  (Smaller is better. MC Std. Err. = 0.2145)

# (4)加入外生协变量
tech_proximity <- read_xlsx("tech_proximity.xlsx",col_names = T)
tech_proximity <- as.matrix(tech_proximity)
net2 %e% "tech_proximity" <- tech_proximity
summary(net2)
## Network attributes:
##   vertices = 17
##   directed = FALSE
##   hyper = FALSE
##   loops = FALSE
##   multiple = FALSE
##   bipartite = FALSE
##  total edges = 27 
##    missing edges = 0 
##    non-missing edges = 27 
##  density = 0.1985294 
## 
## Vertex attributes:
## 
##  patent:
##    numeric valued attribute
##    attribute summary:
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##    5.00   15.00   28.00   72.29   88.00  289.00 
##   vertex.names:
##    character valued attribute
##    17 valid vertex names
## 
## Edge attributes:
## 
##  tech_proximity:
##    numeric valued attribute
##    attribute summary:
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##  0.7101  0.8946  0.9671  0.9301  0.9913  1.0000 
## 
## Network edgelist matrix:
##       [,1] [,2]
##  [1,]    2    1
##  [2,]    3    1
##  [3,]    5    1
##  [4,]    7    1
##  [5,]    8    1
##  [6,]    9    1
##  [7,]    3    2
##  [8,]    5    2
##  [9,]    6    2
## [10,]    8    2
## [11,]    5    3
## [12,]    6    3
## [13,]    8    3
## [14,]    7    4
## [15,]    8    5
## [16,]    9    5
## [17,]   16    5
## [18,]    7    6
## [19,]   12    6
## [20,]   13    6
## [21,]   14    6
## [22,]   13    7
## [23,]   14    7
## [24,]   15   10
## [25,]   17   11
## [26,]   13   12
## [27,]   14   13
model4 <- ergm(net2 ~ edges + gwesp(0.1,T) + 
               nodecov("patent") + edgecov(tech_proximity),
               control=control.ergm(MCMC.samplesize=100, 
                                    MCMC.burnin=100,  
                                    MCMC.interval=100,
                                    seed = 567))
# 模型有效性检验
# (1)拟合优度检验
# 以model4为例
model4_gof <- gof(model4,GOF = ~ degree + espartners+ dspartners,
                  verbose = T, burnin = 10000,
                  interval = 10000)
par(mfrow = c(2,3),mar = c(4,4,4,1),cex.main = .9,cex.lab = .9,cex.axis = .75)
plot(model4_gof,plotlogodds = T)
# (2)MCMC检验
mcmc.diagnostics(model4)
## Sample statistics summary:
## 
## Iterations = 3900:64000
## Thinning interval = 100 
## Number of chains = 1 
## Sample size per chain = 602 
## 
## 1. Empirical mean and standard deviation for each variable,
##    plus standard error of the mean:
## 
##                           Mean       SD Naive SE Time-series SE
## edges                    1.269    5.080   0.2071         0.3192
## gwesp.fixed.0.1          1.104    6.687   0.2726         0.4101
## nodecov.patent         146.668 1051.271  42.8466        74.0038
## edgecov.tech_proximity   1.153    4.761   0.1941         0.3011
## 
## 2. Quantiles for each variable:
## 
##                             2.5%      25%     50%     75%   97.5%
## edges                     -9.000   -2.000   1.000   5.000   11.00
## gwesp.fixed.0.1          -11.336   -3.515   1.056   5.879   13.73
## nodecov.patent         -1829.975 -498.000 140.500 861.000 2185.18
## edgecov.tech_proximity    -8.234   -2.012   1.074   4.326   10.51
## 
## 
## Are sample statistics significantly different from observed?
##                   edges gwesp.fixed.0.1 nodecov.patent edgecov.tech_proximity
## diff.      1.269103e+00     1.103813021   146.66777409           1.1531189085
## test stat. 3.976486e+00     2.691828444     1.98189527           3.8292122414
## P-val.     6.994115e-05     0.007106149     0.04749096           0.0001285541
##            Overall (Chi^2)
## diff.                   NA
## test stat.    3.836589e+01
## P-val.        2.817446e-07
## 
## Sample statistics cross-correlations:
##                            edges gwesp.fixed.0.1 nodecov.patent
## edges                  1.0000000       0.9327347      0.8562209
## gwesp.fixed.0.1        0.9327347       1.0000000      0.8499252
## nodecov.patent         0.8562209       0.8499252      1.0000000
## edgecov.tech_proximity 0.9983408       0.9358446      0.8649161
##                        edgecov.tech_proximity
## edges                               0.9983408
## gwesp.fixed.0.1                     0.9358446
## nodecov.patent                      0.8649161
## edgecov.tech_proximity              1.0000000
## 
## Sample statistics auto-correlation:
## Chain 1 
##                edges gwesp.fixed.0.1 nodecov.patent edgecov.tech_proximity
## Lag 0    1.000000000      1.00000000     1.00000000             1.00000000
## Lag 100  0.406884758      0.38645109     0.49725816             0.41244418
## Lag 200  0.181869851      0.17263880     0.26173079             0.18574456
## Lag 300  0.046687897      0.07951933     0.17795808             0.04866411
## Lag 400  0.033166206      0.04423706     0.14015072             0.03083211
## Lag 500 -0.007513333     -0.02708387     0.05220851            -0.01099762
## 
## Sample statistics burn-in diagnostic (Geweke):
## Chain 1 
## 
## Fraction in 1st window = 0.1
## Fraction in 2nd window = 0.5 
## 
##                  edges        gwesp.fixed.0.1         nodecov.patent 
##                 0.5817                 1.1409                 1.6424 
## edgecov.tech_proximity 
##                 0.6001 
## 
## Individual P-values (lower = worse):
##                  edges        gwesp.fixed.0.1         nodecov.patent 
##              0.5607818              0.2539058              0.1005105 
## edgecov.tech_proximity 
##              0.5484623 
## Joint P-value (lower = worse):  0.3738199 .
## 
## MCMC diagnostics shown here are from the last round of simulation, prior to computation of final parameter estimates. Because the final estimates are refinements of those used for this simulation run, these diagnostics may understate model performance. To directly assess the performance of the final model on in-model statistics, please use the GOF command: gof(ergmFitObject, GOF=~model).
## 至此指数随机图模型的三类变量都依次加入模型中
# 模型整理
library(texreg)
screenreg(list(model1,model2,model3,model4),digits = 3)
## 
## ========================================================================
##                         Model 1      Model 2      Model 3      Model 4  
## ------------------------------------------------------------------------
## edges                    -1.396 ***   -2.715 ***   -3.319 ***   -8.108 *
##                          (0.215)      (0.532)      (0.588)      (3.540) 
## gwesp.fixed.0.1                        1.089 *      0.855        0.889 *
##                                       (0.436)      (0.463)      (0.437) 
## nodecov.patent                                      0.005 **     0.004 *
##                                                    (0.002)      (0.002) 
## edgecov.tech_proximity                                           5.320  
##                                                                 (3.928) 
## ------------------------------------------------------------------------
## AIC                     137.553      129.828      122.490      122.340  
## BIC                     140.466      135.653      131.228      133.991  
## Log Likelihood          -67.777      -62.914      -58.245      -57.170  
## ========================================================================
## *** p < 0.001; ** p < 0.01; * p < 0.05
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-1-2.png" width="672" />
