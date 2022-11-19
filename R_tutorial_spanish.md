# Tutorial

El objetivo principal de igraph es proporcionar un conjunto de tipos de datos y funciones para 1) implementar sin problemas algoritmos de grafos, 2) manejar rápidamente grandes grafos con millones de vértices y aristas, 3) permitir la creación rápida de prototipos mediante lenguajes de alto nivel como R.

## Crear un grafo

La forma más sencilla de crear un grafo es con el constructor `Graph`. Para hacer un grafo vacío:

	> gi <- make_empty_graph()


Para hacer un grafo con 10 nodos (numerados `1` a `10`) y dos aristas que conecten los nodos `1-2` y `1-5`:

	> gi <- graph(edges = c(1,2, 1,5), n=10, directed = FALSE)
	
Podemos imprimir el grafo para obtener un resumen de sus nodos y aristas:

	> print(gi)
	IGRAPH 26b9307 U--- 10 2 -- 
	+ edges from 26b9307:
	[1] 1--2 1--5
	
Tenemos entonces: grafo no dirigido (**U**ndirected)  con **10** vértices y **2** aristas, que se enlistan en la última parte. Si el grafo tiene un atributo [nombre], también se imprime.

Nota

`summary` es similar a `print` pero no enlista las aristas, lo cual es conveniente para grafos grandes con millones de aristas:

	> summary(gi)
	IGRAPH 26b9307 U--- 10 2 -- 
	
## Añadir y borrar vértices y aristas

Empecemos de nuevo con un grafo vacío. Para añadir vértices a un grafo existente, utiliza `add_vertices`:

	> gi <- make_empty_graph()

	> gi <- add_vertices(gi,3)
	
En igraph, los vértices se numeran siempre a partir de uno. El número de un vértice es el *ID del vértice*. Un vértice puede tener o no un nombre.

Del mismo modo, para añadir aristas se utiliza `add_edges`:

	> gi <- add_edges(gi, edges = c(1,2, 1,3))
	
Las aristas se añaden especificando el vértice origen y el vértice destino de cada arista. Esta llamada añade dos aristas, una que conecta los vértices `1` y `2`, y otra que conecta los vértices `1` y `3`. Las aristas también se numeran a partir de uno (el *ID del arista*) y tienen un nombre opcional.

**Atención**

Crear un grafo vacío y añadir vértices y aristas como se muestra aquí puede ser mucho más lento que crear un grafo con sus vértices y aristas como se ha demostrado anteriormente. Si la velocidad es una preocupación, deberías evitar especialmente añadir vértices y aristas *de uno en uno*. Si necesitas hacerlo de todos modos, puedes usar `add_vertex` y `add_edge`.

Si intentas añadir aristas a vértices con IDs no válidos (por ejemplo, intentas añadir una arista al vértice `5` cuando el grafo sólo tiene tres vértices), obtienes un error:

	> gi <- add_edges(gi, edges = c(5,4))
	Error in add_edges(gi, edges = c(5, 4)) : 
  	At core/graph/type_indexededgelist.c:265 : cannot add edges, Invalid vertex id
  	
Añadamos más vértices y aristas a nuestro grafo:

	> gi <- add_edges(gi, edges = c(3,1))
	> gi <- add_vertices(gi, 3)
	> gi <- add_edges(gi, edges = c(3,4, 4,5, 5,6, 6,4))
	> print(gi)
	IGRAPH 891a338 D--- 6 7 -- 
	+ edges from 891a338:
	[1] 1->2 2->3 3->1 3->4 4->5 5->6 6->4
	
Ahora tenemos un grafo dirigido con 6 vértices y 7 aristas. Los IDs de los vértices y aristas son siempre *continuos*, por lo que si eliminas un vértice todos los vértices subsiguientes serán renumerados. Cuando se renumera un vértice, las aristas **no** se renumeran, pero sí sus vértices de origen y destino. Utilice `delete.vertices` y `delete.edges` para realizar estas operaciones. Por ejemplo, para eliminar la arista que conecta los vértices `3-4`:

	> get.edge.ids(gi, c(3,4))
	[1] 4
	gi <- delete.edges(gi, 4)
	
## Generar grafos

igraph incluye generadores de grafos tanto deterministas como estocásticos. Los generadores *deterministas* producen el mismo grafo cada vez que se llama a la función, por ejemplo:

	> gi <- make_empty_graph()
	> gi <- make_tree(127, 2, mode = "undirected")
	> summary(gi)
	IGRAPH 54ab2cf U--- 127 126 -- Tree
	+ attr: name (g/c), children (g/n), mode (g/c)
	
