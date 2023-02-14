# Ejercicio 7

## Explicad en qué consiste el sharding en MongoDB. Intentad montarlo

### Definición y ventajas del Sharding

El sharding en MongoDB es un método para distribuir la carga de datos y la capacidad de procesamiento en múltiples nodos. El sharding es útil cuando una sola instancia de MongoDB no puede manejar altas cargas de trabajo, ya que con este método se puede escalar horizontalmente.

Dicho de otra forma, el Sharding ofrece que la capacidad de procesamiento y almacenamiento de una aplicación funcione manteniendo un alto nivel de rendimiento y disponibilidad. Esto lo hace distribuyendo la carga de datos y la capacidad de procesamiento a través de múltiples máquinas.

En MongoDB, el Sharding se basa en el concepto de fragmentación de los datos en "particiones" o fragmentos más pequeños, conocidos como chunks. Un fragmento o chunk es un conjunto de réplicas que contiene un subconjunto de los datos del clúster.

Cada chunk es almacenado y gestionado por un servidor distinto en la infraestructura de Sharding. La fragmentación se basa en una clave de fragmento, que se utiliza para determinar en qué servidor se almacenará cada chunk, de acuerdo con los rangos de fragmentos y la cantidad de fragmentos.

La clave de fragmento tiene un impacto directo en el rendimiento del clúster y debe elegirse con cuidado. Una clave de fragmento subóptima puede provocar problemas de rendimiento o de escalado debido a una distribución desigual de los fragmentos.

Existen dos estrategias de fragmentación para distribuir datos entre clústeres fragmentados:

- Fragmentación por rangos **(Ranged Sharding)**: divide los datos en rangos en función de los valores de la clave de fragmentación. Luego, a cada fragmento se le asigna un rango basado en los valores de la clave de fragmento. Es más probable que un rango de claves de fragmentos cuyos valores sean "cercanos" residan en el mismo fragmento. Esto permite operaciones dirigidas, ya que mongos puede enrutar las operaciones solo a los fragmentos que contienen los datos requeridos.
- Fragmentación hash **(Hashed Sharding)**: implica calcular un hash del valor del campo de la clave de fragmento. Luego, a cada fragmento se le asigna un rango basado en los valores de clave de fragmento con hash. Si bien un rango de claves de fragmento puede estar "cercano", es poco probable que sus valores hash estén en el mismo fragmento. La distribución de datos basada en valores hash facilita una distribución de datos más uniforme, especialmente en conjuntos de datos donde la clave de fragmento cambia de forma monótona. Sin embargo, la fragmentación hash no proporciona operaciones eficientes basadas en rangos.

**Ventajas del Sharding:**

- Mayor rendimiento de lectura/escritura.
- Mayor capacidad de almacenamiento.
- Alta disponibilidad mediante la distribución geográfica de los datos.

### Cómo implementar el Sharding en MongoDB

Para implementar una configuración de sharding en MongoDB, debemos tener tres componentes clave:

- Un servior **mongos**: estos actúan como intermediarios entre las aplicaciones cliente y los servidores de datos. Los servidores mongos gestionan la distribución de los chunks de datos entre los servidores de datos y redirigen las consultas a los servidores apropiados.

- Servidores de datos fragmentados (**shards**): estos son servidores que almacenan los chunks de datos y ejecutan las operaciones CRUD.

- Al menos un **servidor de configuración** de fragmentación: que mantiene información sobre la configuración de sharding, incluyendo la asignación de chunks a servidores de datos.

Una vez realizada la configuración de los nodos, se debe fragmentar una base de datos o colección específica. Esto se hace asignando una clave de sharding a la colección, que se utiliza para fragmentar los datos en chunks. Para especificar la clave de sharding se utiliza la siguiente sintaxis en la línea de comandos:

```bash
sh.enableSharding("database_name")
db.collection_name.ensureIndex( { shard_key: 1 } )

sh.shardCollection("database_name.collection_name", { shard_key: 1 } )
```

**Shard_key** es la clave de sharding que se utilizará para fragmentar los datos.

Una vez shardeada la colección, los chunks de datos se distribuirán automáticamente entre los servidores de datos y las consultas se redirigirán a los servidores apropiados utilizando los servidores mongos.

