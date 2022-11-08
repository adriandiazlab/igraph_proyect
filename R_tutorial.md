# Tutorial

The main goals of the igraph library is to provide a set of data types and functions for 1) pain-free implementation of graph algorithms, 2) fast handling of large graphs, with millions of vertices and edges, 3) allowing rapid prototyping via high level languages like R.


## Creating a graph

The simplest way to create a graph is the `Graph` constructor. To make an empty graph:

	> gi <- make_empty_graph()

To make a graph with 10 nodes (numbered `1` to `10`) and two edges connecting nodes `1-2` and `1-5`:

	> gi <- graph(edges = c(1,2, 1,5), n=10, directed = FALSE)
	
We can print the graph to get a summary of its nodes and edges:

	> print(gi)
	IGRAPH 26b9307 U--- 10 2 -- 
	+ edges from 26b9307:
	[1] 1--2 1--5
	
This means: **U** ndirected graph with **10** vertices and **2** edges, with the exact edges listed out. If the graph has a [name] attribute, it is printed as well.

Note

`summary` is similar to `print` but does not list the edges, which is convenient for large graphs with millions of edges:

	> summary(gi)
	IGRAPH 26b9307 U--- 10 2 -- 

## Adding/deleting vertices and edges

Let\'s start from the empty graph again. To add vertices to an existing graph, use `add_vertices`:

	> gi <- make_empty_graph()

	> gi <- add_vertices(gi,3)
	
In igraph, vertices are always numbered up from one. The number of a vertex is called the *vertex ID*. A vertex might or might not have a name.

Similarly, to add edges use `add_edges`:

	> gi <- add_edges(gi, edges = c(1,2, 1,3))
	
Edges are added by specifying the source and target vertex for each edge. This call added two edges, one connecting vertices `1` and `2`, and one connecting vertices `1` and `3`. Edges are also numbered up from one (the *edge ID*) and have an optional name.

**Warning**

Creating an empty graph and adding vertices and edges as shown here can be much slower than creating a graph with its vertices and edges as demonstrated earlier. If speed is of concern, you should especially avoid adding vertices and edges *one at a time*. If you need to do it anyway, you can use `add_vertex` and `add_edge`.

If you try to add edges to vertices with invalid IDs (i.e., you try to add an edge to vertex `5` when the graph has only three vertices), you get an `igraph.InternalError` exception:

	> gi <- add_edges(gi, edges = c(5,4))
	Error in add_edges(gi, edges = c(5, 4)) : 
  	At core/graph/type_indexededgelist.c:265 : cannot add edges, Invalid vertex id

The message tries to explain what went wrong (`cannot add edges. -- Invalid vertex id`).


Let us add some more vertices and edges to our graph:

	> gi <- add_edges(gi, edges = c(3,1))
	> gi <- add_vertices(gi, 3)
	> gi <- add_edges(gi, edges = c(3,4, 4,5, 5,6, 6,4))
	> print(gi)
	IGRAPH 891a338 D--- 6 7 -- 
	+ edges from 891a338:
	[1] 1->2 2->3 3->1 3->4 4->5 5->6 6->4

We now have an directed graph with 6 vertices and 7 edges. Vertex and edge IDs are always *continuous*, so if you delete a vertex all subsequent vertices will be renumbered. When a vertex is renumbered, edges are **not** renumbered, but their source and target vertices will. Use `Graph.delete_vertices` and `Graph.delete_edges` to perform these operations. For instance, to delete the edge connecting vertices `3-4`, get its ID and then delete it:


	> get.edge.ids(gi, c(3,4))
	[1] 4
	gi <- delete.edges(gi, 4)

## Generating graphs

igraph includes both deterministic and stochastic graph generators (see `generation`). *Deterministic* generators produce the same graph every time you call the fuction, e.g.:

	> gi <- make_empty_graph()
	> gi <- make_tree(127, 2, mode = "undirected")
	> summary(gi)
	IGRAPH 54ab2cf U--- 127 126 -- Tree
	+ attr: name (g/c), children (g/n), mode (g/c)

Uses `make_tree` to generate a regular tree graph with 127 vertices, each vertex having two children (and one parent, of course). No matter how many times you call `make_tree`, the generated graph will always be the same if you use the same parameters:


	> gi2 <- make_tree(127, 2, mode = "undirected")

	> get.edgelist(gi) == get.edgelist(gi2)
	      [,1] [,2]
 	[1,] TRUE TRUE
 	[2,] TRUE TRUE
 		...

