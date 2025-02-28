# Padding

El **padding** es la extensión que se añade a los textos planos para que puedan ser divididos y operados en tamaños concretos.


## Algunos tipos de padding

### PKCS#5
Cada byte del padding indica el tamaño del último bloque (en bytes).

``... | DD DD DD DD DD DD DD DD | DD DD DD 05 05 05 05 05 |``

En caso de que el último bloque no requiera padding, se añade un bloque entero indicándolo.

``... | DD DD DD DD DD DD DD DD | 08 08 08 08 08 08 08 08 |``

### ANSI X9.23
El último byte indica el tamaño del último bloque (en bytes) y el resto tienen un valor aleatorio (aunque también pueden ser 0).

``... | DD DD DD DD DD DD DD DD | DD DD DD 00 00 00 00 05 |``

``... | DD DD DD DD DD DD DD DD | 00 00 00 00 00 00 00 08 |``


### ISO 10126 y W3C
El último byte indica el tamaño del padding (en bytes), mientras que el resto son números aleatorios.

``... | DD DD DD DD DD DD DD DD | DD DD DD XX XX XX XX 05 |``

``... | DD DD DD DD DD DD DD DD | XX XX XX XX XX XX XX 08 |``
