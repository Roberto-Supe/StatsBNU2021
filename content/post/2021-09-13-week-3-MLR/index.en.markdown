---
title: 'Week 3: Multiple Linear Regression'
author: Roberto Supe
date: '2021-09-22'
slug: []
categories: []
tags: []
lastmod: '2021-09-13T07:29:30+08:00'
keywords: []
description: ''
comment: no
toc: yes
autoCollapseToc: no
postMetaInFooter: no
hiddenFromHomePage: no
contentCopyright: no
reward: no
mathjax: no
mathjaxEnableSingleDollar: no
mathjaxEnableAutoNumber: no
hideHeaderAndFooter: no
flowchartDiagrams:
  enable: no
  options: ''
sequenceDiagrams:
  enable: no
  options: ''
---



# Multiple Linear Regression

This week, we will cover multiple linear regression. As in the previous post and in any model you want to do, we will need to test our assumptions and follow some general steps. 

Procedure to perform Multiple Linear Regressions:

1. Scatterplot matrix
1. Fit multiple linear regression models
1. Calculation estimate parameters, determine their standard errors, obtain tests and confidence intervals.
1. Evaluation and explanation

<!--more-->

Let's have our data


```r
library(PerformanceAnalytics)
library(tidyverse)
library(psych)
set.seed(21) #so we get the same results even with a random selection
df<-data.frame(temperature=rnorm(100,mean = 25,sd=5),
               wind=rnorm(100,mean = 10,sd=1.2),
               radiation=rnorm(100,mean = 5000,sd=1000),
               bacteria=rnorm(100,mean = 50,sd=15),
               groups=c("Day","Night")) #groups
head(df,2)
##   temperature     wind radiation bacteria groups
## 1    28.96507 9.026934  5748.074 32.28645    Day
## 2    27.61126 6.467452  3630.011 31.57719  Night
```


## MLR Assumption (Independency)

Simple linear regression does not require this test because it has one independent variable. This chapter tests the linearity and independence of observations (autocorrelation) to determine that independent variables are not too highly correlated.


```r
cor(df$temperature,df$wind) #function to test the relationship between your independent variables
## [1] -0.0069861
```

## Normality and Linearity Assumptions 

We still need to test the other two assumptions before running our model. A function that will save us time is `chart.Correlation`, we can get **Marginal Response Plots** showing correlations, and the significance of those correlations (independency), histograms and distributions (normality), and bivariate scatter plot with a fitted line (linearity). Although this does not test the normal distribution, you can observe the distribution of your data for all your numeric variables and make some initial estimations with histograms. 


```r
chart.Correlation(df[,-5], #numeric data set
                  histogram = T, #True show histograms
                  method = "pearson") #correlation method "pearson", "kendall", "spearman"
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-3-1.png" width="672" />

Similarly, you can use `psych::corr.test` with a few variables and determine correlation and significance. What we used for the correlation plot in [week 1]().


```r
correlation.test <- psych::corr.test(df[,1:4],use = "pairwise",method = "spearman") #relationship between all your variables
my.correlation<-correlation.test$r # from the test get ($) the correlation values
p.values.correlation<-correlation.test$p # from the test get ($) the correlation p values
```

* Values close to 1 indicate that independent variables are nearly identical, meaning they do not meet the assumptions for multiple linear regression and should be treated or discarded.

## Regression fitting


```r
fit.lm<-lm(bacteria~.,data=df) #all the variables 
#fit.lm<-lm(bacteria~temperature+wind+radiation,data=df) same as before
summary(fit.lm)
## 
## Call:
## lm(formula = bacteria ~ ., data = df)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -44.829 -10.180   2.138  10.238  39.795 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 68.119654  19.742062   3.450 0.000836 ***
## temperature -0.304967   0.317770  -0.960 0.339638    
## wind        -1.469393   1.474082  -0.997 0.321385    
## radiation    0.000886   0.001674   0.529 0.597873    
## groupsNight -0.544984   3.265375  -0.167 0.867805    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 16.14 on 95 degrees of freedom
## Multiple R-squared:  0.0252,	Adjusted R-squared:  -0.01584 
## F-statistic: 0.6141 on 4 and 95 DF,  p-value: 0.6535
```

Breakdown of the element of the result:

* The estimates column `(Estimate)` gives you the change in y caused by each element of the regression. For `Intercept` or the value of `y` assuming that all the rest elements are 0, in this case, 68.12 bacteria units on each test. Similarly, `temperature/rest of the variables` show changes in y (population of bacteria) with one unit increase in `x`. In other words, the temperature in the full model reduces the bacteria population by -0.304 units for each increment in temperature if P < 0.05. Here it is not, so we can not reach that conclusion. 

* The standard error of the estimated values in the second column `Std. Error`.
* The test statistic `t value`.
* The p-value `Pr(>| t | )`, the probability of finding the given t-statistic, and therefore, the calculated `estimate` by chance.


## Model test (Homoscedasticity)

We should ensure that the model fitted meets this last assumption, the same variance, to maintain the model. The error term needs to be the same across all values of the independent variables.


```r
par(mfrow=c(2,2))
plot(fit.lm)
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-6-1.png" width="672" />

```r
par(mfrow=c(1,1))
```

Here we can see that the residual errors are constant across groups, and they are distributed evenly. Look at the `Normal Q-Q` (top right), which under homoscedasticity it should show points along the line.

## Leverage and extrapolation

Leverage h is a quantity solely dependent on the predictors for all the cases in the data without any respect for responses. Leverage hi is called the leverage of observation i. It indicates the **“pull”** an observation has on the regression fit. 

Let's look at that pull for different points.

<iframe height="1000" width="80%" frameborder="no" src="https://roberto-supe.shinyapps.io/leverage_statsBNU/"> </iframe>

>Cases with large leverages should be inspected to ensure no errors in the data or similar problems. We will show some methods to deal with them in the following posts.