Utiliza `make_tree` para generar un grafo regular en forma de árbol con 127 vértices, cada vértice con dos hijos (y un padre, por supuesto). No importa cuántas veces llames a `make_tree`, el grafo generado será siempre el mismo si utilizas los mismos parámetros:

	> gi2 <- make_tree(127, 2, mode = "undirected")

	> get.edgelist(gi) == get.edgelist(gi2)
	      [,1] [,2]
 	[1,] TRUE TRUE
 	[2,] TRUE TRUE
 		...
 		
El fragmento de código anterior también muestra el método `get_edgelist()`, que devuelve una lista de vértices de origen y destino para todas las aristas, ordenados por el ID de la arista. Si imprimes los 10 primeros elementos, obtienes:

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
	
Los generadores *estocásticos* producen un grafo diferente cada vez; por ejemplo, `sample_grg`:

	> gi <- make_empty_graph()
	> gi <- sample_grg(100, 0.2)
	> summary(gi)
	IGRAPH 573863e U--- 100 535 -- Geometric random graph
	+ attr: name (g/c), radius (g/n), torus (g/l)
	
Esto genera un grafo geométrico aleatorio: Se eligen *n* puntos de forma aleatoria y uniforme dentro del cuadrado unitario y los pares de puntos más cercanos entre sí respecto a una distancia predefinida *d* se conectan mediante una arista. Si se generan GRGs con los mismos parámetros, serán diferentes:


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
	
Una forma un poco más relajada de comprobar si los grafos son equivalentes es mediante `isomorphic()`:

	> isomorphic(gi,gi2)
	[1] FALSE
	
Comprobar por el isomorfismo puede llevar un tiempo en el caso de grafos grandes (en este caso, la respuesta puede darse rápidamente comprobando las distribuciones de grados de los dos grafos).

## Establecer y recuperar atributos

Como se ha mencionado anteriormente, en igraph cada vértice y cada arista tienen un ID numérico de `1` en adelante. Por lo tanto, la eliminación de vértices o aristas puede causar la reasignación de los ID de vértices y/o aristas. Además de los IDs, los vértices y aristas pueden tener *atributos* como un nombre, coordenadas para graficar, metadatos y pesos. El propio grafo puede tener estos atributos también (por ejemplo, un nombre, que se mostrará en `print` o `summary`). En cierto sentido, cada `Graph`, vértice y arista puede ser utilizado como una variable R para almacenar y recuperar estos atributos.

Para demostrar el uso de los atributos, creemos una red social sencilla:

	> gi <- make_empty_graph()
	> gi <- graph(edges = c(1,2, 1,3, 3,4, 4,5, 5,3, 3,6, 6,1, 7,4, 6,7), directed = FALSE)

Cada vértice representa una persona, por lo que queremos almacenar nombres, edades y géneros:

	> V(gi)$name <- c("Alice", "Bob", "Claire", "Dennis", "Esther", "Frank", "George")
	> V(gi)$age <- c(25, 31, 18, 47, 22, 23, 50) 
	> V(gi)$gender <- c("f", "m", "f", "m", "f", "m", "m")
	> E(gi)$is_formal <- c(FALSE, FALSE, TRUE, TRUE, TRUE, FALSE, TRUE, FALSE, FALSE)
	
	IGRAPH 0ec8a35 UN-- 7 9 -- 
	+ attr: name (v/c), age (v/n), gender (v/c), is_formal (e/l)
	
`V` y `E` son la forma estándar de obtener una secuencia de todos los vértices y aristas respectivamente. El valor debe ser una lista con la misma longitud que los vértices (para `V`) o aristas (para `E`). Esto asigna un atributo a *todos* los vértices/aristas a la vez.

Para asignar o modificar un atributo para un solo vértice/borde, puedes hacer lo siguiente:

	> E(gi)$is_formal
	[1] FALSE FALSE  TRUE  TRUE  TRUE FALSE  TRUE FALSE FALSE
    
	> E(gi)$is_formal[1] <- c(TRUE)
	
	> E(gi)$is_formal
	[1]  TRUE FALSE  TRUE  TRUE  TRUE FALSE  TRUE FALSE FALSE
	
