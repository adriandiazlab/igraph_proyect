.. include:: include/global.rst

.. _tutorial:

.. currentmodule:: igraph

========
Tutorial
========

Esta página es un tutorial detallado de las capacidades de |igraph| para Python. Para obtener una impresión rápida de lo que |igraph| puede hacer, consulte el :doc:`tutorials/quickstart`. Si aún no ha instalado |igraph|, siga las instrucciones de :doc:`install`.

.. nota::
   Para el lector impaciente, vea la página :doc:`tutorials/index` para ejemplos cortos y autocontenidos.
   
Comenzar con |igraph|
=================

La manera más común de usar |igraph| es como una importanción con nombre dentro de un ambiente de Python (por ejemplo, un simple shell de Python, a `IPython`_ shell , un `Jupyter`_ notebook o una instancia JupyterLab, `Google Colab <https://colab.research.google.com/>`_, o un `IDE <https://www.spyder-ide.org/>`_)::

  $ python
  Python 3.9.6 (default, Jun 29 2021, 05:25:02)
  [Clang 12.0.5 (clang-1205.0.22.9)] on darwin
  Type "help", "copyright", "credits" or "license" for more information.
  >>> import igraph as ig
  
Para llamar a funciones, es necesario anteponerles el prefijo ``ig`` (o el nombre que hayas elegido)::

  >>> import igraph as ig
  >>> print(ig.__version__)
  0.9.8

.. nota::
   Es posible utilizar *importación con asterisco* para |igraph|::

    >>> from igraph import *
    
Pero en general se desaconseja <https://stackoverflow.com/questions/2386714/why-is-import-bad>`_.

Hay una segunda forma de iniciar |igraph|, que consiste en llamar al script :command:`igraph` desde tu terminal::

  $ igraph
  No configuration file, using defaults
  igraph 0.9.6 running inside Python 3.9.6 (default, Jun 29 2021, 05:25:02)
  Type "copyright", "credits" or "license" for more information.
  >>>

.. nota::
   Para los usuarios de Windows encontrarán el script dentro del subdirectorio file:`scripts` de Python 
   y puede que tengan que añadirlo manualmente a su ruta.
   
Este script inicia un intérprete de comandos apropiado (`IPython`_ o `IDLE <https://docs.python.org/3/library/idle.html>`_ si se encuentra, de lo contrario un intérprete de comandos Python puro) y utiliza *importación con asterisco* (véase más arriba). Esto es a veces conveniente para usar las funciones de |igraph|.

.. nota::
   Puede especificar qué shell debe utilizar este script a través 
   :doc:`configuration` de |igraph|.

Este tutorial asumirá que has importado igraph usando el de nombres ``ig``.

Creando un grafo
================

La forma más sencilla de crear un grafo es con el constructor :class:`Graph`. Para hacer un grafo vacío:

  >>> g = ig.Graph()
  
Para hacer un grafo con 10 nodos (numerados ``0`` to ``9``) y dos aristas que conecten los nodos ``0-1`` y `0-5``::

  >>> g = ig.Graph(n=10, edges=[[0, 1], [0, 5]])
  
Podemos imprimir el grafo para obtener un resumen de sus nodos y aristas:

  >>> print(g)
  IGRAPH U--- 10 2 --
  + edges:
  0--1 0--5
  
Tenemos entonces: grafo no dirigido (**U**ndirected)  con **10** vértices y **2** aristas, que se enlistan en la última parte. Si el grafo tiene un atributo "nombre", también se imprime.

.. nota::
   ``summary`` es similar a ``print`` pero no enlista las aristas, lo cual
   es conveniente para grafos grandes con millones de aristas:
   
     >>> summary(g)
     IGRAPH U--- 10 2 --
   
Añadir y borrar vértices y aristas
==================================

Empecemos de nuevo con un grafo vacío. Para añadir vértices a un grafo existente, utiliza :meth:`Graph.add_vertices`::

  >>> g = ig.Graph()
  >>> g.add_vertices(3)
  
En |igraph|, los vértices se numeran siempre a partir de cero El número de un vértice es el *ID del vértice*. Un vértice puede tener o no un nombre.

Del mismo modo, para añadir aristas se utiliza :meth:`Graph.add_edges`::

  >>> g.add_edges([(0, 1), (1, 2)])
  
Las aristas se añaden especificando el vértice origen y el vértice destino de cada arista. Esta llamada añade dos aristas, una que conecta los vértices ``0`` y ``1``, y otra que conecta los vértices ``1`` y ``2``. Las aristas también se numeran a partir de cero (el *ID del arista*) y tienen un nombre opcional.

.. Advertencia::

Crear un grafo vacío y añadir vértices y aristas como se muestra aquí puede ser mucho más lento que crear un grafo con sus vértices y aristas como se ha demostrado anteriormente. Si la velocidad es una preocupación, deberías evitar especialmente añadir vértices y aristas *de uno en uno*. Si necesitas hacerlo de todos modos, puedes usar :meth:`Graph.add_vertex` y :meth:`Graph.add_edge`.

Si intentas añadir aristas a vértices con IDs no válidos (por ejemplo, intentas añadir una arista al vértice `5` cuando el grafo sólo tiene tres vértices), obtienes un error:

  >>> g.add_edges([(5, 4)])
  Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "/usr/lib/python3.10/site-packages/igraph/__init__.py", line 376, in add_edges
      res = GraphBase.add_edges(self, es)
  igraph._igraph.InternalError: Error at src/graph/type_indexededgelist.c:270: cannot add edges. -- Invalid vertex id
  
El mensaje intenta explicar qué ha fallado (``cannot add edges. -- Invalid
vertex id``) junto con la línea correspondiente del código fuente en la que se ha producido el error.

.. nota::
   El rastreo completo, incluida la información sobre el código fuente, es útil cuando
   se informa de errores en nuestro
   `Página de problemas de GitHub <https://github.com/igraph/python-igraph/issues>`_. Por favor, inclúyalo
   completo si crea un nuevo asunto.

