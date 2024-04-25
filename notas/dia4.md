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