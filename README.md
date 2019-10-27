# Práctica BigData Processing
Ignacio Gallegos Sánchez

## Enunciado

El enunciado de la práctica puede descargarse de [aquí](./resources/EspiasBigData.pdf) en formato PDF.

## Parte obligatoria (Spark Streaming)

El enunciado de la parte de streaming solicita obtener los datos de unos archivos CSV con los datos provenientes de los dispositivos IOT.

Para hacerlo más realista, me he tomado la libertad de **obtener, encriptar y encolar** los mensajes desde Twitter a Kafka directamente. Las demás partes del ejercicio son exactamente como se solicitaban en el enunciado (con las peculiaridades propias de la importación de mensajes de twitter, por ejemplo que en vez de disponer del ID de un dispositivo IOT, tendremos un nombre de usuario.

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
 * Cada mensaje es **encriptado** con la clave "sEcReTcL0acaland"
 * Se **envían a Kafka** al topic "keepcoding"

Al ejecutarlo veremos en el consumer que dejamos ejecutando en el paso anterior ha empezado a mostrar mensajes encriptados como los siguientes:

<center><img src="./images/consola-encriptado.png" alt="drawing" width="750"/></center>



Nota: Para que funcione correctamente debemos dar de alta nuestra cuenta de Twitter como desarrollador (en https://dev.twitter.com/apps/new), dar de alta una nueva aplicación, y rellenar los datos en el archivo "TwitterCredentials.scala":

'''scala
package twitterproducer

// Twitter client App credentials
object TwitterCredentials {
  val CONSUMERKEY     = "cWJg5FzMPnrt5cokzlZsoyrGh"
  val CONSUMERSECRET  = "oVBG2zXk4Ief0i2KJCgxynV8irrGeGvCPcXwmKdKbb1Elvm6Mw"
  val APITOKEN        = "3405730379-OLIkayvQ5JupUoEaYP0UQgNsQNWUrj2cevH8oGp"
  val APITOKENSECRET  = "W8r9IeGMJiGtoFrXlNT798TwBou1NUhfCwzC5NUQX7VwO"
}
'''




### Sniffer 

## Parte opcional (GraphX)

<No implementada>
