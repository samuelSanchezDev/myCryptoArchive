# Modos de operación

Un modo de operación es un conjunto de técnicas que adapta un algoritmo de cifrado por bloques para proporcionar seguridad a datos más allá de un solo bloque. Los algoritmos de cifrado por bloques garantizan la confidencialidad de los bloques individuales, pero los modos de operación aseguran su protección como un todo.

## Solo cifrado
Estos modos garantizan la confidencialidad, pero no verifican la integridad de los datos.

| Modo                                     | Paralelizable        | Acceso aleatorio | Tamaños de Bloque |
| ---------------------------------------- | -------------------- | ---------------- | ----------------- |
| Electronic Code-Book (ECB)               | Si                   | Si               | Fijo              |
| Cipher-Block Chaining (CBC)              | Parcial (descifrado) | Si               | Fijo              |
| Propagating Cipher-Block Chaining (PCBC) | No                   | No               | Fijo              |
| Cipher FeedBack (CFB)                    | Parcial (descifrado) | Si               | Variable          |
| Output FeedBack (OFB)                    | No                   | No               | Variable          |
| Counter (CTR)                            | Si                   | Si               | Variable          |

### Electronic Code-Book (ECB)
ECB cifra cada bloque de forma independiente. Esto lo hace rápido y fácil de implementar, pero vulnerable a ciertos tipos de ataques, ya que no oculta patrones dentro de los datos cifrados.

**Cifrado:** Cada bloque se cifra independiente del resto.
```
┌─────────────────────────────────────────────────────────────────┐
│        Bloque Plano #1     Bloque Plano #2     Bloque Plano #3  │
│         █████████████       █████████████       █████████████   │
│          ╔════╩═══╗          ╔════╩═══╗          ╔════╩═══╗     │
│ Clave ═══╣ Cifrar ║ Clave ═══╣ Cifrar ║ Clave ═══╣ Cifrar ║     │
│          ╚════╦═══╝          ╚════╦═══╝          ╚════╦═══╝     │
│         ░░░░░░░░░░░░░       ░░░░░░░░░░░░░       ░░░░░░░░░░░░░   │
│       Bloque Cifrado #1   Bloque Cifrado #2   Bloque Cifrado #3 │
└─────────────────────────────────────────────────────────────────┘
```
**Descifrado:** Cada bloque se descifra independiente del resto.
```
┌────────────────────────────────────────────────────────────────────────┐
│        Bloque Cifrado #1      Bloque Cifrado #2      Bloque Cifrado #3 │
│          ░░░░░░░░░░░░░          ░░░░░░░░░░░░░          ░░░░░░░░░░░░░   │
│          ╔═════╩═════╗          ╔═════╩═════╗          ╔═════╩═════╗   │
│ Clave ═══╣ Descifrar ║ Clave ═══╣ Descifrar ║ Clave ═══╣ Descifrar ║   │
│          ╚═════╦═════╝          ╚═════╦═════╝          ╚═════╦═════╝   │
│          █████████████          █████████████          █████████████   │
│         Bloque Plano #1        Bloque Plano #2        Bloque Plano #3  │
└────────────────────────────────────────────────────────────────────────┘
```
### Cipher-Block Chaining (CBC)
CBC mezcla cada bloque plano con el bloque cifrado anterior y después es procesado por el algoritmo. Esto oculta los patrones del archivo, aunque no es completamente paralelizable.

