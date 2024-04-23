
# Elastic

Es una empresa de software que fabrica un huevo de productos:
- ElasticSearch     \
- Kibana             \
- Logstash           /  Stack ELK
- Familia beats     /

Todos esos productos tienen propósitos diferentes:

- ElasticSearch:    Motor de indexación y búsqueda de contenidos... algo así como un google.
                    NO ES UNA BBDD al uso... Aunque la vamos a usar como tal... en nuestro escenario concreto de uso: OBSERVABILITY
- Kibana:           App web que nos permite hacer operaciones sobre un ElasticSearch:
                       - Mnto / Administración del ElasticSearch
                       - Consultar datos de ese ElasticSearch
                       - ... 
- Logstash          - Enrutador de mensajes hacia un determinado ElasticSearch
                    - Transformar mensajes antes o después del enrutado      
                    Un mensaje puede ser el texto de un documento PDF que quiero indexar con ElasticSearch.

Originalmente el uso planteado para ElasticSearch era el posibilitar a una empresa montarse algo así como su propio google... interno...
La realidad es que hoy en día en 95% de las empresas que usan ElasticSearch ... y las herramientas colindantes, lo hacen para observability.

- Familia beats:    Son programas muy ligeros que permiten recopilar METRICAS / EVENTOS... y mandarlos a: LOGSTASH o ES (esto último no lo haremos NUNCA JAMAS)
    - MetricBeat
    - HeartBeat
    - FileBeat
    - ...

## ElasticSearch

Es un indexador / Motor de búsquedas.

Un indexador es una herramienta que genera INDICES de un contenido, para facilitar posteriores búsquedas sobre ese contenido.
Un indexador guarda el contenido? En principio no... aunque es algo que puedo solicitar:
    Que se guarde una copia del contenido que se solicitó indexar.
    Es más... en ES ésto ocurre por defecto... y de hecho es la clave para su uso en OBSERVABILITY.

### Índice

Una estructura para almacenar información.
Un índice es una copia ORDENADA de unos datos... junto con su ubicación original.
En un índice acabamos con un dato + un puntero a la ubicación de ese dato. Lo que permite una rápida localización de ese datos.

Cuando hablamos de índice, nos referimos por ejemplo al INDICE DE UN LIBRO.

> Libro de recetas:
    Pag 17: tortilla de patatas
    Pag 53: bacalao encebollao
    Pag 103: corderito al horno con papas panaderas
    
    > Quiero saber las recetas que usan patatas? Tengo que ir página a página
    > Quiero saber las recetas de primero platos?
    > Quiero saber las recetas que sonb "fáciles" de preparar?
    
    Evidentemente el ir página a página (FULLSCAN) es muy lento, comparado con otras opciones.
    Para acelerarlo, puedo empezar a crear índices.
    
    INDICE DE INGREDIENTES: pag 307 (INDICE)
        copia del dato          ubicación
        patata                  17, 103
    INDICE POR TIPO DE PLATO
    INDICE POR DIFICULTAD
    
    Crear índices requiere de espacio adicional.
    
#### Tipos de índices?

- Índices directos
- Índices invertidos

Quienes son desde hace más de 50 años, los mayores expertos en generación de índices? BBDD... en concreto las BBDD Relacionales.

Una BBDD relacional, guarda sus datos en una tabla.

    RECETAS
    id  | nombre                | tipo_plato    |   ingrediente pricipal | ...
    1   | tortilla de patatas   | entrante      |   patatas              | ...
    2   | bacalao encebollao    | segundo       |   bacalao              | ...

Si una BBDD necesita buscar información en una tabla, la operación más básica que puede realizar es un FULL SCAN

En que me favorece el tener un índice a la hora de facilitar (hacer más eficiente) la localización de un dato?
Porque puedo cambiar el algoritmo de búsqueda, pasando de un FULLSCAN a una BUSQUEDA BINARIA

#### BUSQUEDA BINARIA

Desde que tenemos 7/8 años... de continuo... cada vez que hemos usado un DICCIONARIO.
En un diccionario puedo hacer búsquedas muy rápido... No necesito h¡aplciar un algoritmo de tipo FULLSCAN para encontrar un dato.
Puedo aplicar algoritmos de tipo BUSQUEDA BINARIA.
- Parto en diccionario en 2...
- Miro la palabra que tengo delante
- Si es la mia: GGUAY, ya la tengo
- Si es anterior (según orden de caracteres): busco avellana y leo manzana:
    Descarto toda la segunda mitad del diccionario
- Si es posterior, descarto la primera mitad del diccionario.
    DICHO DE OTRA FORMA: con 1 única operación he conseguuido descartar la mitad de los elementos de la base de búsqueda

1.000.000 elementos
  500.000
  250.000
  125.000
   62.500
   32.000
   16.000
    8.000
    4.000
    2.000               En 20 operaciones o menos, identifico un elemento entre 1 Millón
    1.000
      500
      250
      125
       63
       32
       16
        8
        4
        2
        1

BRUTAL !!!!
Sobre esto, se aplican optimizaciones adicionales. LLas BBDD guardan "ESTADISTICAS"... y hay que regenerarlas de vez en cuando.

Si me piden la palaba zamburrio, parto el diccionario a la mitad? NI DE COÑA ! me voy al final
Ya que conozco la DISTRIBUCION (ESTADISTICA) De los datos en el diccionario.

Si conozco la distribución de los datos, puedo optimizar el primer particionado de los datos.

