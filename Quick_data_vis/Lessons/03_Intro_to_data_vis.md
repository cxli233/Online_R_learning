# Introduction 

In this lesson, we will cover: 

1. How are data represented in visualizations? 
2. What is "the grammar of graphics"? 
3. Introduction to ggplot: mapping variables to axis and aesthetics. 

# Required packages
```{r}
library(tidyverse)
library(RColorBrewer)
```

The `tidyverse` will be doing all the heavy lifting. 
The `RColorBrewer` will be providing some nice color sets for our graphs. 

# How are data represented in visualizations?
As an example, let's simulate some data: 
```{r}
set.seed(666)
Example_data1 <- rbind(
  rnorm(3, mean = 5, sd = 0.5) %>%
    as.data.frame() %>% 
    mutate(time = 1),
  rnorm(3, mean = 8, sd = 0.5) %>% 
    as.data.frame() %>% 
    mutate(time = 2),
  rnorm(3, mean = 10, sd = 0.5) %>%
    as.data.frame() %>% 
    mutate(time = 3)
) %>% 
  rename(response = ".")

Example_data1
```
|response|time|
|--------|----|
|5.376656|   1|
|6.007177|   1|
|4.822433|   1|
|9.014084|   2|
|6.891563|   2|
|8.379198|   2|
|9.346907|   3|
|9.598740|   3|
|9.103880|   3|

As you can see, the data are stored in this table. This is already a tidy data frame. 
It has two columns, "response" and "time". It appears each time point has 3 observations. 

If you were to just print this table, it will be a perfectly correct representation of the data.  
For example, if you read the numbers, you probably notice the response variable goes up as time progresses.  

But that is NOT the point of data visualization. 
The purpose of data visualization is to create a visual and intuitive representation of the data. 
A good data visualization allows us to gain insight into the data at a glance. 
That being said, what are the ways to visually represent data? 

## Represent data by position
The first way to represent data is by position, such that the data is represented by position along the axis. 
```{r}
Example_data1 %>% 
  ggplot(aes(x = time, y = response)) +
  stat_summary(geom = "line", fun.data = mean_se, 
               size = 1.1, alpha = 0.8, color = "grey60") +
  geom_point(aes(fill = as.factor(time)), shape = 21, color = "grey20",
             alpha = 0.8, size = 3, position = position_jitter(0.1, seed = 666)) +
  stat_summary(geom = "errorbar", fun.data = mean_se,
               width = 0.05, alpha = 0.8) +
  scale_fill_manual(values = brewer.pal(8, "Accent")) +
  scale_x_continuous(breaks = c(1, 2, 3)) +
  labs(x = "Time point",
       y = "Response",
       title = "Dot/line graphs are\nposition based") +
  theme_classic() +
  theme(
    legend.position = "none",
    text = element_text(size = 14, color = "black"),
    axis.text = element_text(color = "black"),
    plot.title = element_text(size = 12)
  )

ggsave("../Results/03_line_plot.png", height = 2.5, width = 2)
```
![line graph](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/03_line_plot.png)

In this case, I graphed time on x-axis, and response on y-axis. 
This visualization is position based. 

## Represent data by length/size 
```{r}
Example_data1 %>% 
  ggplot(aes(x = time, y = response)) +
  stat_summary(geom = "bar", fun.data = mean_se, aes(color = as.factor(time)),
               size = 1.1, alpha = 0.8, fill = NA, width = 0.5) +
  geom_point(aes(fill = as.factor(time)), shape = 21, color = "grey20",
             alpha = 0.8, size = 3, position = position_jitter(0.1, seed = 666)) +
  stat_summary(geom = "errorbar", fun.data = mean_se,
               width = 0.05, alpha = 0.8) +
  scale_fill_manual(values = brewer.pal(8, "Accent")) +
  scale_color_manual(values = brewer.pal(8, "Accent")) +
  scale_x_continuous(breaks = c(1, 2, 3)) +
  labs(x = "Time point",
       y = "\nResponse",
       title = "Bar graphs are\nlength based") +
  theme_classic() +
  theme(
    legend.position = "none",
    text = element_text(size = 14, color = "black"),
    axis.text = element_text(color = "black"),
    plot.title = element_text(size = 12) 
  )

ggsave("../Results/03_bar.png", height = 2.5, width = 2)
```
![bar graph](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/03_bar.png) 

In this case, I graphed time on x-axis, and response on y-axis. 
This visualization is length based. Why?
In bar plots, values are represented by the distance from the x axis, and thus the length of the bar. 

Never confuse position and length based visualizations! 
Can you tell what is wrong with the graph on the right? 

