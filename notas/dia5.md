
    
                                        Cluster ES
                            
                            master1 ------------------- master2
                                |                           |
                            data1*--------------------- data2 ------------------- data3
                                |                           |
                            coordinator1 -------------- coordinator2
                                |                           |
                                --------------------------------------------------
                                    ^                   ^                   ^
                                  kibana            logstash-negocio    logstash-sistemas
                                                        |                   |
                                                        ---------------------
                                                                 ^
                                                            logstash-routing
                                                                v
                                                            kafka1-kafka2-kafka3
                                                                ^           ^
                                                            filebeat     filebeat + apache   
                                                                v
                                                                access_log
                                                                ^
                                                            apache
                                                            
# Kibana

En kibana se maneja el concepto de data view... UNA COLECCION DE INDICES, deben compartir un campo de tipo @timestamp

- Unas pantallas que nos permitirán buscar documentos en bruto
- Dashboards <<<
- Canvas


UBER

GPS -> json -> logstash. Cada coche en flota tiene un ID
Cargar documentos con un mismo ID en versiones

500 vehiculos en flota... cada uno con su id... quiero la ultima version de cada doucmento pintada en el mapa, con el icono de un coche

---

IA                      Programas que simulan comportamiento humano.
    Hoy en día nos referimos a programas que simulan comportamientos bastante complejos.

Machine Learning
    
    Cuando dejo a una computadora que escriba un programa que simule un comportamiento humano.
    Lo único que le doy es el tipo de algoritmo matemático que quiero que utilice:
        - Redes neuronales
        - Arboles de decisión
        - Regresiones logísticas
        Identificar un objeto en una foto
    DeepLearning... cuando se complican mucho los calculos... y necesitamos una GPU
    
DataMining
    Partiendo de una colección de datos, intenta identificar relaciones entre ellos. Qué relaciones? NPI. Búscate la vida.
        
        GET/POST/PUT
        URLS (400)
        BYTES
        CODIGO RESPUESTA
        HORA DIA
        DIA DE LA SEMANA
        MES
        TIPO MAQUINA
        PAIS
        Y tengo 100M de datos.
    
    Categorías: DATA MINING
        A lo mejor tengo muchos grupos que contiene mensajes de respuesta 200, a una hora del día
        A lo mejor tengo un grupo de mensajes 500 (error del servidor) a las 00:00 todos los días / Actualización/backup
    Identifique ANOMALIAS:
        Mensajes que no encajan muy bien con esas categorias que ha identificado a priori
        
    Mensaje de tipo 500 a las 15:00 tarde -> ANOMALIA (PREDICTIVO) : MACHINE LEARNING
---


Cuanto me va a costar un portatil: Asus: 15', procesador i5, 8Gbs Ram... sin gráfica : 500-600€. 400-500€.  ESTIMACION/PREDICCION
    Os aprovechais de vuestra expediencia (datos antiguos) para extrapolarlos y obtener estimaciones a futuro.
computadora, te doy datos de 10000 ordenadores... con sus características... y su precio.
Y ahora, te doy solo un conjunto de caracteríticas ... y estima el precio... en base a todos los datos que te di antes.

Como se pronuncia una frase escrita en español.
Traducciones
Chatgpt

----
fluentd

---


Indice
- 2 Lucenes primarios


^^^^

LUCENE 1
    -------
    tortilla 10 15, 20, 35, 094949459549
    patatas 17,15, 1209

LUCENE 2
    -------
    tortilla 200, 203, 187
    patatas 17,15, 1209

---

Quiero monitorizar si un apache está levantado:
# Una instalación del programa HEARTBEAT (contenedor)
# Un fichero de configuración de HEARTBEAT (qué debe monitorizar (petición http apache))
# Alguien que reciba los datos del HEARTBEAT:
    - ES
    - Logstash <<<<
    - KAFKA
# Una instalación de logstash
# Una configuración para logstash


---

Al trabajar con contenedores, descargo una imagen de contenedor:
Con esa imagen (es un archivo tar descomprimido) 
Genero todos los contenedores que me de la gana... QUE VAN A COMPARTIR LA IMAGEN.
arranco todos los procesos que quiero dentro.

Los contenedores tienen su propio sistema de archivos. Va por capas
CAPA BASE: Capa de la imagen.. Esta capa es readonly. No es alterable.
Cualquier cambio (alta de un archivo, modificación o eliminación) sobre los archivos base, se hace en otra capa.
Lo que veo dentro de un contenedor como su fs es la superposición de todas las capas

La capa base, se comparte entre los contenedores. (ESTO NO OCURRE CON LAS VM)

Es decir, que con una única imagen, atiendo a 50 contenedores.
Cada contenedor ocupará solo en disco, lo que ocupen los cambios que haga sobre el sistema de archivos de la imagen.

---