Añadamos más vértices y aristas a nuestro grafo:

  >>> g.add_edges([(2, 0)])
  >>> g.add_vertices(3)
  >>> g.add_edges([(2, 3), (3, 4), (4, 5), (5, 3)])
  >>> print(g)
  IGRAPH U---- 6 7 --
  + edges:
  0--1 1--2 0--2 2--3 3--4 4--5 3--5
  
Ahora tenemos un grafo no dirigido con 6 vértices y 7 aristas. Los IDs de los vértices y aristas son siempre *continuos*, por lo que si eliminas un vértice todos los vértices subsiguientes serán renumerados. Cuando se renumera un vértice, las aristas **no** se renumeran, pero sí sus vértices de origen y destino. Utilice :meth:`Graph.delete_vertices` y :meth:`Graph.delete_edges` para realizar estas operaciones. Por ejemplo, para eliminar la arista que conecta los vértices ``2-3``, obten sus IDs y luego eliminalos:

  >>> g.get_eid(2, 3)
  3
  >>> g.delete_edges(3)
  
Generar grafos
=================

|igraph| incluye generadores de grafos tanto deterministas como estocásticos. Los generadores *deterministas* producen el mismo grafo cada vez que se llama a la función, por ejemplo:

  >>> g = ig.Graph.Tree(127, 2)
  >>> summary(g)
  IGRAPH U--- 127 126 --
  
Utiliza :meth:`Graph.Tree` para generar un grafo regular en forma de árbol con 127 vértices, cada vértice con dos hijos (y un padre, por supuesto). No importa cuántas veces llames a :meth:`Graph.Tree`, el grafo generado será siempre el mismo si utilizas los mismos parámetros:

  >>> g2 = ig.Graph.Tree(127, 2)
  >>> g2.get_edgelist() == g.get_edgelist()
  True
  
El fragmento de código anterior también muestra el método :meth:`~Graph.get_edgelist()`, que devuelve una lista de vértices de origen y destino para todas las aristas, ordenados por el ID de la arista. Si imprimes los 10 primeros elementos, obtienes:

  >>> g2.get_edgelist()[:10]
  [(0, 1), (0, 2), (1, 3), (1, 4), (2, 5), (2, 6), (3, 7), (3, 8), (4, 9), (4, 10)]
  
Los generadores *estocásticos* producen un grafo diferente cada vez; por ejemplo, :meth:`Graph.GRG`::

  >>> g = ig.Graph.GRG(100, 0.2)
  >>> summary(g)
  IGRAPH U---- 100 516 --
  + attr: x (v), y (v)
  
