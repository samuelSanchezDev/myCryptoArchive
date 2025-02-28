# MD4

**Propiedades:**
- **Nombre:** Message-Digest Algorithm 4
- **Tipo:** Función hash.
- **Digest:** 128 bits.
- **Estructura:** [Esquema de Merkle-Damgård](../Conceptos/Esquema%20Merkle-Damgård.md)
    - **Tamaño de bloque:** 512 bits.
    - **Número de rondas:** 48 (3 bucles de 16 iteraciones).
- **Creador:** Ronald Rivest
- **Publicación:** 1990 ([RFC 1186](https://www.rfc-editor.org/rfc/rfc1186))

**Notación**
- Se usa notación Little-endian.
- Las palabras se componen de 32 bits (4 bytes).
- El símbolo **X << i** indica una rotación a la izquierda a **X** de **i** bits (no un desplazamiento).

---
## 1. Descripción
La función se compone de tres etapas:
1. **Preprocesamiento del mensaje.** El mensaje se expande hasta que sea congruente con el tamaño de bloque.
2. **Función de compresión.** Se procesa cada bloque del mensaje junto con el resultado de la compresión anterior.
3. **Calculo de la salida.** Con el resultado de la última salida, se obtiene el digest.

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

### 1.1 Preprocesamiento del mensaje
La cadena original se expande añadiendo un bit `0b1` seguido de tantos `0b0` hasta que su tamaño en bytes sea congruente con 56 (448 bits) en módulo 64 (512 bits, tamaño de bloque).

> (message + pading) = 56 mod 64

El primer bit se añade incluso si el tamaño original es congruente. En los 8 bytes (64 bits) finales se almacena el tamaño de la cadena original en bits. En caso de que sea mayor que 2<sup>64</sup>, solo se añaden los 64 bits de menor peso del tamaño.

El tamaño se concatena en formato Little-Endian.

```
┌───────────────────────────────────────────────────────────┐
│ 0x87EFCDAB8967452301                                      │
│ ┌0─────┬1─────┬2─────┬3─────┬4─────┬5─────┬6─────┬7─────┐ │
│ │ 0x01 │ 0x23 │ 0x45 │ 0x67 │ 0x89 │ 0xAB │ 0xCD │ 0xEF │ │
│ └──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┘ │
└───────────────────────────────────────────────────────────┘
```

### 1.2 Función de compresión
Esta función toma como argumentos **un bloque (M)** de 512 bits (16 palabras) de la cadena y **un buffer** de 128 bits (4 palabras identificadas con las letras de la **A** a la **D**) que es el resultado de la función de compresión anterior. Cuando se esta procesando el primer bloque, el buffer toma los valores de:
- **A0** = `0x67452301`
- **B0** = `0xEFCDAB89`
- **C0** = `0x98BADCFE`
- **D0** = `0x10325476`

La función de compresión consiste en 3 bucles con 16 rondas cada uno (48 iteraciones). El buffer no se modifica hasta el final de la función, durante la misma se opera una copia del mismo.

Cada bucle (identificado por la variable **t**) tiene asignado una variable **K** y una función que mezcla las palabras **B**, **C** y **D** del buffer.
- **Bucle 1**. Iteraciones 00-15
	- **K** = `0x00000000`
	- **F(B, C, D)** = (B and C)  or ((not B)  and D)
- **Bucle 2**. Iteraciones 16-31:
	- **K** = `0x5A827999` (Es la raíz cuadrada de 2 en hexadecimal).
	- **G(B, C, D)** = (B and C) or (B and D) or (C and D)
- **Bucle 3**. Iteraciones 32-47:
	- **K** = `0x6ED9EBA1` (Es la raíz cuadrada de 3 en hexadecimal).
	- **H(B, C, D)** = B xor C xor D

Cada iteración (identificado por la variable **i**) tiene asignada una variable **g** que indica que palabra del bloque se procesa y una cantidad **s** de bits para una rotación.

La función consiste en las siguientes asignaciones.
- A la palabra **A** se le asigna la palabra **D**.
- A la palabra **B** se le asigna  **(A + F<sub>t</sub>(B,C,D) + M<sub>g</sub> + K<sub>t</sub>) << s<sub>i</sub>**
- A la palabra **C** se le asigna la palabra **B**.
- A la palabra **D** se le asigna la palabra **C**.

```
┌──────────────────────────────────────────────────────────┐
│    ┌────────────┬────────────┬────────────┬────────────┐ │
│    │     A      │     B      │     C      │     D      │ │
│    └████████████┴████████████┴████████████┴████████████┘ │
│          ║            ║            ║            ║        │
│        ╔═╩═╗ ╔═══╦════╣            ║            ║        │
│        ║ + ╠═╣ F ╠════║════════════╣            ║        │
│        ╚═╦═╝ ╚══t╩════║════════════║════════════╣        │
│        ╔═╩═╗          ║            ║            ║        │
│    Mg ═╣ + ║          ║            ║            ║        │
│        ╚═╦═╝          ║            ║            ║        │
│        ╔═╩═╗          ║            ║            ║        │
│    Kt ═╣ + ║          ║            ║            ║        │
│        ╚═╦═╝          ║            ║            ║        │
│      ╔═══╩═══╗        ║            ║            ╚═════╗  │
│      ║ <<<<< ║        ║            ╚════════════╗     ║  │
│      ╚═══╦═Si╝        ╚════════════╗            ║     ║  │
│          ╚════════════╗            ║            ║     ║  │
│          ╔════════════║════════════║════════════║═════╝  │
│          ║            ║            ║            ║        │
│    ┌████████████┬████████████┬████████████┬████████████┐ │
│    │     A      │     B      │     C      │     D      │ │
│    └────────────┴────────────┴────────────┴────────────┘ │
└──────────────────────────────────────────────────────────┘
```

Tras los 3 bucles, el buffer modificado (**X**) se le suma al buffer original (**X0**).
- **A0 = A0 + A**
- **B0 = B0 + B**
- **C0 = C0 + C**
- **D0 = D0 + D**

#### 1.2.1 Valores de 'g'
| Bucle\Iteración | 00  | 01  | 02  | 03  | 04  | 05  | 06  | 07  | 08  | 09  | 0A  | 0B  | 0C  | 0D  | 0E  | 0F  |
| :-------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1               | 00  | 01  | 02  | 03  | 04  | 05  | 06  | 07  | 08  | 09  | 10  | 11  | 12  | 13  | 14  | 15  |
| 2               | 00  | 04  | 08  | 12  | 01  | 05  | 09  | 13  | 02  | 06  | 10  | 14  | 03  | 07  | 11  | 15  |
| 3               | 00  | 08  | 04  | 12  | 02  | 10  | 06  | 14  | 01  | 09  | 05  | 13  | 03  | 11  | 07  | 15  |


#### 1.2.2 Valores de 's' (rotación de bits)
| Bucle\Iteración | 00  | 01  | 02  | 03  | 04  | 05  | 06  | 07  | 08  | 09  | 0A  | 0B  | 0C  | 0D  | 0E  | 0F  |
| :-------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1               | 03  | 07  | 11  | 19  | 03  | 07  | 11  | 19  | 03  | 07  | 11  | 19  | 03  | 07  | 11  | 19  |
| 2               | 03  | 05  | 09  | 13  | 03  | 05  | 09  | 13  | 03  | 05  | 09  | 13  | 03  | 05  | 09  | 13  |
| 3               | 03  | 09  | 11  | 15  | 03  | 09  | 11  | 15  | 03  | 09  | 11  | 15  | 03  | 09  | 11  | 15  |

### 1.3 Calculo de la salida
La salida se calcula concatenando las palabras A, B, C y D empezando por el byte de menor peso de A y acabando por el byte de mayor peso de D.
```
┌────────────────────────────────────────────────────────────────────────┐
│                    │ .... │i-1      0x0123456789ABCDEFFEDCBA9876543210 │
│ word A 0x67452301  │ 0x01 │i ──────────┘ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │
│                    │ 0x23 │i+1 ──────────┘ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │
│                    │ 0x45 │i+2 ────────────┘ │ │ │ │ │ │ │ │ │ │ │ │ │ │
│                    │ 0x67 │... ──────────────┘ │ │ │ │ │ │ │ │ │ │ │ │ │
│ word B 0xEFCDAB89  │ 0x89 │ ───────────────────┘ │ │ │ │ │ │ │ │ │ │ │ │
│                    │ 0xAB │ ─────────────────────┘ │ │ │ │ │ │ │ │ │ │ │
│                    │ 0xCD │ ───────────────────────┘ │ │ │ │ │ │ │ │ │ │
│                    │ 0xEF │ ─────────────────────────┘ │ │ │ │ │ │ │ │ │
│ word C 0x98BADCFE  │ 0xFE │ ───────────────────────────┘ │ │ │ │ │ │ │ │
│                    │ 0xDC │ ─────────────────────────────┘ │ │ │ │ │ │ │
│                    │ 0xBA │ ───────────────────────────────┘ │ │ │ │ │ │
│                    │ 0x98 │ ─────────────────────────────────┘ │ │ │ │ │
│ word D 0x10325476  │ 0x76 │ ───────────────────────────────────┘ │ │ │ │
│                    │ 0x54 │ ─────────────────────────────────────┘ │ │ │
│                    │ 0x32 │...  ───────────────────────────────────┘ │ │
│                    │ 0x10 │i+15 ─────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────────┘
```