Los algoritmos de búsqueda binaria son muy superiores en rendimiento a los algoritmos de tipo FULLSCAN.
Ahora... tienen una cosita... para poder aplciar un algoritmo de búsqueda binaria, necesito que los datos estén ORDENADOS
Si los datos están sin ordenar, NI DE COÑA APLICO UNA BUSQUEDA BINARIA... 
    podría caer en la tentación de ordenar primero los datos y luego aplicar un algoritmo de búsqueda binaria.
    
    Qué tal se le da a los ordenadores, ordenar datos? COMO EL PUTO CULO !!!
    Son de las peores cosas que puedo pedir a un ORDENADOR.
        Al fin y al cabo los ORDENADORES no se llaman ORDENADORES por su capacidad para ORDENAR DATOS.
        Sino por su capacidad de procesar ORDENES.
        
El problema es que cualquier algoritmo de ordenación tarda más que un fullscan.... MUCHO MAS!!!

Cuando trabajo con ÍNDICES, lo que hago es cada vez que se inserta / modifica un dato en la colección, actualizo o añado ese dato al INDICE

Las BBDD Relacionales llevan años creando índices... y hay pocas herramientas tan avanzadas para crear índices como los son las BBDD.

Y entonces, por qué un indexador?
    Porque las BBDD Relacionales son expertas en crear y gestionar (y usar) ÍNDICES DIRECTOS
    Pero no tienen npi de INDICES INVERTIDOS (o poca idea)
    Y en muchos escenarios lo que necesitamos son INDICES INVERTIDOS, que es la gracia de una herramienta como ELASTIC SEARCH.

Sabe ES gestionar índices invertidos? NPI tiene de INDICES INVERTIDOS el ES.
Eso lo delega en otra herrameinta, herramienta Opensource y gratuita, llamada LUCENE (proyecto de la fundación APACHE)
De hecho, podríamos definir a ES como un orquestador de LUCENES... Lucenes que en el mundo ES se denominan: SHARD

    
    TABLA: RECETAS
    
    | id   | nombre                         | tipo_plato     | ingrediente_principal | dificultad | tiempo_preparacion | precio |
    |------|--------------------------------|----------------|-----------------------|------------|--------------------|--------|
    | 1    | Ensalada de pasta              | Entrante       | Pasta                 | Baja       | 20 minutos         | 5€     |
    | 2    | Pasta a la carbonara           | Principal      | Pasta                 | Media      | 30 minutos         | 10€    |
    | 3    | Pasta a la bolonesa            | Principal      | Pasta                 | Media      | 30 minutos         | 10€    |
    | 4    | Tortilla de patata             | Principal      | Huevo                 | Baja       | 20 minutos         | 5€     |
    | 5    | Cordero asado con patatas      | Principal      | Cordero               | Alta       | 2 horas            | 20€    |
    | 6    | Ensalada de pollo y patatitas  | Principal      | Pollo                 | Baja       | 20 minutos         | 5€     |
    | 7    | Patatas al curry               | Principal      | Pollo                 | Media      | 1 hora             | 15€    |
    | 8    | Alcachofas con jamón           | Principal      | Alcachofas            | Baja       | 30 minutos         | 10€    |
    | 9    | Alcachofas con jamón           | Principal      | Alcachofas            | Baja       | 45 minutos         | 10€    |
    
    > BUSQUEDAS POR INGREDIENTE PRINCIPAL
    
    Los tengo ordenados en la tabla? NO... -> FULLSCAN u ordenar los datos 
    Podría haber tenido un INDICE con los ingredientes principales:
    
    | ingrediente principal | ubicacion |
    |-----------------------|-----------|
    |                       |           |
    | Alcachofas            | 8         |
    |                       |           |
    | Cordero               | 5         |
    |                       |           |
    |                       |           |
    | Huevo                 | 4         |
    |                       |           |
    |                       |           |
    | Pasta                 | 1, 2, 3   |
    |                       |           |
    |                       |           |
    |                       |           |
    | Pollo                 | 6, 7      |
    
    Si ahora viniera: Alcachofas con jamón, las añado al final de la tabla... perooooo, si tengo un índice,
    debo de añadirlo... y además en su ubicación correspondiente (conservando el orden)
    Si se llena el fichero... que pasa con la BBDD, qué hace la BBDD tradicional? Crea otro fichero
    Pero la búsqueda entonces se debe empezar a hacer sobre los 2 ficheros... y si se llena el segundo?....
    Por eso en las BBDD Relaciones hay una tarea típica que hacen los DBA que es regenerar los índices cada X tiempo... 
    para evitar problemas de rendimiento
    
    ESTE EJEMPLO ES DE UN INDICE DIRECTO... pero hay otros tipos de índices: INDICES INVERTIDOS
    
> Búsqueda por nombre de la receta

Tengo los nombres ordenados? NO... solución... FULLSCAN o ordenar los datos en un índice
    
    INDICE DIRECTO DEL NOMBRE
    
    | nombre                         | ubicacion |
    |--------------------------------|-----------|
    | Alcachofas con jamón           | 8,9       |
    | Cordero asado con patatas      | 5         |
    | Ensalada de pasta              | 1         |
    | Ensalada de pollo y patatitas  | 6         |
    | Pasta a la bolonesa            | 3         |
    | Pasta a la carbonara           | 2         |
    | Patatas al curry               | 7         |
    | Tortilla de patata             | 4         |

Hago una búsqueda: Tortilla de patata
    Hago una búsqueda binaria en el índice y me dice que está en la ubicación 4
Hago una búsqueda: Que contenga "patata"
    Me sirve de algo ahora el índice? En cualquier caso, la BBDD necesita hacer un FULLSCAN... 
    puede optar según el escenario por hacerlo sobre el índice... Si gana algo.
    RESULTADO en cualquier escenario: BUSQUEDA LENTA DE COJONES
    
    SOLUCION: Haber creado un índice invertido:

