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

Similarly, to add vertices use `add_edges`:

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

[comment]: # (The next paragraphs are not well related with the method that I used in R I think)

If the first positional argument is a callable object (i.e., a function, a bound method or anything that behaves like a function), the object will be called for every vertex that\'s currently in the sequence. If the function returns `True`, the vertex will be included, otherwise it will be excluded:
 
	> graph <- graph.full(n=10)
	> only_odd_vertices <- which(V(graph)%%2==1)
	>  length(only_odd_vertices)
	[1] 5

If the first positional argument is an iterable (i.e., a list, a generator or anything that can be iterated over), it *must* return integers and these integers will be considered as indices into the current vertex set (which is *not* necessarily the whole graph). Only those vertices that match the given indices will be included in the filtered vertex set. Floats, strings, invalid vertex IDs will display an error message:

	
	> seq <- V(graph)[2,3,7]
	> length(seq)
	[1] 3
	> seq
	+ 3/10 vertices, from cc79269:
	[1] 2 3 7
	> seq <- seq[1,3]    #filtering an existing vertex set
	> > seq
	+ 2/10 vertices, from cc79269:
	[1] 2 7
	> seq <- V(graph)[2,3,7,"foo", 3.5]
	Error in simple_vs_index(x, ii, na_ok) : Unknown vertex selected


Operators can be used to filter the vertices based on their attributes or their structural properties. First select the name of the attribute or structural property and then use the `which` function to evaluate each vertex using some operator and another attribute. For instance, the following command gives you people younger than 30 years in our imaginary social network:

	> V(gi)$name[which(V(gi)$age < 30)] 
	[1] "Alice"  "Claire" "Esther" "Frank" 



The possibilities are as follows:

| Operator | Meaning                                                                                                                             |
|----|----|
| `==`        | The attribute/property value must be *equal to*                                                    |
| `!=`        | The attribute/property value must *not be equal to*                                                |
| `<`        | The attribute/property value must be *less than*                                                   |
| `<=`        | The attribute/property value must be *less than or equal to*                                     |
| `>`        | The attribute/property value must be *greater than*                                          |
| `>=`        | The attribute/property value must be *greater than or equal to*                                |
| `%in%`        | The attribute/property value must be *included in*         |
| `%notin% <- Negate(%in%)`     | The attribute/property value must *not be included in*  |
  

Theoretically, it can happen that there exists an attribute and a structural property with the same name (e.g., you could have a vertex attribute named `degree`). In that case, just be careful to reference the attribute correctly and then call the function to calculate the structural property. For instance, we create a new attribute called degree:

	> V(gi)$degree <- c("A", "B", "C", "D", "E", "F", "G")
	> V(gi)$degree[degree(gi) == 3]
	[1] "A" "D" "F"
	> V(gi)$name[degree(gi) == 3]
	[1] "Alice"  "Dennis" "Frank" 

There are also a few special structural properties for selecting edges:

-Using `from` or `to` based on the source vertices of the edges. E.g., to select all the edges originating from Claire (who has vertex index 3):


	> E(gi)[from(3)]
	+ 4/9 edges from e0d557b (vertex names):
	[1] Alice --Claire Claire--Dennis Claire--Esther Claire--Frank
	

Using `to` filter based on the target vertices. This is different from `from` if the graph is directed.

For instance, the following expression selects all the edges between Claire (vertex index 2), Dennis (vertex index 3) and Esther (vertex index 4). The expression %--% is a special operator that can be used to select all edges between two sets of vertices. It ignores the edge directions in directed graphs.

	> E(gi) [ 3:4 %--% 4:5 ]
	+ 3/9 edges from 81fc383 (vertex names):
	[1] Claire--Dennis Dennis--Esther Claire--Esther


You can build lists based on attributes and evaluate the edges that originate with one set and end with the other. E.g., to select all the edges that connect men to women:


	> men <- V(gi)$name[(which(V(gi)$gender == "m"))]
	[1] "Bob"    "Dennis" "Frank"  "George"
	> women <- V(gi)$name[(which(V(gi)$gender == "f"))] 
	[1] "Alice"  "Claire" "Esther"
	E(gi)[men %--% women]
	+ 5/9 edges from 81fc383 (vertex names):
	[1] Alice --Bob    Claire--Dennis Dennis--Esther Claire--Frank  Alice --Frank
	

### Finding a single vertex or edge with some properties


In many cases we are looking for a single vertex or edge of a graph with some properties, and either we do not care which one of the matches is returned if there are multiple matches, or we know in advance that there will be only one match. A typical example is looking up vertices by their names in the `name` property. 

For instance, to look up the vertex corresponding to Claire, one can do this:


	> Claire <- match(c("Claire"),V(gi)$name)
	[1] 3


Looking up an unknown name will yield an exception:

	> match(c("Joe"),V(gi)$name)
	[1] NA


### Looking up vertices by names

Looking up vertices by names is a very common operation, and it is usually much easier to remember the names of the vertices in a graph than their IDs. To this end, igraph treats the `name` attribute of vertices specially; they are indexed such that vertices can be looked up by their names in amortized constant time. E.g, you can simply look up the degree (number of connections) of Dennis as follows:

	> degree(gi,v="Dennis")
	Dennis 
	3 

The mapping between vertex names and IDs is maintained transparently by igraph in the background; whenever the graph changes, igraph also updates the internal mapping. However, uniqueness of vertex names is *not* enforced; you can easily create a graph where two vertices have the same name, but igraph will return only one of them when you look them up by names, the other one will be available only by its index.


## Treating a graph as an adjacency matrix

