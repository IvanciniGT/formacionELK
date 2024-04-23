# ElasticSearch es un sistema que trabaja en cluster

Donde puede repartir tareas entre distintos nodos.
Indexa el documento 1 -> Nodo1
Indexa el documento 2 -> Nodo2
Indexa el documento 3 -> Nodo1

# ElasticSearch es un sistema distribuido

Las tareas de alto nivel se descomponen en tareas de más bajo nivel, 
que se realizan por diferentes componentes, que pueden estar ubicados en máquinas(entornos) diferentes.


# Almacenamiento en entornos de producción

5 Tbs de pelis... Western blue y voy apañao. = 50€
5 Tbs de datos en pro.
    Cada dato en un entorno de pro lo voy a guardar al menos 3: 5Tbs x 3 = 15Tbs
    15 Tbs... de almacenamiento del caro. 150€ x 3
Copias de seguridad? Le hago? o como lo tengo replciado no hace falta:
    Replicas son para: HA del dato
    Copias de seguridad: Recuperación de catastrofes
    
# Indices en ES

Un índice es un conjunto de shards.
No es lo mismo que tenga un índice del que solo voy a querer leer, que un índice del que quiera leer y escribir.
    
                    INDICE A
                        |
    +-------------------+--------------------+
    |                   |                    |
    Shard1              Shard2              Shard3
     = Lucene1           = Lucene 2          = Lucene 3 = Proceso JAVA
     
     
    Llega un documento: doc1: {...} a indexar.
    Lo primero, determinar en que shard (qué lucene) va a indexar ese documento, que se quiere guardar (indexar) en el Índice A
    Esa decisión la toma: EL MAESTRO!
         -> Lucene2
                Hace su magia... aplica todas esas transformaciones para generar los indices invertidos que hablábamos ayer... 
                junto con algunos indices directos.
                        {
                            "id": 1,
                            "nombre": "Ensalada de pasta",      -> Indice invertido
                            "tipo_plato": "Entrante",           -> Indice directo
                            "ingrediente_principal": "Pasta",   -> Indice directo
                            "dificultad": "Baja",               -> Indice directo
                        }                                               ^^^^
                                                                    Esto se define en el ESQUEMA del índice
                                                                        * Nota... esta es otra de esas cosas... 
                                                                          de las que nos arrepentimos si no las hacemos desde el principio bien
                                                                          ES viene preparado para tragarse cualquier cosa. No me pone pegas.
                                                                          Y yo.. que cuando empiezo sé poco.. y aplico la ley del mínimo esfuerzo:
                                                                          Funciona? SI... dejalo!
                                                                          Si yo no defino un esuqema... él lo va definiendo por mi.
                                                                          El problema es que el esque ma que él define, se parecerá al que a mi me gustaría como un huevo a una castaña!
                                                                          Y luego LLORARÉ = PAGARÉ UNA PASTA DE ALMACENAMIENTO
                                                                                          = NO PODRE REALIZAR CIERTAS BUSQUEDAS
                                                                                          = EL SISTEMA IRA LENTO
    Al final... los datos de indexación se deben guardar en unos ficheros.
    Pregunta: Ese proceso... guarda los datos en los ficheros ORDENADOS o sin ORDENAR? Los índices? 
    ES No guarda los datos que va generando de indexación ordenados en el fichero de marras. = UPS !!!!!!
    LUCENE asociado a un índice va creando no uno... sino mogollón de ficheros (SEGMENTOS)... y cuando llegan datos nuevos... los pega al final del fichero
    Ésto lo hace para ahorrar espacio en disco.
    De hecho el proceso es más complejo.
    LUCENE no guarda en disco cada dato que le llega y procesa..
    Hace un flush a Disco cada X tiempo... volcando a disco todos los datos que haya recibido/procesado/generado durante ese periodo.
    El trozo, la porción de datos generados en un periodo de tiempo, va ordenada entre si...
    
    El hecho es que ES no es capaz de usar índices desde ficheros... no como las BBDD.
    Cuando quiere trabajar con un LUCENE, debe cargar sus datos en RAM... para CONSOLIDAR todos esos trozos de fichero... REORDENARLOS Y AGRUPARLOS ENTRE SI
    El SHARD (sus datos) deben entrar en RAM.
    
    Las búsquedas no son tan eficientes en estos escenarios... sobre todo si los datos no los tengo en RAM.
    Pero todos los shards no me entran en RAM... tengo muchos datos, repartidos en muchos HDD.
    Aquí hay un dilema.
    
    En algún momento me interesa consolidar todos esos trozos de fichero... y esos ficheros (segmentos)... y reagruparlo... y reordenarlo internamente.
    De forma que cuando se necesite una búsqueda en ese shard, y los datos no estén en RAM, la lectura / carga de los datos a RAM sea muy rápida.
    ESTE PROCESO ES CARO DE BIGOTES... Me implica: 
        - Restructurar los datos
        - Reescribir el fichero unificado completo.
    En este proceso además, puedo ahorrar espacio en Disco.
    
            Un segmento del indice A
            ------------------------------------------------------------------
            ensalada: doc 17(5) + doc 23(4)
            patatas: doc 18(1) 
            ------------------------------------------------------------------
            ensalada: doc 231(5) + doc 123(4)
            patatas: doc 1128(1) 
    
                        vvvv
    
            ------------------------------------------------------------------
            ensalada: doc 17(5) + doc 23(4) + doc 231(5) + doc 123(4)
            patatas: doc 18(1) + doc 1128(1) 
    
    Cuando me puede interesar hacer ese proceso? La decisión aquí en ES es: Cuando no vaya a cambiar más el contenido del índice.
    
    Y tendremos índices en distintos ESTADOS: OPEN (R+W) ... CERRADOS       CONGELADOS      ....
    Cuando sé que en índice no voy a meter más datos, lo congelo... y en ese momento me trae cuenta el unificar sus datos (el proceso del que hablábamos arriba.)
    
    ¿ Cómo tengo yo garantía de que no voy a añadir cosas en un índice nunca maix?
    ASOCIANDO cada índice a un periodo de tiempo:
        - logs_apache_01_2024
            Cuando llegue febrero? Voy a meter algo nuevo en ese índice? NUNCA MAIX
    Los índices que tengo abierto, los tendré en RAM... los ficheros se usan para PERSISTENCIA.
        Me da igual de cara al rendimiento de la búsqueda que los ficheros estén bien estructurados o mal estructurados... YA QUE SE HACE EN RAM la búsqueda
    Cuando cierre un índice será cuando lo baje de RAM... y solo lo pondré en RAM cuando se quieran buscar datos de ese índice
        Y ahí es donde me interesa que los ficheros estén bien estructurados.. Para que la carga a RAM Sea lo más rápida posible.
    
    El trabajar con muchos índices tiene una ventaja clara en cuanto a mantenibilidad y eficiencia de los procesos /datos.
    
    Y las búsquedas? cuando quiera hacer búsquedas sobre los últimos 16 meses?
        SELECT * FROM TABLA;
                      INDICE;
    ES viene se serie preparado para trabajar con PATRONES DE NOMBRES DE INDICES: 
        SELECT * FROM logs_apache_*;
        SELECT * FROM logs_apache_*_2024;
        SELECT * FROM logs_apache_01_*;
    Adicionalmente a cada INDICE la podemos poner ALIAS para las búsquedas... (sería el equivalente a las VIEWs de una BBDD Relacional.. algo así)
    
    TODO INDICE tiene un esquema asociado... igual que en una BBDD relaciona, toda tabla tiene su esquema: CAMPOS-TIPOS
    
    Podemos montar PLANTILLAS DE ESQUEMA... y asociarlas con patrones de nombres de INDICES:
    
        PLANTILLA DE ESQUEMA para documentos de tipo: log_apache
        QUIERO QUE SE APLIQUE ESA PLANTILLA A todos líndices cuyo nombre sea: apache_log_*

    ---
    
    JAVA es un lenguaje que crea programas con una gestión tremendamente ineficiente de la RAM.
    Dicho de otra forma: El mismo programa hecho en C++ requiere la mitad o menos de RAM que el mismo programa en JAVA
    Esto es bueno o malo? ES UN FEATURE! Java se diseño con esa premisa: Vamos a hacer un lenguaje que destroce la RAM: 
                                                                         Haga un uso inficiente de la RAM
    Cuanto me cuesta hacer el programa en C++:  400 h  x 60€/h = 24000 €
    Cuanto me cuesta hacer el programa en JAVA: 300 h  x 50€/h = 15000 €
                                                                -------> -9000€ y 2 pastillas de RAM pal servidor = 800€ 

