---
layout: post
title: "Effective approach to analyze correlation coefficients"
subtitle: "Learn how to use corrplot and corrr packages"
date: 2020-10-13
author: "Luciano"
featuredImage: "img/autocorrelation.png"
tags:
  - Data exploration
  - Data visualization
  - ggplot2
  - corrr
  - corplot
  - R
categories: [Tutorials]
---

![Plot](/img/correlation/correlation_post.png)

**Correlation analysis** is a key task when you're exploring any dataset. The principal objective is to find linear relationships between features that can help to understanding the big picture.

Probably, the best way to see correlations between variables is to use scatterplots, but in most of time you're working with a high dimensional dataset with a high number of variables, in these situations you have two major problems:

- It's a high computational task to plot lots of scatterplot, specially if you have a big dataset.
- Even if you can plot all scatterplots at once, the readability of the charts will be horrible, and you'll not see nothing useful.

One great approach to solve this problem is calculating the coefficient of correlation and instead of having lots of scatterplots, you'll have a matrix showing how much it correlates your variables, assuming a range from -1 to 1, high negatively correlated to high positively correlated, respectively. This is definitely much faster to plot and easy to interpret.

### Misinterpretation of the coefficient of correlation

Before starting to code, it's important to understand two topics about correlation analysis to drift reliable conclusions.

#### First

Sometimes, we misinterpret the value of coefficient of correlation and establish the cause-and-effect relationship, i.e. one variable causing the variation in the other variable. Actually, we cannot interpret in this way unless we have a powerful motive beside just the coefficient value.

Correlation coefficient gives us a quantitative determination of a relationship between two variables X and Y, not information about the association between the two variables. Causation implies an invariable sequence --- A always leads to B --- whereas correlation is a measure of mutual association between two variables.

![Plot](/img/correlation/correlation_causation.png)

#### Second

Another aspect that we need to be aware of is the factors that influencing the size of the correlation coefficient and can also lead to misinterpretation, like:

- The size of the coefficient is very much dependent upon the variability of measured values in the correlated sample. The greater the variability, the higher will be the correlation, everything else being equal.
- The size of the coefficient is altered when an investigator selects an extreme group of subjects to compare these groups regarding certain behavior. The coefficient got from the combined data of extreme groups would be larger than the coefficient got from a random sample of the same group.
- Addition or dropping the extreme cases from the group can lead to a change in the coefficient's size. Addition of the extreme case may increase the size of correlation, while dropping the extreme cases will lower the value of the coefficient.

With that in mind, it's a good idea to do a standardized step before looking for correlations, to minimize variability and extreme values.

### How to do a correlation analysis in R?

As everything in R, here there are also plenty packages to calculate and plot coefficients of correlation. It's up to you to choose which package is better for your analysis.

The aim of this article is to help you structure a optimized workflow for correlation analysis, creating great charts and consistent functions. For that will be necessary, especially two specific packages, `corrplot` and `corrr`.

### Data

As a dataset for this tutorial, I'll use some data from a personal project in development. I collected the data from a public API and from an electronic game called **League of Legends**.

![lol](/img/correlation/lol.jpg)

_League of Legends_ is one of the most popular video games in the world. It is played by over 100 million active users every single month. Each team has a base they must guard from their opponents while simultaneously attacking their opponent's base, there is the Blue team, whose base is in the lower left part of the map, and the Red team, whose base is in the upper right part of the map, at the back of each team's base there is a building called The Nexus. You win the game by destroying the enemy team's Nexus.

During the match, there are a lot of statistics about performing each player (5 in each team) and also from the match itself, so, I created a script to collect all this information.

The game also has a ranking by country and according to your performance you get a specific tier, that can be: iron, bronze, silver, gold, platinum, diamond, master, grandmaster and challenger, following the batter to worse order, respectively.

Just to simplify the example, I'll show just the data for one particular tier of users, diamond players.

