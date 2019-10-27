# Práctica BigData Processing
Ignacio Gallegos Sánchez

## Enunciado

El enunciado de la práctica puede descargarse de [aquí](./resources/EspiasBigData.pdf) en formato PDF.

## Spark Streaming (obligatoria)

El enunciado de la parte de streaming solicita obtener los datos de unos archivos CSV con los datos provenientes de los dispositivos IOT.

Para hacerlo más realista, me he tomado la libertad de **obtener, encriptar y encolar** los mensajes desde Twitter a Kafka directamente. De este modo tenemos un flujo continuo de mensajes totalmente aleatorio en tiempo real, viendo así una aplicación directa de Spark Streaming. Las demás partes del ejercicio son exactamente como se solicitaban en el enunciado (con las peculiaridades propias de la importación de mensajes de twitter, por ejemplo que en vez de disponer del ID de un dispositivo IOT, tendremos un nombre de usuario).

A grandes rasgos tendremos una arquitectura como la siguiente:

<center><img src="./images/Kafka-general.png" alt="drawing" width="750"/></center>



### Kafka

Lo primero que haremos será descargar kafka y escribir unos comandos básicos para ejecutarlo en nuestro ordenador.

Primero arrancaremos kafka con la configuración básica:

```bash
./bin/zookeeper-server-start.sh config/zookeeper.properties
./bin/kafka-server-start.sh config/server.properties
```

Una vez arrancado crearemos el **topic** usado en nuestra práctica, en mi caso "**keepcoding**":

```bash
./bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic keepcoding
```

Por último arrancamos un consumer de consola y lo dejamos a la escucha:

```bash
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic keepcoding --from-beginning
```



### Twitter producer

El producer está en el proyecto paralelo de mi repositorio:

https://github.com/ignaciogallegossanchez/TwitterKafkaProducer

Como puede verse en la función principal del código:

```scala
package twitterproducer

object Main {
  def main(args: Array[String]): Unit = {
    println("Twitter Kafka Producer")

    val keepCodingReadWriter = new KeepCodingReadWriter(
      in = new TwitterReader(
        termsToTrack = List("#bigdata", "#keepcoding", "datos", "bigdata", "developer")
      ),
      out = new SparkProducer(
        servers = "localhost:9092",
        defaultTopic = "keepcoding"
      ).addEncryption(
        new AESEncryption(Array[Byte]('s','E','c','R','e','T','c','L','0','a','c','a','l','a','n','d')))
    )

    keepCodingReadWriter.start()
  }
}
```

Lo que hace es:
 * **Recibe los mensajes** de twitter (buscando los términos "#bigdata", "#keepcoding", "datos", "bigdata" y "developer")
 * Cada mensaje es **encriptado** con la clave "sEcReTcL0acaland" con algoritmo **AES-128**
 * Se **envían a Kafka** al topic "keepcoding"

Al ejecutarlo veremos en el consumer que dejamos ejecutando en el paso anterior ha empezado a mostrar mensajes encriptados como los siguientes:

<center><img src="./images/consola-encriptado.png" alt="drawing" width="750"/></center>



_Nota_: Para que funcione correctamente debemos dar de alta nuestra cuenta de Twitter como desarrollador (en https://dev.twitter.com/apps/new), dar de alta una nueva aplicación, y rellenar los datos en el archivo "TwitterCredentials.scala":

```scala
package twitterproducer

// Twitter client App credentials
object TwitterCredentials {
  val CONSUMERKEY     = "cWJg5FzMPnrt5cokzlZsoyrGh"
  val CONSUMERSECRET  = "oVBG2zXk4Ief0i2KJCgxynV8irrGeGvCPcXwmKdKbb1Elvm6Mw"
  val APITOKEN        = "3405730379-OLIkayvQ5JupUoEaYP0UQgNsQNWUrj2cevH8oGp"
  val APITOKENSECRET  = "W8r9IeGMJiGtoFrXlNT798TwBou1NUhfCwzC5NUQX7VwO"
}
```


### Sniffer 

Para esta parte de la práctica, para facilitar la corrección y visualización del ejercicio, he dejado el grueso del código en la función "main" aunque en un entorno de producción no sería lo más recomendable.

El código en: AAAAAAAAQUUUUUUUUUUUIIIIIIIIIIIIII

El sniffer es el encargado de:
 * Desencriptar los mensajes
 * Formatear correctamente los datos recibidos
 * Tratarlos (dividir frases en palabras)
 * Filtrarlos (eliminar las que no tienen valor)
 * Hacer conteo de las más utilizadas en un rango temporal (1h)
 * Cruzar las 10 mas utilizadas con las que están en una lista negra
 * En caso de que el resultado contenga alguna palabra (es decir, de las 10 más utilizadas en la ventana temporal alguna estaba en la lista negra) enviar una notificación.
 
__¿Cómo abordar el ejercicio?__

Dado que en el enunciado se especifica que se debe procesar una ventana de una hora, **mi primera aproximación** fue la de utilizar el soporte de ventanas temporal que Spark nos brinda (usando "window" sobre un timestamp). Algo como lo siguiente:

```scala
.groupBy(window(col("timestamp"), "1 hour")).count()
```

Aunque es perfectamente usable, los **problemas** llegan al intentar quedarnos con los n-primeros elementos, probando con:
 * limit(10)
 * head(10)
 * Generando una columna indice y filtrando por ".where(col(index) < 10)"

En todos los casos obtengo un error del framework Spark indicando que no se soportan esas operaciones en modo streaming.





## GraphX (opcional)

No implementada