**Cifrado:** El primer bloque se mezcla con un vector de inicialización (IV). Los bloques posteriores dependen del bloque cifrado previo.
```
┌──────────────────────────────────────────────────────────────────────────────┐
│                 Bloque Plano #1       Bloque Plano #2       Bloque Plano #3  │
│                  █████████████         █████████████         █████████████   │
│                     ╔══╩══╗               ╔══╩══╗               ╔══╩══╗      │
│ IV ■■■■■■■■■■■■■ ═══╣ XOR ║  ╔════════════╣ XOR ║  ╔════════════╣ XOR ║      │
│                     ╚══╦══╝  ║            ╚══╦══╝  ║            ╚══╦══╝      │
│                   ╔════╩═══╗ ║          ╔════╩═══╗ ║          ╔════╩═══╗     │
│          Clave ═══╣ Cifrar ║ ║ Clave ═══╣ Cifrar ║ ║ Clave ═══╣ Cifrar ║     │
│                   ╚════╦═══╝ ║          ╚════╦═══╝ ║          ╚════╦═══╝     │
│                        ╠═════╝               ╠═════╝               ╠═════... │
│                  ░░░░░░░░░░░░░         ░░░░░░░░░░░░░         ░░░░░░░░░░░░░   │
│                Bloque Cifrado #1     Bloque Cifrado #2     Bloque Cifrado #3 │
└──────────────────────────────────────────────────────────────────────────────┘
```
**Descifrado:** Para descifrar, cada bloque cifrado pasa primero por el algoritmo de descifrado y luego se mezcla con el bloque cifrado anterior. El primer bloque se mezcla con un vector de inicialización (IV).
```
┌────────────────────────────────────────────────────────────────────────────────────┐
│                Bloque Cifrado #1        Bloque Cifrado #2        Bloque Cifrado #3 │
│                  ░░░░░░░░░░░░░            ░░░░░░░░░░░░░            ░░░░░░░░░░░░░   │
│                        ╠═══════╗                ╠═══════╗                ╠═════... │
│                  ╔═════╩═════╗ ║          ╔═════╩═════╗ ║          ╔═════╩═════╗   │
│         Clave ═══╣ Descifrar ║ ║ Clave ═══╣ Descifrar ║ ║ Clave ═══╣ Descifrar ║   │
│                  ╚═════╦═════╝ ║          ╚═════╦═════╝ ║          ╚═════╦═════╝   │
│                     ╔══╩══╗    ║             ╔══╩══╗    ║             ╔══╩══╗      │
│ IV ■■■■■■■■■■■■■ ═══╣ XOR ║    ╚═════════════╣ XOR ║    ╚═════════════╣ XOR ║      │
│                     ╚══╦══╝                  ╚══╦══╝                  ╚══╦══╝      │
│                  █████████████            █████████████            █████████████   │
│                 Bloque Plano #1          Bloque Plano #2          Bloque Plano #3  │
└────────────────────────────────────────────────────────────────────────────────────┘
```
### Propagating Cipher-Block Chaining (PCBC)
PCBC es similar al CBC, pero mezcla el bloque plano con una combinación del bloque plano y cifrado previo, y después es procesado por el algoritmo.

