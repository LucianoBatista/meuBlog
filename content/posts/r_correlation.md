---
layout: post
title: "How to Perform Correlation Analysis in Time Series data using R?"
date: 2020-09-15
author: "Luciano"
featuredImage: "img/cor_ts.png"
tags:
  - Time Series
  - Data Science
  - Autocorrelation
  - R
categories: [Tutorials]
---

# What is it correlation analysis?

The concept of correlation is the same used in non-time series data: _identify and quantify the relationship between two variables._ Due to the continuous and chronologically ordered nature of time series data, there is a likelihood that there will be some degree of correlation between the series observations.

Measuring and analyzing the correlation between two variables, in the context of time series analysis, can be understood by two different aspects:

- Analyzing the correlation between a series and its lags, as some of the past lags may contain predictive information, which can be utilized to forecast events of the series. One of the most popular methods for measuring the level of correlation between a series and its lags is the **autocorrelation function and partial autocorrelation function.**

- Analyzing the correlation between two series in order to identify exogenous factors or predictors, which can explain the variation of the series over time. In this case, the measurement of correlation is typically done with the **cross-correlation function**.

Looking at these characteristics can be very useful to find new features to use in the modeling step, also to understand patterns of behavior throughout the time.

# How to Perform Correlation Analysis in Time Series Data Using R?

Finding a way to do this in R can be overwhelming, because of the vast quantity of packages, and each package use one different kind of object. The objective of this article is to walk you through three different ways of doing the correlation analysis, which the last one is a general (tidy) way, and also what I prefer.

