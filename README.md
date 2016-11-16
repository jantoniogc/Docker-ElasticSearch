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

### Plugins

Puede instalar uno o más plugins antes de iniciar compartiendo una lista de complementos separados por comas.

-e PLUGINS=ID[,ID]

In this example, it will install the Marvel plugin

-e PLUGINS=elasticsearch/marvel/latest

Many more plugins are available here.
Publish As

Since the container gives the Elasticsearch software an isolated perspective of its networking, it will most likely advertise its published address with a container-internal IP address. This can be overridden with a physical networking name and port using:

-e PUBLISH_AS=DOCKERHOST:9301

Author Note: I have yet to hit a case where this was actually necessary. Other
than the cosmetic weirdness in the logs, Elasticsearch seems to be quite tolerant.
Node Name

Rather than use the randomly assigned node name, you can indicate a specific one using:

-e NODE_NAME=Docker

Node Type

If you refer to the Node section
of the Elasticsearch reference guide, you'll find that there's three main types of nodes: master-eligible, data, and client.

In larger clusters it is important to dedicate a small number (>= 3) of master nodes. There are also cases where a large cluster may need dedicated gateway nodes that are neither master nor data nodes and purely operate as "smart routers" and have large amounts of CPU and memory to handle client requests and search-reduce.

To simplify all that, this image provides a TYPE variable to let you amongst these combinations. The choices are:

    (not set, the default) : the default node type which is both master-eligible and a data node
    MASTER : master-eligible, but holds no data. It is good to have three or more of these in a
    large cluster
    DATA (or NON_MASTER) : holds data and serves search/index requests. Scale these out for elastic-y goodness.
    GATEWAY : only operates as a client node or a "smart router". These are the ones whose HTTP port 9200 will need to be exposed

A Docker Compose file will serve as a good example of these three node types:

version: '2'

services:
  gateway:
    image: itzg/elasticsearch
    environment:
      UNICAST_HOSTS: master
      TYPE: GATEWAY
    ports:
      - "9200:9200"

  master:
    image: itzg/elasticsearch
    environment:
      UNICAST_HOSTS: gateway
      TYPE: MASTER
      MIN_MASTERS: 2

  data:
    image: itzg/elasticsearch
    environment:
      UNICAST_HOSTS: master,gateway
      TYPE: DATA

Minimum Master Nodes

In combination with the TYPE variable above, you will also want to configure the minimum master nodes to avoid split-brain during network outages.

The minimum, which can be calculated as (master_eligible_nodes / 2) + 1, can be set with the MIN_MASTERS variable.

Using the Docker Compose file above, a value of 2 is appropriate when scaling the cluster to 3 master nodes:

docker-compose scale master=3

Auto transport/http discovery with Swarm Mode

When using Docker Swarm mode (starting with 1.12), the overlay and ingress network interfaces are assigned
multiple IP addresses. As a result, it creates confusion for the transport publish logic even when using
the special value _eth0_.

To resolve this, add

-e DISCOVER_TRANSPORT_IP=eth0

replacing eth0 with another interface within the container, if needed.

The same can be done for publish/binding of the http module by adding:

-e DISCOVERY_HTTP_IP=eth2