**Cifrado:** El primer bloque se mezcla con un vector de inicialización (IV). Los bloques posteriores dependen de los bloques cifrado y plano previos.
```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                 Bloque Plano #1         Bloque Plano #2         Bloque Plano #3         │
│                  █████████████           █████████████           █████████████          │
│                        ╠═══════╗               ╠═══════╗               ╠═══════╗        │
│                     ╔══╩══╗ ╔══╩══╗         ╔══╩══╗ ╔══╩══╗         ╔══╩══╗ ╔══╩══╗     │
│ IV ■■■■■■■■■■■■■ ═══╣ XOR ║ ║ XOR ╠═════════╣ XOR ║ ║ XOR ╠═════════╣ XOR ║ ║ XOR ╠═... │
│                     ╚══╦══╝ ╚══╦══╝         ╚══╦══╝ ╚══╦══╝         ╚══╦══╝ ╚══╦══╝     │
│                   ╔════╩═══╗   ║          ╔════╩═══╗   ║          ╔════╩═══╗   ║        │
│          Clave ═══╣ Cifrar ║   ║ Clave ═══╣ Cifrar ║   ║ Clave ═══╣ Cifrar ║   ║        │
│                   ╚════╦═══╝   ║          ╚════╦═══╝   ║          ╚════╦═══╝   ║        │
│                        ╠═══════╝               ╠═══════╝               ╠═══════╝        │
│                  ░░░░░░░░░░░░░           ░░░░░░░░░░░░░           ░░░░░░░░░░░░░          │
│                Bloque Cifrado #1       Bloque Cifrado #2       Bloque Cifrado #3        │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```
**Descifrado:** Para descifrar, cada bloque cifrado pasa primero por el algoritmo de descifrado y luego se mezcla con los bloques cifrado y plano previos.
```
┌────────────────────────────────────────────────────────────────────────────────────────────┐
│                 Bloque Cifrado #1        Bloque Cifrado #2        Bloque Cifrado #3        │
│                   ░░░░░░░░░░░░░            ░░░░░░░░░░░░░            ░░░░░░░░░░░░░          │
│                         ╠═══════╗                ╠═══════╗                ╠═══════╗        │
│                   ╔═════╩═════╗ ║          ╔═════╩═════╗ ║          ╔═════╩═════╗ ║        │
│          Clave ═══╣ Descifrar ║ ║ Clave ═══╣ Descifrar ║ ║ Clave ═══╣ Descifrar ║ ║        │
│                   ╚═════╦═════╝ ║          ╚═════╦═════╝ ║          ╚═════╦═════╝ ║        │
│                      ╔══╩══╗ ╔══╩══╗          ╔══╩══╗ ╔══╩══╗          ╔══╩══╗ ╔══╩══╗     │
│ IV ■■■■■■■■■■■■■ ════╣ XOR ║ ║ XOR ╠══════════╣ XOR ║ ║ XOR ╠══════════╣ XOR ║ ║ XOR ╠═... │
│                      ╚══╦══╝ ╚══╦══╝          ╚══╦══╝ ╚══╦══╝          ╚══╦══╝ ╚══╦══╝     │
│                         ╠═══════╝                ╠═══════╝                ╠═══════╝        │
│                   █████████████            █████████████            █████████████          │
│                  Bloque Plano #1          Bloque Plano #2          Bloque Plano #3         │
└────────────────────────────────────────────────────────────────────────────────────────────┘
```
### Cipher FeedBack (CFB)
En CFB, cada bloque plano se mezcla con el resultado de procesar el bloque cifrado anterior de nuevo por el algoritmo de cifrado. Esto permite convertir un cifrado por bloques en un cifrado por flujo y solo usar el algoritmo en **modo cifrar**.