I choose this dataset because we have a lot of variables, high dimensionality, and look for linear correlations can be useful to build our model in the next stage of the project.

Let's begin!

```r
# libraries used in this tutorial
library(tidyverse)
library(corrr)
library(corrplot)
library(tidyquant) # color pallete
library(plotly)

# dataset cleaned
# download from kaggle: https://www.kaggle.com/dsluciano/league-of-legends-match-statistics/download
lol_diamond_tbl <- read_csv("data/lol_diamond_numeric_final.csv")


lol_diamond_tbl %>% glimpse()
```

```

    Rows: 99,727
    Columns: 68
    $ totalDamageDealt                 <dbl> -0.2930711279, 2.5773485960, 0.2419916679, 0.7503652702, 0.9952605522, -0.04…
    $ kills                            <dbl> 0.0153183, -0.2410509, 1.9523597, 0.9610752, 0.6312293, -0.5386355, 0.961075…
    $ deaths                           <dbl> -0.1644351, -0.5187119, -1.8349225, -0.9050923, -0.1644351, -0.9050923, -1.3…
    $ assists                          <dbl> -1.81311786, -0.06615988, -0.49817594, 0.46319384, 0.12308021, 0.61792164, 0…
    $ largestKillingSpree              <dbl> 0.32609942, -0.07849633, 2.84784752, 0.94628600, 0.32609942, -0.07849633, 0.…
    $ largestMultiKill                 <dbl> -0.472330, -0.472330, 0.865404, 0.865404, -0.472330, -0.472330, 0.865404, 0.…
    $ killingSprees                    <dbl> 0.68996109, -0.08011612, 0.68996109, 1.24585393, 0.68996109, -0.08011612, 1.…
    $ longestTimeSpentLiving           <dbl> 0.04987467, 1.71337317, -0.42769388, 1.73915199, 0.89870412, 1.01482638, 1.2…
    $ doubleKills                      <dbl> -0.7575416, -0.7575416, 1.6941396, 1.1340473, -0.7575416, -0.7575416, 1.5003…
    $ tripleKills                      <dbl> -0.2604183, -0.2604183, -0.2604183, -0.2604183, -0.2604183, -0.2604183, -0.2…
    $ quadraKills                      <dbl> -0.1065788, -0.1065788, -0.1065788, -0.1065788, -0.1065788, -0.1065788, -0.1…
    $ pentaKills                       <dbl> -0.04710454, -0.04710454, -0.04710454, -0.04710454, -0.04710454, -0.04710454…
    (...)
```