A short disclaimer before start, this article is not meant to explain all the theory behind the correlation analysis and for a more complete explanation, you can access this great reference: [Forecasting: Principles and Pratice](https://otexts.com/fpp3/graphics.html).

# The Different Approaches

Let's set our environment:

```r
# packages for this tutorial
# first approach
library(feasts)
library(tsibble)
library(lubridate)

# second approach
library(TSstudio)
library(plotly)

# third approach
library(tidyverse)
library(timetk)
library(lubridate)
```

## Data

We'll use a dataset from Stack Overflow, that have the numbers of questions for each month from 2009 to 2019, in different topics. You can access the data using this [link](https://www.kaggle.com/aishu200023/stackindex).

The dataset contains 9 different features regarding keywords used in Stack Overflow questions, but here, we'll use just `r` and `python` columns.

```r
# reading the dataset
stackoverflow_raw <- read_csv("00_data/MLTollsStackOverflow.csv")

# selecting columns to use in this tutorial
stackoverflow_tbl <- stackoverflow_raw %>%
  select(month, r, python)

```

I'll do one basic transformation in this dataset that must help-us during the tutorial and in all methods.

```r
# convert to date
stackoverflow_prep_tbl <- stackoverflow_tbl %>%
  mutate(date = str_glue("20{month}"),
         date = yearmonth(date) %>% as_date()) %>%
  select(date, r, python, -month)

```

```
# A tibble: 132 x 3
   date           r python
   <date>     <dbl>  <dbl>
 1 2009-01-01     8    631
 2 2009-02-01     9    633
 3 2009-03-01     4    766
 4 2009-04-01    12    768
 5 2009-05-01     2   1003
 6 2009-06-01     5   1046
 7 2009-07-01    51   1165
 8 2009-08-01    47   1143
 9 2009-09-01   139   1169
10 2009-10-01    73   1424
# … with 122 more rows
```

Now we can start!!

## First Approach

The first method that I want to show you use the `ts` object, a built-in format in R for time series. I choose this approach because with the `ts` object we have a good integration with `TSstudio` package and nice interactive functions to use.

```r
# ts function is responsible to convert to ts object
stackOverflow_ts <- ts(data = stackoverflow_prep_tbl[, c("python", "r")], # selecting 2 variables
   start = c(2009, 01), # start date
   end = c(2019, 12), # end date
   frequency = 12)

```

Now you can see all information about the time series that was created with the `ts` function.

```r
ts_info(stackOverflow_ts)
```

```
 The stackOverflow_ts series is a mts object with 2 variables and 132 observations
 Frequency: 12
 Start time: 2009 1
` End time: 2019 12
```

With this summary, it's possible to see that we have two variables, that represent two different time series. Each time series represent one feature (r and python).

Lets visualize the time series.

```r
ts_plot(stackOverflow_ts,
        title = "Monthly questions on Stack Overflow platform",
        Ytitle = "# Questions",
        Xtitle = "Year")

```

![Plot](/img/ts_01.png)

With this graph, we see that python has a more pronounced trend than R regarding the number of questions made in the stack overflow platform. The time series doesn't have a seasonal pattern, but it's possible to see some cyclicality in both time series.

Let's look at the correlation plots!

### Lags Analysis I

The `TSstudio` package doesn't have any function to plot the ACF and PACF measures, and for that, we use the built-in functions `acf` and `pacf`. The downside here, for me, is that the object generated by these functions is not a `ggplot2` object, and we lose all beautiful features from the grammar of graphics.

Obviously, you can go through the data and try to plot in the `ggplot2` by yourself but will be a little burdensome.

For R time series:

```r
par(mfrow = c(1, 2))
# acf R time series
stackOverflow_ts[, c("r")] %>%
  acf(lag.max = 300,
      main = "Autocorrelation Plot - R")

# pacf R time series
stackOverflow_ts[, c("r")] %>%
  pacf(lag.max = 300,
       main = "Partial Autocorrelation Plot - R")
```

![Plot](/img/ts_02.png)

For Python time series:

```r
# acf Python time series
stackOverflow_ts[, c("python")] %>%
  acf(lag.max = 300,
      main = "Autocorrelation Plot - Python")

# pacf Python time series
stackOverflow_ts[, c("python")] %>%
  pacf(lag.max = 300,
       main = "Python Autocorrelation Plot - Python")
```

![Plot](/img/ts_03.png)

In all graphics we have similar characteristics that need explanation:

- **blue lines:** These lines are referent to a significance interval, the bars that run through these lines have statistical meaning.
- **y-axis:** Correlation scores.
- **x-axis:** Here we have the indication of the lags. Each lag corresponds 12 months, so, almost 11 lags.

Note that looking at ACF plots, both for R and Python time series, we have a greater correlation with more recent lags, which is lost over time. With more distant lags it is possible to see that we have a negative correlation with recent data.

The Partial Autocorrelation is a little different, this "partial" correlation between two variables is the amount of correlation between them which is not explained by their mutual correlations with a specified set of other variables.

**For example**, if we are regressing a variable Y on other variables X1, X2, and X3, the partial correlation between Y and X3 is the amount of correlation between Y and X3 that is not explained by their common correlations with X1 and X2.

To summarize, when we look at the PACF plot, we want to know each lag that has relevant information to use as a predictor in a future forecast. How much greater the PACF score, the better.

And, in our plots, we see that both time series have one lag (closer to lag 1) that may be useful.

Just know which correlation score has the higher score is not enough, it's important to see visually how these points are distributed. In that, `TSstudio` has a great interactive function, called `ts_lags`.

```r
# Looking at lag plots
ts_lags(stackOverflow_ts,
        lags = c(2, 20, 30, 40, 50, 80)) %>% # choosing what lags to plot
  layout(title = "Series vs Lags")
```

![Plot](/img/ts_04.png)

### Causality analysis I

Just look at the past pattern within the series is not always a good idea. The main pitfall of this method is that it will fail whenever the changes in the series derive from the exogenous factors. The goal of causality analysis, in the context of time series analysis, is to identify whether a causality relationship exists between the series we wish to forecast other potential exogenous factors.

Be careful about the fact that correlation doesn't imply causation, we just looking for something that could help the forecast model. In our current case, we just can see if exist some lag in R time series correlated to Python time series.

```r
# ccf time series
par(mfrow=c(1,1))
ccf(stackOverflow_ts[, c("r")], stackOverflow_ts[, c("python")],
    lag.max = 300,
    main = "Cros-Correlation Plot",
    ylab = "CCF")

```

![Plot](/img/ts_05.png)

We see that the higher score is in the recent lags, and some scores between 5 and 10 have a negative relationship.

The graphic is saying to us that currently (at the date of the dataset) the growth of R questions is highly correlated to Python questions.

## Second Approach

This second approach uses what is called tsibble, like a tibble with an implicit date index. So, let's convert our tibble to a tsibble.

```r
stackoverflow_prep_tsbl <- stackoverflow_prep_tbl %>%
  mutate(date = yearmonth(date)) %>%
  pivot_longer(cols = c(r, python), names_to = "names", values_to = "value") %>%
  as_tsibble(key = names,index = date) # specifying the time series in the object
```

Differently of what we did before, here the data was converted to a long format, better to plot.

Looking at the object created, we see the indication of an interval of these observations [1M] (monthly). And also, we see the key variable, that it's a way to indicate how many time series exist in this object. It's possible to have different combinations of features to do a new time series in the object.

As we have the objects ready, we can now perform the correlation analysis by this second approach. Attempt that the graphic interpretation was already made, and here I'll just explain the difference between the methods.

### Lag Analysis II

Unlike `TSstudio`, we do not have interactivity plots, all plots are static, but they are objects of ggplot2 and allow us to use the concepts of the grammar of graphics.

A problem with these functions is that they may behave differently than expected. Plugging the data to the acf and pacf functions, we are able to automatically see the faceted plots between all time series within the object. But if we try to simply visualize the series over time, it will not be possible to facet. It's not a big problem, and you can plot using ggplot normally (see the code).

```r
# looking at the data
stackoverflow_prep_tsbl %>%
  feasts::autoplot()
```

![Plot](/img/ts_06.png)

```r

# looking at the data facet way
stackoverflow_prep_tsbl %>%
  ggplot(aes(x = date,
             y = value,
             color = names)) +
  geom_line() +
  facet_wrap(~names, scales = "free_y")
```

![Plot](/img/ts_07.png)

```r

# lag plots
stackoverflow_prep_tsbl %>%
  filter(names == "python") %>%
  gg_lag(value, geom="point")

```

![Plot](/img/ts_08.png)

```r

# acf
stackoverflow_prep_tsbl %>%
  ACF(value, lag_max = 300) %>%
  autoplot()
```

![Plot](/img/ts_09.png)

```r
# pacf
stackoverflow_prep_tsbl %>%
  PACF(value, lag_max = 300) %>%
  autoplot()
```

![Plot](/img/ts_10.png)

### Casuality Analysis II

```r
# ccf
stackoverflow_prep_tsbl %>%
  pivot_wider(names_from = names, values_from = value) %>%
  CCF(python, r, lag_max = 300) %>%
  autoplot()

```

![Plot](/img/ts_11.png)

## Third Approach (Tidy Approach)

Until now we have changed our original object several times, but using the `timetk` package you can perform all these analyzes using just the original object, a tibble data frame. Making analysis much more intuitive, and also integrating with all `tidyverse` packages.

### Lag Analysis III

You basically use 2 functions to perform all the correlation analysis with much less code and more flexibility. The only thing that you need to do is put the dataset in a long format.

```r
# passing to a long format
stackoverflow_prep_long_tbl <- stackoverflow_prep_tbl %>%
  pivot_longer(cols = c(r, python), names_to = "names", values_to = "value")
```

```r
# the series
stackoverflow_prep_long_tbl %>%
  plot_time_series(.date_var = date,
                   .value = value,
                   .facet_vars = names)
```

![Plot](/img/ts_12.png)

```r
# acf/pacf plots
stackoverflow_prep_long_tbl %>%
  group_by(names) %>%
  plot_acf_diagnostics(.date_var = date,
                       .value = value,
                       .show_white_noise_bars = T)
```

![Plot](/img/ts_13.png)

### Causality Analysis III

In the real world, the time series you want to compare will probably not be in the same range as here, and you will probably need to do some transformation to put in the same interval and only then calculate the ccf score.

```r
# CCF plot
stackoverflow_prep_tbl %>%
  plot_acf_diagnostics(.date_var = date,
                       .value = r,
                       .ccf_vars = python,
                       .show_ccf_vars_only = T)


```

![Plot](/img/ts_14.png)

So, the main takeaways in this tutorial are that the `timetk` package is a great boost for your exploratory data analysis in time series context. The functions are consistent and we have interactivity in all plots if we want.

`timetk` is able to do also, other different analysis:

- Seasonality
- Anomaly
- STL Decompostion
- Regression Plots

I hope you find something useful with this tutorial, if you want to contact me, check the links:

- [LinkedIn](https://www.linkedin.com/in/lucianobatistads/)
- [gitHub](https://github.com/LucianoBatista)

=]