**Cifrado:** El primer bloque se mezcla con el resultado de cifrar un vector de inicialización (IV). Los bloques posteriores dependen del bloque cifrado previo, que es vuelto a cifrar.
```
┌──────────────────────────────────────────────────────────────────────────────────────────┐
│             IV ■■■■■■■■■■■■■                                                             │
│                      ║     ╔══════════════════════╗     ╔══════════════════════╗         │
│                 ╔════╩═══╗ ║                 ╔════╩═══╗ ║                 ╔════╩═══╗     │
│        Clave ═══╣ Cifrar ║ ║        Clave ═══╣ Cifrar ║ ║        Clave ═══╣ Cifrar ║     │
│                 ╚════╦═══╝ ║                 ╚════╦═══╝ ║                 ╚════╦═══╝     │
│ Bloque Plano #1   ╔══╩══╗  ║ Bloque Plano #2   ╔══╩══╗  ║ Bloque Plano #3   ╔══╩══╗      │
│  █████████████ ═══╣ XOR ║  ║  █████████████ ═══╣ XOR ║  ║  █████████████ ═══╣ XOR ║      │
│                   ╚══╦══╝  ║                   ╚══╦══╝  ║                   ╚══╦══╝      │
│                      ╠═════╝                      ╠═════╝                      ╠═...     │
│                ░░░░░░░░░░░░░                ░░░░░░░░░░░░░                ░░░░░░░░░░░░░   │
│              Bloque Cifrado #1            Bloque Cifrado #2            Bloque Cifrado #3 │
└──────────────────────────────────────────────────────────────────────────────────────────┘
```
**Descifrado:** Para descifrar, cada bloque cifrado se mezcla con el resultado de cifrar de nuevo el bloque cifrado previo. El primer bloque se mezcla con el resultado de cifrar el vector de inicialización (IV).
```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│      IV ■■■■■■■■■■■■■                                                                               │
│               ║     ╔═══════════════════════╗     ╔═══════════════════════╗     ╔═════════════...   │
│          ╔════╩═══╗ ║                  ╔════╩═══╗ ║                  ╔════╩═══╗ ║                   │
│ Clave ═══╣ Cifrar ║ ║         Clave ═══╣ Cifrar ║ ║         Clave ═══╣ Cifrar ║ ║                   │
│          ╚════╦═══╝ ║                  ╚════╦═══╝ ║                  ╚════╦═══╝ ║                   │
│            ╔══╩══╗  ║ Bloque Cifrado #1  ╔══╩══╗  ║ Bloque Cifrado #2  ╔══╩══╗  ║ Bloque Cifrado #3 │
│            ║ XOR ╠══╩══ ░░░░░░░░░░░░░    ║ XOR ╠══╩══ ░░░░░░░░░░░░░    ║ XOR ╠══╩══ ░░░░░░░░░░░░░   │
│            ╚══╦══╝                       ╚══╦══╝                       ╚══╦══╝                      │
│         █████████████                 █████████████                 █████████████                   │
│       Bloque Plano #1               Bloque Plano #2               Bloque Plano #3                   │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```
### Output FeedBack (OFB)
OFB es similar a CFB, pero cada bloque plano se mezcla con el resultado de procesar de nuevo el resultado anterior del algoritmo, antes de ser mezclado con el bloque plano. Esto permite convertir un cifrado por bloques en un cifrado por flujo y solo usar el algoritmo en **modo cifrar**.

**Cifrado:** El primer bloque se mezcla con el resultado de cifrar un vector de inicialización (IV). Los bloques posteriores dependen del resultado del cifrado previo sin mezclar, que es vuelto a cifrar.
```
┌──────────────────────────────────────────────────────────────────────────────────────────┐
│             IV ■■■■■■■■■■■■■                                                             │
│                      ║     ╔══════════════════════╗     ╔══════════════════════╗         │
│                 ╔════╩═══╗ ║                 ╔════╩═══╗ ║                 ╔════╩═══╗     │
│        Clave ═══╣ Cifrar ║ ║        Clave ═══╣ Cifrar ║ ║        Clave ═══╣ Cifrar ║     │
│                 ╚════╦═══╝ ║                 ╚════╦═══╝ ║                 ╚════╦═══╝     │
│                      ╠═════╝                      ╠═════╝                      ╠═...     │
│ Bloque Plano #1   ╔══╩══╗    Bloque Plano #2   ╔══╩══╗    Bloque Plano #3   ╔══╩══╗      │
│  █████████████ ═══╣ XOR ║     █████████████ ═══╣ XOR ║     █████████████ ═══╣ XOR ║      │
│                   ╚══╦══╝                      ╚══╦══╝                      ╚══╦══╝      │
│                ░░░░░░░░░░░░░                ░░░░░░░░░░░░░                ░░░░░░░░░░░░░   │
│              Bloque Cifrado #1            Bloque Cifrado #2            Bloque Cifrado #3 │
└──────────────────────────────────────────────────────────────────────────────────────────┘
```
**Descifrado:** Para descifrar, cada bloque cifrado se mezcla con el resultado de cifrar el resultado del cifrado previo El primer bloque se mezcla con el resultado de cifrar el vector de inicialización (IV).
```
┌───────────────────────────────────────────────────────────────────────────────────────────┐
│              IV ■■■■■■■■■■■■■                                                             │
│                       ║     ╔══════════════════════╗     ╔══════════════════════╗         │
│                  ╔════╩═══╗ ║                 ╔════╩═══╗ ║                 ╔════╩═══╗     │
│         Clave ═══╣ Cifrar ║ ║        Clave ═══╣ Cifrar ║ ║        Clave ═══╣ Cifrar ║     │
│                  ╚════╦═══╝ ║                 ╚════╦═══╝ ║                 ╚════╦═══╝     │
│                       ╠═════╝                      ╠═════╝                      ╠═...     │
│ Bloque Cifrado #1  ╔══╩══╗   Bloque Cifrado #2  ╔══╩══╗   Bloque Cifrado #3  ╔══╩══╗      │
│   ░░░░░░░░░░░░░ ═══╣ XOR ║     ░░░░░░░░░░░░░ ═══╣ XOR ║     ░░░░░░░░░░░░░ ═══╣ XOR ║      │
│                    ╚══╦══╝                      ╚══╦══╝                      ╚══╦══╝      │
│                 █████████████                █████████████                █████████████   │
│                Bloque Plano #1              Bloque Plano #2              Bloque Plano #3  │
└───────────────────────────────────────────────────────────────────────────────────────────┘
```
### Counter (CTR)
En CTR, se utiliza un contador único que se cifra para generar un flujo de bloques que se mezclan con los bloques planos para generar bloques cifrados. Esto permite convertir un cifrado por bloques en un cifrado por flujo, solo usar el algoritmo en **modo cifrar** y poder ser paralelizable.

