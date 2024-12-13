# README

## ¿Qué es una función hash?
Una función hash es cualquier función que permita asignar a entradas de un tamaño variable una salida de tamaño fijo (esto último es discutible debido a que existen algunas funciones hash de salida variable).

## ¿Qué es una función hash criptográfica?
Son funciones hash destinadas a la criptografía, por lo que tienen que ser seguras de usar, que a grandes rasgos significa que tienen que cumplir las siguientes propiedades.
- **Ser determinista.** Una misma entrada tiene que dar siempre la misma salida.
- **Ser rápida.** Esta propiedad esta relacionada en que, ya sea sobre hardware o software, tiene que tener un rendimiento que permita su uso.
- **Seguir el efecto avalancha.** Los más ligeros cambios en el input generan un output completamente diferente. Idealmente, cada bit del output (o digest) tiene una probabilidad del 0.5 de cambiar cuando cambia un bit de input.
- **Ser resistente al calculo de la primera pre-imagen (funciones One-way).** Dado un hash **h**, tiene que ser complicado encontrar un mensaje **m** tal que **hash(m)=h**.
- **Ser resistente al calculo de la segunda pre-imagen (Weak Collision Resistance).** Dado un input **m**, tiene que ser complicado encontrar otro input **m'** tal que **hash(m)=hash(m')**.
- **Ser resistente a colisiones (Strong Collision Resistance).** Tiene que ser complicado encontrar un par de inputs **m** y **m'** tal que **hash(m)=hash(m')**.

En las últimas 3 propiedades, el que sea complicado de calcular significa que no haya métodos mejores que la fuerza bruta.

## Lista de funciones hash descritas.

|                   Nombre                   | Digest   | Año  |
| :----------------------------------------: | -------- | ---- |
|          [MD4](Algoritmos/MD4.md)          | 128 bits | 1990 |
|          [MD5](Algoritmos/MD5.md)          | 128 bits | 1992 |
|        [SHA-0](Algoritmos/SHA-0.md)        | 160 bits | 1993 |
|        [SHA-1](Algoritmos/SHA-1.md)        | 180 bits | 1995 |
|   [SHA-256 (SHA-2)](Algoritmos/SHA-2.md)   | 256 bits | 2002 |
|   [SHA-384 (SHA-2)](Algoritmos/SHA-2.md)   | 384 bits | 2002 |
|   [SHA-512 (SHA-2)](Algoritmos/SHA-2.md)   | 512 bits | 2002 |
|   [SHA-224 (SHA-2)](Algoritmos/SHA-2.md)   | 224 bits | 2004 |
| [SHA-512/224 (SHA-2)](Algoritmos/SHA-2.md) | 224 bits | 2008 |
| [SHA-512/256 (SHA-2)](Algoritmos/SHA-2.md) | 256 bits | 2008 |

## Conceptos relacionados con las funciones hash.
- [Esquema Merkle-Damgård](Conceptos/Esquema%20Merkle-Damgård.md). Se trata de una estructura que se usa habitualmente para construir funciones hash. Algunas funciones hash que tienen esta estructura son [MD5](Algoritmos/MD5.md), [SHA-1](Algoritmos/SHA-1.md) o [SHA-2](Algoritmos/SHA-2.md)