# Indices invertidos:

Son índices que generan las BBDD o los indexadores, pero con un preprocesado profundo de los datos :
    
    PARTIMOS DE: 
        | 1    | Ensalada de pasta              |
        | 2    | Pasta a la carbonara           |
        | 3    | Pasta a la bolonesa            |
        | 4    | Tortilla de patata             |
        | 5    | Cordero asado con patatas      |
        | 6    | Ensalada de pollo y patatitas  |
        | 7    | Patatas al curry               |
        | 8    | Alcachofas con jamón           |
        | 9    | Alcachofas con jamón           |
    
    1º TOKENIZAR LOS TEXTOS: Espacios en blanco, comas, puntos, paréntesis, números
        | 1    | Ensalada-de-pasta              |
        | 2    | Pasta-a-la-carbonara           |
        | 3    | Pasta-a-la-bolonesa            |
        | 4    | Tortilla-de-patata             |
        | 5    | Cordero-asado-con-patatas      |
        | 6    | Ensalada-de-pollo-y-patatitas  |
        | 7    | Patatas-al-curry               |
        | 8    | Alcachofas-con-jamón           |
        | 9    | Alcachofas-con-jamón           |
    
    2º ELIMINAR LO QUE SE DENOMINAN STOP-WORDS: Palabras que no aportan valor a la búsqueda
        | 1    | Ensalada-?-pasta              |
        | 2    | Pasta-?-?-carbonara           |
        | 3    | Pasta-?-?-bolonesa            |
        | 4    | Tortilla-?-patata             |
        | 5    | Cordero-asado-?-patatas       |
        | 6    | Ensalada-?-pollo-?-patatitas  |
        | 7    | Patatas-?-curry               |
        | 8    | Alcachofas-?-jamón            |
        | 9    | Alcachofas-?-jamón            |
    
    3º CALCULAR LA RAIZ ETIMOLOGICA DE LA PALABRA / NORMALIZACION DE LA PALABRA (quitar acentos, minusculas)
        | 1    | Ensalada-?-pasta             -> | ensalad-?-past |
        | 2    | Pasta-?-?-carbonara          -> | past-?-?-carbon |
        | 3    | Pasta-?-?-bolonesa           -> | past-?-?-bolon |
        | 4    | Tortilla-?-patata            -> | tortill-?-patat |
        | 5    | Cordero-asado-?-patatas      -> | corder-asad-?-patat |
        | 6    | Ensalada-?-pollo-?-patatitas -> | ensalad-?-poll-?-patat |
        | 7    | Patatas-?-curry              -> | patat-?-curry |
        | 8    | Alcachofas-?-jamón           -> | alcachof-?-jamon |
        | 9    | Alcachofas-?-jamón           -> | alcachof-?-jamon |
    
    4º CONSOLIDO LOS TERMINOS
        | ensalad | id(1, 1) + id(6, 1) |
        | past    | id(1, 3) + id(2, 1) + id(3,1)|
        | carbon  | id(2, 4)|
        | bolon   | id(3, 4)|
        | tortill | id(4, 1)|
        | patat   | id(4, 3) + id(5, 4) + id(6, 5) + id(7, 1)|
        | corder  | id(5, 1)|
        | asad    | id(5, 2)|
        | poll    | id(6, 3)|
        | curry   | id(7, 3)|
        | alcachof| id(8, 1) + id(9, 1)|
        | jamon   | id(8, 3) + id(9, 3)|
    
    5º ORDENO LOS TERMINOS
        | alcachof| id(8, 1) + id(9, 1)|
        | asad    | id(5, 2)|
        | bolon   | id(3, 4)|
        | carbon  | id(2, 4)|
        | corder  | id(5, 1)|
        | curry   | id(7, 3)|
        | ensalad | id(1, 1) + id(6, 1) |
        | jamon   | id(8, 3) + id(9, 3)|
        | past    | id(1, 3) + id(2, 1) + id(3,1)|
        | patat   | id(4, 3) + id(5, 4) + id(6, 5) + id(7, 1)|
        | poll    | id(6, 3)|
        | tortill | id(4, 1)|
    
        ^^^^ ESTE ES EL INDICE INVERTIDO
    
        Si los indices directos suponen una gran penalización a la hora de actualizar una BBDD, 
        los indices invertidos suponen una DESMESURADAMENTE ENORME GIGANTIDAD De penalización a la hora de insertar datos en la BBDD

ESTO es lo que las BBDD relaciones hacen de aquella manera... si es que llegan a hacerlo: ORACLE... más o menos (ORACLE TEXT)
En SQL Server, también hay algo parecido, pero no es tan potente como en Oracle
En otras BBDD: Mysql, postgresql... tienen algo... pero es muy cutre.

Y esto es lo que nos aporta ElasticSearch... que tiene un motor de indexación que es una pasada...
Aunque realmente, TODO Este trabajo, no lo hace ES... lo hace Lucene... 