# Instalación de ES

## FASE 1:

Cluster con 3 nodos, todos iguales entre si... que tengan todos los roles
Kibana

## FASE 2:

Cluster con 4 nodos: 2 master + 2 resto tareas ( 1 de ellos con votación)
Kibana

## FASE 3:

Cluster con 4 nodos: 2 master + 2 resto tareas ( 1 de ellos con votación) + 2 coordinadores
Kibana

---

# Modelos de instalación de software

## Modelo tradicional "A HIERRO"
    
         App1 + App2 + App3
    ----------------------------
         Sistema Operativo
    ----------------------------
               HIERRO
               
## Modelo basado en VMs
    
        App1  |  App2 + App3
    ----------------------------
        SO1   |     SO2
    ----------------------------
        VM1   |     VM2
    ----------------------------
            Hipervisor
    ----------------------------
         Sistema Operativo
    ----------------------------
               HIERRO          
               
## Modelo basado en Contenedores
    
        App1  |  App2 + App3
    ----------------------------
        C1    |     C2
    ----------------------------
        Gestor de contenedores:
        Docker, CRIO, 
        ContainerD, Podman
    ----------------------------
      Sistema Operativo LINUX
    ----------------------------
               HIERRO


# Contenedores

Entorno aislado (dentro de un máquina que corra un kernel Linux) donde ejecutar procesos.
Aislado, en cuanto a qué?
- Tiene su propio sistema de archivos (filesystem)
- Tiene su propia configuración de red -> Su propia IP
- Tiene sus propias variables de entorno
- Puede tener limitaciones de acceso a los recursos del hierro (HW: CPU, RAM...)