The above code snippet also shows you that the `get_edgelist()` method, which returns a list of source and target vertices for all edges, sorted by edge ID. If you print the first 10 elements, you get:

	> head(get.edgelist(gi2),10)
	      [,1] [,2]
	[1,]    1    2
	[2,]    1    3
	[3,]    2    4
	[4,]    2    5
	[5,]    3    6
	[6,]    3    7
	[7,]    4    8
	[8,]    4    9
	[9,]    5   10
	[10,]    5   11


*Stochastic* generators produce a different graph each time, e.g. `sample_grg`:

	> gi <- make_empty_graph()
	> gi <- sample_grg(100, 0.2)
	> summary(gi)
	IGRAPH 573863e U--- 100 535 -- Geometric random graph
	+ attr: name (g/c), radius (g/n), torus (g/l)


This generates a geometric random graph: *n* points are chosen randomly and uniformly inside the unit square and pairs of points closer to each other than a predefined distance *d* are connected by an edge. If you generate GRGs with the same parameters, they will be different:

	> gi2 <- make_empty_graph()
	> gi2 <- sample_grg(100, 0.2)

	> gi_a <- head(get.edgelist(gi), 10) 
	> gi2_a <-head(get.edgelist(gi2), 10)
	
	> gi_a == gi2_a
	       [,1]  [,2]
 	[1,]  TRUE FALSE
 	[2,]  TRUE FALSE
 	[3,]  TRUE  TRUE
 	[4,]  TRUE FALSE
 	[5,]  TRUE FALSE
 	[6,]  TRUE FALSE
 	[7,] FALSE FALSE
 	[8,]  TRUE FALSE
 	[9,]  TRUE FALSE
	[10,]  TRUE FALSE

A slightly looser way to check if the graphs are equivalent is via `isomorphic()`:

	> isomorphic(gi,gi2)
	[1] FALSE
	
Checking for isomorphism can take a while for large graphs (in this case, the answer can quickly be given by checking the degree distributions of the two graphs).


## Setting and retrieving attributes

As mentioned above, in igraph each vertex and each edge have a numeric id from `1` upwards. Deleting vertices or edges can therefore cause reassignments of vertex and/or edge IDs. In addition to IDs, vertex and edges can have *attributes* such as a name, coordinates for plotting, metadata, and weights. The graph itself can have such attributes too (e.g. a name, which will show in `print` or `summary`). In a sense, every `Graph`, vertex and edge can be used as a R variable to store and retrieve these attributes.

To demonstrate the use of attributes, let us create a simple social network:

	> gi <- make_empty_graph()
	> gi <- graph(edges = c(1,2, 1,3, 3,4, 4,5, 5,3, 3,6, 6,1, 7,4, 6,7), directed = FALSE)

Each vertex represents a person, so we want to store names, ages and genders:

	> V(gi)$name <- c("Alice", "Bob", "Claire", "Dennis", "Esther", "Frank", "George")
	> V(gi)$age <- c(25, 31, 18, 47, 22, 23, 50) 
	> V(gi)$gender <- c("f", "m", "f", "m", "f", "m", "m")
	> E(gi)$is_formal <- c(FALSE, FALSE, TRUE, TRUE, TRUE, FALSE, TRUE, FALSE, FALSE)

	IGRAPH 0ec8a35 UN-- 7 9 -- 
	+ attr: name (v/c), age (v/n), gender (v/c), is_formal (e/l)

`V` and `E` are the standard way to obtain a sequence of all vertices and edges, respectively. The value must be a list with the same length as the vertices (for `V`) or edges (for `E`). This assigns an attribute to *all* vertices/edges at once.


To assign or modify an attribute for a single vertex/edge, you can use indexing:

	> E(gi)$is_formal
	[1] FALSE FALSE  TRUE  TRUE  TRUE FALSE  TRUE FALSE FALSE
    
	> E(gi)$is_formal[1] <- c(TRUE)
	
	> E(gi)$is_formal
	[1]  TRUE FALSE  TRUE  TRUE  TRUE FALSE  TRUE FALSE FALSE

In fact, a single vertex is represented via the class `Vertex`, and a single edge via `Edge`. Both of them plus `Graph` can all be keyed like a dictionary to set attributes, e.g. to add a date to the graph:


	> gi <- set_graph_attr(gi, "date", "2022-02-11")
	> graph_attr(gi, "date")
	[1] "2022-02-11"