Ahora, al hacer una búsqueda
> patatas
    LO PRIMERO, le aplico al término (o términos) la misma normalización que le he aplicado a los términos del índice
    1 Separo por token
    2 Elimino stop-words
    3 Normalizo
    Y una vez que tengo eso, el resultado "patat", busco en el índice invertido (MEDIANTE BUSQUEDA BINARIA)
    y me devuelve los ids de los registros que contienen ese término
    
    TABLA: RECETAS
    
    | id   | nombre                         | tipo_plato     | ingrediente_principal | dificultad | tiempo_preparacion | precio |
    |------|--------------------------------|----------------|-----------------------|------------|--------------------|--------|
    | 1    | Ensalada de pasta              | Entrante       | Pasta                 | Baja       | 20 minutos         | 5€     |
    | 2    | Pasta a la carbonara           | Principal      | Pasta                 | Media      | 30 minutos         | 10€    |
    | 3    | Pasta a la bolonesa            | Principal      | Pasta                 | Media      | 30 minutos         | 10€    |
    | 4    | Tortilla de patata             | Principal      | Huevo                 | Baja       | 20 minutos         | 5€     |
    | 5    | Cordero asado con patatas      | Principal      | Cordero               | Alta       | 2 horas            | 20€    |
    | 6    | Ensalada de pollo y patatitas  | Principal      | Pollo                 | Baja       | 20 minutos         | 5€     |
    | 7    | Patatas al curry               | Principal      | Pollo                 | Media      | 1 hora             | 15€    |
    | 8    | Alcachofas con jamón           | Principal      | Alcachofas            | Baja       | 30 minutos         | 10€    |
    | 9    | Alcachofas con jamón           | Principal      | Alcachofas            | Baja       | 45 minutos         | 10€    |

    > BUSQUEDAS POR INGREDIENTE PRINCIPAL
    
    Los tengo ordenados en la tabla? NO... -> FULLSCAN u ordenar los datos 
    Podría haber tenido un INDICE con los ingredientes principales:
    
    | ingrediente principal | ubicacion |
    |-----------------------|-----------|   De datos tengo 10Kbs... pero de blancos tengo 30Kbs = Fichero 40Kbs en disco
    |                       |           |
    | Alcachofas            | 8         |
    |                       |           |
    | Cordero               | 5         |
    |                       |           |
    |                       |           |
    | Huevo                 | 4         |
    |                       |           |
    |                       |           |
    | Pasta                 | 1, 2, 3   |
    |                       |           |
    |                       |           |
    |                       |           |
    | Pollo                 | 6, 7      |

    Si ahora viniera: Alcachofas con jamón, las añado al final de la tabla... perooooo, si tengo un índice,
    debo de añadirlo... y además en su ubicación correspondiente (conservando el orden)
    Si se llena el fichero... que pasa con la BBDD, qué hace la BBDD tradicional? Crea otro fichero
    Pero la búsqueda entonces se debe empezar a hacer sobre los 2 ficheros... y si se llena el segundo?....
    Por eso en las BBDD Relaciones hay una tarea típica que hacen los DBA que es regenerar los índices cada X tiempo... 
    para evitar problemas de rendimiento

    ESTE EJEMPLO ES DE UN INDICE DIRECTO... pero hay otros tipos de índices: INDICES INVERTIDOS

> Búsqueda por nombre de la receta

Tengo los nombres ordenados? NO... solución... FULLSCAN o ordenar los datos en un índice

    INDICE DIRECTO DEL NOMBRE

    | nombre                         | ubicacion |
    |--------------------------------|-----------|
    | Alcachofas con jamón           | 8,9       |
    | Cordero asado con patatas      | 5         |
    | Ensalada de pasta              | 1         |
    | Ensalada de pollo y patatitas  | 6         |
    | Pasta a la bolonesa            | 3         |
    | Pasta a la carbonara           | 2         |
    | Patatas al curry               | 7         |
    | Tortilla de patata             | 4         |

Hago una búsqueda: Tortilla de patata
    Hago una búsqueda binaria en el índice y me dice que está en la ubicación 4
Hago una búsqueda: Que contenga "patata"
    Me sirve de algo ahora el índice? En cualquier caso, la BBDD necesita hacer un FULLSCAN... 
    puede optar según el escenario por hacerlo sobre el índice... Si gana algo.
    RESULTADO en cualquier escenario: BUSQUEDA LENTA DE COJONES

    SOLUCION: Haber creado un índice invertido:

# Indices invertidos:

