---
layout: post
title: "Hypothesis Testing by Computational Methodology - Part 1"
date: 2020-09-20
author: "Luciano"
featuredImage: "img/testing.png"
tags:
  - Hypothesis Testing
  - Data Science
  - Statistics
  - R
  - Infer
categories: [Tutorials]
---

# Introduction

This is the first of two articles that we'll talk about two different approaches to perform hypotheses tests, covering the _classical_ and _computational_ methodologies. In the end I'll show you one R package (`Infer`) capable to execute any of these methods in an easy, flexible, and less error-prone way.

In the second article, we'll go deeper in a hands-on experiment using the Infer package, if you already know the package and want to see more code than text, click **here**.

# What is Hypothesis Test?

Before start to talk about the hypothesis test I think that it's important to know where we are in the 'StatsVerse'. One didactic way is to start looking for the two major areas of Statistics:

- **Descriptive Statistics:** correspond to methods for collecting, describing, and analyzing a set of data _without drawing any conclusions_ about a large group.

- **Inferential Statistics:** correspond to methods for the analysis of a subset of data leading to _predictions or inferences about the entire set of data._

That been said, hypothesis tests are part of Inferential Statistics and allow us to take a sample of data from a population and infer about the plausibility of competing hypotheses. Sadly, here we have a lot of terminologies, notations, and definitions, and all that is important to understand the following topics of the article, because of this I prepare the table bellow with some short explanations you can use, if needed, to remember some of these concepts.

![Plot](/img/hp_04.jpeg)

Now you already refresh your mind about the hypothesis test, we can think about how to perform them. In that case, there is a classical methodology, based on very well known tests (t, Z, Chi-squared...), and there is the other one I didn't know until coming across with this incredible article **"There is Only one Test"**, based on a general framework and resampling techniques (permutation and bootstrap), called _computational methodology_.

So, you may ask me _"Why should I care about this computational approach?"_. And that is the answer I intend to give you at the end of the article =D.

Let's see now how we can proceed in order to perform a hypothesis test using these different tests.

# Classical Methodology

The way that you do a hypothesis test by the classical approach is based on these following steps:

1. State the hypotheses.
1. **Identify the test statistic and its probability distribution.**
1. Specify the significance level.
1. State the decision rule.
1. Collect the data and perform the calculations.
1. Make the statistical decision.
1. Make the business decision.

You probably noticed that the second step is bold, and that is because the most difficult part is identifying how the test should be used based on the data you have, it's not a trivial task, and you may probably end up googling and find images like this:

![Plot](/img/hp_1.jpg)

or this:

![Plot](/img/hp_2.jpg)

or even worse...

All these tests are based on assumptions that you need to follow in order to get a significative response. Otherwise, your answer will be misleading, that is, you may find a relationship between variables that is not true and vice-versa.

So, if existed one way to generalize this second step? Would be great, right? In fact, there is and we go see now!

# Computational Methodology

This computational methodology is based on a framework developed by Allen Downey, a computer scientist, author of Think Stats, and other books. Allen is a Professor of Computer Science at Olin College of Engineering. He has a Ph.D. in Computer Science from U.C. Berkeley and Master's and Bachelor's degrees from MIT.

He observed a pattern in the way that hypotheses tests are applied and came up with one more generalized approach to perform these tests.

The author believes that when computation was expensive, the shortcuts offered by all kinds of statistical tests were very important and necessary to calculate p-values, but now that computation is in a completely different level, they lost your weight.

The solution brought by him is to use **simulation**. That is, if you can construct a model of the null hypothesis, you can estimate p-values just by counting!

**Let's see how it works!!**

## Framework

The figure below it's showing the framework proposed by Allen Downey.

![Plot](/img/hp_3.png)

In order to describe the elements of this framework, I'll put here what was wrote by the author:

1. _Given a dataset, you compute a test statistic that measures the size of the apparent effect. For example, if you are describing a difference between the two groups, the test statistic might be the absolute difference in means. He calls the test statistic from the observed data δ_.\*

1. _Next, you define a null hypothesis, which is a model of the world under the assumption that the effect is not real. For example, if you think there might be a difference between the two groups, the null hypothesis would assume that there is no difference._

1. _Your model of the null hypothesis should be stochastic. That is, capable of generating random datasets similar to the original dataset._

1. _Now, the **goal of classical hypothesis testing is to compute a p-value**, which is the probability of seeing an effect as big as δ under the null hypothesis. You can estimate the p-value by using your model of the null hypothesis to generate many simulated datasets. For each simulated dataset, compute the same test statistic you used on the actual data._

1. _Finally, **count the fraction of times the test statistic from simulated data exceeds** δ. This fraction approximates the p-value. If it's sufficiently small, you can conclude that the apparent effect is unlikely to be due to chance._

That's it, all hypothesis tests fit into this framework.

## Pros & Cons

Like everything we always have pros and cons, and here there is no difference.

| Pros             | Cons                            |
| :--------------- | :------------------------------ |
| Easy             | Easy just for programmers       |
| Less error-prone | Big data can take longer to run |
| Flexible         |                                 |
| Fast enough      |                                 |

Look that these two _cons_ can be easily managed if you want:

- if you are not comfortable with programming, the `Infer` package makes it easier to execute this framework.
- although be fast, if you are working with big data, tasks like permutation can be very computationally expensive. But, nothing stops you to get a sample from your big dataset to perform hypotheses tests.

I'm not saying that the classical methodology it's not necessary anymore, but just showing another way (less error-prone) we can take advantage of all computational power to optimize this task.

## Infer Package

This package was developed by Andrew Bray and collaborators, and the objective is exactly to perform statistical inference using an expressive statistical grammar that coheres with the design framework showed before. The package is centered around 4 main verbs, supplemented with many utilities to visualize and extract value from their outputs.

- `specify()`: allows you to specify the variable or relationship between variables, that you’re interested in.
- `hypothesize()`: allows you to declare the null hypothesis.
- `generate()`: allows you to generate data reflecting the null hypothesis.
- `calculate()`: allows you to calculate a distribution of statistics from the generated data to form the null distribution.

The flow is straightforward, like the image below:

![](/img/hp_5.png)

To learn more about the principles underlying the package, stay tuning to the next article!

**Sources**

- [Allen Downey article](http://allendowney.blogspot.com/2016/06/there-is-still-only-one-test.html)
- [Hypothesis testing theory](https://moderndive.com/9-hypothesis-testing.html#understanding-ht)
- [Andrew Bray RStudio talk](https://rstudio.com/resources/rstudioconf-2018/infer-a-package-for-tidy-statistical-inference/)