I cleaned the dataset before, and the script used for that is in my [GitHub repo](https://github.com/LucianoBatista/articles). Besides that, you can find the description of the variables used here on this [kaggle kernel](https://www.kaggle.com/dsluciano/league-of-legends-match-statistics/) created by me. If you have any questions, please let me know.

### Using corplot package

This is a well-known library, that R users normally choose. The downside from `corrplot` package is we don't have a ggplot object at the output, and because of that is difficult to customize the plot with something that you want.

But the procedure to plot the correlation coefficients using `corrplot` is fairly simple:

1.  calculate the correlation matrix
2.  apply the `corrplot()` function
3.  customize

Although the simplicity of this process, the final result it's not so beautiful and it's not the best chart to put in your report.

```r
# basic procedure
corr_matrix_train_mtx <- lol_diamond_tbl %>% cor()
corrplot(corr_matrix_train_mtx)
```

![lol](/img/correlation/corr_01.png)

One feature that most people don't know is the possibility to group variables, creating clusters regarding the correlation coefficient. Definitely this can improve the visualization and interpretability of the result.

In the following sequence of steps I'll show you how to improve the final result:

1.  Change the method parameter to put a square shape in the circle place (more easy to read). This way we can get the first plot.

2.  But as a personal taste I don't like the grid, and so I change from square to color (second plot).

3.  Here we have a problem, as a correlation matrix is mirrored, you end up with too much information to analyze. To solve that, we take just the inferior triangle and get the plot 3.

4.  We can agree that red text is not the better color to put in your chart, for that we can change tl.col to black and also decrease the font size with tl.cex (plot 4).

5.  And the most important, we can rearrange the variables using clustering methods, the corrplot function accept 5 different ways: AOE, FPC, hclust and alphabet (look the documentation to see more).

```r
# first plot
corrplot(corr_matrix_train_mtx, method = "square")

# second plot
corrplot(corr_matrix_train_mtx, method = "color")

# third plot
corrplot(corr_matrix_train_mtx, type = "lower", method = "color")

# fourth plot
corrplot(corr_matrix_train_mtx, method = "color", type = "lower",
         tl.col = "black", tl.cex = 0.5)

# final plot
corrplot(corr_matrix_train_mtx, method = "color", type = "lower",
         tl.col = "black", tl.cex = 0.5,
         order = "hclust",)
```

![lol](/img/correlation/corr_01_02.png) ![lol](/img/correlation/corr_03_04.png)

Look that the final result (figure below) really helps you see distinct groups of variables that have similar correlations, and can be a start point to investigate specific groups later.

![lol](/img/correlation/correlation_06.png)

### Using corrr package

This second approach is the most tidy way to perform a correlation analysis. To facilitate the readability of this workflow, I drew this flowchart below:

![flowchart](/img/correlation/correlation_07.png)

```r
# plot 1
lol_diamond_tbl %>%
    correlate() %>%
    rearrange() %>%
    shave() %>%
    # rplot need to receive a correlation matrix
    rplot()
```

It's incredibly straightforward, and you just need to tune specific parameters to achieve a astonish chart. The standard configuration is shown in the next figure:

![flowchart](/img/correlation/correlation_08.png)

Yes, it's a mess and difficult to read. Let's improve this plot following some customizations:

1.  Set the PCA method for rearrange the variables.

2.  Setting `shape = 15`, will put squares shape in circle places. You can also try different number to different shapes.

3.  I'll choose better colors. And at the end, I will set the x-axis label angle to 45 and `hjust = 1`, to put the axis text in right place.

```r
lol_diamond_tbl %>%
    correlate(use = "pairwise.complete.obs") %>%
    rearrange(method = "PCA") %>%
    shave() %>%
    # rplot need to receive a correlation matrix
    rplot(shape = 15, colours = c("darkorange", "white", "darkcyan")) +
    theme_minimal() +
    theme(
        axis.text.x = element_text(angle = 45, hjust = 1)
    )
```

This is a much better chart! =)

![flowchart](/img/correlation/correlation_09.png)