Son índices que generan las BBDD o los indexadores, pero con un preprocesado profundo de los datos :

    PARTIMOS DE: 
        | 1    | Ensalada de pasta              |
        | 2    | Pasta a la carbonara           |
        | 3    | Pasta a la bolonesa            |
        | 4    | Tortilla de patata             |
        | 5    | Cordero asado con patatas      |
        | 6    | Ensalada de pollo y patatitas  |
        | 7    | Patatas al curry               |
        | 8    | Alcachofas con jamón           |
        | 9    | Alcachofas con jamón           |
    
    1º TOKENIZAR LOS TEXTOS: Espacios en blanco, comas, puntos, paréntesis, números
        | 1    | Ensalada-de-pasta              |
        | 2    | Pasta-a-la-carbonara           |
        | 3    | Pasta-a-la-bolonesa            |
        | 4    | Tortilla-de-patata             |
        | 5    | Cordero-asado-con-patatas      |
        | 6    | Ensalada-de-pollo-y-patatitas  |
        | 7    | Patatas-al-curry               |
        | 8    | Alcachofas-con-jamón           |
        | 9    | Alcachofas-con-jamón           |
    
    2º ELIMINAR LO QUE SE DENOMINAN STOP-WORDS: Palabras que no aportan valor a la búsqueda
        | 1    | Ensalada-?-pasta              |
        | 2    | Pasta-?-?-carbonara           |
        | 3    | Pasta-?-?-bolonesa            |
        | 4    | Tortilla-?-patata             |
        | 5    | Cordero-asado-?-patatas       |
        | 6    | Ensalada-?-pollo-?-patatitas  |
        | 7    | Patatas-?-curry               |
        | 8    | Alcachofas-?-jamón            |
        | 9    | Alcachofas-?-jamón            |
    
    3º CALCULAR LA RAIZ ETIMOLOGICA DE LA PALABRA / NORMALIZACION DE LA PALABRA (quitar acentos, minusculas)
        | 1    | Ensalada-?-pasta             -> | ensalad-?-past |
        | 2    | Pasta-?-?-carbonara          -> | past-?-?-carbon |
        | 3    | Pasta-?-?-bolonesa           -> | past-?-?-bolon |
        | 4    | Tortilla-?-patata            -> | tortill-?-patat |
        | 5    | Cordero-asado-?-patatas      -> | corder-asad-?-patat |
        | 6    | Ensalada-?-pollo-?-patatitas -> | ensalad-?-poll-?-patat |
        | 7    | Patatas-?-curry              -> | patat-?-curry |
        | 8    | Alcachofas-?-jamón           -> | alcachof-?-jamon |
        | 9    | Alcachofas-?-jamón           -> | alcachof-?-jamon |

    4º CONSOLIDO LOS TERMINOS
        | ensalad | id(1, 1) + id(6, 1) |
        | past    | id(1, 3) + id(2, 1) + id(3,1)|
        | carbon  | id(2, 4)|
        | bolon   | id(3, 4)|
        | tortill | id(4, 1)|
        | patat   | id(4, 3) + id(5, 4) + id(6, 5) + id(7, 1)|
        | corder  | id(5, 1)|
        | asad    | id(5, 2)|
        | poll    | id(6, 3)|
        | curry   | id(7, 3)|
        | alcachof| id(8, 1) + id(9, 1)|
        | jamon   | id(8, 3) + id(9, 3)|
    
    5º ORDENO LOS TERMINOS
        | alcachof| id(8, 1) + id(9, 1)|
        | asad    | id(5, 2)|
        | bolon   | id(3, 4)|
        | carbon  | id(2, 4)|
        | corder  | id(5, 1)|
        | curry   | id(7, 3)|
        | ensalad | id(1, 1) + id(6, 1) |
        | jamon   | id(8, 3) + id(9, 3)|
        | past    | id(1, 3) + id(2, 1) + id(3,1)|
        | patat   | id(4, 3) + id(5, 4) + id(6, 5) + id(7, 1)|
        | poll    | id(6, 3)|
        | tortill | id(4, 1)|

        ^^^^ ESTE ES EL INDICE INVERTIDO

        Si los indices directos suponen una gran penalización a la hora de actualizar una BBDD, 
        los indices invertidos suponen una DESMESURADAMENTE ENORME GIGANTIDAD De penalización a la hora de insertar datos en la BBDD

ESTO es lo que las BBDD relaciones hacen de aquella manera... si es que llegan a hacerlo: ORACLE... más o menos (ORACLE TEXT)
En SQL Server, también hay algo parecido, pero no es tan potente como en Oracle
En otras BBDD: Mysql, postgresql... tienen algo... pero es muy cutre.

Y esto es lo que nos aporta ElasticSearch... que tiene un motor de indexación que es una pasada...
Aunque realmente, TODO Este trabajo, no lo hace ES... lo hace Lucene... 

Ahora, al hacer una búsqueda
> patatas
    LO PRIMERO, le aplico al término (o términos) la misma normalización que le he aplicado a los términos del índice
    1 Separo por token
    2 Elimino stop-words
    3 Normalizo
    Y una vez que tengo eso, el resultado "patat", busco en el índice invertido (MEDIANTE BUSQUEDA BINARIA)
    y me devuelve los ids de los registros que contienen ese término

Así lo hace google

Ahora bien... computacionalmente, el proceso de generar un índice invertido es muy costoso...


ElasticSearch es una herramienta que:
    
    - Va a recibir datos para indexar, a través de un API HTTP REST.
    - Los datos que recibe ES son DOCUMENTOS JSON (texto plano)
    - ES toma ese texto (documento JSON) y se lo pasa a un LUCENE, junto con unas instrucciones, acerca de cómo indexarlo:
        - Qué StopWords debe usar, qué tokenización debe usar... si debe discriminar mayúsculas/minúsculas...
    - Y Lucene hace la indexación... y guarda el resultado en un conjunto de INDICES INVERTIDOS que internamente gestiona...
        Ya que Lucene no va a generar un UNICO indice invertido... generá normalmente UNO POR CAMPO dentro del JSON...
        A veces menos... a veces más... SEGUN LO CONFIGURE
        
            {
                "id": 1,
                "nombre": "Ensalada de pasta",
                "tipo_plato": "Entrante",
                "ingrediente_principal": "Pasta",
                "dificultad": "Baja",
            }

            INDICES INVERSOS QUE GENERA EL LUCENE:
                nombre: 
                    | ensalad | id(1, 1) |
                    | past    | id(1, 3) |
                tipo_plato:
                    | entrant | id(1, 1) |
                ingrediente_principal:
                    | past    | id(1, 1) |
                ...
Pero este proceso, hemos dicho que es muy muy costoso.... por lo que en algunos escenarios, si tengo alta demanda de indexaciones,
Me puede interesar paralelizar el trabajo, teniendo muchos LUCENES que vayan indexando por separado distintos documentos...

ES Se convierte en un ORQUESTADOR de LUCENES.
Esa colección de LUCENES (en ES, a cada LUCENE se le denomina SHARD... más o menos) es a lo que en ES denominamos INDICE

    
    ElasticSearch
            |
        -------------- INDICE A -------------- y se encarga de indexar cierto tipo de documentos
        Lucene 1         Lucene 2       Lucene 3        Shards primarios            | Pero en muchas ocasiones, en ES 
        Lucene 1(r1)     Lucene 2(r1)   Lucene 3(r1)    Shards de replicación       |   A una columna le denominan SHARD
        Lucene 1(r2)     Lucene 2(r2)   Lucene 3(r2)    Shards de replicación       |
        ---------------------------------------------                   
        Shard 1          Shard 2        Shard 3
    
        Un SHARD es un LUCENE

