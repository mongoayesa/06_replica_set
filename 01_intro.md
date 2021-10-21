# Intro a Replica Set

- Un replica set ó cluster es un grupo de servidores que mantienen el mismo set de datos sincronizado
para:

    - ALTA DISPONIBILIDAD
    - Incremento de la capacidad de lectura
    - Copias adicionales de los datos para propósitos dedicados
        - reporting
        - recuparación de desastre (miembro delayed)
        - backup
        - ...

- ¿Es el replica set un sistema de escalado horizontal?

En principio no, porque las operaciones de escritura solo se producen en uno de los miembros.