De hecho, un solo vértice se representa mediante la clase `Vertex`, y una sola arista mediante `Edge`. Ambos, junto con `Graph`, pueden ser tecleados como un diccionario para establecer atributos, por ejemplo, para añadir una fecha al grafo:

	> gi <- set_graph_attr(gi, "date", "2022-02-11")
	> graph_attr(gi, "date")
	[1] "2022-02-11"
	
Para recuperar un diccionario de atributos, puedes utilizar `graph_attr`, `vertex_attr` y `edge_attr`. Para encontrar el ID de un vértice puedes utilizar la función `match`:

	> match(c("George"),V(gi)$name)
	[1] 7

Para asignar atributos a un subconjunto de vértices o aristas, puedes utilizar el corte:

	> V(gi)$name[1:3] <- c("Alejandra", "Bruno", "Carmina")
	> V(gi)
	+ 7/7 vertices, named, from 0ec8a35:
	[1] Alejandra  Bruno Carmina Dennis  Esther  Frank   George 
	
Para eliminar atributos:

	> gi <- delete_vertex_attr(gi,"name")
	> V(gi)$name
	NULL
	
## Propiedades estructurales de los grafos

Además de las funciones simples de manipulación de grafos y atributos descritas anteriormente, igraph proporciona un amplio conjunto de métodos para calcular varias propiedades estructurales de los grafos. Está más allá del alcance de este tutorial documentar todos ellos, por lo que esta sección sólo presentará algunos de ellos con fines ilustrativos. Trabajaremos con la pequeña red social que construimos en la sección anterior.

Probablemente, la propiedad más sencilla en la que se puede pensar es el "grado del vértice". El grado de un vértice es igual al número de aristas incidentes a él. En el caso de los grafos dirigidos, también podemos definir el `grado de entrada` (el número de aristas que apuntan hacia el vértice) y el `grado de salida` (el número de aristas que se originan en el vértice). 

	> degree(gi)
	[1] 3 1 4 3 2 3 2

Si el grafo fuera dirigido, habríamos podido calcular los grados de entrada y salida por separado utilizando `degree(mode="in")` y `degree(mode="out")`. También puedes usar un único ID de un vértice o una lista de ID de los vértices a `degree` si quieres calcular los grados sólo para un subconjunto de vértices:

	> degree(gi,7)
	[1] 2
	> degree(gi, v = c(3,4,5))
	[1] 4 3 2
	
Este procedimiento se aplica a la mayoría de las propiedades estructurales que igraph puede calcular. Para las propiedades de los vértices, los métodos aceptan un ID o una lista de IDs de los vértices (y si se omiten, el valor predeterminado es el conjunto de todos los vértices). Para las propiedades de las aristas, los métodos también aceptan un único ID de o una lista de IDs de aristas. Más adelante, en el próximo capítulo <consulta de vértices y aristas>, aprenderás a restringirlos exactamente a los vértices o aristas que quieras.

Nota

Para algunos casos, no tiene sentido realizar el calculo sólo para unos pocos vértices o aristas en lugar de todo el grafo, ya que de todas formas se tardaría el mismo tiempo. En este caso, los métodos no aceptan IDs de vértices o aristas, pero se puede restringir la lista resultante más tarde usando operadores estándar de indexación y de corte. Un ejemplo de ello es la centralidad de los vectores propios (`evcent()`).

Además de los grados, igraph incluye rutinas integradas para calcular muchas otras propiedades de centralidad, como la intermediación de vértices y aristas o el PageRank de Google (`Graph.pagerank`), por nombrar algunas. Aquí sólo ilustramos la interrelación de aristas:

	> edge_betweenness(gi)
	[1] 6 6 4 2 4 3 4 3 4

La mayoría de las propiedades estructurales también pueden ser obtenidas para un subconjunto de vértices o aristas o para un solo vértice o arista llamando al método apropiado en el objeto `Vertex` o `Edge` de interés:

	> degree(gi)
	[1] 3 1 4 3 2 3 2
   	> degree(gi)[3]
	[1] 4
	
## Busqueda de vértices y aristas basada en atributos

### Selección de vértices y aristas

Tomando como ejemplo la red social anterirormente creada, te gustaría averiguar quién tiene el mayor grado o centralidad de intermediación. Puedes hacerlo con las herramientas presentadas hasta ahora y algunos conocimientos básicos de R:

	> which.max(degree(gi))
	Claire 
     	3 

[comment]: # (The next paragraphs are not well related with the method that I used in R I think)