Os había dicho que un shard +- equivale a un LUCENE... pero no es exactamente así... O SI.

Imaginad que pierdo un LUCENE (un shard)... qué ocurre? El original (los datos) no los pierdo ... siempre y cuando los tenga almacenados en otro sitio. Lo que desde luego siempre pierdo es la capacidad de hacer búsquedas sobre esos datos. ES ADMISIBLE? Para un buscador NO.
Entonces? HA: Redundancia: REPLICAS

Cuando me hablan de SHARD ... sin apellido... entendemos que nos hablan del conjunto de SHARD PRIMARIO + SUS REPLICAS
Pero si me dicen SHARD PRIMARIO, se refieren dentro de una columna, al que opera como primario
Y por SHARD DE REPLICACION, se refieren dentro de una columna, a los que no operan como primario

Los shards primarios son exactamente iguales a los shards de replicación. Simplemente uno de ellos ES lo nombre PRIMARIO.
De ese es del que se lee la información (QUERIES)
Si un SHARD primario se cae, se elige uno de los SHARD de replicación para que pase a ser el PRIMARIO

Qué pasa con las búsquedas?
> Dame la receta de la tortilla de patatas
Yo se lo voy a pedir a ES... por HTTP.
    ES debe encontrar el shard adecuado que alberga esa información. Puede ser que lo sepa de antemano.
    Puede ser que no... en cuyo caso, tendrá que preguntar a todos los shards... y luego consolidar la información, lo cual será siempre más lento.

Cómo elige ES el shard que va a almacenar un determinado documento.
ES, cuando creamos un índice, le voy a decir anticipadamente el número de shards que quiero tener asociados a ese índice.
Le diré también cuántas replicas quiero tener.... Quiero 3 shards, con 2 replicas (incluyen el primario)

Cuando llega un documento, se le aplica un algoritmo de enrutamiento que le dice a ES en qué shard debe almacenar ese documento.
- Por defecto, lo que se hace es un hash del ID del documento... un hash numérico
  - Y se toma el resto de la división entera (MODULO) del ese hash entre el número de shards totales (entendidos como columnas)
    - Esto da lugar siempre a un número entre: RESTO = [0 - (número de shards -1)]
      32098238092309823498742896237908230987 | 3
                                             +------------ 
                                    RESTO     Resultado
    Ese el el shard al que se envía el documento. Los shards dentro de un INDICE (ES) se numeran de 0 a N-1
Ese es el algoritmo por defecto de enrutamiento de ES... que puedo cambiar:
Coge y haz la huella del tipo_de_plato, en lugar del ID...
    Esto me asegura que: Todos los documentos (recetas) del mismo "tipo_de_plato" van a parar al mismo shard

Y si hago muchas búsquedas por "tipo_de_plato" solo tendré que buscar en un shard... y no en 50...

Si busco un documento... la cosa no es tan compleja...
El problema es cuando una búsqueda da lugar a tropecientos (4) resultados.. Sobre todo si ES no sabe a priori donde buscar (y es lo habitual).
    ES necesitará consultar a todos los SHARDS (Lucenes) y luego consolidar la información.
    El consolidar la información es otro proceso BIEN COMPLEJO (costoso computacionalmente)... Por qué?
        Porque cuando me piden datos... me los van a pedir siempre ORDENADOS!
        Y cada Lucene tiene los datos ordenados... internamente.
        Pero cuando yo cojo los datos de 15 lucenes... antes de devolverlos, me toca REORDENARLOS.
            Diego, las facturas del Cliente A
            Jesús, las facturas del Cliente B
            Joaquín, las facturas del Cliente C
            Luis, las facturas del Cliente D

            Búsqueda por 2024, ordenada por fecha.
            Cada uno me podéis dar las facturas de vuestro cliente, ordenadas por fecha...
            Pero al consolidarlas... necesito reordenarlas... y eso es un proceso costoso
            VAYA FOLLON !!!!!

Y aquí empezamos con otro concepto.
Dentro de un cluster de ES, vamos a tener DISTINTOS TIPOS DE NODOS:
- master
- coordinadores
- data
- ml
- ingest

Para arrancar un cluster de ES, a priori necesito:
- 3 nodos [master, coordinador, data, ingest...]... ESTO NO ES MUY HABITUAL

La tarea de master es muy importante... y voy a dedicar nodos en exclusiva a ese trabajo.
- 3 master (2 y medio)
- 2 x resto de cosas [coordinadores, data, ml, ingest].. Digo 2 ... por tener un mínimo de HA

Según el cluster va creciendo.... voy añadiendo nodos... y los voy especializando.
- Si veo que tengo mucho curro de coordinación, desgajaré ese trabajo en nodos independientes... al menos 2

    Cluster ES
        master1 (IP: 192.168.0.100)
            |
        master2 (IP: 192.168.0.101)
            |
        master3 (IP: 192.168.0.102) (que luego no será así)
            |
        data1    (IP: 192.168.0.103)
            |
        data2    (IP: 192.168.0.104)
            |
        coordinador1 (IP: 192.168.0.105)        \
            |                                       Balanceador de carga (192.168.1.100)
        coordinador2 (IP: 192.168.0.106)        /
            |
        ingest1 (IP: 192.168.0.107)             \ 
            |                                       Balanceador de carga (192.168.1.101)
        ingest2 (IP: 192.168.0.108)             / 
    
    Los clientes que ataquen al cluster para pedir datos, atacarán a que IP?
        2 opciones: 
            Opción 1: Darle una lista de IPs: 192.168.0.5 + 192.168.0.6
            Opción 2: Montar un balanceador de carga externo: 192.168.1.100
    Los clientes que ataquen al cluster para indexar datos, atacarán a que IP?
        2 opciones: 
            Opción 1: Darle una lista de IPs: 192.168.0.7 + 192.168.0.8
            Opción 2: Montar un balanceador de carga externo: 192.168.1.101

