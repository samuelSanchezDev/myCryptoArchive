# MD5

**Propiedades:**
- **Nombre:** Message-Digest Algorithm 5.
- **Tipo:** Función hash.
- **Digest:** 128 bits.
- **Estructura:** [Esquema de Merkle-Damgård](../Conceptos/Esquema%20Merkle-Damgård.md) (Es una mejora del algoritmo [MD4](MD4.md))
    - **Tamaño de bloque:** 512 bits.
    - **Número de rondas:** 64 (4 bucles con 16 iteraciones).
- **Creador:** Ronald Rivest
- **Publicación:** 1992 ([RFC 1321](https://www.rfc-editor.org/rfc/rfc1321))

**Notación**
- Se usa notación Little-endian.
- Las palabras se componen de 32 bits (4 bytes).
- El símbolo **X << i** indica una rotación a la izquierda a **X** de **i** bits (no un desplazamiento).

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

La función de compresión consiste en 4 bucles de 16 iteraciones cada uno (64 iteraciones). El buffer no se modifica hasta el final de la función, durante la misma se opera una copia del mismo.

Cada bucle (identificado por la variable **t**) tiene asignado dos funciones: una para calcula un índice **g** que indica que palabra del bloque se procesa y otra que mezcla la palabras **B**, **C** y **D** del buffer.
- **Bucle 1**. Iteraciones (**i**) 00-15:
	- **g** = i mod 16
	- **F(B, C, D)** = (B and C) or ((not B) and D)
- **Bucle 2**. Iteraciones (**i**) 16-31:
	- **g** = (5 × i + 1) mod 16
	- **G(B, C, D)** = (B and D) or (C and (not D))
- **Bucle 3**. Iteraciones (**i**) 32-47:
	- **g** = (3 × i + 5) mod 16
	- **H(B, C, D)** = B xor C xor D
- **Bucle 4**.  Iteraciones (**i**) 48-63:
	- **g**= (7 × i) mod 16
	- **I(B, C, D)** = C xor (B or (not D))

Cada iteración (identificado por la variable **i**) tiene asignada una constante **K** equivalente al valor absoluto del seno del número de la iteración en radianes y una cantidad **s** de bits para una rotación. Consiste en las siguientes asignaciones.
- A la palabra **A** se le asigna la palabra **D**.
- A la palabra **B** se le asigna  **((A + F<sub>t</sub>(B,C,D) + M<sub>g</sub> + K<sub>i</sub>) << s<sub>i</sub>) + B**.
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
│    Ki ═╣ + ║          ║            ║            ║        │
│        ╚═╦═╝          ║            ║            ║        │
│      ╔═══╩═══╗        ║            ║            ║        │
│      ║ <<<<< ║        ║            ║            ║        │
│      ╚═══╦═Si╝        ║            ║            ║        │
│        ╔═╩═╗          ║            ║            ╚═════╗  │
│        ║ + ╠══════════╣            ╚════════════╗     ║  │
│        ╚═╦═╝          ╚════════════╗            ║     ║  │
│          ╚════════════╗            ║            ║     ║  │
│                       ║            ║            ║     ║  │
│          ╔════════════║════════════║════════════║═════╝  │
│          ║            ║            ║            ║        │
│    ┌████████████┬████████████┬████████████┬████████████┐ │
│    │     A      │     B      │     C      │     D      │ │
│    └────────────┴────────────┴────────────┴────────────┘ │
└──────────────────────────────────────────────────────────┘
```


Tras los 4 bucles, el buffer modificado se le suma al buffer original.
- **A0 = A0 + A**
- **B0 = B0 + B**
- **C0 = C0 + C**
- **D0 = D0 + D**

#### 1.2.1 Senos pre-calculados

| Iteraciones |              |              |              |              |              |              |              |              |
| :---------: | ------------ | ------------ | ------------ | ------------ | ------------ | ------------ | ------------ | ------------ |
|   00...07   | `0xD76AA478` | `0xE8C7B756` | `0x242070DB` | `0xC1BDCEEE` | `0xF57C0FAF` | `0x4787C62A` | `0xA8304613` | `0xFD469501` |
|   08...15   | `0x698098D8` | `0x8B44F7AF` | `0xFFFF5BB1` | `0x895CD7BE` | `0x6B901122` | `0xFD987193` | `0xA679438E` | `0x49B40821` |
|   16...23   | `0xF61E2562` | `0xC040B340` | `0x265E5A51` | `0xE9B6C7AA` | `0xD62F105D` | `0x02441453` | `0xD8A1E681` | `0xE7D3FBC8` |
|   24...31   | `0x21E1CDE6` | `0xC33707D6` | `0xF4D50D87` | `0x455A14ED` | `0xA9E3E905` | `0xFCEFA3F8` | `0x676F02D9` | `0x8D2A4C8A` |
|   32...39   | `0xFFFA3942` | `0x8771F681` | `0x6D9D6122` | `0xFDE5380C` | `0xA4BEEA44` | `0x4BDECFA9` | `0xF6BB4B60` | `0xBEBFBC70` |
|   40...47   | `0x289B7EC6` | `0xEAA127FA` | `0xD4EF3085` | `0x04881D05` | `0xD9D4D039` | `0xE6DB99E5` | `0x1FA27CF8` | `0xC4AC5665` |
|   48...55   | `0xF4292244` | `0x432AFF97` | `0xAB9423A7` | `0xFC93A039` | `0x655B59C3` | `0x8F0CCC92` | `0xFFEFF47D` | `0x85845DD1` |
|   56...63   | `0x6FA87E4F` | `0xFE2CE6E0` | `0xA3014314` | `0x4E0811A1` | `0xF7537E82` | `0xBD3AF235` | `0x2AD7D2BB` | `0xEB86D391` |

#### 1.2.2 Valores de 's' (rotación de bits)

| Bucle\Iteración | 00  | 01  | 02  | 03  | 04  | 05  | 06  | 07  | 08  | 09  | 0A  | 0B  | 0C  | 0D  | 0E  | 0F  |
| :-------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1               | 07  | 12  | 17  | 22  | 07  | 12  | 17  | 22  | 07  | 12  | 17  | 22  | 07  | 12  | 17  | 22  |
| 2               | 05  | 09  | 14  | 20  | 05  | 09  | 14  | 20  | 05  | 09  | 14  | 20  | 05  | 09  | 14  | 20  |
| 3               | 04  | 11  | 16  | 23  | 04  | 11  | 16  | 23  | 04  | 11  | 16  | 23  | 04  | 11  | 16  | 23  |
| 4               | 06  | 10  | 15  | 21  | 06  | 10  | 15  | 21  | 06  | 10  | 15  | 21  | 06  | 10  | 15  | 21  |


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
