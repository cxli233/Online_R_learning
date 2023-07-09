
# Introduction
In this lesson we will explore the best practice for mean separation visualizations. 
Mean separation is one of the most common graph types in scientific publications. 
In these graphs, you have two or more groups.
These groups can be experimental treatments, genotypes, conditions, and so on. 
Within each group, there are multiple observations. 
The goal of the visualization is to represent the mean (the central tendency) and spread of the data. 

We will explore: 

1. Is your bar plot hiding data from you?  
2. What are the optimal graph type given the number of observations? 
3. How to handle mean separation plots in multifactorial experiments? 

# Packages 
```{r}
library(tidyverse)
library(readxl)
library(RColorBrewer)
library(ggbeeswarm)
```

We have a new package `ggbeeswarm`.
`ggbeeswarm` is useful for mean separation graphs with many overlapping dots. 
You will see that in a moment. 

# Is your bar plot hiding data from you? 
For a first example, let's simulate some data, then make a plain old bar plot. 
You can ignore this code chunk. 
```{r}
set.seed(666)
group1 <- rnorm(n = 100, mean = 1, sd = 1)
group2 <- rlnorm(n = 100, 
                 meanlog = log(1^2/sqrt(1^2 + 1^2)), 
                 sdlog = sqrt(log(1+(1^2/1^2))))

groups_long <- cbind(
  group1,
  group2
) %>% 
  as.data.frame() %>% 
  pivot_longer(cols = 1:2, names_to = "group", values_to = "response")

head(groups_long)
```

I simulated two groups, each has 100 observations. Now let's make a bar plot. 
To make a bar plot, we will use `geom_bar()`. 

```{r}
groups_long %>% 
  ggplot(aes(x = group, y = response)) +
  geom_bar(stat = "summary", fun = mean, width = 0.5,
           aes(fill = group)) +
  stat_summary(geom = "errorbar", fun.data = mean_se, 
               width = 0.1) +
  scale_fill_manual(values = brewer.pal(8, "Accent")) +
  theme_classic() +
  theme(
    legend.position = "none"
  )

ggsave("../Results/04_bar.png", height = 2.5, width = 2)
```

![bar plot](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/04_bar.png)

`stat = "summary", fun = mean` inside `geom_bar()` calculates the mean for each group,
such that the bar lengths are the mean, not the individual observations. 
The `stat_summary()` layer adds the means +/- standard errors (SE) as error bars,
which is specified by `fun.data = mean_se`. 

As you can see in this simple bar plot, the two groups are _very_ similar in terms of mean and SE.
But is this the full story? Let's make another plot. 

```{r}
groups_long %>% 
  ggplot(aes(x = group, y = response)) +
  geom_boxplot(aes(fill = group), width = 0.5) +
  scale_fill_manual(values = brewer.pal(8, "Accent")) +
  theme_classic() +
  theme(
    legend.position = "none"
  )

ggsave("../Results/04_box.png", height = 2.5, width = 2)
```

![box plot](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/04_box.png)

Now I've made a box plot. 
In a box plot, the center line is median. 
The box spans interquartile range (from 25th percentile to 75th percentile). 
The whiskers span 1.5 IQR or (min and max, whichever is less extreme). 

Looking at the box plots, we can notice that group2 is more skewed (more outliers on the upper end).
In this case, our simple bar plot was hiding data from us. 

For mean separation plots, my go-to is plotting all dots with lateral offsets based on density. 
What does that even mean? Let's graph it first and I will explain. 
```{r}
groups_long %>% 
  ggplot(aes(x = group, y = response)) +
  geom_quasirandom(aes(color = group), alpha = 0.8) +
  scale_color_manual(values = brewer.pal(8, "Accent")) +
  theme_classic() +
  theme(
    legend.position = "none"
  )

ggsave("../Results/04_dots.png", height = 2.5, width = 2)
```

![dot plot](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/04_dots.png)