Adjacency matrix is another way to form a graph. In adjacency matrix, rows and columns are labeled by graph vertices: the elements of the matrix indicate whether the vertices *i* and *j* have a common edge (*i, j*). The adjacency matrix for the example graph is:

	> get.adjacency(gi)
	7 x 7 sparse Matrix of class "dgCMatrix"
	         Alice Bob Claire Dennis Esther Frank George
	Alice      .   1      1      .      .     1      .
	Bob        1   .      .      .      .     .      .
	Claire     1   .      .      1      1     1      .
	Dennis     .   .      1      .      1     .      1
	Esther     .   .      1      1      .     .      .
	Frank      1   .      1      .      .     .      1
	George     .   .      .      1      .     1      .

For example, Claire (`[1, 0, 0, 1, 1, 1, 0]`) is directly connected to Alice (who has vertex index 1), Dennis (index 4), Esther (index 5), and Frank (index 6), but not to Bob (index 2) nor George (index 7).


## Layouts and plotting 

A graph is an abstract mathematical object without a specific representation in 2D or 3D space. This means that whenever we want to visualise a graph, we have to find a mapping from vertices to coordinates in two- or three-dimensional space first, preferably in a way that is pleasing for the eye. A separate branch of graph theory, namely graph drawing, tries to solve this problem via several graph layout algorithms. igraph implements quite a few layout algorithms and is also able to draw them onto the screen or to a PDF, PNG or SVG file using the [Cairo library](https://www.cairographics.org).


### Layout algorithms

The layout methods in igraph are to be found in the `Graph` object, and they always start with `layout`. The following table summarises them:

| Method name  |  Algorithm description |
| --- | --- |
|`layout_in_circle` `layout.circle`  | Deterministic layout that places the vertices on a circle |
|`layout_with_drl` `layout.drl`  | The [Distributed Recursive Layout]() algorithm for large graphs |
|`layout.fruchterman.reingold`| Fruchterman-Reingold force-directed algorithm |
|`layout_with_kk` `layout.kamada.kawai` | Kamada-Kawai force-directed algorithm |
|`layout_with_lgl` `layout.lgl` | The [Large Graph Layout]() algorithm for large graphs | 
|`layout.random`| Places the vertices completely randomly |
|`layout_randomly`| Places the vertices completely randomly in 3D |
|`layout.reingold.tilford`| Reingold-Tilford tree layout, useful for (almost) tree-like graphs |
|`layout_as_tree`| Reingold-Tilford tree layout with a polarcoordinate post-transformation, useful for (almost) tree-like graphs |
|`layout_on_sphere` `layout.sphere`  | Deterministic layout that places the vertices evenly on the surface of a sphere |


Layout algorithms can either be called directly:

	> layout <- layout.kamada.kawai(gi)
	> layout <- layout_with_kk(gi)


For instance, the following two calls are completely equivalent:

	> layout <- layout.reingold.tilford(gi, root= 2)
	> layout <- layout_as_tree(gi, root = 2)

Layout methods return a `Layout` object which behaves mostly like a list of lists. Each list entry in a `Layout` object corresponds to a vertex in the original graph and contains the vertex coordinates in the 2D or 3D space. `Layout` objects also contain some useful methods to translate, scale or rotate the coordinates in a batch. However, the primary utility of `Layout` objects is that you can pass them to the `plot` function along with the graph to obtain a 2D drawing.


### Drawing a graph using a layout

For instance, we can plot our imaginary social network with the Kamada-Kawai layout algorithm as follows:

	> layout <- layout_with_kk(gi)
	> plot(gi, layout = layout)

This should open an external image viewer showing a visual representation of the network, something like the one on the following figure (although the exact placement of nodes may be different on your machine since the layout is not deterministic):

![alt text](https://github.com/adriandiazlab/igraph_proyect/blob/main/Images/tutorial_social_network_R_1.png?raw=true)

Our social network with the Kamada-Kawai layout algorithm

Hmm, this is not too pretty so far. A trivial addition would be to use the names as the vertex labels and to color the vertices according to the gender. Vertex labels are taken from the `label` attribute by default and vertex colors are determined by the `color` attribute. For the first example, it is important to take into account that the attributes that we select as a condition (i.e gender) to select a color are integers and not strings.

So we can simply create these attributes and re-plot the graph:

V(gi)$gender_num <- c(1,2,1,2,1,2,2)
colors <- c("yellow", "red")
V(gi)$color <- colors[V(gi)$gender_num]
plot(gi, layout = layout)

Anothers ways to approach this example:

	> V(gi)$color <- ifelse(V(gi)$gender == "m", "red","yellow")
	> plot(gi, layout = layout)

	> plot(gi, layout= layout, vertex.color=as.factor(V(gi)$gender))
	
	> plot(gi, layout= layout, vertex.color=c( "yellow", "red")[1+(V(gi)$gender == "f")])

The result is:

![alt text](https://github.com/adriandiazlab/igraph_proyect/blob/main/Images/tutorial_social_network_R_2.png?raw=true)

Our social network - with names as labels and genders as colors

Instead of specifying the visual properties as vertex and edge attributes, you can also give them as arguments to `plot`:

	> plot(gi, layout =layout, vertex.size = 20, margin = 0.5,
	       vertex.color=c( "red", "yellow")[1+(V(gi)$gender == "m")], 
	       edge.width=c(1,3)[1+(E(gi)$is_formal == TRUE)],
	       )

This latter approach is preferred if you want to keep the properties of the visual representation of your graph separate from the graph itself. The final plot shows the formal ties with thick lines while informal ones with thin lines:

![alt text](https://github.com/adriandiazlab/igraph_proyect/blob/main/Images/tutorial_social_network_R_3.png?raw=true)









	