![What is wrong?](https://github.com/cxli233/FriendsDontLetFriends/blob/main/Results/Position_and_length_based_visualizations.svg)

## Represent data by color 
Finally, data can be represented by color. 
We will explore this more when we talk about heatmaps and whatnot. 

# ggplot: data visualization using the grammar of graphics
As you can see in the examples, I've been using `ggplot` to produce graphs. 
The "gg" in `ggplot` stands for grammar of graphics, a theoretical framework to visualize data. 
In grammar of graphics, each visualization has the following layers: 

1. Coordinates: what are on x and y axis? 
2. Geometric objects ("geom"): e.g., bars, points, lines... 
3. Mapping: mapping colors, size, and transparency to geometric objects.

For more resources, you can look at the [ggplot cheat sheet](https://statsandr.com/blog/files/ggplot2-cheatsheet.pdf). 

Let's build a scatter plot for our example data. 
Usually I start with the data table and pipe it into `ggplot()` using the pipe `%>%` syntax.
Inside the `ggplot()` function, there is this `aes()` argument. 
Inside the `aes()` argument, it takes the x and y variables. 
Not intuitive, but it is what it is. 

```{r}
Example_data1 %>% 
  ggplot(aes(x = time, y = response)) +
  theme_classic()

ggsave("../Results/03_scatter_1.png", height = 2.5, width = 2)
```
![step1](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/03_scatter_1.png)

If we just do that, we get a blank graph with time on x axis and response on y axis. 
The `theme_classic()` only change the appearance of the background, read more [here](https://ggplot2.tidyverse.org/reference/ggtheme.html).
ggplot comes with a collection of themes. My go-to is "classic". It has white background and axis lines, as you can see in this example. 

The graph is blank because we haven't added any geometric objects (geom) yet. 
For the sake of this example only, let's make a scatter plot. So we need some points. 
The command to add points in `ggplot` is `geom_poit()`. 

```{r}
Example_data1 %>% 
  ggplot(aes(x = time, y = response)) +
  geom_point() +
  theme_classic()

ggsave("../Results/03_scatter_2.png", height = 2.5, width = 2)
```
![step2](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/03_scatter_2.png)

This is how the graph looks like after we added points. 
To include more layers, you literally add (using the `+` syntax) layers to the existing code. 
After the `ggplot()` line which initiates the graph, I usually add the geom layers first, then other layers, then lastly the `theme` layer. 

For a minimal plot, we are done! 

# Mapping variables to ggplot geom
The example data we are using only have 2 variables: time and response. What if we have more than 2? 
To demonstrate that, let's use another example. 

I am going to read in child mortality, children/women, and income data. 
We will use those data for another example. For the sake of this lesson, we will just use year 1945. 

First, read in data. 
```{r}
child_mortality <- read_csv("../Data/child_mortality_0_5_year_olds_dying_per_1000_born.csv", col_types = cols()) 
babies_per_woman <- read_csv("../Data/children_per_woman_total_fertility.csv", col_types = cols()) 
income <- read_csv("../Data/income_per_person_gdppercapita_ppp_inflation_adjusted.csv", col_types = cols()) 
```

Second, re-shape data into tidy format. 
```{r}
babies_per_woman_tidy <- babies_per_woman %>% 
  pivot_longer(names_to = "year", values_to = "birth", cols = c(2:302))  

child_mortality_tidy <- child_mortality %>% 
  pivot_longer(names_to = "year", values_to = "death_per_1000_born", cols = c(2:302))  

income_tidy <- income %>% 
  pivot_longer(names_to = "year", values_to = "income", cols = c(2:242))  
```

Third, join them together. 
```{r}
example2_data <- babies_per_woman_tidy %>% 
  inner_join(child_mortality_tidy, by = c("country", "year")) %>% 
  inner_join(income_tidy, by = c("country", "year"))

head(example2_data)
```

Fourth, filter for year 1945.
```{r}
example2_data_1945 <- example2_data %>% 
  filter(year == "1945")

head(example2_data_1945)
```


You should be able to do all that relatively easily after completing [Intro_to_tidy_data](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Lessons/02_Intro_to_tidy_data.md). 
If not, practice more! 

Say we want birth rate on x axis, mortality on y axis, make a scatter plot, and color the dots by income, what should we do? 
Let's make a plain scatter plot first. 

```{r}
example2_data_1945 %>% 
  ggplot(aes(x = birth, y = death_per_1000_born)) +
  geom_point() +
  labs(x = "No. children per woman",
       y = "Child mortality/1000 born",
       title = "Year 1945") +
  theme_classic()

ggsave("../Results/03_scatter_3.png", width = 2.5, height = 2.5)
```
![basic scatter](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/03_scatter_3.png)

That looks fine. 
To color dots (or any geom) based on a variable in the data frame, you open an `aes()` argument inside the `geom()`.

```{r}
example2_data_1945 %>% 
  ggplot(aes(x = birth, y = death_per_1000_born)) +
  geom_point(aes(color = log10(income))) +
  labs(x = "No. children per woman",
       y = "Child mortality/1000 born",
       title = "Year 1945") +
  theme_classic()

ggsave("../Results/03_scatter_4.png", width = 3, height = 2.5)
```
![scatter, colored](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/03_scatter_4.png)

Note: income is quite unevenly distributed, so I log-transformed it. 
Now we have dots colored by income (in log10 scale).
The problem is the default color scale in ggplot looks bad. 
We can use a different set of colors from the `RColorBrewer` package. 
You can look at all the available color scales in `RColorBrewer` [here](https://r-graph-gallery.com/38-rcolorbrewers-palettes.html). 
Let's use something easier to see than the default ggplot blue. 

```{r}
example2_data_1945 %>% 
  ggplot(aes(x = birth, y = death_per_1000_born)) +
  geom_point(aes(color = log10(income))) +
  scale_color_gradientn(colours = brewer.pal(9, "YlGnBu")) +
  labs(x = "No. children per woman",
       y = "Child mortality/1000 born",
       title = "Year 1945") +
  theme_classic()

ggsave("../Results/03_scatter_5.png", width = 3, height = 2.5)
```
![scatter, colored, better](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/03_scatter_5.png)

To provide a custom color scale that produce a color gradient, use `scale_color_gradientn())`.
Here is a whole family of `scale` functions in ggplot, which allow you to control the scales for colors and axis.  
You can read more about scales on the [ggplot cheat sheet](https://statsandr.com/blog/files/ggplot2-cheatsheet.pdf). 
I quite like the yellow-green-blue color gradient from `RColorBrewer`, which is what I used here. 
As you can see, low income countries have more children/woman and also higher child mortality. 

# Exercise 
Graph income (in log10 scale) on x axis, child mortality on y axis, and color with children/woman in year 2010. 
Were the trend similar to year 1945? 
Save the graph using `ggsave()`. 

 

