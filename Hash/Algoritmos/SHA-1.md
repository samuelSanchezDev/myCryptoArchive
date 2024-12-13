# SHA-1

**Propiedades:**
- **Nombre:** Secure Hash Algorithms 1
- **Tipo:** Función hash.
- **Digest:** 160 bits.
- **Estructura:** [Esquema de Merkle-Damgård](../Conceptos/Esquema%20Merkle-Damgård.md)
    - **Tamaño de bloque:** 512 bits.
    - **Número de rondas:** 80 (En 4 bucles de 20 iteraciones).
- **Creador:** La NSA (National Security Agency)
- **Publicación:** 1995 ([FIPS 180-1](https://csrc.nist.gov/pubs/fips/180-1/final)) Desde 2011 su uso a sido restringido por la NSA al ya no ser considerada completamente segura.

**Seguridad:**
- **Resistencia a colisiones:** Esta rota, se requieren 2<sup>61.2</sup> operaciones en vez de las 2<sup>160</sup> teóricas.
- **Resistencia al cálculo de la pre-imagen**: Esta rota, se requieren 2<sup>63.4</sup> operaciones en vez de las 2<sup>80</sup> teóricas.

**Notación**
- Se usa notación Little-endian.
- Las palabras se componen de 32 bits (4 bytes).
- El símbolo ROTL<sup>i</sup>(X) indica una rotación a la izquierda a **X** de **i** bits

Tiene una fuerte inspiración en el algoritmo [MD5](MD5.md)

---
## 1. Descripción
Se compone de tres partes:
1. **Preprocesamiento del mensaje.** El mensaje se expande hasta que sea congruente con el tamaño de bloque.
2. **Función de compresión.** Se procesa cada bloque del mensaje junto con el resultado de la compresión anterior.
3. **Calculo de la salida.** Con el resultado de la última salida, se obtiene el hash.

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│        ┌─────────────────────────── ─ ─ ─ ─ ───┬─────────┐                              │
│        │ MESSAGE                               │ PADDING │                              │
│        ├███████████████████████████ █ █ █ █ ███┴█████████┤                              │
│        .                                                 .                              │
│        .                                                 .                              │
│        ┌─────────────┬─────────────┬─ ─ ─ ─┬─────────────┐                              │
│        │  Bloque #1  │  Bloque #2  │       │  Bloque #N  │                              │
│        └█████████████┴█████████████┴─ ─ ─ ─┴█████████████┘                              │
│               ╚══╦═══╗      ╚══╦═══╗              ╚══╦═══╗   ╔══════════════╗    Hash   │
│                  ║ F ╠═╗       ║ F ╠═╗               ║ F ╠═══╣ Finalisation ╠═ ■■■■■■■■ │
│ (IV) ■■■■■■■■■ ══╩═══╝ ╚═══════╩═══╝ ╚═ ═ ═ ═ ═ ═ ═ ═╩═══╝   ╚══════════════╝           │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```



### 1.1 Preprocesamiento del mensaje.
La cadena original se expande añadiendo un bit `0b1` seguido de tantos `0b0` hasta que su tamaño en bytes sea congruente con 56 (448 bits) en módulo 64 (512 bits, tamaño de bloque).

> (message + pading) = 56 mod 64

El primer bit se añade incluso si el tamaño original es congruente. En los 8 bytes (64 bits) finales se almacena el tamaño de la cadena original en bits. En caso de que sea mayor que 2<sup>64</sup>, solo se añaden los 64 bits de menor peso del tamaño.

El tamaño se concatena en formato Big-Endian.

```
┌───────────────────────────────────────────────────────────┐
│ 0xEFCDAB8967452301                                        │
│ ┌0─────┬1─────┬2─────┬3─────┬4─────┬5─────┬6─────┬7─────┐ │
│ │ 0xEF │ 0xCD │ 0xAB │ 0x89 │ 0x67 │ 0x45 │ 0x23 │ 0x01 │ │
│ └──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┘ │
└───────────────────────────────────────────────────────────┘
```

### 1.2 Función de compresión.
Esta función toma como argumentos **un bloque (M)** de 512 bits (16 palabras) de la cadena y **un buffer** de 160 bits (5 palabras identificadas con las letras de la **A** a la **E**) que es el resultado de la función de compresión anterior. Cuando se esta procesando el primer bloque, el buffer toma los valores de:
- **A** = `0x67452301`
- **B** = `0xEFCDAB89`
- **C** = `0x98BADCFE`
- **D** = `0x10325476`
- **E** = `0xC3D2E1F0`

La función de compresión consiste en la preparación del bloque y 4 bucles de 20 iteraciones cada uno (80 iteraciones). El buffer no se modifica hasta el final de la función, durante la misma se opera una copia del mismo.

#### 1.2.1 Preparación del bloque
La preparación del bloque consiste en extenderlo de 512 bits (16 palabras) hasta 2.560 bits (80 palabras), estando identificado por **W**. De forma que en cada iteración (**i**) se use una palabra (**W<sub>i</sub>**) del mismo. Se realiza de la siguiente forma:
- Para las palabras 0 y 15
    - **W<sub>i</sub> = M<sub>i</sub>** (Se carga en big endian)

- Para las palabras 16 y 79
    - **W<sub>i</sub> = ROTL<sup>1</sup>(W<sub>i-3</sub> xor W<sub>i-8</sub> xor W<sub>i-14</sub> xor W<sub>i-16</sub>)**.

#### 1.2.2 Bucle de compresión.
Cada bucle (identificado por la variable **t**) tiene asignado una variable **K** y una función **F** que mezcla las palabras **B**, **C** y **D** del buffer.
- **Bucle 1**. Iteraciones 00-19:
	- **K** = `0x5A827999`
	- **Ch(B, C, D)** = (B and C) xor (not(B) and D)
- **Bucle 2**. Iteraciones 20-39:
	- **K** = `0x6ED9EBA1`
	- **Parity(B, C, D)** = B xor C xor D
- **Bucle 3**. Iteraciones 40-59:
    - **K** = `0x8F1BBCDC`
	- **Maj(B, C, D)** = (B and C) xor (B and D) xor (C and D)
- **Bucle 4**. Iteraciones 60-79:
    - **K** = `0xCA62C1D6`
	- **Parity(B, C, D)** = B xor C xor D

Cada iteración (identificado por la variable **i**) consiste en las siguientes asignaciones.
- A la palabra **A** se le asigna **F<sub>t</sub>(B, C, D) + E + W<sub>i</sub> + K<sub>t</sub> + ROTL<sup>5</sup>(A)**.
- A la palabra **B** se le asigna la palabra **A**.
- A la palabra **C** se le asigna **ROTL<sup>30</sup>(B)**.
- A la palabra **D** se le asigna la palabra **C**.
- A la palabra **E** se le asigna la palabra **D**.

```
┌────────────────────────────────────────────────────────────────────┐
│ ┌────────────┬────────────┬────────────┬────────────┬────────────┐ │
│ │     A      │     B      │     C      │     D      │     E      │ │
│ └████████████┴████████████┴████████████┴████████████┴████████████┘ │
│       ║            ║            ║            ║            ║        │
│       ║            ║            ║            ╠═══╦═══╗  ╔═╩═╗      │
│       ║            ║            ╠════════════║═══╣ F ╠══╣ + ║      │
│       ║            ╠════════════║════════════║═══╩══t╝  ╚═╦═╝      │
│       ║            ║            ║            ║          ╔═╩═╗      │
│       ║        ╔═══╩═══╗        ║            ║          ║ + ╠═  Wi │
│       ╠══════╗ ║ <<<<< ║        ║            ║          ╚═╦═╝      │
│       ║      ║ ╚═══╦═30╝        ║            ║          ╔═╩═╗      │
│       ║  ╔═══╩═══╗ ║            ║            ║          ║ + ╠═  Kt │
│       ║  ║ <<<<< ║ ║            ║            ║          ╚═╦═╝      │
│       ║  ╚═══╦══5╝ ║            ║            ║          ╔═╩═╗      │
│       ║      ╚═════║════════════║════════════║══════════╣ + ║      │
│       ║            ║            ║            ║          ╚═╦═╝      │
│       ║            ║            ║            ║            ╚═════╗  │
│       ║            ║            ║            ╚════════════╗     ║  │
│       ║            ║            ╚════════════╗            ║     ║  │
│       ║            ╚════════════╗            ║            ║     ║  │
│       ╚════════════╗            ║            ║            ║     ║  │
│                    ║            ║            ║            ║     ║  │
│       ╔════════════║════════════║════════════║════════════║═════╝  │
│       ║            ║            ║            ║            ║        │
│ ┌████████████┬████████████┬████████████┬████████████┬████████████┐ │
│ │     A      │     B      │     C      │     D      │     E      │ │
│ └────────────┴────────────┴────────────┴────────────┴────────────┘ │
└────────────────────────────────────────────────────────────────────┘
```

Tras los 4 bucles, el buffer modificado se le suma al buffer original.
- **A0 = A0 + A**
- **B0 = B0 + B**
- **C0 = C0 + C**
- **D0 = D0 + D**
- **E0 = E0 + E**

### 1.3 Calculo de la salida
La salida se calcula concatenando las palabras A, B, C, D y E empezando por el byte de mayor peso de A y acabando por el byte de menor peso de E.

```
┌────────────────────────────────────────────────────────────────────────────────┐
│                    │ .... │i-1      0x67452301EFCDAB8998BADCFE10325476C3D2E1F0 │
│ word A 0x67452301  │ 0x01 │i ──────────│─│─│─┘ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │
│                    │ 0x23 │i+1 ────────│─│─┘   │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │
│                    │ 0x45 │i+2 ────────│─┘     │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │
│                    │ 0x67 │... ────────┘       │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │
│ word B 0xEFCDAB89  │ 0x89 │ ───────────────────│─│─│─┘ │ │ │ │ │ │ │ │ │ │ │ │ │
│                    │ 0xAB │ ───────────────────│─│─┘   │ │ │ │ │ │ │ │ │ │ │ │ │
│                    │ 0xCD │ ───────────────────│─┘     │ │ │ │ │ │ │ │ │ │ │ │ │
│                    │ 0xEF │ ───────────────────┘       │ │ │ │ │ │ │ │ │ │ │ │ │
│ word C 0x98BADCFE  │ 0xFE │ ───────────────────────────│─│─│─┘ │ │ │ │ │ │ │ │ │
│                    │ 0xDC │ ───────────────────────────│─│─┘   │ │ │ │ │ │ │ │ │
│                    │ 0xBA │ ───────────────────────────│─┘     │ │ │ │ │ │ │ │ │
│                    │ 0x98 │ ───────────────────────────┘       │ │ │ │ │ │ │ │ │
│ word D 0x10325476  │ 0x76 │ ───────────────────────────────────│─│─│─┘ │ │ │ │ │
│                    │ 0x54 │ ───────────────────────────────────│─│─┘   │ │ │ │ │
│                    │ 0x32 │ ───────────────────────────────────│─┘     │ │ │ │ │
│                    │ 0x10 │ ───────────────────────────────────┘       │ │ │ │ │
│ word E 0xC3D2E1F0  │ 0xF0 │ ───────────────────────────────────────────│─│─│─┘ │
│                    │ 0xE1 │ ───────────────────────────────────────────│─│─┘   │
│                    │ 0xD2 │... ────────────────────────────────────────│─┘     │
│                    │ 0xC3 │i+19 ───────────────────────────────────────┘       │
└────────────────────────────────────────────────────────────────────────────────┘
```