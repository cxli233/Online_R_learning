# Introduction
Network analyses are useful in many fields. 
They are very powerful in modeling relationship data, which have a wide range of applications. 
We will explore some of the applications below.
Given its prevalence and usefulness, network (and the underlying relationship data) should have a lesson of its own.  

In this lesson, we will explore: 

1. What underlies a network?
2. How to handle the visualization of network? 

# Packages 
```{r}
library(tidyverse)
library(igraph)
library(ggraph)
library(readxl)
library(RColorBrewer)
```

We have two new packages, `igraph` and `ggraph`. 
`igraph` is a network analysis package that can do a lot of heavy-lifting when it comes of dealing with networks. 
`ggraph` is a `ggplot` extension for network visualization. 

# What underlies a network? 
![insert network here](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/07_my_network1.png)

A network consists of nodes of edges. 
Nodes are points, and edges are lines connecting nodes. 
Nodes are in fact observations or entities. 
Edges are the relationship between nodes. 
When nodes and edges are combined to make a network, 
the network becomes an object that contains both observational data (from the nodes) and relationship data (from the edges). 
(In mathematics, a network is also referred to as a "graph", thus the "graph" in `igraph`.
Nodes are sometimes also referred to as "vertices".) 

Thus, a network is fundamentally two tables: 

1. an edge table - a data frame that records how nodes are connected to each other. 
2. a node table - data frame that records attributes of nodes. 

An additional requirement for the node table is that 
each node in the edge table must appear in the node table once and only once. 

An edge table can look like this:

| From | To | Additional info |  
|:----:|:--:|:---------------:|  
| Robin| Li | Started 09/2021 |  

* The first column of the edge table must be `from`: where the edge starts.
* The second column of the edge table must be `to`: where the edge ends.
* Additional information can be added as additional  columns.

In this case, this table has a single edge going from `Robin` to `Li`. 

A node table can look like this: 

| Node | Additional info |  
|:----:|:---------------:|  
|Robin |    PI           |  
|   Li |    Postdoc      |  

In a node table, the first column must be the names of the nodes. 
Each row is a node. In this case, we have a table of two nodes. 

# How to visualize a network? 
## Make a network object from edges and nodes 
We will use an example. Here is a hypothetical network that I made up. 
```{r}
example_network_edges <- read_excel("../Data/Example_network_edges.xlsx")
```
Without a given node table, 
you can actually produce a node table from the edge table. 
The node table is just a non-redundant list of all members of `from` and `to` in the edge table. 

```{r}
example_network_nodes <- data.frame(
  nodes = unique(c(example_network_edges$From, example_network_edges$To))
) %>% 
  mutate(nodes = as.character(nodes))
```

`unique(c(example_network_edges$From, example_network_edges$To)` takes the members of `from` and `to` from `example_network_edges` and take the non-redundant set. 

To make a network, use the `graph_from_data_frame()` function from the `igraph` package. 
```{r}
my_network <- graph_from_data_frame(
  d = example_network_edges,
  vertices = example_network_nodes,
  directed = F
)
```

The `d` in `graph_from_data_frame()` specifies the edge table.
`vertices` species the node table. 
In this example I set `directed` to `FALSE`.
This means from node 1 to node 2 is the same from node 2 to node 1. 

## Using ggraph 
To make a network diagram, we will use `ggraph`.
```{r}
ggraph(my_network, layout = "kk") +
  geom_edge_diagonal(color = "grey70") +
  geom_node_point(size = 3, shape = 21, color = "white", fill = "grey20") +
  geom_node_text(repel = T, aes(label = example_network_nodes$nodes)) +
  theme_void()

ggsave("../Results/07_my_network1.png", height = 3, width = 3.5, bg = "white")
```
![network 1](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/07_my_network1.png) 

There are 3 fundamental elements to a network diagram. 

1. Layout, which controls how nodes are placed in the 2-D plane. In this diagram I used "kk" or "Kamada-Kawai" layout algorithm. More about layout algorithms is discussed below. 
2. Edges. In this example, I used `geom_edge_diagonal()`, which draws curves between nodes.
3. Nodes, which is provided by `geom_node_point()`, draws each node as a point. 

