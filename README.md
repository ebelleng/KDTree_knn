# Estructuras espaciales

Si hicieramos un pequeño recorrido de historia, debemos mencionar el origen de lo que hoy se conoce como el oro negro del siglo XXI: _los datos_.
Para entender como esta pequeña unidad de información se ha convertido en lo más cotizado en estos días, debemos conocer su origen y su creciente evolución.

## El internet de las cosas

Este termino aparece por primera vez en 1999, cuando un profesor del MIT usó la expresión de forma publica por primera vez, desde entonces ha tenido un crecimiento de forma exponencial. Pero, ¿a qué se refiere? Básicamente es conectar todos los objetos con los cuales interactuamos y que existen en nuestro planeta. Es decir, que tengan una dirección IP  (Protocolo  de  Internet)  para  que  puedan  generar  información  y  transferir  datos  mediante la red, sin la intervención de los seres humanos o de la interacción personas-computadoras.

Una de las consecuencias de esta nueva tendencia es la generación de grandes volumenes de datos. El 90% de los datos en servidores de todo el mundo se han creado en los últimos años y por eso aparece con tanta fuerza el termino _Big Data_.

## Big data

Cuando hablamos de Big Data nos referimos a conjuntos de datos o combinaciones de conjuntos de datos cuyo tamaño (volumen), complejidad (variabilidad) y velocidad de crecimiento (velocidad) dificultan su captura, gestión, procesamiento o análisis mediante tecnologías y herramientas convencionales, tales como bases de datos relacionales y estadísticas convencionales o paquetes de visualización, dentro del tiempo necesario para que sean útiles.

## KD Tree

Se ha decidido implementar la estructura espacial KD Tree con el nodo raíz y la dimensión del vector que se almacenará en el nodo.

    class KD_Tree:
    def __init__(self, dimensions):
      self.root = None
      self.D = dimensions
      
#### Nodo
 
 Para la estructura del nodo se ha implementado la siguiente estructura con el fin de almacenar el vector (point) y el id del registro.
 
    class KD_Node:
      def __init__(self, id_obj, point, cutting_dimension=0):
        self.left = None
        self.right = None
        self.parent = None
        self.point = point
        self.cd = cutting_dimension
        self.id = id_obj
        
#### Métodos

Para insertar los nodos en el arbol, se implementó la siguiente función, la cual busca la posición segun las coordenadas (point) del vector y lo insserta donde corresponde.
    
    def insert(self, point, id_obj=None):

Para buscar los vecinos más cercanos se utiliza esta función que recibe un vector y genera los _k_ vecinos más prometedores (que recibe como parametro) según la distancia entre ellos.

    def k_nearest_neighbors(self, point, k=1):

Para buscar un nodo en el arbol se utiliza esta función, la cual recorre el arbol hasta encontrar el vector ingresado por parametro.

    def search(self, point):
    
Para obtener la distancia entre dos puntos se compara posición por posición de las coordenadas del vector mediante el método de distancia euclideana.

    def get_distance(self, point_one, point_two):
    
### Ejemplo

Se muestra el siguiente ejemplo para probar la estructura.

  
    from kd_tree import KD_Tree

    kdt = KD_Tree(2)

    nodes_to_insert = [(30,40), (5,25), (10,12), (70,70), (50,30), (35,45)]
    #nodes_to_insert = [(1,1), (-2,1), (3,-1), (1,4), (-5,1), (-1,6)]

    for i in range(len(nodes_to_insert)):
      kdt.insert(nodes_to_insert[i], i)

    # Print the resultant tree (level by level)
    #kdt.showTree()

    node_to_search = (70, 70)
    print(kdt.search(node_to_search))

    point = (0, 0)
    knn = kdt.k_nearest_neighbors(point, 4)

    for i in range(len(knn)):
    print(knn[i].point, knn[i].id)
 
 Obteniendo como resultado los 4 vecinos más cercanos al punto (0,0):
 
    True
    (10, 12) 2
    (5, 25) 1 
    (30, 40) 0
    (35, 45) 5
    
## Set de datos

Una vez implementado nuestra estructura espacial, se trabajó en el formato de los vectores extrayendo información desde un [csv](/src/dataset/movies.csv).

### Elección del set de datos

Para este desafío se encontró un dataset sobre peliculas con el siguiente formato:

    >> print(df_movies.columns)
    
    Index(['Rank', 'Title', 'Genre', 'Description', 'Director', 'Actors', 'Year', 
    'Runtime (Minutes)', 'Rating', 'Votes', 'Revenue (Millions)',
    'Metascore'], dtype='object')
    
A continuación se detalla los campos que llamaron más la atención por su pontencial para vectorizar.

| Campo          | Descripción                                                           |
|----------------|-----------------------------------------------------------------------|
| **'Rank'**     | Corresponde al id de cada pelicula                                    |
| **'Title'**    | Corresponde al nombre o titulo de la pelicula                         |
| **'Genre'**    | Es una lista de los generos que categorizan al video separado por ',' |
| **'Director'** | Corresponde al director del film                                      |
| **'Actors'**   | Es una lista de los actores que conforman el reparto                  |
| **'Votes'**    | Corresponde a los votos para calcular el rating                       |
| **'Rating'**   | Corresponde al promedio del puntaje dado por el publico               |

### ETL: Extraer, Transformar y Cargar

De los atributos seleccionados anteriormente se tomaron las siguientes decisiones:

* El 'Rank' se utilizará como identificador de la pelicula, se insertará en la estructura nodo. 
* El 'Title' se descartó ya que no se considera util a la hora de generar una distancia espacial.
* El 'Genre' se puede incluir en el vector ya que los generos son finitos y agrupables entre ellos.
* El 'Director' se consideró para incluir en el vector ya que existe una preferencia por peliculas dirigidas por la misma persona.
* El 'Actors' se trabajará de igual manera que 'Genre' ya que existen preferencias por peliculas de cierto actor
* El 'Votes' se descartará para la generación del vector ya que no tiene sentido una distancia espacial entre estos elementos.
* El 'Rating' se agregó al vector ya que una pelicula bien evaluada, generará en los usuarios buscar peliculas así. También se puede ver que si una pelicula tiene bajo rating, significa que está más cerca en distancia que una de alto rating, por lo tanto puede considerarse perfectamente como un atributo a incluir.

La transformación de las listas de elementos como generos y actores se realizó en el siguiente [archivo](/src/dataset/ETL.ipynb). Se utilizó un jypiter notebook para mostrar de forma grafica la transformación del dataframe y el formato de salida. 

### Funciones para trabajar vector

