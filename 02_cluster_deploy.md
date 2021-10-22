# Despliegue de cluster MongoDB (local)

## Cluster de 3 servidores

1.- Crear 3 directorios de datos

server1, server2, server3 (en el usuario)

2.- Levantaremos los 3 servidores

mongod --replSet ayesaCluster --dbpath server1 --port 27101
mongod --replSet ayesaCluster --dbpath server2 --port 27102
mongod --replSet ayesaCluster --dbpath server3 --port 27103

3.- Configuración e inicialización del Replica Set

Conectamos con shell a uno de los miembros

mongo --port 27101

Y utilizamos el método initiate con la conf sobre rs

rs.initiate({
    _id: "ayesaCluster",
    members: [
        {_id: 0, host: "localhost:27101"},
        {_id: 1, host: "localhost:27102"},
        {_id: 2, host: "localhost:27103"},
    ]
})

4.- Para monitorizar

rs.status()

## Estados 

PRIMARY Todas las operaciones del cluster se dirigen al miembro en estado primario
SECONDARY Cada secondary adicional recibe una réplica de cada operación que se ejecuta en el primario (oplog)

¿Cuando se producen cambios de estado?

El mecanismo automatic failover comprueba permanentemente la conexión entre los miembros (ping cada 10s) y si 
detecta que uno de los miembros no está disponible, desencadena un proceso de elecciones.

Elecciones => reasignar el primario y reconfigurar el funcionamiento del cluster con los miembros disponibles.

## Configurar la prioridad de los miembros

rs.reconfig(<configuracion>)

1.- Descargar la configuración.

let configuration = rs.conf() // Devuelve el objeto de configuración del cluster

2.- Cambiar la prioridad (con ese objeto JavaScript)

configuration.members[2].priority = 2 // El valor por defecto es 1

3.- Reconfiguramos el cluster con ese objeto

rs.reconfig(configuration)

## Operaciones permitidas en los miembros

Por defecto, solo están permitidas las operaciones en el primario.

Cambiando la configuración por defecto, se podrán realizar operaciones en los secundarios (para propósitos
dedicados) de lectura, nunca de escritura.

Los permisos de lectura se asignan a nivel de base de datos con el método

use <base-datos>
db.setSecondaryOk()

## Añadir un nuevo miembro al cluster

1.- Crear directorio para el nuevo miembro

mkdir server4

2.- Levantamos servidor con

mongod --replSet ayesaCluster --dbpath server4 --port 27104

3.- Reconfigurar desde el Primario

rs.add({
    host: "localhost:27104",
    priority: 0,
    votes: 0
})

4.- Volvemos a obtener la configuración (seguimos en PRIMARIO)

let configuration = rs.conf()

5.- Modificamos prioridad y votos del nuevo miembro y reconfiguramos

configuration.members[3].priority = 1;
configuration.members[3].votes = 1;

rs.reconfig(configuration)

## Tolerancia a fallos ¡Ojo certificación!

- Para determinar la tolerancia a fallos debemos tener en cuenta que no pueden
existir dos primarios al mismo tiempo.

- Para que se produzcan elecciones y por tanto se elija un nuevo primario, debe haber en los miembros disponibles
mayoría contabilizando para esa mayoría el total de los miembros del cluster incluyendo los caidos o no disponibles.
(los vivos deben ser mayoría simple respecto al total)

Como entonces es necesario tener mayoría para que se produzcan elecciones la tolerancia a fallos aumentará siempre
en los cluster impares

Tabla de tolerancia a fallos

nº Miembros totales         Mayoría (elecciones)    Tolerancia a fallos resultante

        2                           2                       0
        3                           2                       1
        4                           3                       1
        5                           3                       2
        6                           4                       2
        7                           4                       3   
        // Nº maximo de miembros con voto y prioridad (que pueden llegar a ser primarios)
        8
        ...
        50 // Nº máximo de miembros totales

## Añadir un miembro árbitro (sirve para aumentar la tolerancia a fallos con muy pocos recursos)

1.- Crear un directorio

server5

2.- Levantamos

mongod --replSet ayesaCluster --dbpath server5 --port 27105

3.- Añadimos desde el primario como árbitro

rs.addArb("localhost:27105")

El árbitro nunca será primario ni secundario

## Añadir o reconfigurar un miembro como delayed (recuperación de desastres)

El miembro debe ser oculto para que no reciba lecturas y prioridad debe establecerse en 0 para que no pueda llegar a ser primario.

1.- En el primario, recuperamos la conf del cluster

let configuration = rs.conf()

2.- Seteamos los valores del miembro a convertir en delayed

configuration.members[3].priority = 0; 
configuration.members[3].hidden = true; 
configuration.members[3].secondaryDelaySecs = 120; // Entero con los segundos de delay que tendrán las operaciones 

3.- Reconfiguramos

rs.reconfig(configuration)

Practicar en casa, como el delayed no admite lecturas, usar mongoexport.

## Miembros con votos 0

configuration.members[n].votes = 0; // Miembros adicionales al 7

## Retirada de miembros

1.- Apagar el servidor del miembro a retirar

2.- En el primario

rs.remove(<host>)

rs.remove("localhost:27104");
rs.remove("localhost:27105");