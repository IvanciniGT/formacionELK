input: beats
output: logstash1
output: logstash2   input: logstash / output: es
LogstashRouter  >   LogstashNegocio
                        Manda los datos buenos a un indice
                    LogstashSistemas (input: logstash / output: es)
                        Manda todos los datos (sin geoposicionar) a otro indice
                        
                        
                        
Nuestro proceso de logstash recibia datos de 
    input:  beat: 5044
    output: es
    
    
-----

# En kubernetes
POD1: 
    apache
        \
        Compartan almacenamiento en RAM
        /
    filebeat (Contenedor sidecar)
POD2:
    logstash-routing
POD3:
    logstash-negocio
POD4:
    logstash-sistemas

En Kubernetes NO SE CONFIGURAN CONTENEDORES.
En kubernetes se trabajan con PODs:
    Un pod es un conjunto de contenedores (puede ser solo 1), que:
        - Se despliegan en el mismo host
        - Y escalan juntos
        

Docker < --- Podman (REDHAT) (Pod manager)