.. nota::
   `+ attr`` muestra atributos para vértices (v) y aristas (e), en este caso dos atributos de   
   vértice y ningún atributo de arista.

Esto genera un grafo geométrico aleatorio: Se eligen *n* puntos de forma aleatoria y uniforme dentro del cuadrado unitario y los pares de puntos más cercanos entre sí respecto a una distancia predefinida *d* se conectan mediante una arista. Si se generan GRGs con los mismos parámetros, serán diferentes:

  >>> g2 = ig.Graph.GRG(100, 0.2)
  >>> g.get_edgelist() == g2.get_edgelist()
  False
  
Una forma un poco más relajada de comprobar si los grafos son equivalentes es mediante :meth:`~Graph.isomorphic()`::

  >>> g.isomorphic(g2)
  False
  
Comprobar por el isomorfismo puede llevar un tiempo en el caso de grafos grandes (en este caso, la respuesta puede darse rápidamente comprobando las distribuciones de grados de los dos grafos).

Establecer y recuperar atributos
=================================

Como se ha mencionado anteriormente, en |igraph| cada vértice y cada arista tienen un ID numérico de `0` en adelante. Por lo tanto, la eliminación de vértices o aristas puede causar la reasignación de los ID de vértices y/o aristas. Además de los IDs, los vértices y aristas pueden tener *atributos* como un nombre, coordenadas para graficar, metadatos y pesos. El propio grafo puede tener estos atributos también (por ejemplo, un nombre, que se mostrará en ``print`` o ``summary``). En cierto sentido, cada :class:`Graph`, vértice y arista pueden utilizarse como un diccionario de Python para almacenar y recuperar estos atributos.

Para demostrar el uso de los atributos, creemos una red social sencilla:

  >>> g = ig.Graph([(0,1), (0,2), (2,3), (3,4), (4,2), (2,5), (5,0), (6,3), (5,6)])
  
Cada vértice representa una persona, por lo que queremos almacenar nombres, edades y géneros:

  >>> g.vs["name"] = ["Alice", "Bob", "Claire", "Dennis", "Esther", "Frank", "George"]
  >>> g.vs["age"] = [25, 31, 18, 47, 22, 23, 50]
  >>> g.vs["gender"] = ["f", "m", "f", "m", "f", "m", "m"]
  >>> g.es["is_formal"] = [False, False, True, True, True, False, True, False, False]
  
:attr:`Graph.vs` y :attr:`Graph.es` son la forma estándar de obtener una secuencia de todos los vértices y aristas respectivamente. El valor debe ser una lista con la misma longitud que los vértices (para :attr:`Graph.vs`) o aristas (para :attr:`Graph.es`). Esto asigna un atributo a *todos* los vértices/aristas a la vez.

Para asignar o modificar un atributo para un solo vértice/borde, puedes hacer lo siguiente:

 >>> g.es[0]["is_formal"] = True

De hecho, un solo vértice se representa mediante la clase :class:`Vertex`, y una sola arista mediante :class:`Edge`. Ambos, junto con :class:`Graph`, pueden ser tecleados como un diccionario para establecer atributos, por ejemplo, para añadir una fecha al grafo:

  >>> g["date"] = "2009-01-10"
  >>> print(g["date"])
  2009-01-10

Para recuperar un diccionario de atributos, puedes utilizar :meth:`Graph.attributes`, :meth:`Vertex.attributes` y :meth:`Edge.attributes`.

Además, cada :class:`Vertex` tiene una propiedad especial, :attr:`Vertex.index`, que se utiliza para averiguar el ID de un vértice. Cada :class:`Edge` tiene :attr:`Edge.index` más dos propiedades adicionales, :attr:`Edge.source` y :attr:`Edge.target`, que se utilizan para encontrar los IDs de los vértices conectados por esta arista. Para obtener ambas propiedades a la vez, puedes utilizar :attr:`Edge.tuple`.

Para asignar atributos a un subconjunto de vértices o aristas, puedes utilizar el corte:

  >>> g.es[:1]["is_formal"] = True
  
La salida de ``g.es[:1]`` es una instancia de :class:`~seq.EdgeSeq`, mientras que :class:`~seq.VertexSeq` es la clase equivalente que representa subconjuntos de vértices.

Para eliminar atributos:

  >>> g.vs[3]["foo"] = "bar"
  >>> g.vs["foo"]
  [None, None, None, 'bar', None, None, None]
  >>> del g.vs["foo"]
  >>> g.vs["foo"]
  Traceback (most recent call last):
    File "<stdin>", line 25, in <module>
  KeyError: 'Attribute does not exist'
  
.. Advertencia::
   Los atributos pueden ser objetos arbitrarios de Python, pero si está guardando grafos en un 
   archivo, sólo se conservarán los atributos de cadena ("string") y numéricos. Consulte el 
   módulo :mod:`pickle` de la biblioteca estándar de Python si busca una forma de guardar otros 
   tipos de atributos. Puede hacer un pickle de sus atributos individualmente, almacenarlos como 
   cadenas y guardarlos, o puedes hacer un pickle de todo el :class:`Graph` si sabes que quieres 
   cargar el grafo en Python.
   
   
   
   
Propiedades estructurales de los grafos
===============================

Además de las funciones simples de manipulación de grafos y atributos descritas anteriormente, |igraph| proporciona un amplio conjunto de métodos para calcular varias propiedades estructurales de los grafos. Está más allá del alcance de este tutorial documentar todos ellos, por lo que esta sección sólo presentará algunos de ellos con fines ilustrativos. Trabajaremos con la pequeña red social que construimos en la sección anterior.

Probablemente, la propiedad más sencilla en la que se puede pensar es el "grado del vértice". El grado de un vértice es igual al número de aristas incidentes a él. En el caso de los grafos dirigidos, también podemos definir el ``grado de entrada`` (el número de aristas que apuntan hacia el vértice) y el ``grado de salida`` (el número de aristas que se originan en el vértice). 

  >>> g.degree()
  [3, 1, 4, 3, 2, 3, 2]
  
Si el grafo fuera dirigido, habríamos podido calcular los grados de entrada y salida por separado utilizando ``g.degree(mode="in")`` y ``g.degree(mode="out")``. También puedes usar un único ID de un vértice o una lista de ID de los vértices a :meth:`~Graph.degree` si quieres calcular los grados sólo para un subconjunto de vértices:

  >>> g.degree(6)
  2
  >>> g.degree([2,3,4])
  [4, 3, 2]
  
Este procedimiento se aplica a la mayoría de las propiedades estructurales que |igraph| puede calcular. Para las propiedades de los vértices, los métodos aceptan un ID o una lista de IDs de los vértices (y si se omiten, el valor predeterminado es el conjunto de todos los vértices). Para las propiedades de las aristas, los métodos también aceptan un único ID de o una lista de IDs de aristas. En lugar de una lista de IDs, también puedes proporcionar una instancia :class:`VertexSeq` o una instancia :class:`EdgeSeq` apropiadamente. Más adelante, en el próximo capítulo "consulta de vértices y aristas", aprenderás a restringirlos exactamente a los vértices o aristas que quieras.

.. nota::

   Para algunos casos, no tiene sentido realizar el calculo sólo para unos pocos vértices o 
   aristas en lugar de todo el grafo, ya que de todas formas se tardaría el mismo tiempo. En 
   este caso, los métodos no aceptan IDs de vértices o aristas, pero se puede restringir la 
   lista resultante más tarde usando operadores estándar de indexación y de corte. Un ejemplo de 
   ello es la centralidad de los vectores propios (:meth:`Graph.evcent()`)

Además de los grados, |igraph| incluye rutinas integradas para calcular muchas otras propiedades de centralidad, como la intermediación de vértices y aristas o el PageRank de Google (:meth:`Graph.pagerank`), por nombrar algunas. Aquí sólo ilustramos la interrelación de aristas:

  >>> g.edge_betweenness()
  [6.0, 6.0, 4.0, 2.0, 4.0, 3.0, 4.0, 3.0. 4.0]

Ahora también podemos averiguar qué conexiones tienen la mayor centralidad de intermediación
con un poco de magia de Python:

  >>> ebs = g.edge_betweenness()
  >>> max_eb = max(ebs)
  >>> [g.es[idx].tuple for idx, eb in enumerate(ebs) if eb == max_eb]
  [(0, 1), (0, 2)]
  
La mayoría de las propiedades estructurales también pueden ser obtenidas para un subconjunto de vértices o aristas o para un solo vértice o arista llamando al método apropiado de la clase :class:`VertexSeq` o :class:`EdgeSeq` de interés:

  >>> g.vs.degree()
  [3, 1, 4, 3, 2, 3, 2]
  >>> g.es.edge_betweenness()
  [6.0, 6.0, 4.0, 2.0, 4.0, 3.0, 4.0, 3.0. 4.0]
  >>> g.vs[2].degree()
  4
  
Busqueda de vértices y aristas basada en atributos
===============================================

Selecting vertices and edges
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Tomando como ejemplo la red social anterirormente creada, te gustaría averiguar quién tiene el mayor grado o centralidad de intermediación. Puedes hacerlo con las herramientas presentadas hasta ahora y conocimientos básicos de Python, pero como es una tarea común seleccionar vértices y aristas basándose en atributos o propiedades estructurales, |igraph| te ofrece una forma más fácil de hacerlo:

  >>> g.vs.select(_degree=g.maxdegree())["name"]
  ['Claire']

La sintaxis puede parecer un poco rara a primera vista, así que vamos a tratar de interpretarla paso a paso. meth:`~VertexSeq.select` es un método de :class:`VertexSeq` y su único propósito es filtrar un :class:`VertexSeq` basándose en las propiedades de los vértices individuales. La forma en que filtra los vértices depende de sus argumentos posicionales y de palabras clave. Los argumentos posicionales (los que no tienen un nombre explícito como ``_degree`` siempre se procesan antes que los argumentos de palabra clave de la siguiente manera:

- Si el primer argumento posicional es ``None``, se devuelve una secuencia vacía (que no contiene vértices):

    >>> seq = g.vs.select(None)
    >>> len(seq)
    0

- Si el primer argumento posicional es un objeto invocable (es decir, una función, un método vinculado o cualquier cosa que se comporte como una función), el objeto será llamado para cada vértice que esté actualmente en la secuencia. Si la función devuelve ``True``, el vértice será incluido, en caso contrario será excluido:

    >>> graph = ig.Graph.Full(10)
    >>> only_odd_vertices = graph.vs.select(lambda vertex: vertex.index % 2 == 1)
    >>> len(only_odd_vertices)
    5

- Si el primer argumento posicional es un iterable (es decir, una lista, un generador o cualquier cosa sobre la que se pueda iterar), *debe* devolver enteros y estos enteros se considerarán como índices del conjunto de vértices actual (que *no* es necesariamente todo el grafo). Sólo se incluirán en el conjunto de vértices filtrados los vértices que coincidan con los índices dados. Los numero flotantes, las cadenas y los ID de vértices no válidos seran omitidos:

    >>> seq = graph.vs.select([2, 3, 7])
    >>> len(seq)
    3
    >>> [v.index for v in seq]
    [2, 3, 7]
    >>> seq = seq.select([0, 2])         # filtering an existing vertex set
    >>> [v.index for v in seq]
    [2, 7]
    >>> seq = graph.vs.select([2, 3, 7, "foo", 3.5])
    >>> len(seq)
    3

- Si el primer argumento posicional es un número entero, se espera que todos los demás argumentos sean también números enteros y se interpretan como índices del conjunto de vértices actual. Esto solo es "azucar sintáctica", se podría conseguir un efecto equivalente pasando una lista como primer argumento posicional, de esta forma se pueden omitir los corchetes:

    >>> seq = graph.vs.select(2, 3, 7)
    >>> len(seq)
    3
    
Los argumentos clave ("keyword argument") pueden utilizarse para filtrar los vértices en función de sus atributos o sus propiedades estructurales. El nombre de cada argumento clave consiste como máximo de dos partes: el nombre del atributo o propiedad estructural y el operador de filtrado. El operador puede omitirse; en ese caso, automáticamente se asume el operador de igualdad. Las posibilidades son las siguientes (donde *name* indica el nombre del atributo o propiedad):

================ ================================================================
Keyword argument Significado
================ ================================================================
``name_eq``      El valor del atributo/propiedad debe ser *igual* a 
---------------- ----------------------------------------------------------------
``name_ne``      El valor del atributo/propiedad debe *no ser igual* a
---------------- ----------------------------------------------------------------
``name_lt``      El valor del atributo/propiedad debe ser *menos* que
---------------- ----------------------------------------------------------------
``name_le``      El valor del atributo/propiedad debe ser *inferior o igual a*
---------------- ----------------------------------------------------------------
``name_gt``      El valor del atributo/propiedad debe ser *mayor que*
---------------- ----------------------------------------------------------------
``name_ge``      El valor del atributo/propiedad debe ser *mayor o igual a* 
---------------- ----------------------------------------------------------------
``name_in``      El valor del atributo/propiedad debe estar *incluido en*, el cual tiene que ser
                 una secuencia en este caso
---------------- ----------------------------------------------------------------
``name_notin``   El valor del atributo/propiedad debe *no estar incluido en* ,
                 el cual tiene que ser una secuencia en este caso
================ ================================================================

Por ejemplo, el siguiente comando te da las personas menores de 30 años en nuestra red social imaginaria:

  >>> g.vs.select(age_lt=30)

.. nota::
   Debido a las restricciones sintácticas de Python, no se puede utilizar la sintaxis más 
   sencilla de ``g.vs.select(edad < 30)``, ya que en Python sólo se permite que aparezca el 
   operador de igualdad en una lista de argumentos.
   
Para ahorrarte algo de tecleo, puedes incluso omitir el método :meth:`~VertexSeq.select` si
desea::

  >>> g.vs(age_lt=30)