La única restricción es al menos tener 3 maestros. De los cuales unicamente 1 actuará como tal... los otros 2 serán de reserva.
    Tengo un cluster ACTIVO / PASIVO de maestros
    Es decir... que tiro 2 puñeteras máquinas a la basura... no las uso... salvo en caso de catástrofe... LA HA es CARA !

Por qué 3? y no 2 o 4 o 7?
Siempre impares... por qué? Para evitar el BRAIN Splitting
    Si tengo 3 nodos... y pierden red entre si:
        maestro1
        maestro2
        ----
        maestro3 <<< ESTE NO QUIERO QUE SE PONGA A ACTUAR COMO MAESTRO

Lo que pasa es que es una putada... ya que tiro 2 nodos a la basura... y ahí nos sale un concepto que tiene ES: Maestro de mentirijilla

Lo que defino son 2 nodos master
Y luego a un nodo (data) le digo que también es maestro... pero no para ejercer como tal... sino solo con permiso para votar.
De esa forma ahorro una máquina... y es la conf. habitual de un ES.

Los master son los que controlan las máquinas que hay operativas en el cluster (o miran si alguna se cae)
Los que deciden a que máquinas deben ir los shards (tanto primarios como de replicación) de cada índice.

Los nodos de datos, se encargan de ejecutar los Lucenes... y de almacenar los datos (pero esto también lo hace lucene)
A éstos nodos, según mi cluster se hace más grande, les voy quitando responsabilidades adicionales.
Empiezo con un cluster:
    2 master
    2 para todo lo demás
        Y a uno de ellos le permito votar (maestro de mentirijilla)

Según el cluster va recibiendo más y más carga de trabajo, mediante monitorización, identifico como expandirlo:
- A veces meto otro nodo de esos que hacen de todo.
- A veces meto nodos de coordinación y le quitamos responsabilidades a los nodos de datos
- A veces meto nodos de ingestión... y le quito responsabilidades a los nodos de datos
- Cuando ya tengo nodos repartidos.. a veces meto más nodos de datos.. o de ingesta... o de coordinación... YO QUE SE !!


DECISIONES DE LAS QUE ME ARREPIENTO... que cambiarlas es un follón de huevos:
- Cambiar el algoritmo de routing
- Cambiar el proceso de indexación (los stop words... la tokenización... la normalización)
- Número de shards de un índice... No se puede cambiar a futuro.
  - Podré dividir los shards...
  - Podré juntar los shards...
  - Pero no cambiar alegremenete el número de shards.
     5 x 2 = 10
        De 10, podré pasar de nuevo a 5... pero no a 7
     Pero no a 2

---

# Donde entra en todo esto el tema de la monitorización?

Porque no tiene pinta ES de ser un sistema de monitorización al uso, no?
Y si tengo herramientas que han nadido para ser sistemas de monitorización.
Qué pinta ES en todo esto? ES al final, solo es un indexador de datos... y luego hacer búsquedas sobre ellos.
Esos datos pueden ser de monitorización por ejemplo...
Hay alguna gracia? algo especial que tenga los datos de monitorización, que hagan de ES una buena opción para su gestión?
Los datos de monitorización presentan algunas características:
- SON DATOS sobre los que voy a querer hacer búsquedas complejas:
  - No son datos que una BBDD RELACIONA vaya a gestionar bien.
        IGUAL QUE CON EL TITULO/ NOMBRE de mi receta de cocina.
        "10:00:00  12/12/2020 Error crítico en el sistema"
        "10:00:01  12/12/2020 No se ha podido conectar con el servidor: error de acceso a la red"
    YO QUIERO BUSCAR POR "ERROR"
  - 192.168.2.102 < ERROR
    > Quiero hacer una búsqueda por: 192.168.0.0/16