The offset dots layer is provided by `geom_quasirandom()` of the `ggbeeswarm` package.
The name comes from the spread of dots looks like a swarm of bees.
As you can see, this is not just a dot plot. 
The dots are offset to the left or right when there are multiple observations around the same value.
The term "quasirandom" means the dots are offset more if the density is higher. 
The result is that the dots lay themselves out into the distribution of the data. 

A final note is that we have 100 observations per group. 
Even with `geom_quasirandom()` and offsets, there are still overlapping dots. 
This is called "overplotting" in data visualization. 
I turned on `alpha = 0.8` inside `geom_quasirandom()` to make the dots a bit transparent. 
`alpha = 0.8` means 80% opacity, or 20% transparency. 
This way, when dots do overlap, we can at least see through them. 

Comparing the bar plot, the box plot, and the dot plot, 
we can see that while the mean and SE are similar between the groups, 
their underlying distributions are very different. 
The bar plot itself cannot convey this aspect of our data. 

**Take home message**: I _highly_ recommend plotting all data points in mean separation graphs. 

# What are the optimal graph type given the number of observations? 
Be very careful with data with low number of observations (n < 30), and very large number of observations (n > 100). 

For data with large number of observations, as we just saw in the dot plot, 
even with lateral offset, you might still get overlapping dots.
A good practice is turn on transparency of the dots. 
Another solution is to make a violin plot. 

```{r}
groups_long %>% 
  ggplot(aes(x = group, y = response)) +
  geom_quasirandom(aes(color = group), alpha = 0.8) +
  geom_violin(fill = NA) +
  geom_boxplot(width = 0.1, fill = NA) +
  scale_color_manual(values = brewer.pal(8, "Accent")) +
  theme_classic() +
  theme(
    legend.position = "none"
  )

ggsave("../Results/04_dots_violin.png", height = 2.5, width = 2)
```

![dot and violin plot](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/04_dots_violin.png)

In this case, I overlaid both violin plot and box plot onto the dots. 
The width of the violin plot shows the distribution, 
which as you can see follows the offset of the dots.
A quick note: I turned on `fill = NA` in `geom_violin()` and `geom_boxplot()` so that we can see through them and see the dots under them. 
The default has white (non-transparent) fill. 