Los contenedores los creamos desde imágenes de contenedor.

## Imagen de contenedor

Es un triste fichero comprimido: .tar que contiene:
 - Una estructura de carpetas compatible con POSIX ( /bin /etc /tmp /home)
 - Unos progrmas YA INSTALADOS DE ANTEMANO Y PRECONFIGURADOS POR ALGUIEN
 - Con sus dependencias
 - Y otros programas que puedan ser de utilidad.

Las encontramos en registros de repositorios de imagene sde contenedor: docker hub... quay.io (redhat)

##€ Redireccion de puertos
    
        ------------------------ red amazon ---------------------------------------
        |                                                                       |
        172.31.34.165       :8888 -> 172.17.0.2:80                        ???????????
        |                                                                       |
        IvanPC- 172.17.0.1 ------------+--------- red docker                MenchuPC
        |                              | 
        127.0.0.1                      +- 172.17.0.2   minginx: 80
        
## Usos de los volumenes en contenedores:

- Inyectar archivos al contenedor (configuraciónes, datos)
- Persistir datos(archivos) del contenedor tras su eliminación
- Compartir datos(archivos) entre contenedores



Contenedor: Apache  /var/logs/apache ----> RAM ( 100Kbs )
        vv                                  |
    access.log (rotado 2 ficheros de 50Kbs) |
        ^^                                  |
Contenedor: Filebeat /datos < --------------+


Docker es una herramienta para jugar... no es apta para entorno de producción.

Los entornos de producción:
- HA                \
- Escalabilidad     / REDUNDANCIA: Al menos 2 hosts

Docker es una herrameinta que trabaja en 1 host (menos ese enjendro que existe en docker llamado swarm... que no usa ni el tato)

Cuando quiero trabajar con varios nodos: KUBERNETES
    
    Kubernetes < Apache+Filebeat
    Maquina 1
        crio/containerd
    Maquina 2
        crio/containerd
            - Apache+Filebeat

Kubernetes es un gestor de gestores de contenedores apto para entornos de producción: 
    - Balanceadores de carga    SERVICE
    - Proxy reversos            INGRESS CONTROLLER
    - Regras de firewall        NETWORK POLICY
    - Escalabilidad             HPA
    
## Formación del cluster

Cuando se forma un cluster de ES, los nodos se tienen que empezar a comunicar entre si.
En las primeras versiones de ES, se hacian comunicaciones MULTICAST: Cad nodo daba un grito al aire en la red

Esto se cambió... y hoy en día estas comuniociones se hacen UNICAST:
Yo nodo X voy a buscar a un Nodo Y para ver si le encuentro y hacerme amigo de él
    
    Nodo01 -> Nodo02, Nodo03
    Nodo02 -> Nodo03            Por si aca, el Node01 no levanta. REDUNDANTE
    Nodo03
    
    Esto sería una configuración más que suficiente en cualquier escenario para dyscovery nodes

    
    master01
      - cluster.initial_master_nodes=master01, master02
      - discovery.seed_hosts=master02
      - node.roles=[master]
    master02
      - cluster.initial_master_nodes=master01, master02
      - discovery.seed_hosts=master01
      - node.roles=[master]
    data01
      - discovery.seed_hosts=master01, master02
      - node.roles=[data,master,voting-only] # Maestro de mentirijilla
    data02
      - discovery.seed_hosts=master01, master02
      - node.roles=[data]
    data03
      - discovery.seed_hosts=master01, master02
      - node.roles=[data]
    coordinator01
      - discovery.seed_hosts=master01, master02
      - node.roles=[]
    coordinator02
      - discovery.seed_hosts=master01, master02
      - node.roles=[]
    
    
    