To retrieve a dictionary of attributes, you can use `graph_attr`, `vertex_attr`, and `edge_attr`. To find the ID of a vertex you can use the function `match`:

	> match(c("George"),V(gi)$name)
	[1] 7


<!---
I don't know how to find the index of edges and the functions Edge.source`, `Edge.target` and Edge.tuple I can't find them either.
-->

To assign attributes to a subset of vertices or edges, you can use slicing:

	> V(gi)$name[1:3] <- c("Jimena", "Claudio", "Carmina")
	> V(gi)
	+ 7/7 vertices, named, from 0ec8a35:
	[1] Jimena  Claudio Carmina Dennis  Esther  Frank   George 


To delete attributes:

	> gi <- delete_vertex_attr(gi,"name")
	> V(gi)$name
	NULL

<!---
Does this warning also apply to R?: Attributes can be arbitrary Python objects, but if you are saving graphs to a file, only string and numeric attributes will be kept. See the `pickle` module in the standard Python library if you are looking for a way to save other attribute types. You can either pickle your attributes individually, store them as strings and save them, or you can pickle the whole `Graph` if you know that you want to load the graph back into Python only.
-->

## Structural properties of graphs

Besides the simple graph and attribute manipulation routines described above, igraph provides a large set of methods to calculate various structural properties of graphs. It is beyond the scope of this tutorial to document all of them, hence this section will only introduce a few of them for illustrative purposes. We will work on the small social network we built in the previous section.

Probably the simplest property one can think of is the `vertex degree`. The degree of a vertex equals the number of edges adjacent to it. In case of directed networks, we can also define `in-degree(the number of edges pointing towards the vertex) and `out-degree` (the number of edges originating from the vertex). igraph is able to calculate all of them using a simple syntax:

	> degree(gi)
	[1] 3 1 4 3 2 3 2

If the graph was directed, we would have been able to calculate the in- and out-degrees separately using `degree(mode="in")` and `degree(mode="out")`. You can also pass a single vertex ID or a list of vertex IDs to `degree` if you want to calculate the degrees for only a subset of vertices:

	> degree(gi,7)
	[1] 2
	> degree(gi, v = c(3,4,5))
	[1] 4 3 2

This calling convention applies to most of the structural properties igraph can calculate. For vertex properties, the methods accept a vertex ID or a list of vertex IDs (and if they are omitted, the default is the set of all vertices). For edge properties, the methods accept a single edge ID or a list of edge IDs. Instead of a list of IDs, you can also supply a `VertexSeq` or an `EdgeSeq` instance appropriately. Later in the `next chapter <querying_vertices_and_edges>`, you will learn how to restrict them to exactly the vertices or edges you want.

Note

For some measures, it does not make sense to calculate them only for a few vertices or edges instead of the whole graph, as it would take the same time anyway. In this case, the methods won\'t accept vertex or edge IDs, but you can still restrict the resulting list later using standard list indexing and slicing operators. One such example is eigenvector centrality (`evcent()`).

Besides degree, igraph includes built-in routines to calculate many other centrality properties, including vertex and edge betweenness (`Graph.betweenness <GraphBase.betweenness>`, `Graph.edge_betweenness <GraphBase.edge_betweenness>`) or Google\'s PageRank (`Graph.pagerank`) just to name a few. Here we just illustrate edge betweenness:

	> edge_betweenness(gi)
	[1] 6 6 4 2 4 3 4 3 4
	
<!---
how to do this Python magic?:

    >>> ebs = g.edge_betweenness()
    >>> max_eb = max(ebs)
    >>> [g.es[idx].tuple for idx, eb in enumerate(ebs) if eb == max_eb]
    [(0, 1), (0, 2)]-->

Most structural properties can also be retrieved for a subset of vertices or edges or for a single vertex or edge by calling the appropriate method on `Vertex` or `Edge` object of interest:

	> degree(gi)
	[1] 3 1 4 3 2 3 2
   	> degree(gi)[3]
	[1] 4
	
## Querying vertices and edges based on attributes

### Selecting vertices and edges

Imagine that in a given social network (use the original example), you would like to find out who has the largest degree or betweenness centrality. You can do that with the tools presented so far and some basic R knowledge:

	> which.max(degree(gi))
	Claire 
     	3 


<!---
Another way to do it is:

	> V(gi)$name[degree(gi)==max(degree(gi))]
	[1] "Claire"  
	
But this solution was obtained from stackoverflow 
-->












	