Si el primer argumento posicional es un objeto invocable (es decir, una función, un método vinculado o cualquier cosa que se comporte como una función), el objeto será llamado para cada vértice que esté actualmente en la secuencia. Si la función devuelve `Verdadero`, el vértice será incluido, en caso contrario será excluido:

	> graph <- graph.full(n=10)
	> only_odd_vertices <- which(V(graph)%%2==1)
	>  length(only_odd_vertices)
	[1] 5
	
Si el primer argumento posicional es un iterable (es decir, una lista, un generador o cualquier cosa sobre la que se pueda iterar), *debe* devolver enteros y estos enteros se considerarán como índices del conjunto de vértices actual (que *no* es necesariamente todo el grafo). Sólo se incluirán en el conjunto de vértices filtrados los vértices que coincidan con los índices dados. Los numero flotantes, las cadenas y los ID de vértices no válidos mostrarán un mensaje de error:

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

Se pueden utilizar operadores para filtrar los vértices en función de sus atributos o sus propiedades estructurales. Primero se selecciona el nombre del atributo o la propiedad estructural y luego se utiliza la función `which` para evaluar cada vértice utilizando algún operador u otro atributo. Por ejemplo, el siguiente comando te da las personas menores de 30 años en nuestra red social imaginaria:

	> V(gi)$name[which(V(gi)$age < 30)] 
	[1] "Alice"  "Claire" "Esther" "Frank" 

Las posibilidades son las siguientes:

| Operador | Significado              |
|------------------|-----------------------|
| `==`        | El valor del atributo/propiedad debe ser *igual* a      |
| `!=`        | El valor del atributo/propiedad debe *no ser igual* a   |
| `<`        | El valor del atributo/propiedad debe ser *menos* que  |
| `<=`        | El valor del atributo/propiedad debe ser *inferior o igual a*  |
| `>`        | El valor del atributo/propiedad debe ser *mayor que*   |
| `>=`        | El valor del atributo/propiedad debe ser *mayor o igual a*  |
| `%in%`        | El valor del atributo/propiedad debe estar *incluido en*      |
| `%notin% <- Negate(%in%)` (primero crea la función)    | El valor del atributo/propiedad debe *no estar incluido en* |

Teóricamente, puede ocurrir que exista un atributo y una propiedad estructural con el mismo nombre (por ejemplo, podrías tener un atributo de vértice llamado "grado"). En ese caso, sólo hay que tener cuidado de referenciar correctamente el atributo y luego llamar a la función para calcular la propiedad estructural. Por ejemplo, creamos un nuevo atributo llamado grado:

	> V(gi)$degree <- c("A", "B", "C", "D", "E", "F", "G")
	> V(gi)$degree[degree(gi) == 3]
	[1] "A" "D" "F"
	> V(gi)$name[degree(gi) == 3]
	[1] "Alice"  "Dennis" "Frank" 

También hay algunas propiedades estructurales especiales para seleccionar los aristas:

Utilizando `from` o `to` en función de los vértices de donde se originan las aristas. Por ejemplo, para seleccionar todas las aristas procedentes de Claire (que tiene el índice de vértice 3):

	> E(gi)[from(3)]
	+ 4/9 edges from e0d557b (vertex names):
	[1] Alice --Claire Claire--Dennis Claire--Esther Claire--Frank
	
Usarr el filtro `to` en base a los vértices de destino. Esto es diferente de `from` si el grafo es dirigido.

Por ejemplo, la siguiente expresión selecciona todas las aristas entre Claire (índice de vértice 2), Dennis (índice de vértice 3) y Esther (índice de vértice 4). La expresión %--% es un operador especial que puede utilizarse para seleccionar todas las aristas entre dos conjuntos de vértices. Ignora las direcciones de las aristas en los grafos dirigidos.

	> E(gi) [ 3:4 %--% 4:5 ]
	+ 3/9 edges from 81fc383 (vertex names):
	[1] Claire--Dennis Dennis--Esther Claire--Esther

Puede construir listas basadas en atributos y evaluar las aristas que se originan en un conjunto y terminan en el otro. Por ejemplo, para seleccionar todas las aristas que conectan a los hombres con las mujeres:

	> men <- V(gi)$name[(which(V(gi)$gender == "m"))]
	[1] "Bob"    "Dennis" "Frank"  "George"
	> women <- V(gi)$name[(which(V(gi)$gender == "f"))] 
	[1] "Alice"  "Claire" "Esther"
	E(gi)[men %--% women]
	+ 5/9 edges from 81fc383 (vertex names):
	[1] Alice --Bob    Claire--Dennis Dennis--Esther Claire--Frank  Alice --Frank
	
