# Open ElasticSearch

Esta imagen Docker nos permite una sencilla consiguración de uno o varios nodos de Elasticsearch. 
## Modo de Uso Básico

Para arrancar un nodo de datos de Elasticsearch y que esté disponible en el puerto estandard de nuestro host:
```sh
$ docker run -d -p 9200:9200 -p 9300:9300 cross/elasticsearch
```
Ahora debería tener habilitado la conexión a Elasticsearh vía HTTP, para confirmar que está arrancado:

[http://localhost:9200](http://localhost:9200/)

```sh
{
  "Name": "682f9c6432e4",
  "Cluster_name": "cross_db",
  "Cluster_uuid": "l16MnaadTQG2scIM-5vksA",
  "versión" : {
    "Número": "5.0.0",
    "Build_hash": "253032b",
    "Fecha_compilación": "2016-10-26T04: 37: 51.531Z",
    "Build_snapshot": false,
    "Lucene_version": "6.2.0"
  },
  "Lema": "Usted sabe, para la búsqueda"
}
```
Donde localhost es el nombre o IP de tu máquina local, es decir, donde está corriendo el servicio Docker.
Para arrancar un cluster multinodo (3 nodos en este ejemplo) simplemente tenemos que hacer lo siguiente:
```sh
docker run -d --name es0 -p 9200:9200                    es
docker run -d --name es1 --link es0 -e UNICAST_HOSTS=es0 es
docker run -d --name es2 --link es0 -e UNICAST_HOSTS=es0 es
```
Para comprobar la correcta salud del cluster, es tan sencillo como:

[http://localhost:9200/_cluster/health?pretty](http://localhost:9200/_cluster/health?pretty)

```sh
{
  "Cluster_name": "cross_db",
  "Status": "amarilla",
  "Timed_out": false,
  "number_of_nodes": 7,
  "number_of_data_nodes": 3,
  "active_primary_shards": 7,
  "active_shards": 20,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 6,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "Number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "Active_shards_percent_as_number": 76.92307692307693
}
```

## Resumen de Configuración
### Puertos

    9200 - HTTP REST
    9300 - Native transport

### Volumenes

    /data - localización de path.data
    /conf - localizacion de path.conf

## Detalles de Configuración
Las siguientes opciones de configuración se especifican utilizando variables de entorno de ejecución (-e) como por ejemplos

    docker run ... -e NAME=VALUE ... cross/elasticsearch

Dado que las configuraciones -e de Docker se incorporan a la definición del contenedor, esta imagen proporciona una función adicional para cambiar cualquiera de las configuraciones para un contenedor existente.

Las opciones de configuración que no se admiten explícitamente se pueden especificar a través de la variable de entorno OPTS. Por ejemplo, por defecto la variable de entorno OPTS se establece con:

    OPTS=-Dnetwork.bind_host=_non_loopback_

>NOTA: Esta opción es un valor predeterminado ya que bind_host tiene como valor por >defecto localhost a partir de 2.0, lo que no es útil para puertos de mapeo fuera del >contenedor.

### Cluster Name

Si se une a un clúster preexistente, es posible que deba especificar un nombre de clúster distinto del predeterminado **elasticsearch**:

    -e CLUSTER=dockers

### Zen Unicast Hosts

Al unirse a un clúster multi-físico-host, la multidifusión puede no ser compatible con la red física. En ese caso, su nodo puede hacer referencia a uno o más hosts específicos en el clúster a través de la capacidad de **Zen Unicast Host** como una lista separada por comas de pares [HOST:PORT]:

    -e UNICAST_HOSTS=HOST:PORT[,HOST:PORT]

como esto

    -e UNICAST_HOSTS=192.168.0.100:9300