The `corrr` library can also use a lot of different clustering methods for rearranging the variables, and you can get the full list from the [seriate package documentation](https://www.rdocumentation.org/packages/seriation/versions/1.2-8/topics/seriate).

One more convenience in use `corrr` is that you can create `plotly` interactive plots and hover through specific squares to investigate the values of correlation coefficients and the variable in x and y axis [play the video](https://www.youtube.com/watch?v=4Xv4x7wklSM).

```r
g <- lol_diamond_tbl %>%
    correlate(use = "pairwise.complete.obs") %>%
    rearrange(method = "PCA") %>%
    corrr::shave() %>%
    # rplot need to receive a correlation matrix
    rplot(shape = 15, colours = c("darkorange", "white", "darkcyan")) +
    theme_minimal() +
    theme(
        axis.text.x = element_text(angle = 45, hjust = 1)
    )


plotly::ggplotly(g)

```

Now that you already have a fully understanding in how to plot a good chart to investigate the correlation between your variables, I'll show you one specific approach to a more applicable task in your modeling workflow.

See, the correlation between all variables is useful, but as we are investigating a correlation against one specific variable (target), a better way is to extract the correlation coefficients to that specific variable and plot it.

### Creating the plot_cor() function

In this last section, I'll show you two functions you can use to create beautiful plots and see which variable can contribute more to your correlation analysis.

First, we need the `get_cor()` function, that will return the correlation matrix. After, we plug into `plot_cor()` function and get our awesome chart.

```r

# getting the correlation matrix
get_cor <- function(data, target, use = "pairwise.complete.obs",
         fct_reorder = FALSE, fct_rev = FALSE) {

    # meta programming to capture the variables
    # like tidy functions
    feature_expr <- enquo(target)
    feature_name <- quo_name(feature_expr)

    # get the corralating matrix
    # and also ensuring that the data is in the
    # right format
    data_cor <- data %>%
        mutate(across(where(is.character), as_factor)) %>%
        mutate(across(where(is.factor), as.numeric)) %>%
        cor(use = use) %>%
        as_tibble() %>%
        mutate(feature = names(.)) %>%
        select(feature, !! feature_expr) %>%
        filter(!(feature == feature_name))

    # conditionals to sort the variables
    # very usefull to plot
    if (fct_reorder) {
        data_cor <- data_cor %>%
            mutate(feature = fct_reorder(feature, !! feature_expr)) %>%
            arrange(feature)
    }

    if (fct_rev) {
        data_cor <- data_cor %>%
            mutate(feature = fct_rev(feature)) %>%
            arrange(feature)

    }

    return(data_cor)

}

```

Second, we need the plot function.

```r
# this function plot the correlation scores in order of values
# and have a lot of parameters that can be used to
# fully customize your plot as you want
plot_cor <- function(data, target, fct_reorder = FALSE, fct_rev = FALSE,
                     include_lbl = TRUE, lbl_precision = 2, lbl_position = "outward",
                     size = 2, line_size = 1, vert_size = 1,
                     color_pos = palette_light()[[1]],
                     color_neg = palette_light()[[2]]) {

    # meta programming to capture the variables
    # like tidy functions
    feature_expr <- enquo(target)
    feature_name <- quo_name(feature_expr)


    data_cor <- data %>%
        get_cor(!! feature_expr, fct_reorder = fct_reorder, fct_rev = fct_rev) %>%

        # used as label, and also putting the precision of the numbers
        mutate(feature_name_text = round(!! feature_expr, lbl_precision)) %>%

        # labeling the correlation as negative and positive
        mutate(Correlation = case_when(
            (!! feature_expr) >= 0 ~ "Positive",
            TRUE ~ "Negative") %>% as.factor())

    g <- data_cor %>%
        ggplot(aes_string(x = feature_name, y = "feature", group = "feature")) +
        geom_point(aes(color = Correlation), size = size) +
        geom_segment(aes(xend = 0, yend = feature, color = Correlation), size = line_size) +
        geom_vline(xintercept = 0, color = palette_light()[[1]], size = vert_size) +
        expand_limits(x = c(-1, 1)) +
        theme_tq() +
        scale_color_manual(values = c(color_neg, color_pos))

    if (include_lbl) g <- g + geom_label(aes(label = feature_name_text), hjust = lbl_position)

    return(g)

}


plot_cor(lol_diamond_tbl, totalDamageDealt, fct_reorder = T)
```

![final_result](/img/correlation/correlation_10.png)

And that is the final result. As we can see, goldEarned and goldSpent is more related to totalDamageDealt (target variable), and make sense because you earn gold when you kill/destroy players/minions/objectives in the game, hence, with more gold you spent more gold.

Look, you can change the target variable and see the correlation between other variables too, just need to change the target parameter. I encourage you to explore different customization and change the default values for this function, and if you find any question, please let me know.

I hope you enjoyed this tutorial, bye! ^^

LinkedIn: https://www.linkedin.com/in/lucianobatistads/

GitHub: https://github.com/LucianoBatista

source: https://www.yourarticlelibrary.com/statistics-2/correlation-meaning-types-and-its-computation-statistics/92001
