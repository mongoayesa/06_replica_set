# Despliegue de cluster MongoDB (local)

## Cluster de 3 servidores

1.- Crear 3 directorios de datos

server1, server2, server3 (en el usuario)

2.- Levantaremos los 3 servidores

mongod --replSet ayesaCluster --dbpath server1 --port 27101
mongod --replSet ayesaCluster --dbpath server2 --port 27102
mongod --replSet ayesaCluster --dbpath server3 --port 27103