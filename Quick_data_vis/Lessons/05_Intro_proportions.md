# Introduction 
In this lesson we will explore the best practice for presenting proportional data. 
The presentation of proportional data is very common, and thus warrant a lesson of its own. 
Proportional data are data that range from 0 to 1. 
More importantly, proportional data should add up to 1 if they are derived from the same entity. 
For example, let's say on a corn ear, there are yellow and white kernels (and no other colors). 
The proportion of yellow and white kernels should add up to 1. 
Let's say if 75% of the kernel are yellow, it implies the proportion of white ones is 25%. 

Goals might differ when we are presenting proportional data. Here are two common objectives:

1. Showing different proportions of an entity add up to 1. 
2. Comparing proportions of different entities. 

These two objectives may not be mutually exclusive. 
We will explore what are the best practice to achieve these objectives. 

# Packages 
```{r}
library(tidyverse)
library(readxl)
library(RColorBrewer)
```
 
# What really is a pie chart? 
The most common visualization for proportional data is a pie chart. 
You literally see them all the time. 
Pie chart is a common type of visualization for proportional data, where proportions add up to 100%.
This is achieved by dividing a circle into sectors, and the sectors add up to a full circle.
This all sounds fine, but how do we construct a pie chart using ggplot? 
The short answer is we make a stacked bar chart and wrap it into a circle. 

Let's make a pie chart. 
Let's say we have a corn ear that has been open pollinated. 
As a result, there are purple, yellow, and white kernels on the ear. 
Let's say the proportions are 15% purple, 70% yellow, and 15% white. 

```{r}
ear_1 <- data.frame(
  colors = c("purple", "yellow", "white"),
  proportions = c(0.15, 0.7, 0.15)
)

head(ear_1)
```

```{r}
example_1_stacked <- ear_1 %>% 
  ggplot(aes(x = "A Corn Ear", y = proportions)) +
  geom_bar(stat = "identity", aes(fill = colors),
           color = "black", width = 0.5) +
  scale_fill_identity() +
  labs(x = NULL) +
  theme_classic()

example_1_stacked

ggsave("../Results/05_stack1.png", width = 2, height = 2.5) 
```

Note that there are two coloring options in `ggplot`.
There is `color`, which is the color of the outline. 
In this case, the outline of the stacks is black. 
There is also `fill`, which is the color of the interior. 
In this case, the interior of the stacks is colored by the color of the corn kernel. 

So now we have a stacked bar plot. 
Technically, we are done. The proportions are presented by the heights of the stacks within the bar.
And this is a perfectly fine visualization of our proportional data. 
But what if we want to make a pie chart? 
Well, we will use the polar coordinate.

```{r}
example_1_stacked +
  coord_polar(theta = "y") +
  theme_void()

ggsave("../Results/05_pie1.png", width = 2, height = 2.5) 
```
There are just two extra lines of code to convert a stack bar into a pie chart.
First, `coord_polar(theta = "y")` wraps the y axis into a circle. 
Second, `theme_void()` turns off the x and y axis. 
Axis are not all that informative in for pie charts anyway. 
So if you want to make a pie chart, this is what you do. 

A final note about pie chart: what are the data represented by? 
In this case, the data are represented by the arc length. 
So a pie chart is a length based visualization. 
Due the properties of a circle, the data are also presented by the arc angle and sector area.   

# What is the best visualization for comparing proportional data? 
However, oftentimes we don't want to just present proportions of one entity. 
Instead, we want to compare the proportions of multiple entities.
For this purpose, pie chart is probably not the best option. 
Pie charts have been criticized, because we are much worse in reading angles, areas, and lengths of arcs than reading lengths of straight lines. 

