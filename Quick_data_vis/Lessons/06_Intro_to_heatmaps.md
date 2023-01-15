# Introduction
Heatmap is another very common type of visualization in scientific publications. 
In biology they are extremely common in omics papers. 
However, extra care must be taken to not produce an uninterpretable or misleading heatmap. 

We will cover: 

1. What is a heat map?
2. Are you using an appropriate color scale? 
3. Are you scaling the data correctly to best visualize your data? 
4. How to handle outliers in heatmaps? 
5. Should you reorder rows and columns of a heatmap? 

# Packages 
```{r}
library(tidyverse)
library(RColorBrewer)
library(viridis) 
```
We have new package `viridis`. 
`viridis` is a collection of beautiful color scales for heatmaps. 
Read more [here](https://cran.r-project.org/web/packages/viridis/vignettes/intro-to-viridis.html). 

# What is a heat map? 
A heatmap is a representation of 3 dimension data. 
Say we have 3 dimensions x, y, z. 
In a heatmap, the z dimension is visualized by different colors. 
Let's look at an example: 

```{r}
abc_1 <- expand.grid(
  a = c(1:10),
  b = c(1:10)
) %>% 
  mutate(c = a + b)

head(abc_1)
```
In this example, we have 3 dimensions: a, b, and c. 
a and b ranges from 1 to 10. 
And the c is the sum of a and b. 

```{r}
abc_1 %>% 
  ggplot(aes(x = a, y = b)) +
  geom_tile(aes(fill = c)) +
  scale_fill_gradientn(colors = viridis(100)) +
  labs(title = "c = a + b") +
  theme_classic() +
  coord_fixed()

ggsave("../Results/06_abc_1.png", height = 3, width = 3)
```
![heatmap1](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/06_abc_1.png)

Now this is a heatmap. 
In ggplot, heatmap can be made using `geom_tile()`. 
To map values to the tiles, use `aes(fill = ...)`. 
In this case, we want to color the interior of tiles using the values of `c`. 
We do `aes(fill = c)` inside `geom_tile()`. 

# Choosing the correct color scale 
Note the `scale_fill_gradientn(colors = viridis(100))` line in the previous code chunk. 
What it does is it maps values of z to the `viridis` color scale, a purple-blue-green-yellow color gradient, where smaller values are darker purple, and larger values are mapped to progressively lighter and yellower colors. 

Color scales are pretty, but we must be extra careful. 
There are a couple important concepts regarding choosing a color scale. 

## 1. Never use both red/green colors for heatmaps. 
Yes, I repeat, _NEVER_ use red and green in the same heat map. 
Red/green color blindness occurs in about 1/16 in male and 1/256 in female. 
People who are red/green colorblind cannot distinguish between red and green; they see brown instead. 
They will not be able to read a heatmap where numerical values are represented by a gradient of red and green.  

## 2. Correctly use unidirectional and bidirectional color scales.
There are two types of color scales:

1. Unidirectional or sequential: the color gradient goes from darkest to lightest.The viridis color gradient is a unidirectional color scale. 
2. Bidirectional or divergent: the color gradient has a single lightest point at the center and two darkest points at the extremes. 

You should _NEVER_ use a bidirectional color scale for unidirectional data. 
![Color scales](https://github.com/cxli233/FriendsDontLetFriends/blob/main/Results/ColorScales.svg)

You might not have thought about this one, and people make this mistake all the time. 
In this example, the two plots on the right use a bidirectional color scale. 
Why? Because the blue and red are darker than white. 
The first three plots are fine, but last one is not okay. 
You might ask, what's wrong with it? 

When color scales (or color gradients) are used to represent numerical data, 
the darkest and lightest colors should have special meanings. 
You can decide what those special meanings are: e.g., max, min, mean, zero. 
But they should represent something meaningful. 
A data visualization sin for heat maps/color gradients is when the lightest or darkest colors are some arbitrary numbers. 
_This is as bad as the longest bar in a bar chart not being the largest value._ Can you imagine that?

# Same heatmap, same scale  
The next issue with heatmap is scaling. 
Let's use an example to illustrate this. 
In this hypothetical example, I have the expression level of 3 genes across 5 stages of development.
I want to visualize their expression pattern across development. 
You can ignore this chunk. 
```{r}
genes123 <- data.frame(
  stages = c(1:5),
  gene1 = c(15, 25, 35, 45, 55),
  gene2 = c(100, 200, 300, 400, 500),
  gene3 = c(100, 80, 70, 60, 50)
) %>% 
  pivot_longer(cols = !stages, names_to = "gene", values_to = "expression") 

head(genes123)
```

If I just make a heatmap, what will happen? 
```{r}
genes123 %>% 
  ggplot(aes(x = stages, y = gene)) +
  geom_tile(aes(fill = expression), color = "grey90") +
  scale_fill_gradientn(colors = brewer.pal(9, "YlGnBu")) +
  theme_classic() +
  theme(
    legend.position = "top"
  )

ggsave("../Results/06_genes_1.png", height = 2.5, width = 3)
```
![gene_heatmap1](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/06_genes_1.png)

We have stages on x axis, genes on y axis, and tiles colored by expression. 
If you look at the heatmap, you might be tempted to say, gene2 is going up during development, 
and genes 1 and 3 are not changing that much. 
But is that so? 

In this example, gene2 ranges from 100-500, whereas the rest of the two genes ranges from 10-100. 
The large difference in range implies that the lower range values are not visualized well. 

A good way to scale the data is using z score. 
A z score is the difference between a value and the mean, divided by the standard deviation. 
In this example, we will calculate z score for each gene. 
Remember our good friend `group_by()`? 

```{r}
genes123 <- genes123 %>% 
  group_by(gene) %>% 
  mutate(z = (expression - mean(expression))/sd(expression)) %>% 
  ungroup()

head(genes123)
```

What happened here is we `group_by(gene)` first,
such that z score calculation is done relative to each gene, not the entire data. 
Then `mutate(z = (expression - mean(expression))/sd(expression))` quite literally calculates the z score. 
Lastly, we `ungroup()`, and we are done. 

Now we make a new heatmap, but color tiles with z scores. 
```{r}
genes123 %>% 
  ggplot(aes(x = stages, y = gene)) +
  geom_tile(aes(fill = z), color = "grey90") +
  scale_fill_gradientn(colors = brewer.pal(9, "YlGnBu")) +
  theme_classic() +
  theme(legend.position = "top")

ggsave("../Results/06_genes_2.png", height = 2.5, width = 3)
```
![gene_heatmap2](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/06_genes_2.png)

In this version of the heatmap, it is obvious that genes 1 and 2 have the same expression pattern.
They both go up during development, while gene3 has the opposite pattern. 

# Be aware of outliers 
This is related to the previous point. 
Outliers can drastically change the appearance of a heatmap. 
Here is an example: 
![check outliers for heatmap](https://github.com/cxli233/FriendsDontLetFriends/blob/main/Results/Check_outliers_for_heatmap.svg) 

In this example, I have 2 observations. 
For each observations, I measured 20 features. 
Without checking for outliers, it may appear that the 2 observations are overall similar, except at 2 features. 
However, after maxing out the color scale around the 95th percentile of the data, it reveals that the two observations are distinct across all features.

The point here is always check for outliers when making heatmaps. 
If there are outliers, it may be a good idea to clip outliers at the extremes. 

# Reordering rows and columns 
In most cases, the ordering of rows and columns are arbitrary.
For example, nothing says the y axis has to be in order of gene1, followed by gene2, then gene3. 
However, if the heatmap reflects a physical map (x and y axis maps to physical locations in space), 
then you can't reorder rows and columns. 

The ordering of the x and y axis has a _HUGE_ impact on how useful a heatmap is, especially a large heatmap.  
Let's look at an example: 
![recorder rows and columns](https://github.com/cxli233/FriendsDontLetFriends/blob/main/Results/Reorder_rows_and_columns_for_heatmap.png) 

In this example, I have cells as columns and features as rows. 
Each tile is showing z scores. 
It is impossible to get anything useful out of the heatmap without reordering rows and columns.
We can reorder rows and columns, such that features that are enriched in cell type 1 appears first.
Then features that are enriched in cell type 2 appears second. Data from: [Li et al., 2022, BioRxiv](https://www.biorxiv.org/content/10.1101/2022.07.04.498697v1.abstract)

When rows and columns of a heatmap is properly ordered, you can see a clear signal across the diagonal. For example: 

![Beautiful heatmap](https://github.com/cxli233/FriendsDontLetFriends/blob/main/Results/Abstract_R_2022_11_24.svg)

The coding underlying reordering rows and columns of a heatmap is actually quite complex. 
I would say this is a bit outside the scope of this introductory lesson. 
That being said, there are packages that produces heatmaps, and they also handles reordering rows and columns automatically.  
For example, [tidyHeatmap](https://github.com/stemangiola/tidyHeatmap) is a good one.
If you ever need to make heatmaps, you should explore how to reorder rows and columns, or explore how to use a package like `tidyHeatmap`. 

# Exercise 
## Q1
Here is the rainbow color scale: 
```{r}
abc_1 %>% 
  ggplot(aes(x = a, y = b)) +
  geom_tile(aes(fill = c)) +
  scale_fill_gradientn(colors = rainbow(20)) +
  labs(title = "c = a + b") +
  theme_classic() +
  coord_fixed()

ggsave("../Results/06_abc_2.png", height = 3, width = 3)
```
![rainbow](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/06_abc_2.png)

Can you explain why the rainbow color scale is inappropriate for heatmaps?  

## Q2
You can ignore the following chunk. It generates the data. 
```{r}
Q2_data <- rbind(
  c(10, 20, 30, 40, 50, 60), # 6
  c(0, 10, 30, 10, 5, 0), # 3
  c(100, 200, 300, 400, 500, 550), # 6
  c(50, 45, 40, 30, 20, 0),  # 1
  c(0, 200, 300, 500, 200, 100), # 4
  c(10, 500, 200, 100, 10, 0), # 2 
  c(0, 0, 0, 10, 500, 0) # 5
) %>%
  as.data.frame() %>% 
  cbind(gene = c("a", "b", "c", "d", "e", "f", "g")) %>% 
  pivot_longer(cols = !gene, names_to = "V", values_to = "expression") %>% 
  mutate(stage = str_remove(V, "V"))

Q2_data %>% 
  ggplot(aes(x = stage, y = gene)) +
  geom_tile(aes(fill = expression)) +
  geom_text(aes(label = expression)) +
  scale_fill_gradientn(colors = rev(brewer.pal(11, "RdBu")))+
  theme_classic()

ggsave("../Results/06_genes_3.png", height = 3.5, width = 4.2)
```
![bad heatmap](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/06_genes_3.png)

What is (are) wrong with this heatmap? 
Hint: there is a lot going wrong with this heatmap. 