- SON DATOS MUERTOS. No se actualizan. Solo admiten como operaciones: 
  - Insert
  - Delete
  - Select
  - Update???? NO.. ya que van asociados a una FECHA... y cuando ocurren... ya ocurrieron... no puedo cambiar el pasado!
  Qué impacto tiene esto en el contexto de ES?
  Al principio de hoy os he dicho que: Un indexador guarda copias del dato original que se pide indexar? Es su misión?
    NO... aunque lo puedo solicitar (de hecho en ES viene de serie)
    También lo hace google.

    En condiciones normales... hablando en genérico de cualquier tipo de dato. El indexador no nos asegura tener la última versión del dato... así pasa con google.
    Pero eso con datos de monitorización no ocurre. Un evento que se registro, no cambia en el futuro. El registro (log) se hizo... y fué el que fué... En el futuro habrá otro dato... asociado a otro timestamp... pero el de ahora... no cambia.

    No penseis en un log como un fichero... sino como cada entrada de ese fichero... una medición o evento que ha ocurrido en un momento dado del tiempo... y que nuedo cambiar ese HECHO que aconteció.

    De esta forma, el dato cacheado que tengo en el indexador (ES) se convierte en el dato REAL...
    NO NECESITAMOS DE UNA BBDD INDEPENDIENTE!

    Dicho de otra forma, si estuviera guardando los eventos que registro en una BBDD
    Y posteriormente los mando a ES para su indexación.
    Habría posibilidad de que el dato cambie a posteriori en la BBDD sin que sentere ES? NO.. ya el dato no cambia... por ser un dato muerto... un evento registrado en el pasado.
    No hay posibilidad de que el dato se DESSINCRONICE entre ES y el repositorio donde guardo el dato.
        Cosa que con google y una web si me ocurre.
            Tengo garantía de que el dato cacheado en Google sea el contenido REAL actual de la WEB? NO
            Tengo garantía de que un EVENTO indexado en ES y cacheado sea el contenido REAL ACTUAL de ese dato guardado en una BBDD? SI

        Entonces: PARA QUE COJONES QUIERO LA BBDD? El dato cacheado en ES me sirve como copia del dato REAL... inalterable.

    ES no es una BBDD al uso... pero guarda una copia del dato cuando se indexa... y dado que el dato a futuro es IMPOSIBLE por definición que cambie... Se me acaba de convertir en un REPOSITORIO del dato original.... 
    No siendo una BBDD.... en este escenario, me hace las veces, por la cache!
    De hecho, por so pYA PUEDO PRESCINDIR DEL DATO ORIGINAL donde se generó... me la pela... que de ahí se borre.

El punto es que:
- Necesito un idexador por cojones... porque quiero hacer búsquedas que las BBDD relacionales no me dan.
- Y si el indexador me permite guardar una copia del dato... lo que ya no me hace falta es una BBDD independiente.

Y como a día de hoy no hay nadie (ninguna herramienta) con las capacidades de indexación y búsquedas que ofrece ES,
Se ha convertido en la mejor alternatiiva para guardar datos de REGISTROS/OBSERVABILITY, MONITORIZACION, METRICAS

Me permite explotar esos datos posteriormente (mediante peticiones HTTP)... = VAYA COÑAZO !!!!!
Pues menos mal que tenemos KIBANA, que es una SUPERHERRAMIENTA para explotar datos guardados en un ES:
- Hacer queries
- Generar dashboards
- Mostrar pantallas con todos los datos que se vayan recopilando en TR.

Pero... captura datos el KIBANA? No... y el ES? tampoco

Pues necesito alguien que capture datos: BEATS FAMILY
- FileBeat: Va mirando cada linea que se escribe en un fichero de texto... y la manda
- MetricBeat: Va mirando unas métricas (de HW: %Uso CPU, de SW: # conexiones abiertas en el Apache) y las manda
- HeartBeat: Va mirando si alguien esta vivo o no.... y lo manda

Y donde lo mandan? Lo pueden mandar a ES? SI... pero qué problemas tendría?
- Va el dato tal cual aparece en el fichero... o en origen... Y esto me interesa?
  - No siempre... depende el uso que vaya a hacer del dato.
  Y esas herramientas no permiten hacer algún tipo de transformación a los datos? SIII... montonón de transformación.
  PERO NO LO HACEMOS NUNCA:
    - Lo primero, quiero sacar los datos lo más rápido que pueda del sitio original:
       httpd
        VVVVV escribiendo
        access.log 
        ^^^^^
        Filebeat procesando
    - Siempre quiero el mismo procesamiento para los datos? PUEDE QUE NO
- Me puede interesa el dato mandarlo a:
  - Un sistema de monitorización para los tios raros de sistemas... pa' que miren si el apache se ha caído... o si tiene algún error.
  - Me puede interesar mandar el dato a negocio... para que vean el uso actual del sistema: Gente que hay dentro, que productos están mirando,
  - Analizar el éxito de una campaña comercial que hemos lanzado.

Por todo ello, al final montamos entre medias de los agentes BEAT y el ES, un LOGSTASH
    
    Máquina 1
        httpd
            v
          access.log
            ^                                   Logstash2
        filebeat  >>>           KAFKA      <<<  Logstash1 (routing)      >>>> Logstash2 (transformación 1)        >> ES1 (Sistemas)
                                  ^                                                      filtro cosas que no sean error
    Máquina 2                     ^                                      >>>> Logstash2 (transformación 2)        >> ES2 (Negocio)
        httpd                     ^                                                      lo que me interesa es lo que no son errores
            v                     ^
          access.log              ^
            ^                     ^
        filebeat  >>>>>>>>>>>>>>>>>           

Stack ELK (nos falta la B=BEATS)... pero me falta otra letra ahí... OTRA K... KAFKA
      BKLEK = NOMBRE GUAY... aunque nos ahogaríamos cada vez que quisieramos decirlo

Logstash NO ofrece HA... como se caiga voy bien jodido...
Los datos se me empiezan a acumular en los servidores... hasta que petan... y/o pierdo datos.

KAFKA si trabaja con HA.. siempre se monta en cluster. 
Si eso está bien montao, es imposible perder datos de un KAFKA. que un KAFKA se caiga.
Lo que quiero es que los BEATS, tener la certeza, que son capaces de desaguar los datos a toda ostia y sin complicaciones.
Y ya luego... 









---

# Algoritmo de HASH / HUELLA

LA LETRA DEL DNI es un HASH (huella) del número del DNI

Una huella o hash es un valor que obtengo desde un dato... con algunas características:
- Debe haber poca probabilidad de colisión ... es decir, poca probabilidad de que 2 datos diferentes generen la misma huella.
- El mismo dato, siempre debe dar lugar a la misma huella
    12345678 -> W
    98765432 -> W
- Desde la huella es imposible recuperar el dato original