Veremos más detenidamente cuáles son los pasos a seguir en el siguiente ejemplo de implementación.

### Ejemplo de implementación

En nuestro caso vamos a crear un cluster shardeado (fragmentado) usando Docker-Compose, por lo que el primer paso será instalar el paquete.

```bash
sudo apt install docker-compose
```

Comenzamos el servicio docker:

```bash
sudo systemctl start docker
```

Vamos a descargar los ficheros del siguiente repositorio: [MongoDB Sharding with Docker](https://github.com/kayne87/mongodb-sharding-docker)

Entramos en el repositorio descargado. Si queremos modificar las instancias que se crearán, nombres o eliminar/añadir algún servidor shard o de configuración, podemos ver el fichero docker-compose.yml del repositorio. Para nuestro caso no voy a modificar nada, ya que esto es un ejemplo de creación de configuración de sharding. Así pues, iniciamos docker-compose.

```bash
cd Descargas/mongodb-sharding-docker-master

sudo docker-compose up -d
```

![docker-compose](/img/capturas-arantxa/99.png)
![docker-compose](/img/capturas-arantxa/100.png)

Entramos en el servidor de configuración **mongocfg1** e iniciamos el Replica Set (conjunto de réplicas) mediante `rs.initiate()`. Este comando iniciará la replicación con la configuración predeterminada inferida por el servidor MongoDB, así que si quiero configurar un conjunto de réplicas que consta de varios servidores independientes, tengo que poner algunos datos: como el nombre del Replica Set, el valor configsvr en true y la información de los hosts creados.

```bash
docker exec -it mongocfg1 /bin/bash

mongosh

rs.initiate({_id: "cfg", configsvr: true, members: [{_id: 0, host: "mongocfg1"},{_id: 1, host: "mongocfg2"}, {_id: 2, host : "mongocfg3"}]})
```

Comprobamos el resultado con `rs.status()`. Debería salirnos algo parecido a lo siguiente:

```bash
cfg [direct: primary] test> rs.status()
{
  set: 'cfg',
  date: ISODate("2023-02-14T12:58:11.779Z"),
  myState: 1,
  term: Long("1"),
  syncSourceHost: '',
  syncSourceId: -1,
  configsvr: true,
  heartbeatIntervalMillis: Long("2000"),
  majorityVoteCount: 2,
  writeMajorityCount: 2,
  votingMembersCount: 3,
  writableVotingMembersCount: 3,
  optimes: {
    lastCommittedOpTime: { ts: Timestamp({ t: 1676379491, i: 1 }), t: Long("1") },
    lastCommittedWallTime: ISODate("2023-02-14T12:58:11.204Z"),
    readConcernMajorityOpTime: { ts: Timestamp({ t: 1676379491, i: 1 }), t: Long("1") },
    appliedOpTime: { ts: Timestamp({ t: 1676379491, i: 1 }), t: Long("1") },
    durableOpTime: { ts: Timestamp({ t: 1676379491, i: 1 }), t: Long("1") },
    lastAppliedWallTime: ISODate("2023-02-14T12:58:11.204Z"),
    lastDurableWallTime: ISODate("2023-02-14T12:58:11.204Z")
  },
  lastStableRecoveryTimestamp: Timestamp({ t: 1676379464, i: 1 }),
  electionCandidateMetrics: {
    lastElectionReason: 'electionTimeout',
    lastElectionDate: ISODate("2023-02-14T12:57:54.926Z"),
    electionTerm: Long("1"),
    lastCommittedOpTimeAtElection: { ts: Timestamp({ t: 1676379464, i: 1 }), t: Long("-1") },
    lastSeenOpTimeAtElection: { ts: Timestamp({ t: 1676379464, i: 1 }), t: Long("-1") },
    numVotesNeeded: 2,
    priorityAtElection: 1,
    electionTimeoutMillis: Long("10000"),
    numCatchUpOps: Long("0"),
    newTermStartDate: ISODate("2023-02-14T12:57:55.024Z"),
    wMajorityWriteAvailabilityDate: ISODate("2023-02-14T12:57:56.248Z")
  },
  members: [
    {
      _id: 0,
      name: 'mongocfg1:27017',
      health: 1,
      state: 1,
      stateStr: 'PRIMARY',
      uptime: 566,
      optime: { ts: Timestamp({ t: 1676379491, i: 1 }), t: Long("1") },
      optimeDate: ISODate("2023-02-14T12:58:11.000Z"),
      lastAppliedWallTime: ISODate("2023-02-14T12:58:11.204Z"),
      lastDurableWallTime: ISODate("2023-02-14T12:58:11.204Z"),
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: '',
      electionTime: Timestamp({ t: 1676379474, i: 1 }),
      electionDate: ISODate("2023-02-14T12:57:54.000Z"),
      configVersion: 1,
      configTerm: 1,
      self: true,
      lastHeartbeatMessage: ''
    },
    {
      _id: 1,
      name: 'mongocfg2:27017',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 27,
      optime: { ts: Timestamp({ t: 1676379490, i: 1 }), t: Long("1") },
      optimeDurable: { ts: Timestamp({ t: 1676379490, i: 1 }), t: Long("1") },
      optimeDate: ISODate("2023-02-14T12:58:10.000Z"),
      optimeDurableDate: ISODate("2023-02-14T12:58:10.000Z"),
      lastAppliedWallTime: ISODate("2023-02-14T12:58:11.204Z"),
      lastDurableWallTime: ISODate("2023-02-14T12:58:11.204Z"),
      lastHeartbeat: ISODate("2023-02-14T12:58:10.988Z"),
      lastHeartbeatRecv: ISODate("2023-02-14T12:58:10.495Z"),
      pingMs: Long("0"),
      lastHeartbeatMessage: '',
      syncSourceHost: 'mongocfg1:27017',
      syncSourceId: 0,
      infoMessage: '',
      configVersion: 1,
      configTerm: 1
    },
    {
      _id: 2,
      name: 'mongocfg3:27017',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 27,
      optime: { ts: Timestamp({ t: 1676379490, i: 1 }), t: Long("1") },
      optimeDurable: { ts: Timestamp({ t: 1676379490, i: 1 }), t: Long("1") },
      optimeDate: ISODate("2023-02-14T12:58:10.000Z"),
      optimeDurableDate: ISODate("2023-02-14T12:58:10.000Z"),
      lastAppliedWallTime: ISODate("2023-02-14T12:58:11.204Z"),
      lastDurableWallTime: ISODate("2023-02-14T12:58:11.204Z"),
      lastHeartbeat: ISODate("2023-02-14T12:58:10.988Z"),
      lastHeartbeatRecv: ISODate("2023-02-14T12:58:10.497Z"),
      pingMs: Long("0"),
      lastHeartbeatMessage: '',
      syncSourceHost: 'mongocfg1:27017',
      syncSourceId: 0,
      infoMessage: '',
      configVersion: 1,
      configTerm: 1
    }
  ],
  ok: 1,
  lastCommittedOpTime: Timestamp({ t: 1676379491, i: 1 }),
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1676379491, i: 1 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1676379491, i: 1 })
}
```

Salimos del servidor mongocfg1 y entramos en **mongoshard11** para iniciar el clúster del conjunto de réplicas para el primer fragmento (shard1).

```bash
exit 
exit

docker exec -it mongoshard11 /bin/bash

mongosh

rs.initiate({_id : "shard1", members: [{ _id : 0, host : "mongoshard11" },{ _id : 1, host : "mongoshard12" },{ _id : 2, host : "mongoshard13" }]})
```

Comprobamos el resultado:

```bash
shard1 [direct: secondary] test> rs.status()
{
  set: 'shard1',
  date: ISODate("2023-02-14T13:01:15.130Z"),
  myState: 1,
  term: Long("1"),
  syncSourceHost: '',
  syncSourceId: -1,
  heartbeatIntervalMillis: Long("2000"),
  majorityVoteCount: 2,
  writeMajorityCount: 2,
  votingMembersCount: 3,
  writableVotingMembersCount: 3,
  optimes: {
    lastCommittedOpTime: { ts: Timestamp({ t: 1676379667, i: 6 }), t: Long("1") },
    lastCommittedWallTime: ISODate("2023-02-14T13:01:07.258Z"),
    readConcernMajorityOpTime: { ts: Timestamp({ t: 1676379667, i: 6 }), t: Long("1") },
    appliedOpTime: { ts: Timestamp({ t: 1676379667, i: 6 }), t: Long("1") },
    durableOpTime: { ts: Timestamp({ t: 1676379667, i: 6 }), t: Long("1") },
    lastAppliedWallTime: ISODate("2023-02-14T13:01:07.258Z"),
    lastDurableWallTime: ISODate("2023-02-14T13:01:07.258Z")
  },
  lastStableRecoveryTimestamp: Timestamp({ t: 1676379655, i: 1 }),
  electionCandidateMetrics: {
    lastElectionReason: 'electionTimeout',
    lastElectionDate: ISODate("2023-02-14T13:01:05.716Z"),
    electionTerm: Long("1"),
    lastCommittedOpTimeAtElection: { ts: Timestamp({ t: 1676379655, i: 1 }), t: Long("-1") },
    lastSeenOpTimeAtElection: { ts: Timestamp({ t: 1676379655, i: 1 }), t: Long("-1") },
    numVotesNeeded: 2,
    priorityAtElection: 1,
    electionTimeoutMillis: Long("10000"),
    numCatchUpOps: Long("0"),
    newTermStartDate: ISODate("2023-02-14T13:01:05.757Z"),
    wMajorityWriteAvailabilityDate: ISODate("2023-02-14T13:01:07.179Z")
  },
  members: [
    {
      _id: 0,
      name: 'mongoshard11:27017',
      health: 1,
      state: 1,
      stateStr: 'PRIMARY',
      uptime: 750,
      optime: { ts: Timestamp({ t: 1676379667, i: 6 }), t: Long("1") },
      optimeDate: ISODate("2023-02-14T13:01:07.000Z"),
      lastAppliedWallTime: ISODate("2023-02-14T13:01:07.258Z"),
      lastDurableWallTime: ISODate("2023-02-14T13:01:07.258Z"),
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: '',
      electionTime: Timestamp({ t: 1676379665, i: 1 }),
      electionDate: ISODate("2023-02-14T13:01:05.000Z"),
      configVersion: 1,
      configTerm: 1,
      self: true,
      lastHeartbeatMessage: ''
    },
    {
      _id: 1,
      name: 'mongoshard12:27017',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 19,
      optime: { ts: Timestamp({ t: 1676379667, i: 6 }), t: Long("1") },
      optimeDurable: { ts: Timestamp({ t: 1676379667, i: 6 }), t: Long("1") },
      optimeDate: ISODate("2023-02-14T13:01:07.000Z"),
      optimeDurableDate: ISODate("2023-02-14T13:01:07.000Z"),
      lastAppliedWallTime: ISODate("2023-02-14T13:01:07.258Z"),
      lastDurableWallTime: ISODate("2023-02-14T13:01:07.258Z"),
      lastHeartbeat: ISODate("2023-02-14T13:01:13.731Z"),
      lastHeartbeatRecv: ISODate("2023-02-14T13:01:13.243Z"),
      pingMs: Long("0"),
      lastHeartbeatMessage: '',
      syncSourceHost: 'mongoshard11:27017',
      syncSourceId: 0,
      infoMessage: '',
      configVersion: 1,
      configTerm: 1
    },
    {
      _id: 2,
      name: 'mongoshard13:27017',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 19,
      optime: { ts: Timestamp({ t: 1676379667, i: 6 }), t: Long("1") },
      optimeDurable: { ts: Timestamp({ t: 1676379667, i: 6 }), t: Long("1") },
      optimeDate: ISODate("2023-02-14T13:01:07.000Z"),
      optimeDurableDate: ISODate("2023-02-14T13:01:07.000Z"),
      lastAppliedWallTime: ISODate("2023-02-14T13:01:07.258Z"),
      lastDurableWallTime: ISODate("2023-02-14T13:01:07.258Z"),
      lastHeartbeat: ISODate("2023-02-14T13:01:13.731Z"),
      lastHeartbeatRecv: ISODate("2023-02-14T13:01:13.242Z"),
      pingMs: Long("0"),
      lastHeartbeatMessage: '',
      syncSourceHost: 'mongoshard11:27017',
      syncSourceId: 0,
      infoMessage: '',
      configVersion: 1,
      configTerm: 1
    }
  ],
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1676379667, i: 6 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1676379667, i: 6 })
}
```

Salimos y hacemos lo mismo con el segundo shard. Para ello tendremos que entrar en **mongoshard21** e iniciar el conjunto de réplicas de shard2.

```bash
exit
exit

docker exec -it mongoshard21 /bin/bash

mongosh

rs.initiate({_id : "shard2", members: [{ _id : 0, host : "mongoshard21" },{ _id : 1, host : "mongoshard22" },{ _id : 2, host : "mongoshard23" }]})
```

Comprobamos el resultado:

```bash
shard2 [direct: secondary] test> rs.status()
{
  set: 'shard2',
  date: ISODate("2023-02-14T13:02:27.343Z"),
  myState: 1,
  term: Long("1"),
  syncSourceHost: '',
  syncSourceId: -1,
  heartbeatIntervalMillis: Long("2000"),
  majorityVoteCount: 2,
  writeMajorityCount: 2,
  votingMembersCount: 3,
  writableVotingMembersCount: 3,
  optimes: {
    lastCommittedOpTime: { ts: Timestamp({ t: 1676379736, i: 1 }), t: Long("-1") },
    lastCommittedWallTime: ISODate("2023-02-14T13:02:16.836Z"),
    readConcernMajorityOpTime: { ts: Timestamp({ t: 1676379736, i: 1 }), t: Long("-1") },
    appliedOpTime: { ts: Timestamp({ t: 1676379747, i: 7 }), t: Long("1") },
    durableOpTime: { ts: Timestamp({ t: 1676379747, i: 7 }), t: Long("1") },
    lastAppliedWallTime: ISODate("2023-02-14T13:02:27.304Z"),
    lastDurableWallTime: ISODate("2023-02-14T13:02:27.304Z")
  },
  lastStableRecoveryTimestamp: Timestamp({ t: 1676379736, i: 1 }),
  electionCandidateMetrics: {
    lastElectionReason: 'electionTimeout',
    lastElectionDate: ISODate("2023-02-14T13:02:27.197Z"),
    electionTerm: Long("1"),
    lastCommittedOpTimeAtElection: { ts: Timestamp({ t: 1676379736, i: 1 }), t: Long("-1") },
    lastSeenOpTimeAtElection: { ts: Timestamp({ t: 1676379736, i: 1 }), t: Long("-1") },
    numVotesNeeded: 2,
    priorityAtElection: 1,
    electionTimeoutMillis: Long("10000"),
    numCatchUpOps: Long("0"),
    newTermStartDate: ISODate("2023-02-14T13:02:27.274Z")
  },
  members: [
    {
      _id: 0,
      name: 'mongoshard21:27017',
      health: 1,
      state: 1,
      stateStr: 'PRIMARY',
      uptime: 821,
      optime: { ts: Timestamp({ t: 1676379747, i: 7 }), t: Long("1") },
      optimeDate: ISODate("2023-02-14T13:02:27.000Z"),
      lastAppliedWallTime: ISODate("2023-02-14T13:02:27.304Z"),
      lastDurableWallTime: ISODate("2023-02-14T13:02:27.304Z"),
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: '',
      electionTime: Timestamp({ t: 1676379747, i: 1 }),
      electionDate: ISODate("2023-02-14T13:02:27.000Z"),
      configVersion: 1,
      configTerm: 1,
      self: true,
      lastHeartbeatMessage: ''
    },
    {
      _id: 1,
      name: 'mongoshard22:27017',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 10,
      optime: { ts: Timestamp({ t: 1676379736, i: 1 }), t: Long("-1") },
      optimeDurable: { ts: Timestamp({ t: 1676379736, i: 1 }), t: Long("-1") },
      optimeDate: ISODate("2023-02-14T13:02:16.000Z"),
      optimeDurableDate: ISODate("2023-02-14T13:02:16.000Z"),
      lastAppliedWallTime: ISODate("2023-02-14T13:02:16.836Z"),
      lastDurableWallTime: ISODate("2023-02-14T13:02:16.836Z"),
      lastHeartbeat: ISODate("2023-02-14T13:02:27.244Z"),
      lastHeartbeatRecv: ISODate("2023-02-14T13:02:27.249Z"),
      pingMs: Long("0"),
      lastHeartbeatMessage: '',
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: '',
      configVersion: 1,
      configTerm: 0
    },
    {
      _id: 2,
      name: 'mongoshard23:27017',
      health: 1,
      state: 2,
      stateStr: 'SECONDARY',
      uptime: 10,
      optime: { ts: Timestamp({ t: 1676379736, i: 1 }), t: Long("-1") },
      optimeDurable: { ts: Timestamp({ t: 1676379736, i: 1 }), t: Long("-1") },
      optimeDate: ISODate("2023-02-14T13:02:16.000Z"),
      optimeDurableDate: ISODate("2023-02-14T13:02:16.000Z"),
      lastAppliedWallTime: ISODate("2023-02-14T13:02:16.836Z"),
      lastDurableWallTime: ISODate("2023-02-14T13:02:16.836Z"),
      lastHeartbeat: ISODate("2023-02-14T13:02:27.244Z"),
      lastHeartbeatRecv: ISODate("2023-02-14T13:02:27.249Z"),
      pingMs: Long("0"),
      lastHeartbeatMessage: '',
      syncSourceHost: '',
      syncSourceId: -1,
      infoMessage: '',
      configVersion: 1,
      configTerm: 0
    }
  ],
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1676379747, i: 7 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1676379747, i: 7 })
}
```

Salimos y entramos en la instancia **mongos1** para añadir los clústeres fragmentados. Esto lo haremos con `sh.addShard()`.

```bash
exit
exit

docker exec -it mongos1 /bin/bash

mongosh

sh.addShard("shard1/mongoshard11")
sh.addShard("shard2/mongoshard21")
```

![docker-compose](/img/capturas-arantxa/102.png)

Comprobamos los resultados de añadir los fragmentos.

```bash
[direct: mongos] test> sh.status()

shardingVersion
{
  _id: 1,
  minCompatibleVersion: 5,
  currentVersion: 6,
  clusterId: ObjectId("63eb85532608d03f4afe05a8")
}
---
shards
[
  {
    _id: 'shard1',
    host: 'shard1/mongoshard11:27017,mongoshard12:27017,mongoshard13:27017',
    state: 1,
    topologyTime: Timestamp({ t: 1676379868, i: 5 })
  },
  {
    _id: 'shard2',
    host: 'shard2/mongoshard21:27017,mongoshard22:27017,mongoshard23:27017',
    state: 1,
    topologyTime: Timestamp({ t: 1676379880, i: 6 })
  }
]
---
active mongoses
[ { '6.0.4': 1 } ]
---
autosplit
{ 'Currently enabled': 'yes' }
---
balancer
{
  'Currently enabled': 'yes',
  'Failed balancer rounds in last 5 attempts': 0,
  'Currently running': 'no',
  'Migration Results for the last 24 hours': 'No recent migrations'
}
---
databases
[
  {
    database: { _id: 'config', primary: 'config', partitioned: true },
    collections: {}
  }
]
```

#### Sharded database

Ya podremos usar libremente el servidor de consultas mongos para trabajar con bases de datos, documentos y colecciones tal como se haría con una base de datos típica sin fragmentar. Sin embargo, si no se realiza ninguna configuración más, cualquier dato que se agregue al clúster terminará almacenándose solo en el fragmento principal, no se dividirá automáticamente y no experimentará ninguno de los beneficios que proporciona la fragmentación.

Para utilizar el clúster MongoDB fragmentado que hemos creado en todo su potencial, se debe habilitar la fragmentación para una base de datos dentro del clúster. Una base de datos MongoDB solo puede contener colecciones fragmentadas si tiene habilitada la fragmentación.

Para ello vamos a trabajar con la base de datos shardedDB y config desde el servidor mongos1:

```bash
use shardedDB
sh.enableSharding("shardedDB")
```

Al activar la fragmentación debería aparecer lo siguiente:

```bash
{
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1676379934, i: 4 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1676379934, i: 2 })
}
```

Vamos a config y consultamos las bases de datos.

```bash
use config
db.databases.find()
```

Nos debe aparecer algo parecido a lo siguiente:

```bash
[
  {
    _id: 'shardedDB',
    primary: 'shard2',
    partitioned: false,
    version: {
      uuid: new UUID("9926816f-0ed8-428c-a170-cbd14b270c5e"),
      timestamp: Timestamp({ t: 1676379933, i: 2 }),
      lastMod: 1
    }
  }
]
```

Creamos a continuación la colección fragmentada en la base de datos shardedDB. En este caso vamos a utilizar la estrategia de fragmentación mediante hash (Hashed Sharding). La otra forma de hacerlo sería mediante fragmentación por rangos (Ranged Sharding).

```bash
use shardedDB
db.shardedCollection.ensureIndex({_id: "hashed"})
sh.shardCollection("shardedDB.shardedCollection", {"_id": "hashed"})
```

```bash
{
  collectionsharded: 'shardedDB.shardedCollection',
  ok: 1,
  '$clusterTime': {
    clusterTime: Timestamp({ t: 1676380085, i: 37 }),
    signature: {
      hash: Binary(Buffer.from("0000000000000000000000000000000000000000", "hex"), 0),
      keyId: Long("0")
    }
  },
  operationTime: Timestamp({ t: 1676380085, i: 33 })
}
```

Insertamos algún registro a la colección.

```bash
for(var i = 0; i < 1500; i++) db.shardedCollection.insertOne({x: i})
```

```bash
{
  acknowledged: true,
  insertedId: ObjectId("63eb8845b958d280e786eb46")
}
```

Por último comprobamos la distribución de los datos guardados en shard1 y shard2.

```bash
db.shardedCollection.getShardDistribution()
```

El resultado nos muestra que los datos se han fragmentado en ambos shards. Tenemos 4 chunks, dos en cada shard. También vemos en Totals el porcentaje de los datos guardados en cada uno.

```bash
Shard shard2 at shard2/mongoshard21:27017,mongoshard22:27017,mongoshard23:27017
{
  data: '44KiB',
  docs: 1554,
  chunks: 2,
  'estimated data per chunk': '22KiB',
  'estimated docs per chunk': 777
}
---
Shard shard1 at shard1/mongoshard11:27017,mongoshard12:27017,mongoshard13:27017
{
  data: '40KiB',
  docs: 1446,
  chunks: 2,
  'estimated data per chunk': '20KiB',
  'estimated docs per chunk': 723
}
---
Totals
{
  data: '84KiB',
  docs: 3000,
  chunks: 4,
  'Shard shard2': [
    '51.8 % data',
    '51.8 % docs in cluster',
    '29B avg obj size on shard'
  ],
  'Shard shard1': [
    '48.2 % data',
    '48.2 % docs in cluster',
    '29B avg obj size on shard'
  ]
}
```

Si volvemos a hacer sh.status() nos aparecerá la información de la base de datos creada y los datos de las colecciones.

```bash
...
databases
[
  {
    database: { _id: 'config', primary: 'config', partitioned: true },
    collections: {
      'config.system.sessions': {
        shardKey: { _id: 1 },
        unique: false,
        balancing: true,
        chunkMetadata: [ { shard: 'shard1', nChunks: 1024 } ],
        chunks: [
          'too many chunks to print, use verbose if you want to force print'
        ],
        tags: []
      }
    }
  },
  {
    database: {
      _id: 'shardedDB',
      primary: 'shard2',
      partitioned: false,
      version: {
        uuid: new UUID("9926816f-0ed8-428c-a170-cbd14b270c5e"),
        timestamp: Timestamp({ t: 1676379933, i: 2 }),
        lastMod: 1
      }
    },
    collections: {
      'shardedDB.shardedCollection': {
        shardKey: { _id: 'hashed' },
        unique: false,
        balancing: true,
        chunkMetadata: [
          { shard: 'shard1', nChunks: 2 },
          { shard: 'shard2', nChunks: 2 }
        ],
        chunks: [
          { min: { _id: MinKey() }, max: { _id: Long("-4611686018427387902") }, 'on shard': 'shard1', 'last modified': Timestamp({ t: 1, i: 0 }) },
          { min: { _id: Long("-4611686018427387902") }, max: { _id: Long("0") }, 'on shard': 'shard1', 'last modified': Timestamp({ t: 1, i: 1 }) },
          { min: { _id: Long("0") }, max: { _id: Long("4611686018427387902") }, 'on shard': 'shard2', 'last modified': Timestamp({ t: 1, i: 2 }) },
          { min: { _id: Long("4611686018427387902") }, max: { _id: MaxKey() }, 'on shard': 'shard2', 'last modified': Timestamp({ t: 1, i: 3 }) }
        ],
        tags: []
      }
    }
  }
]
```

#### Base de datos con uno de los campos de la colección fragmentado

Vamos a hacerlo con otra base de datos de ejemplo.

Activamos la fragmentación para la base de datos **populations**.

```bash
sh.enableSharding("populations")
```

Creamos la colección **cities** con el campo **country** como su clave de fragmentación.

```bash
sh.shardCollection("populations.cities", { "country": "hashed" })
```

Ahora insertamos varios documentos a la colección cities.

```bash
use populations

db.cities.insertMany([
    {"name": "Seoul", "country": "South Korea", "continent": "Asia", "population": 25.674 },
    {"name": "Mumbai", "country": "India", "continent": "Asia", "population": 19.980 },
    {"name": "Lagos", "country": "Nigeria", "continent": "Africa", "population": 13.463 },
    {"name": "Beijing", "country": "China", "continent": "Asia", "population": 19.618 },
    {"name": "Shanghai", "country": "China", "continent": "Asia", "population": 25.582 },
    {"name": "Osaka", "country": "Japan", "continent": "Asia", "population": 19.281 },
    {"name": "Cairo", "country": "Egypt", "continent": "Africa", "population": 20.076 },
    {"name": "Tokyo", "country": "Japan", "continent": "Asia", "population": 37.400 },
    {"name": "Karachi", "country": "Pakistan", "continent": "Asia", "population": 15.400 },
    {"name": "Dhaka", "country": "Bangladesh", "continent": "Asia", "population": 19.578 },
    {"name": "Rio de Janeiro", "country": "Brazil", "continent": "South America", "population": 13.293 },
    {"name": "São Paulo", "country": "Brazil", "continent": "South America", "population": 21.650 },
    {"name": "Mexico City", "country": "Mexico", "continent": "North America", "population": 21.581 },
    {"name": "Delhi", "country": "India", "continent": "Asia", "population": 28.514 },
    {"name": "Buenos Aires", "country": "Argentina", "continent": "South America", "population": 14.967 },
    {"name": "Kolkata", "country": "India", "continent": "Asia", "population": 14.681 },
    {"name": "New York", "country": "United States", "continent": "North America", "population": 18.819 },
    {"name": "Manila", "country": "Philippines", "continent": "Asia", "population": 13.482 },
    {"name": "Chongqing", "country": "China", "continent": "Asia", "population": 14.838 },
    {"name": "Istanbul", "country": "Turkey", "continent": "Europe", "population": 14.751 }
    ])
```

Comprobamos la distribución de los fragmentos.

```bash
db.cities.getShardDistribution()
```

```bash
Shard shard2 at shard2/mongoshard21:27017,mongoshard22:27017,mongoshard23:27017
{
  data: '943B',
  docs: 9,
  chunks: 2,
  'estimated data per chunk': '471B',
  'estimated docs per chunk': 4
}
---
Shard shard1 at shard1/mongoshard11:27017,mongoshard12:27017,mongoshard13:27017
{
  data: '1KiB',
  docs: 11,
  chunks: 2,
  'estimated data per chunk': '567B',
  'estimated docs per chunk': 5
}
---
Totals
{
  data: '2KiB',
  docs: 20,
  chunks: 4,
  'Shard shard2': [
    '45.4 % data',
    '45 % docs in cluster',
    '104B avg obj size on shard'
  ],
  'Shard shard1': [
    '54.59 % data',
    '55 % docs in cluster',
    '103B avg obj size on shard'
  ]
}
```

!!! note "Más info"
    Para más información y aprender a configurar el Sharding (fragmentación) en MongoDB sin utilizar Docker se puede seguir el siguiente [tutorial](https://www.digitalocean.com/community/tutorials/how-to-use-sharding-in-mongodb).