# Mecanismo de sincronización de datos en el cluster

## Colección oplog

El oplog es una colección en la que se registran las operaciones del cluster, siendo la
fuente de sincronización de los secundarios.

- Es una colección limitada en tamaño (capped) y que no se puede modificar.
- Todas las operaciones registradas en el oplog son convertidas en idempotentes.

## Write concern

Write concern es un sistema de reconocimiento de escritura de los miembros de un replica set
que garantiza que una operación de escritura se registre en un determinado número de miembros.

- Nivel global
- Nivel colección
- Nivel operación

Sintaxis (a nivel operación)

writeConcern: {
    w: <entero | 'majority'>, número de miembros que reconocen la escritura
    j: <boolean>, si el reconocimiento implica cuando se escribe en el journal
    wtimeout: <entero-milisegundos> establece el timeout de las operaciones
}

w:  Numero de miembros en los que se tendrá que persistir la operación de escritura para
    que el cluster devuelva ok al servidor de aplicación.
        -majority (menor de los dos siguientes valores)    . Mayoría de los miembros con voto i/arbitro
                                                           . Mayoría de todos los miembros con datos y voto

Desde MongoDB 5 el valor global por defecto en los cluster será majority

## Rollback ¡Ojo certificación!

Con el reconocimiento de escritura en majority evitamos la aparición del mecanismo de Rollback.

Si en un primario que queda fuera del cluster debido a una partición de red se producen operaciones de escritura 
como, durante el proceso de elecciones del nuevo primario, este cluster no puede enviar esas operaciones, al no sincronizarse
en el nuevo primario y secundarios, cuando el antiguo cluster se incorpora de nuevo, esas operaciones se deshacen (rollback) para
evitar inconsistencias con las registradas en el primario actual y secundarios.

Esas operaciones que se deshacen se envian al directorio en un directorio que incluye 'rollback'. Una vez que ha desehecho esas
operaciones, se sincroniza con el primario actual y posteriormente pasará de nuevo a ser el primario o se mantendrá como
secundario de acuerdo a su configuración (priority).

El rollback se evitará con writeConcern majority.