The best way to visualize proportions from multiple entities is stacked bars. 
Here is an example. 
![Just make a stacked bar chart](https://github.com/cxli233/FriendsDontLetFriends/blob/main/Results/dont_pie_chart.svg)   

In this example, we have two groups, each contains 4 sub-category. 
In classic pie charts, the angles (and thus arc lengths & sector area) represent the data. 
The problem is that it is very difficult to compare between entities. 
We can visually simplify the pie chart into donut charts, where the data are now represented by arc lengths. 
However, if we want to use lengths to represent the data, why don't we just unwrap the donut and make stacked bars? 
In stacked bar graphs, bars are shown side-by-side and thus easier to compare across entities. 

Let's try an example on our own. 
Let's say we have another open pollinated corn ear. 
This ear has 20% purple kernels, 75% yellow kernels, and 5% white kernels.
```{r}
ear_2 <- data.frame(
  colors = c("purple", "yellow", "white"),
  proportions = c(0.20, 0.75, 0.05)
)

ears_1_and_2 <- rbind(
  ear_1 %>% 
    mutate(ear = "ear1"), 
  ear_2 %>% 
    mutate(ear = "ear2")
)

head(ears_1_and_2)
```
Say now we want to make a graph to compare the proportions of these two ears. 
The code is in fact quite straight forward. 

```{r}
ears_1_and_2 %>% 
  ggplot(aes(x = ear, y = proportions)) +
  geom_bar(stat = "identity", aes(fill = colors), 
           color = "black", width = 0.5) +
  scale_fill_identity() +
  theme_classic()

ggsave("../Results/05_stack2.png", width = 2, height = 2.5)
```
That's it! 
I would say stacked bars are my go-to visualization for comparing proportions of multiple entities. 
A key advantage is that side-by-side stacked bars are more space efficient. 
Imagine you are comparing proportions of hundreds of entities. 
It is unrealistic to present hundreds of pie or donut charts, let alone comparing across them. 
But stacked bars make the task easy. 

# Donut charts? 
Donut charts are good alternatives of pie charts for presenting proportions of a small number of entities. 
We won't cover how to make donut charts here. 
If you are interested, you can explore on your own. 
But what you should _never_ do is making concentric donuts. 
Here is an example. 
![concentric donuts](https://github.com/cxli233/FriendsDontLetFriends/blob/main/Results/dont_concentric_donuts.svg) 
For concentric donuts, you might be tempted to say the data are represented by the arc lengths, 
which is in fact inaccurate. 
The arc lengths on the outer rings are much longer than those in the inner rings. 
Group 2 and Group 3 have the same exact values, but the arc lengths of Group 3 are much longer. 
In fact the data are represented by the arc angles, which we are bad at reading.

Since outer rings are longer, the ordering of the groups (which group goes to which ring) has a big impact on the impression of the plot. 
It can lead to the apparent paradox where larger values have shorter arcs. 
The better (and simpler!) alternative is just unwrap the donuts and make a good old stacked bar plot. 

# Should you use log scale when preseneting proportional data? 
This might be something you have not thought about. 
Let's look an example. 
In this hypothetical example, we quantified biodiversity of a rain forest relative to year 1960. 
The data are normalized to the value of year 1960. 

```{r}
example_2 <- data.frame(
 year = c(1960, 1970, 1980, 1990, 2000, 2010, 2020),
 relative_biodiversity = c(1, 0.6, 0.3, 0.2, 0.15, 0.1, 0.01)
)

head(example_2)
```

```{r}
example_2 %>% 
  ggplot(aes(x = year, y = relative_biodiversity)) +
  geom_point(size = 3) +
  labs(y = "biodiversity\n(relative to 1960)") +
  theme_classic()

ggsave("../Results/05_dot1.png", width = 3, height = 2.5)
```
Looking at this graph, you would probably say the loss of biodiversity has stabilized in the last 4 decades.
But is that so? 

A related concept of proportion is odds. 
Odds = proportion /  (1 - proportion). 
For example, if p = 0.5, then the odds is 1:1. 
If p = 0.1, then the odds is 1:9. 
If p = 0.01, then then odds is 1:99.  
A property of proportions is that is bound between 0 and 1. 
Thus, any changes near 0 and 1 will appear small by definition. 
However, if you think about it, going from 0.1 to 0.01 is 10 fold change in proportion and 11 fold change in odds, 
We can capture the relative changes using the log scale. 
We can use the log10 or nature log scale. It doesn't matter. 

```{r}
example_2 %>% 
  mutate(log_odds = log(relative_biodiversity/(1-relative_biodiversity))) %>% 
  ggplot(aes(x = year, y = log_odds)) +
  geom_point(size = 3) +
  labs(y = "biodiversity\n(relative to 1960\nlog odds scale)") +
  theme_classic()

ggsave("../Results/05_dot2.png", width = 3, height = 2.5)
```
In this case, presenting proportional data in the log odds scale paints a different picture. 
Biodiversity has decreased sharply relative to the previous decade.
It makes sense: 
From 2000 to 2010, the change was 0.15 to 0.1, or 0.1/0.15 = 0.67, or 33% decrease from 2000. 
However, from 2010 to 2020, the change was 0.1 to 0.01, which is a 90% decrease from 2010. 
In practice, you will need to think very carefully about if you need to present your proportional data in log scale. 

# Exercise 
As an exercise, let's visualize this example. 
We have two groups, each contains 4 categories. 
```{r}
group1 <- data.frame(
  "Type" = c("Type I", "Type II", "Type III", "Type IV"),
  "Percentage" = c(15, 35, 30, 20)
)
group2 <- data.frame(
  "Type" = c("Type I", "Type II", "Type III", "Type IV"),
  "Percentage" = c(10, 25, 35, 30)
)
```

Use `ggsave` to save your visualization. 

Hint: look in this lesson to see what I did to combine two entities into one data frame while giving each a unique identifier. 