**Cifrado:** Se cifra el contador y se mezcla con el bloque plano.
```
┌──────────────────────────────────────────────────────────────────────────────────────────┐
│                Nonce + 0x001                Nonce + 0x002                Nonce + 0x003   │
│                ■■■■■■■■■■■■■                ■■■■■■■■■■■■■                ■■■■■■■■■■■■■   │
│                 ╔════╩═══╗                   ╔════╩═══╗                   ╔════╩═══╗     │
│        Clave ═══╣ Cifrar ║          Clave ═══╣ Cifrar ║          Clave ═══╣ Cifrar ║     │
│                 ╚════╦═══╝                   ╚════╦═══╝                   ╚════╦═══╝     │
│ Bloque Plano #1   ╔══╩══╗    Bloque Plano #2   ╔══╩══╗    Bloque Plano #3   ╔══╩══╗      │
│  █████████████ ═══╣ XOR ║     █████████████ ═══╣ XOR ║     █████████████ ═══╣ XOR ║      │
│                   ╚══╦══╝                      ╚══╦══╝                      ╚══╦══╝      │
│                ░░░░░░░░░░░░░                ░░░░░░░░░░░░░                ░░░░░░░░░░░░░   │
│              Bloque Cifrado #1            Bloque Cifrado #2            Bloque Cifrado #3 │
└──────────────────────────────────────────────────────────────────────────────────────────┘
```
**Descifrado:** Se cifra el contador y se mezcla con el bloque cifrado.
```
┌───────────────────────────────────────────────────────────────────────────────────────────┐
│                 Nonce + 0x001                Nonce + 0x002                Nonce + 0x003   │
│                 ■■■■■■■■■■■■■                ■■■■■■■■■■■■■                ■■■■■■■■■■■■■   │
│                  ╔════╩═══╗                   ╔════╩═══╗                   ╔════╩═══╗     │
│         Clave ═══╣ Cifrar ║          Clave ═══╣ Cifrar ║          Clave ═══╣ Cifrar ║     │
│                  ╚════╦═══╝                   ╚════╦═══╝                   ╚════╦═══╝     │
│ Bloque Cifrado #1  ╔══╩══╗   Bloque Cifrado #2  ╔══╩══╗   Bloque Cifrado #3  ╔══╩══╗      │
│   ░░░░░░░░░░░░░ ═══╣ XOR ║     ░░░░░░░░░░░░░ ═══╣ XOR ║     ░░░░░░░░░░░░░ ═══╣ XOR ║      │
│                    ╚══╦══╝                      ╚══╦══╝                      ╚══╦══╝      │
│                 █████████████                █████████████                █████████████   │
│                Bloque Plano #1              Bloque Plano #2              Bloque Plano #3  │
└───────────────────────────────────────────────────────────────────────────────────────────┘
```