## Try different network layouts
There are multiple layout algorithms for network diagrams. 
Layouts can drastically change the appearance of networks, making them easier or harder to interpret. 
Here are 3 network diagrams from the same data using different layout algorithms. 
They look very different from each other. Data from: [Li et al., 2022, BioRxiv](https://www.biorxiv.org/content/10.1101/2022.07.04.498697v1.abstract) 

My go-to is "kk", which generally gives good results.
But I encourage you to read more about different layouts [here](https://www.data-imaginist.com/2017/ggraph-introduction-layouts/) and [here](https://r-graph-gallery.com/247-network-chart-layouts.html). 

## Mapping variables to networks
![insert network here](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/07_my_network1.png)

If you look at the network that we made earlier, all the dots are just black. 
What if we want to color nodes based on additional attributes? 
This is when additional columns of the node table become useful. 
The method is like any other `ggplot` layers, you open an `aes()` argument inside `geom_node_point()`. 

As a simple example, let's use Robin and Li. 
```{r}
small_network_edges <- data.frame(
  from = "Robin",
  to = "Li",
  info = "started 09/2021"
)

small_network_nodes <- data.frame(
  node = c("Robin", "Li"),
  role = c("PI", "Postdoc")
)

small_network <- graph_from_data_frame(
  d = small_network_edges,
  vertices = small_network_nodes,
  directed = T
)
```

Here I wrote the edge and node table, and make a network object from them. 
Now let's make a network diagram, and color nodes by `role`.
```{r}
ggraph(small_network, 
       layout = rbind(c(1, 1),
                      c(1, 0))) +
  geom_edge_link(arrow = arrow(length = unit(4, "mm")), 
                 end_cap = circle(4, "mm"), 
                 start_cap = circle(4, "mm"),
                 aes(label = info),
                 angle_calc = "along",
                 vjust = -0.5) +
  geom_node_point(size = 3, aes(color = role)) +
  geom_node_text(repel = T, aes(label = small_network_nodes$node)) +
  theme_void() 

ggsave("../Results/07_small_network1.png", height = 2.5, width = 2, bg = "white")
```
![small_network](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/07_small_network1.png)

Now the network diagram is colored by our roles, PI or postdoc. 
`geom_edge_link()` makes straight lines between nodes. 
There are multiple additional parameters in `geom_edge_link()` to make the arrow and edge label. 
You can look into what these arguments do by removing them and see what happens to the diagram. 

Note that you can provide manual layout for the network if you want to. 
In this example, I provided a matrix using `layout = rbind(c(1, 1),c(1, 0))`.
This specifies the node `Robin` to be at coordinate x = 1, y = 1;
and node `Li` to be at x = 1, y = 0. 
For a small network you can get away with this. 

# How to deconvolute a large network into smaller ones? 
Networks can be very large, to better understand them we might have to deconstruct them into smaller sub-networks. 
These sub-networks are often referred to as modules. 
There are multiple ways to achieve that. I will introduce the two most common approaches below. 

## Deconvolute using neigbors of nodes of interest
The first approach is to pull out neighbors of nodes of interest. 
Let's look at the example network we made earlier. 
![insert network here](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/07_my_network1.png) 

Say we are interested in nodes 1, 14, and 23.
And we want to look at what are their direct network neighbors. 
We can do that using the `neighbors()` function in the `igraph` package. 

```{r}
neighbors_of_interst <- c(
  names(neighbors(my_network, v = "1")),  # pull neighbors of node 1
  names(neighbors(my_network, v = "14")), # pull neighbors of node 14 
  names(neighbors(my_network, v = "23")), # pull neighbors of node 23
  c(1, 14, 23) # don't forget 1, 14, 23 themselves
) %>% 
  unique() # remove redundant nodes, if any. 
  
neighbors_of_interst
```
We can make a sub-network by filtering nodes from the node table,
as well as filtering for edges that connect them. 

```{r}
my_subnetwork_nodes <- example_network_nodes %>% 
  filter(nodes %in% neighbors_of_interst)

my_subnetwork_edges <- example_network_edges %>% 
  filter(From %in% neighbors_of_interst &
           To %in% neighbors_of_interst)

my_subnetwork <- graph_from_data_frame(
  d = my_subnetwork_edges,
  vertices = my_subnetwork_nodes,
  directed = F
)
```

We made the sub-network object. 
Now we can make a new diagram. 
```{r}
ggraph(my_subnetwork, layout = "kk") +
  geom_edge_diagonal(color = "grey70") +
  geom_node_point(size = 3, shape = 21, color = "white", fill = "grey20") +
  geom_node_text(repel = T, aes(label = my_subnetwork_nodes$nodes)) +
  theme_void()

ggsave("../Results/07_my_network2.png", height = 3, width = 3, bg = "white")
```
![network2](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/07_my_network2.png)

Now we are left with nodes 1, 14, and 23 and their direct neighbors. 

## Deconvolute using module detection 
A very common method of network deconvolution is clustering. 
Clustering looks for highly interconnected nodes and assigns modules accordingly. 
The `igraph` package comes with clustering algorithms, such as `leiden`. 
You can break a network into multiple highly interconnected parts using `cluster_leiden()`. 

```{r}
my_netowrk_modules <- cluster_leiden(graph = my_network,  
                                     objective_function = "modularity") 

node_table_clustered <- data.frame(
  nodes = my_netowrk_modules$names,   # Pull node names 
  Module = my_netowrk_modules$membership # Pull membership  
) %>% 
  arrange(as.numeric(nodes)) %>% 
  inner_join(example_network_nodes, by = "nodes") # add membership info to original node table  

head(node_table_clustered)
```
What I did in this chunk is I ran the leiden clustering algorithm on my network.
Then I made a new data frame that contains the nodes and which module they belong to.
Finally, I joined the membership information to the original node table. 

Now we can make a new network object with the updated node table. 
```{r}
my_network_clustered <- graph_from_data_frame(d = example_network_edges,
                                         vertices = node_table_clustered,
                                         directed = F)
```

```{r}
ggraph(my_network_clustered, layout = "kk") +
  geom_edge_diagonal(color = "grey70") +
  geom_node_point(size = 3, shape = 21, color = "white", 
                  aes(fill = as.factor(Module))) +
  geom_node_text(repel = T, aes(label = node_table_clustered$nodes)) +
  scale_fill_manual(values = brewer.pal(8, "Set2")) +
  labs(fill = "Module") +
  theme_void() +
  theme(
    legend.position = c(0.2, 0.25)
  )

ggsave("../Results/07_my_network3.png", height = 3, width = 3.5, bg = "white")
```
![network3](https://github.com/cxli233/Online_R_learning/blob/master/Quick_data_vis/Results/07_my_network3.png) 

The key addition in this code chunk is `aes(fill = as.factor(Module)` inside `geom_node_point()`. 
Now the nodes are colored by which module they belong to. 
You can see that the clustering actually makes sense: 
Nodes that are highly interconnected are assigned in the same cluster. 
Incidentally, this network has 3 modules. 
While there are multiple connections _within_ each module, 
there is only one edge _between_ each module. 

One application of network analysis is in gene expression. 
In gene co-expression networks, each node is a gene, 
and an edge is drawn between two genes if two genes are co-expressed (having very similar expression patterns). 
Gene co-expression modules represent groups of genes that may function in the same biological process. 

# Homework 
## Q1 
Metabolic pathways can be modeled by networks, where each node is a metabolite and each edge is a metabolic enzyme. 
You can read more about how to represent metabolic pathways [here](https://github.com/cxli233/ggpathway). 

Here is an example:
```{r}
calvin_cycle_edges <- read_excel("../Data/Calvin_cycle_edges.xlsx")
calvin_cycle_nodes <- read_excel("../Data/Calvin_cycle_nodes.xlsx")

head(calvin_cycle_edges)
head(calvin_cycle_nodes)
```
Make a network diagram for the Calvin cycle. 
Color each node by how many carbons they have (the `carbon` column in the node table).
Label each metabolite using `geom_node_text(aes(label = name), hjust = 0.5, repel = T)`. 
Save the figure using `ggsave()`.
Use `bg = "white"` inside `ggsave()` to add a white background when exporting the png file.  

 
## Q2 
[Phylogenetic trees](https://en.wikipedia.org/wiki/Phylogenetic_tree) and [dendrogram](https://en.wikipedia.org/wiki/Dendrogram) can be modeled by a network. 
Say we have a tree like this: 

```    
    I
    |
    G - H
    |
A - B - C 
    |   
    D - E
    |
    F
``` 

Make a network diagram and label each node.
Save the diagram using `ggave()`.
Use `bg = "white"` inside `ggsave()` to add a white background when exporting the png file.  
Hint: You can write the edge table in Excel and import it into R if that's easier. 