You can read more about fancy mean separation plots (such as "raincloud plot") [here](https://z3tt.github.io/Rainclouds/). 


Violin and box plots are great for data with many observations. 
However, they can be misleading in small datasets. 
Here is an example: 

![what's wrong here?](https://github.com/cxli233/FriendsDontLetFriends/blob/main/Results/Beware_of_small_n_box_violin_plot.svg) 

Violin plots (or any sort of smoothed distribution curves) and box plots make no sense for small n.
Distributions and quartiles can vary widely with small n, 
even if the underlying observations are similar.
Distribution and quartiles are only meaningful with large n. 
I did an experiment before, where I sampled the same normal distribution several times and computed the quartiles for each sample. 
The quartiles only stabilize when n gets larger than 50.

**Take home message**: pay attention to the data set size when designing a mean separation plot! 

# Mean separation in multifactorial experiments. 
Multifactorial experiments are very common in science. 
To explain this, let's use a real world example. 
Data from [Matand et al., 2020, BMC Plant Biology](https://link.springer.com/article/10.1186/s12870-020-2243-7).

```{r}
tissue_culture <- read_excel("../Data/LU-Matand - Stem Data.xlsx")
head(tissue_culture)
```

In this experiment, the authors looked at the ability of stem pieces of [daylily](https://en.wikipedia.org/wiki/Daylily) to regrow into a full plant. 

They have 19 cultivars of the plant, 3 explant type, and 3 hormone treatment. 
They want to look at the best regeneration condition for each cultivar that they have. 
The ability to regenerate is quantified by the number of buds or shoots. 

Multifactorial experiments like this are _very_ common. 
However, we need to pay extra attention to the design of the graphs. 

```{r}
tissue_culture %>% 
  ggplot(aes(x = Variety, y = Buds_Shoots)) +
  geom_bar(stat = "summary", 
           aes(fill = interaction(Explant, Treatment)), 
           fun = mean,
           position = position_dodge2()) +
  labs(x = NULL,
       y = "Num buds and shoots",
       fill = NULL) +
  theme_classic() + 
  theme(axis.text.x = element_text(angle = 45, hjust = 1))

ggsave("../Results/04_multi_bar.png", height = 3, width = 6)
```
![very bad bar plot](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/04_multi_bar.png)

I have cultivar on x axis, buds/shoots on y axis, 
and color bars by the combination of explant and treatment. 
This is very horrendous! It is extremely difficult to interpret what is going on. 

To overcome this problem, I am introducing you to your new friend, facets! 
Facetting is a method to layout your plot into subplots and better direct the attention of your readers. 
Let me show you an example: 

```{r}
tissue_culture %>% 
  mutate(Treatment = factor(Treatment, 
                            levels = c("T1", "T5", "T10"))) %>% 
  ggplot(aes(x = Treatment, y = Buds_Shoots)) +
  facet_wrap(~ Variety, scales = "free") +
  geom_quasirandom(aes(color = Explant), alpha = 0.8) +
  scale_colour_manual(values = brewer.pal(8, "Set2")) +
  labs(x = "Hormone Treatment",
       y = "Num buds and shoots",
       fill = NULL) +
  theme_classic() +
  theme(legend.position = "bottom") 

ggsave("../Results/04_multi_dots.png", height = 6, width = 9)
```

![much better](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/04_multi_dots.png) 

Now this is a lot better. 
Since the authors want to find the best conditions for each cultivar, we make each cultivar a subplot. 
This is provided by `facet_wrap()`. 
The value after `~` is the column name to make subplot from, which is "Variety" in the table. 
From this new plot, it is obvious that the split stem explant and T10 treatment gave the best results for most cultivars, with some exceptions in a cultivar-specific manner. 

**Take home message**: facets are very powerful and you should learn to use them! Read more about facets [here](https://ggplot2.tidyverse.org/reference/facet_grid.html). 

# Exercise
## Q1
Notice the `scales = "free` inside `facet_wrap()`. 
Make a new graph but remove the `scales = "free` inside `facet_wrap()`.

Can you explain what happened? 
Can you explain in what scenario is more appropriate to have `scales = "free"` turned on vs off? 

## Q2 
Here is an example data for you to practice: 
```{r}
M1 <- data.frame(
  conc = rnorm(n = 8, mean = 0.03, sd = 0.01)
) %>% 
  mutate(group = "ctrl") %>% 
  rbind(
    data.frame(
      conc = rnorm(n = 6, mean = 0.25, sd = 0.02)
    ) %>% 
      mutate(group = "trt")
  ) %>% 
  mutate(pest = "Pest 1")

M2 <- data.frame(
  conc = rnorm(n = 8, mean = 6, sd = 1)
) %>% 
  mutate(group = "ctrl") %>% 
  rbind(
    data.frame(
      conc = rnorm(n = 6, mean = 5.5, sd = 1.1)
    ) %>% 
      mutate(group = "trt")
  ) %>% 
  mutate(pest = "Pest 2")

M3 <- data.frame(
  conc = rnorm(n = 8, mean = 20, sd = 0.5)
) %>% 
  mutate(group = "ctrl") %>% 
  rbind(
    data.frame(
      conc = rnorm(n = 6, mean = 19.5, sd = 1.2)
    ) %>% 
      mutate(group = "trt")
  ) %>% 
  mutate(pest = "Pest 3")

Spray <- rbind(
  M1, M2, M3
)

head(Spray)
```

In this hypothetical experiment, I have two treatments: ctrl vs. trt. 
And I measured the occurrence of three different pests after spraying either treatments. 
Make a mean separation plot for this experiment. 

Hints: 

* Is this a multifactorial experiment? 
* How many observations are there in each group? 

Was the treatment effective in controlling any of the three pests? 
Save the graph using `ggsave()`. 

 