### Encontrar un solo vértice o arista con algunas propiedades

En muchos casos buscamos un solo vértice o arista de un grafo con algunas propiedades, sin importar cuál de las coincidencias se devuelve o que sólo sea una coincidencia. Un ejemplo típico es buscar vértices por su nombre en la propiedad `name`.

Por ejemplo, para buscar el vértice correspondiente a Claire, se puede hacer lo siguiente:

	> Claire <- match(c("Claire"),V(gi)$name)
	[1] 3
	
La búsqueda de un nombre desconocido dará lugar a una excepción:

	> match(c("Joe"),V(gi)$name)
	[1] NA 
	
### Búsqueda de vértices por nombres

Buscar vértices por su nombre es una operación muy común, y normalmente es mucho más fácil recordar los nombres de los vértices de un grafo que sus IDs. Para ello, igraph trata el atributo `name` de los vértices de forma especial; se indexan de forma que los vértices se puedan buscar por sus nombres. Por ejemplo, puedes buscar el grado (número de conexiones) de Dennis de la siguiente manera:

	> degree(gi,v="Dennis")
	Dennis 
	3 

El mapeo entre los nombres de los vértices y los IDs es mantenido de forma transparente por igraph en segundo plano; cada vez que el grafo cambia, igraph también actualiza el mapeo interno. Sin embargo, la singularidad de los nombres de los vértices *no* se impone; puedes crear fácilmente un grafo en el que dos vértices tengan el mismo nombre, pero igraph sólo devolverá uno de ellos cuando los busques por nombres, el otro sólo estará disponible por su índice.

## Tratar un grafo como una matriz de adyacencia 

La matriz de adyacencia es otra forma de formar un grafo. En la matriz de adyacencia, las filas y columnas están etiquetadas por los vértices del grafo: los elementos de la matriz indican si los vértices *i* y *j* tienen una arista común (*i, j*). La matriz de adyacencia del grafo de nuestra red social imaginaria es:

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
	
Por ejemplo, Claire (`[1, 0, 0, 1, 1, 1, 0]`) está directamente conectada con Alice (que tiene el índice 1), Dennis (índice 4), Esther (índice 5) y Frank (índice 6), pero no con Bob (índice 2) ni con George (índice 7).

## Diseños ("layouts") y graficación

