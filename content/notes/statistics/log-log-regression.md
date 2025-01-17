---
title: log log regression
creation date: 2021-06-14 09:37
---

_sourced somewhere from Twitter_

When using linear regression, when should you log-transform your data?

Many people seem to think that any non-Gaussian, continuous variables should be transformed so that the data “look more normal.” Linear regressionrégression does in fact assume the errorsles erreurs are normally distributed, but it is fairly robust to violations of this assumption, and there are no such assumptions regarding the predictor variables. What is often ignored or misunderstood is the impact that variable transformations have on the linearity assumption of regression models, and on coefficient interpretation.

The are a variety of optionsoptio for transforming data, and simply taking the logarithim may be the most popular, given that your data doesn’t include valuesvaleurs equal to zero. We will thus focus on linear regression when the outcome and one predictor are both log transformed.

## Data

The data for this exercise are from a small sample of older, in-hospital patients with information on average daily step count (measured over 5 days with a pedometer) and their length of stay in the hospital. The data can be down loaded from github.

```
  library(ggfortify)
  library(ggplot2)
  library(readr)
  library(dplyr)
  library(tidyr)
  library(viridis)
  library(ggthemes)
  library(ggalt)
  library(car)

  data <- read_csv("https://dantalus.github.io/public/steps.csv")
```

## Variables

The distributions for average daily step count (Steps) and hospital length of stay (LOS) and their repective log transformed values are plotted below. The original values are right skewed and bounded on the left side at zero. As expected, the log transformed values are more symetrical.

```
  gather(data, variable, value, avg.steps, los,  log.avg.steps, log.los) %>%
  mutate(variable = factor(variable, levels = c("avg.steps", 
                                                "log.avg.steps", 
                                                "los", 
                                                "log.los"))) %>%
  ggplot(aes(x = value, fill = variable)) +
    geom_bkde() +
    geom_rug() +
    scale_fill_viridis(guide = FALSE, discrete = TRUE) +
    facet_wrap(~variable, scales = "free") +
    theme_base()
```

![](https://darrendahly.github.io/post/loglog_files/figure-html/unnamed-chunk-3-1.svg)

## Linearity

The real challenge however is that the relationship between Steps and LOS is clearly not linear. This is illustrated in the plot below, where the solid line is from the linear regression of LOS on Steps, and the dashed line is from a loess smoother.

```
  ggplot(data, aes(x = avg.steps, y = los)) +
    geom_jitter() +
    geom_smooth(method = "lm", color = viridis(1, begin = 1),   se = FALSE) +
    geom_smooth(span   = 1,    color = viridis(1, begin = 0.6), se = FALSE, linetype =
                "dashed") +
    theme_base()
```

![](https://darrendahly.github.io/post/loglog_files/figure-html/unnamed-chunk-4-1.svg)

## Linear regression

We can estimate the linear regression shown in the previous plot as follows:

```
  lr <- lm(los ~ avg.steps, data)
  
  summary(lr)
```

```
## 
## Call:
## lm(formula = los ~ avg.steps, data = data)
## 
## Residuals:
##    Min     1Q Median     3Q    Max 
## -7.632 -3.650 -1.227  1.623 19.275 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  9.4672380  0.6484016  14.601  < 2e-16 ***
## avg.steps   -0.0017605  0.0006241  -2.821  0.00546 ** 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.342 on 146 degrees of freedom
##   (6 observations deleted due to missingness)
## Multiple R-squared:  0.05168,    Adjusted R-squared:  0.04518 
## F-statistic: 7.956 on 1 and 146 DF,  p-value: 0.00546
```

## Diagnostic plots

```
  autoplot(lr, which = 1:6, ncol = 3, label.size = 3)
```

![](https://darrendahly.github.io/post/loglog_files/figure-html/unnamed-chunk-6-1.svg)

## Log transformed variables

As we saw above, the distributions of Steps and LOS “look more normal” after transformation. More importantly however, the relationship between the log transformed variables is also linear.

```
  ggplot(data, aes(x = log.avg.steps, y = log.los, group)) +
    geom_jitter(alpha = 0.5) +
    geom_smooth(method = "lm", color = viridis(1, begin = 1),   se = FALSE) +
    geom_smooth(span   = 1,    color = viridis(1, begin = 0.6), se = FALSE, 
                linetype = "dashed") +
    theme_base()
```

![](https://darrendahly.github.io/post/loglog_files/figure-html/unnamed-chunk-7-1.svg)

Note that the extreme outlier in the first scatter plot is not an outlier in log-log space.

## Log-Log linear regression

A regression model where the outcome and at least one predictor are log transformed is called a log-log linear model. Here are the model and results:

```
  log.log.lr <- lm(log.los ~ log.avg.steps, data)
  
  summary(log.log.lr)
```

```
## 
## Call:
## lm(formula = log.los ~ log.avg.steps, data = data)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -1.92304 -0.44851  0.00306  0.32693  1.39440 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept)    3.01997    0.29400  10.272  < 2e-16 ***
## log.avg.steps -0.17800    0.04653  -3.825 0.000193 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.5748 on 146 degrees of freedom
##   (6 observations deleted due to missingness)
## Multiple R-squared:  0.09109,    Adjusted R-squared:  0.08486 
## F-statistic: 14.63 on 1 and 146 DF,  p-value: 0.0001931
```

## Diagnostic plots

```
  autoplot(log.log.lr, which = 1:6, ncol = 3, label.size = 3)
```

![](https://darrendahly.github.io/post/loglog_files/figure-html/unnamed-chunk-9-1.svg)

## Ploting the results in the original scales

We will again scatter plot the Steps and LOS variables with fit lines, but this time we will add the line from the log-log linear regression model we just estimated. Importantly, the regression line in log-log space is straight (see above), but in the space defined by the original scales, it’s curved, as shown by the purple line below.

```
  ggplot(data, aes(x = avg.steps, y = los)) +
    geom_jitter(alpha = 0.5) +
    geom_smooth(method = "lm", color = viridis(1, begin = 1), se = FALSE, 
                linetype = "dashed") +
    geom_line(data = data.frame(x = exp(log.log.lr$model$log.avg.steps),
                                y = exp(predict(log.log.lr))),
              aes(x = x, y = y),
              color = viridis(1, end = 0), size = 0.7) +
    geom_smooth(span = 1, color = viridis(1, begin = 0.6), size = 0.7, linetype = "dashed",
                se = FALSE) +
    theme_base()
```

![](https://darrendahly.github.io/post/loglog_files/figure-html/unnamed-chunk-10-1.svg)

## Model interpretation

So how do we interpret the regression coefficients from a log-log model? The best explanation I have found for interpreting the regression coefficients can be found here: [http://www.kenbenoit.net/courses/ME104/logmodels2.pdf](http://www.kenbenoit.net/courses/ME104/logmodels2.pdf)

In a nutshell, a 1% increase in the predictor is associated with a Beta% change in the outcome. This is an approximation however. To be exact, we can say that an x% increase in the predictor is associated with a change in the outcome equivalent to multiplying it by 2.71^((log((100+x)/100)) \* Beta)). For the model above, the approximation is a 1% increase in Steps is associated with an approximately 0.18% decrease in LOS. Using the exact method, we can calculate the a 50% increase in Steps is associated with a 2.71^((log((100 + 50)/100)) \* -0.18)) = 0.93 \* LOS = a 7% decrease in LOS.

The important thing to understand is that by working with logarithms, we have moved from talking about absolute differences (e.g. 600 steps = 1 less day in the hospital) to relative differences (given a 1% increase in Steps, we would expect an 0.18% decrease in hospital length of stay). Why is this? Remember from math class that adding log(x) + log(y) = log(x \* y)? When we start adding things in log space, we are multiplying them in absolute space. This is the same reason that a one-unit increase in log(odds) space results in an odds-ratio after exponentiation in logistic regression models. This shift from thinking in absolute to thinking in relative terms is important for understanding a number of analytical techniques (e.g. what is the difference between an additive and a multiplicative interaction?). Here is a entertaining introduction to the topic: [https://www.youtube.com/watch?v=Pxb5lSPLy9c](https://www.youtube.com/watch?v=Pxb5lSPLy9c).

One last, critical point we must consider is whether the proportional interpretation is reasonable for each variable. In the case of Steps, it makes sense to me that an increase of 50 steps per day would mean something different for a completely sedentary patient vs. a very active patient. How to think about LOS is less obvious to me. Do I expect a change in Steps to be associated with same amount of change in LOS regardless of how long the stay would have been otherwise? I’ll have to ask the physiotherapist about this one.