Un grafo es un objeto matemático abstracto sin una representación específica en el espacio 2D o 3D. Esto significa que cuando queremos visualizar un grafo, tenemos que encontrar primero un trazado de los vértices a las coordenadas en el espacio bidimensional o tridimensional, preferiblemente de una manera que sea agradable a la vista. Una rama separada de la teoría de grafos, denominada dibujo de grafos, trata de resolver este problema mediante varios algoritmos de disposición de grafos. igraph implementa varios algoritmos de diseño y también es capaz de dibujarlos en la pantalla o en un archivo PDF, PNG o SVG utilizando la [biblioteca Cairo](https://www.cairographics.org).

### Algoritmos de diseños ("layouts")

Los métodos de diseño en igraph se encuentran en el objeto `graph`, y siempre comienzan con `layout`. La siguiente tabla los resume:

| Nombre del metodo |  Descripción |
| --- | --- |
|`layout_in_circle` `layout.circle`  | Disposición determinista que coloca los vértices en un círculo |
|`layout_with_drl` `layout.drl`  | El algoritmo [Distributed Recursive Layout] para grafos grandes |
|`layout.fruchterman.reingold`| El algoritmo dirigido Fruchterman-Reingold |
|`layout_with_kk` `layout.kamada.kawai` | El algoritmo dirigido Kamada-Kawai |
|`layout_with_lgl` `layout.lgl` | El algoritmo [Large Graph Layout] para grafos grandes | 
|`layout.random`| Coloca los vértices de forma totalmente aleatoria |
|`layout_randomly`| Coloca los vértices de forma totalmente aleatoria en 3D |
|`layout.reingold.tilford`| Diseño de árbol de Reingold-Tilford, útil para grafos (casi) arbóreos |
|`layout_as_tree`| Diseño de árbol de Reingold-Tilford con una post-transformación de coordenadas polares, útil para grafos (casi) arbóreos |
|`layout_on_sphere` `layout.sphere`  | Disposición determinista que coloca los vértices de manera uniforme en la superficie de una esfera |

Los algoritmos de diseño pueden ser llamados directamente:

	> layout <- layout.kamada.kawai(gi)
	> layout <- layout_with_kk(gi)
	
Por ejemplo, las dos llamadas siguientes son completamente equivalentes:

	> layout <- layout.reingold.tilford(gi, root= 2)
	> layout <- layout_as_tree(gi, root = 2)
	
Los métodos de diseño devuelven un objeto `layout` que se comporta principalmente como una lista de listas. Cada entrada de la lista en un objeto `layout` corresponde a un vértice en el grafo original y contiene las coordenadas del vértice en el espacio 2D o 3D. Los objetos `layout` también contienen algunos métodos útiles para traducir, escalar o rotar las coordenadas en un lote. Sin embargo, la principal utilidad de los objetos `layout` es que puedes pasarlos a la función `plot` junto con el grafo para obtener un dibujo en 2D.

### Dibujar un grafo utilizando un diseño ("layout")

Por ejemplo, podemos trazar nuestra red social imaginaria con el algoritmo de distribución Kamada-Kawai de la siguiente manera:

	> layout <- layout_with_kk(gi)
	> plot(gi, layout = layout)
	
Esto debería abrir un visor de imágenes externo que muestre una representación visual de la red, algo parecido a lo que aparece en la siguiente figura (aunque la colocación exacta de los nodos puede ser diferente en su máquina, ya que la disposición no es determinista):

![alt text](https://github.com/adriandiazlab/igraph_proyect/blob/main/Images/tutorial_social_network_R_1.png?raw=true)

Nuestra red social con el algoritmo de distribución Kamada-Kawai

Hmm, esto no es demasiado bonito hasta ahora. Una adición trivial sería usar los nombres como etiquetas de los vértices y colorear los vértices según el género. Las etiquetas de los vértices se toman del atributo `label` por defecto y los colores de los vértices se determinan por el atributo `color`. Para el ejemplo, es importante tener en cuenta que los atributos que seleccionamos como condición (por ejemplo, el género) para seleccionar un color son enteros y no cadenas:

Así que podemos crear simplemente estos atributos y volver a trazar el grafo:

	> V(gi)$gender_num <- c(1,2,1,2,1,2,2)
	> colors <- c("yellow", "red")
	> V(gi)$color <- colors[V(gi)$gender_num]
	> plot(gi, layout = layout)
	
Otras formas de enfocar este ejemplo:

	> V(gi)$color <- ifelse(V(gi)$gender == "m", "red","yellow")
	> plot(gi, layout = layout)

	> plot(gi, layout= layout, vertex.color=as.factor(V(gi)$gender))
	
	> plot(gi, layout= layout, vertex.color=c( "yellow", "red")[1+(V(gi)$gender == "f")])
	
El resultado es:

![alt text](https://github.com/adriandiazlab/igraph_proyect/blob/main/Images/tutorial_social_network_R_2.png?raw=true)

Nuestra red social - con nombres como etiquetas y géneros como colores

En lugar de especificar las propiedades visuales como atributos de vértices y aristas, también puedes darlas como argumentos a `plot`:

	> plot(gi, layout =layout, vertex.size = 20, margin = 0.5, 
	vertex.color=c( "red", "yellow")[1+(V(gi)$gender == "m")],
	edge.width=c(1,3)[1+(E(gi)$is_formal == TRUE)]
	)

Este último enfoque es preferible si quiere mantener las propiedades de la representación visual de su gráfico separadas del propio gráfico. El gráfico final muestra los vínculos formales con líneas gruesas y los informales con líneas finas:

![alt text](https://github.com/adriandiazlab/igraph_proyect/blob/main/Images/tutorial_social_network_R_3.png?raw=true)

Para resumirlo todo: hay propiedades especiales de vértices y aristas que corresponden a la representación visual del grafo. Estos atributos anulan la configuración por defecto de igraph (es decir, el color, el peso, el nombre, la forma, el diseño, etc.). Las dos tablas siguientes resumen los atributos visuales más utilizados para los vértices y las aristas, respectivamente:

### Atributos de los vértices que controlan los gráficos

| Nombre del atributo | Argumento | Propósito |
|---|---|----|
| `color` |`vertex.color` | Color del vertice |
| `font` |   `vertex.label.font`   |   Familia tipográfica del vértice |
|  `label`  | `vertex.label`   |  Etiqueta del vértice. Se convertirán en caracteres. Especifique NA para omitir las etiquetas de los vértices. Las etiquetas de vértices por defecto son los ids de los vértices. |
| `label angle`  | `vertex.label.degree` | Define la posición de las etiquetas de los vértices, en relación con el centro de los mismos. Se interpreta como un ángulo en radianes, cero significa 'a la derecha', y 'pi' significa a la izquierda, arriba es -pi/2 y abajo es pi/2. El valor por defecto es -pi/4 |
|  `label color`  | `vertex.label.color`    |    Color de la etiqueta del vértice |
|  `label dist`  | `vertex.label.dist`   |   Distancia de la etiqueta del vértice desde el propio vértice, en relación con el tamaño del vértice |
|  `label size`  |  `vertex.label.size`   |  Tamaño de letra de la etiqueta de vértice |
|  `shape`  | `vertex.shape`    | La forma del vértice, actualmente “circle”, “square”, “csquare”, “rectangle”, “crectangle”, “vrectangle”, “pie” (ver vertex.shape.pie), ‘sphere’ y “none” son admitidos, y sólo por el comando plot.igraph. |
| `size` | `vertex.size`  | El tamaño del vértice, un escalar numérico o vector, en este último caso el tamaño de cada vértice puede ser diferente |

### Atributos de las aristas que controlan los gráficos

| Nombre del atributo | Argumento  | Propósito |
|-----------|-------------|------|
|  `color`   |    `edge.color`      |      Color de la arista |
|  `curved`    |     `edge.curved`     |  Un valor numérico especifica la curvatura de la arista; una curvatura cero significa aristas rectas, valores negativos significan que la arista se curva en el sentido de las agujas del reloj, valores positivos lo contrario. TRUE significa curvatura 0.5, FALSE significa curvatura cero |
| `font`  |  `edge.font`    |  Familia tipográfica del borde |
| `arrow size`  |   `edge.arrow.size`   |   Actualmente es una constante, por lo que es la misma para todas las aristas. Si se presenta un vector, sólo se utiliza el primer elemento, es decir, si se toma de un atributo de arista, sólo se utiliza el atributo de la primera arista para todas las flechas  |
|`arrow_width`    |     `edge.arrow.width`    |   El ancho de las flechas. Actualmente es una constante, por lo que es la misma para todas las aristas |
|  `width` |    `edge.width`    |   Anchura del borde en píxeles  |
|  `label`    |   `edge.label`   |  Si se especifica, añade una etiqueta al borde |

### Argumentos genéricos de `plot()`

Estos ajustes se pueden especificar como argumentos de palabra clave a la función `plot` para controlar la apariencia general del gráfico

|  Keyword argument   |       Purpose   |
|----|-----|
|  `autocurve.edges`   |  Determinación automática de la curvatura de las aristas en grafos con múltiples aristas |
|  `layout`           |    El diseño que se va a utilizar. Puede ser una instancia de `layout`, una lista de tuplas que contengan coordenadas X-Y, o el nombre de un algoritmo de diseño. El valor por defecto es `auto`, que selecciona un algoritmo de diseño automáticamente basado en el tamaño y la conectividad del grafo. |
|  `margin`           |   La cantidad de espacio vacío debajo, encima, a la izquierda y a la derecha del gráfico, es un vector numérico de longitud cuatro |

### Especificación de colores en los gráficos

igraph entiende las siguientes especificaciones de color siempre que espera un color (por ejemplo, colores de aristas, vértices o etiquetas en los respectivos atributos):

***Nombres de colores X11***

Consulta la [lista de nombres de colores X11](https://en.wikipedia.org/wiki/X11_color_names) en Wikipedia para ver la lista completa. Los nombres de los colores no distinguen entre mayúsculas y minúsculas en igraph, por lo que "DarkBLue" puede escribirse también como "darkblue".

***Especificación del color en la sintaxis CSS***

Se trata de una cadena según uno de los siguientes formatos (donde *R*, *G* y *B* denotan los componentes rojo, verde y azul, respectivamente):

-   `#RRGGBB`, los componentes van de 0 a 255 en formato hexadecimal. Ejemplo: `"#0088ff"`
-   `#RGB`, los componentes van de 0 a 15 en formato hexadecimal. Ejemplo: `"#08f"`
-   `rgb(R, G, B)`, los componentes van de 0 a 255 o de 0% a 100%. Ejemplo: `"rgb(0, 127, 255)"` o `"rgb(0%, 50%, 100%)"`.

### Guardar gráficos

igraph puede usarse para crear gráficos de calidad de publicación. El formato preferido se deduce de la extensión. igraph puede guardar en cualquier cosa que soporte Cairo, incluyendo archivos SVG, PDF y PNG. Los archivos SVG o PDF pueden ser convertidos posteriormente al formato PostScript (`.ps`) o PostScript encapsulado (`.eps`) si lo prefieres, mientras que los archivos PNG pueden ser convertidos a TIF (`.tif`):

	> png("social_network.png", 600, 600) 
	> plot(gi, layout =layout, vertex.size = 20, margin = 0.5,
     		vertex.color=c( "red", "yellow")[1+(V(gi)$gender == "m")], 
     		edge.width=c(1,3)[1+(E(gi)$is_formal == TRUE)]
		)
	> dev.off()

## igraph y el mundo exterior

Ningún módulo de grafos estaría completo sin algún tipo de funcionalidad de importación/exportación que permita al paquete comunicarse con programas y kits de herramientas externos. igraph no es una excepción: proporciona funciones para leer los formatos de grafos más comunes y para guardar objetos `Graph` en archivos que obedezcan estas especificaciones de formato. La siguiente tabla resume los formatos que igraph puede leer o escribir:

| Formato | Nombre corto | Metodo de lectura |  Metodo de escritura |
 |---|---|----|----|
 | Adjacency list (a.k.a. [LGL](https://lgl.sourceforge.net/#FileFormat))   |   `lgl`  |     `read_graph(file, format = c("lgl"))`    |     `write_graph(graph, file, format = c("lgl"))` |
 | Adjacency matrix   |     `adjacency`   |  `graph_from_adjacency_matrix(adjmatrix, mode = c("directed", "undirected", "max", "min", "upper","lower", "plus"), weighted = NULL, diag = TRUE, add.colnames = NULL, add.rownames = NA)`   |  `as.matrix(graph, "adjacency")` |
 |DIMACS   |       `dimacs`  |  `read_graph(file, format = c("dimacs"))`   |  `write_graph(graph, file, format = c("dimacs"))"` |
 | DL       |  `dl`        |                  `Graph.Read_DL`       |     not supported yet |
 | Edge list |   `edgelist` |  `read_graph(file, format = c("edgelist"))`    |  `write_graph(graph, file, format = c("edgelist"))` |
 | [GraphViz](https://www.graphviz.org)   |    `dot`      |       not supported yet     |         `write_graph(graph, file, formati = c("dot"))` |
 | GML   |   `gml`   |    `read_graph(file, format = c("gml"))`  |  `write_graph(graph, file, format = c("gml"))` |
 | GraphML  |    `graphml`   |        `read_graph(file, format = c("graphml"))`   |    `write_graph(graph, file, format = c("graphml"))` |
 | LEDA     |         `leda`       |    not supported yet    |       `write_graph(graph, file, format = c("leda"))` |
 | Labeled edgelist (a.k.a. [NCOL](https://lgl.sourceforge.net/#FileFormat))  | `ncol`     |                   `read_graph(file, format = c("ncol"))`     |    `write_graph(graph, file, format = c("ncol"))` |
 | [Pajek](http://mrvar.fdv.uni-lj.si/pajek/) format   |     `pajek`   | `read_graph(file, format = c("pajek"))`   |   `write_graph(graph, file, format = c("pajek"))` |


Nota

La mayoría de los formatos tienen sus propias limitaciones; por ejemplo, no todos pueden almacenar atributos. Tu mejor opción es probablemente GraphML o GML si quieres guardar los grafos de igraph en un formato que pueda ser leído desde un paquete externo y quieres preservar los atributos numéricos y de cadena. La lista de aristas y NCOL también están bien si no tienes atributos (aunque NCOL soporta nombres de vértices y pesos de aristas).

## Dónde ir a continuación

Este tutorial sólo ha arañado la superficie de lo que igraph puede hacer. Los planes a largo plazo son ampliar este tutorial para convertirlo en una documentación adecuada de estilo manual para igraph en los próximos capítulos. Un buen punto de partida es la documentación de la clase `Graph`. Si te quedas atascado, intenta preguntar primero en nuestro [grupo Discourse](https://igraph.discourse.group) - quizás haya alguien que pueda ayudarte inmediatamente.





