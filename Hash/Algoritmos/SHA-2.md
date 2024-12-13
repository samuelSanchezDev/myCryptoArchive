# SHA-2

SHA-2 no es un algoritmo, sino una familia de funciones hash creadas por la NSA (National Security Agency).

|   Nombre    |  Digest  | Tamaño de bloque | Tamaño de palabra | Numero de rondas |                                      Publicación                                       |
| :---------: | :------: | :--------------: | :---------------: | :--------------: | :------------------------------------------------------------------------------------: |
|   SHA-256   | 256 bits |     512 bits     |      32 bits      |        64        |            2002 ([FIPS 180-2](https://csrc.nist.gov/pubs/fips/180-2/final))            |
|   SHA-384   | 384 bits |    1024 bits     |      64 bits      |        80        |            2002 ([FIPS 180-2](https://csrc.nist.gov/pubs/fips/180-2/final))            |
|   SHA-512   | 512 bits |    1024 bits     |      64 bits      |        80        |            2002 ([FIPS 180-2](https://csrc.nist.gov/pubs/fips/180-2/final))            |
|   SHA-224   | 224 bits |     512 bits     |      32 bits      |        64        | 2004 ([Actualización de FIPS 180-2](https://csrc.nist.gov/pubs/fips/180-2/upd1/final)) |
| SHA-512/224 | 224 bits |    1024 bits     |      64 bits      |        80        |            2012 ([FIPS 180-4](https://csrc.nist.gov/pubs/fips/180-4/final))            |
| SHA-512/256 | 256 bits |    1024 bits     |      64 bits      |        80        |            2012 ([FIPS 180-4](https://csrc.nist.gov/pubs/fips/180-4/final))            |

**Notación**
- Se usa notación Little-endian.
- El símbolo **ROTR<sup>i</sup>(X)** indica una rotación a la derecha a **X** de **i** bits
- El símbolo **SHR<sup>i</sup>(X)** indica un desplazamiento a la derecha a **X** de **i** bits
- **Ch(X, Y, Z) = (X and Y) xor (not(X) and Z)**
- **Parity(X, Y, Z) = X xor Y xor Z**
- **Maj(X, Y, Z) = (X and Y) xor (X and Z) xor (Y and Z)**
- Los símbolos **SIGMA<sub>0</sub>(X)**, **SIGMA<sub>1</sub>(X)**, **sigma<sub>0</sub>(X)** y **sigma<sub>0</sub>(X)** representan 4 funciones que varían según el algoritmo, pero que están presentes en todos.


## 1. Descripción general de SHA2
Todas las funciones tienen como **estructura** un [esquema de Merkle-Damgård](../Conceptos/Esquema%20Merkle-Damgård.md) y siguen una estructura similar.

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

Las funciones se componen de tres partes:

### 1 **Preprocesamiento del mensaje.**

El mensaje se expande hasta que sea congruente con el tamaño de bloque. Esta expansión se realiza añadiendo un bit `0b1` seguido de tantos `0b0` hasta que su tamaño en bytes sea congruente con un tamaño **X** (diferente según la función) y en el bloque restante se almacena el tamaño de la cadena original en bits.

El tamaño se concatena en formato Big-Endian.

```
┌───────────────────────────────────────────────────────────┐
│ 0xEFCDAB8967452301                                        │
│ ┌0─────┬1─────┬2─────┬3─────┬4─────┬5─────┬6─────┬7─────┐ │
│ │ 0xEF │ 0xCD │ 0xAB │ 0x89 │ 0x67 │ 0x45 │ 0x23 │ 0x01 │ │
│ └──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┘ │
└───────────────────────────────────────────────────────────┘
```

### 2 **Función de compresión.**

Esta función toma como argumentos **un bloque (M)** de 16 palabras y **un buffer** de 8 palabras identificadas con las letras de la **A** a la **H** que es el resultado de la función de compresión anterior (para la primera iteración hay constantes predefinidas).

La función de compresión consiste en la preparación del bloque y **N** rondas (iteraciones) para mezclar. El buffer no se modifica hasta el final de la función, durante la misma se opera una copia del mismo.

#### 2.1 **La preparación del bloque**

La preparación del bloque consiste en extenderlo de **16 palabras** hasta **N palabras**, estando identificadas por **W**. De forma que en cada iteración (**i**) se use una palabra (**W<sub>i</sub>**) del mismo. Se realiza de la siguiente forma:
- Para las palabras 0 y 15
    - **W<sub>i</sub> = M<sub>i</sub>** (Se carga en big endian)
- Para las palabras 16 y **N-1**
    - **W<sub>i</sub> = sigma<sub>1</sub>(W<sub>i-2</sub>) + W<sub>i-7</sub> + sigma<sub>0</sub>(W<sub>i-15</sub>) + W<sub>i-16</sub>**.

#### 2.2 **Bucle de compresión.**

En cada iteración (**i**) se realiza la siguientes operaciones (cada iteración de bucle tiene asignada una constante **K** diferente según la función **SHA2**):
- **T<sub>1</sub> = H + SIGMA<sub>1</sub>(E) + Ch(e, f, g) + K<sub>i</sub> + W<sub>i</sub>**
- **T<sub>2</sub> = SIGMA<sub>0</sub>(A) + Maj(a, b, c)**

Y las siguientes asignaciones:
- A la palabra **A** se le asigna **T<sub>1</sub> + T<sub>2</sub>**
- A la palabra **B** se le asigna **A**
- A la palabra **C** se le asigna **B**
- A la palabra **D** se le asigna **C**
- A la palabra **E** se le asigna **D + T<sub>1</sub>**
- A la palabra **F** se le asigna **E**
- A la palabra **G** se le asigna **F**
- A la palabra **H** se le asigna **G**

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ ┌────────────┬────────────┬────────────┬────────────┬────────────┬────────────┬────────────┬────────────┐       │
│ │     A      │     B      │     C      │     D      │     E      │     F      │     G      │     H      │       │
│ └████████████┴████████████┴████████████┴████████████┴████████████┴████████████┴████████████┴████████████┘       │
│       ║            ║            ║            ║            ║            ║            ║            ║              │
│       ║            ║            ║            ║            ║            ║            ╠═══╦════╗ ╔═╩═╗  ╔═══╗     │
│       ║            ║            ║            ║            ║            ╠════════════║═══╣ CH ╠═╣ + ╠══╣ + ╠═ Wi │
│       ║            ║            ║            ║            ╠════════════║════════════║═══╩════╝ ╚═╦═╝  ╚═╦═╝     │
│       ║            ║            ║            ║            ║            ║            ║   ╔════╗ ╔═╩═╗    Ki      │
│       ║            ║            ║            ║            ║            ║            ║═══╣ E1 ╠═╣ + ║            │
│       ║            ║            ║            ║            ║            ║            ║   ╚════╝ ╚═╦═╝            │
│       ║            ║            ║          ╔═╩═╗          ║            ║            ║            ║              │
│       ║            ║            ║          ║ + ╠══════════║════════════║════════════║════════════╣              │
│       ║            ║            ║          ╚═╦═╝          ║            ║            ║            ║              │
│       ╠════════════║════════════║════════════║════════════║════════════║════════════║═══╦════╗ ╔═╩═╗            │
│       ║            ╠════════════║════════════║════════════║════════════║════════════║═══╣ Ma ╠═╣ + ║            │
│       ║            ║            ╠════════════║════════════║════════════║════════════║═══╩════╝ ╚═╦═╝            │
│       ║            ║            ║            ║            ║            ║            ║   ╔════╗ ╔═╩═╗            │
│       ║            ║            ║            ║            ║            ║            ║═══╣ E0 ╠═╣ + ║            │
│       ║            ║            ║            ║            ║            ║            ║   ╚════╝ ╚═╦═╝            │
│       ║            ║            ║            ║            ║            ║            ║            ╚═════╗        │
│       ║            ║            ║            ║            ║            ║            ╚════════════╗     ║        │
│       ║            ║            ║            ║            ║            ╚════════════╗            ║     ║        │
│       ║            ║            ║            ║            ╚════════════╗            ║            ║     ║        │
│       ║            ║            ║            ╚════════════╗            ║            ║            ║     ║        │
│       ║            ║            ╚════════════╗            ║            ║            ║            ║     ║        │
│       ║            ╚════════════╗            ║            ║            ║            ║            ║     ║        │
│       ╚════════════╗            ║            ║            ║            ║            ║            ║     ║        │
│                    ║            ║            ║            ║            ║            ║            ║     ║        │
│       ╔════════════║════════════║════════════║════════════║════════════║════════════║════════════║═════╝        │
│       ║            ║            ║            ║            ║            ║            ║            ║              │
│ ┌████████████┬████████████┬████████████┬████████████┬████████████┬████████████┬████████████┬████████████┐       │
│ │     A      │     B      │     C      │     D      │     E      │     F      │     G      │     H      │       │
│ └────────────┴────────────┴────────────┴────────────┴────────────┴────────────┴────────────┴────────────┘       │
└─────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```
Al final, el buffer modificado se le suma al buffer original.
- **A0 = A0 + A**
- **B0 = B0 + B**
- **C0 = C0 + C**
- **D0 = D0 + D**
- **E0 = E0 + E**
- **F0 = F0 + F**
- **G0 = G0 + G**
- **H0 = H0 + H**

### 3. **Calculo de la salida.**
La salida se calcula concatenando las palabras del buffer empezando por el byte de mayor peso de A y acabando un byte diferente dependiendo de la función SHA2.

```
┌────────────────────────────────────────────────────────────┐
│                    │ .... │i-1      0x67452301EFCDAB899....│
│ word A 0x67452301  │ 0x01 │i ──────────│─│─│─┘ │ │ │ │ │ │ │
│                    │ 0x23 │i+1 ────────│─│─┘   │ │ │ │ │ │ │
│                    │ 0x45 │i+2 ────────│─┘     │ │ │ │ │ │ │
│                    │ 0x67 │... ────────┘       │ │ │ │ │ │ │
│ word B 0xEFCDAB89  │ 0x89 │ ───────────────────│─│─│─┘ │ │ │
│                    │ 0xAB │ ───────────────────│─│─┘   │ │ │
│                    │ 0xCD │ ───────────────────│─┘  ...... │
│                    │ 0xEF │ ───────────────────┘    ...... │
│ word C 0x98......  │ 0x.. │ ────────────────────────...... │
│                    │ 0x.. │ ────────────────────────...... │
│                    │ .... │ ────────────────────────...... │
└────────────────────────────────────────────────────────────┘
```

## 2. SHA-256 y SHA-224
Ambas funciones son muy similares ya que SHA-224 se derivo a partir de SHA-256. Únicamente difieren en el buffer inicial y en el calculo de la salida.

### Notación
- **SIGMA<sub>0</sub>(X) = ROTR<sup>2</sup>(X) xor ROTR<sup>13</sup>(X) xor ROTR<sup>22</sup>(X)**
- **SIGMA<sub>1</sub>(X) = ROTR<sup>6</sup>(X) xor ROTR<sup>11</sup>(X) xor ROTR<sup>25</sup>(X)**
- **sigma<sub>0</sub>(X) = ROTR<sup>7</sup>(X) xor ROTR<sup>18</sup>(X) xor SHR<sup>3</sup>(X)**
- **sigma<sub>1</sub>(X) = ROTR<sup>17</sup>(X) xor ROTR<sup>19</sup>(X) xor SHR<sup>10</sup>(X)**

### 2.1 Preprocesamiento del mensaje.
La cadena original se expande añadiendo un bit `0b1` seguido de tantos `0b0` hasta que su tamaño en bytes sea congruente con 56 (448 bits) en módulo 64 (512 bits, tamaño de bloque).

> (message + pading) = 56 mod 64

El primer bit se añade incluso si el tamaño original es congruente. En los 8 bytes (64 bits) finales se almacena el tamaño de la cadena original en bits. En caso de que sea mayor que 2<sup>64</sup>, solo se añaden los 64 bits de menor peso del tamaño.

El tamaño se concatena en formato Big-Endian.

### 2.2 Función de compresión.
Los valores iniciales del buffer son:


| Buffer | SHA256       | SHA224       |
| :----: | ------------ | ------------ |
|   A    | `0x6A09E667` | `0xC1059ED8` |
|   B    | `0xBB67AE85` | `0x367CD507` |
|   C    | `0x3C6EF372` | `0x3070DD17` |
|   D    | `0xA54FF53A` | `0xF70E5939` |
|   E    | `0x510E527F` | `0xFFC00B31` |
|   F    | `0x9B05688C` | `0x68581511` |
|   G    | `0x1F83D9AB` | `0x64F98FA7` |
|   H    | `0x5BE0CD19` | `0xBEFA4FA4` |

Las constante **K** representan la parte decimal de la raíz cuadrada de los 64 primeros números primos. Son las siguientes:

| Iteraciones |              |              |              |              |              |              |              |              |
| :---------: | ------------ | ------------ | ------------ | ------------ | ------------ | ------------ | ------------ | ------------ |
|   00...07   | `0x428A2F98` | `0x71374491` | `0xB5C0FBCF` | `0xE9B5DBA5` | `0x3956C25B` | `0x59F111F1` | `0x923F82A4` | `0xAB1C5ED5` |
|   08...15   | `0xD807AA98` | `0x12835B01` | `0x243185BE` | `0x550C7DC3` | `0x72BE5D74` | `0x80DEB1FE` | `0x9BDC06A7` | `0xC19BF174` |
|   16...23   | `0xE49B69C1` | `0xEFBE4786` | `0x0FC19DC6` | `0x240CA1CC` | `0x2DE92C6F` | `0x4A7484AA` | `0x5CB0A9DC` | `0x76F988DA` |
|   24...31   | `0x983E5152` | `0xA831C66D` | `0xB00327C8` | `0xBF597FC7` | `0xC6E00BF3` | `0xD5A79147` | `0x06CA6351` | `0x14292967` |
|   32...39   | `0x27B70A85` | `0x2E1B2138` | `0x4D2C6DFC` | `0x53380D13` | `0x650A7354` | `0x766A0ABB` | `0x81C2C92E` | `0x92722C85` |
|   40...47   | `0xA2BFE8A1` | `0xA81A664B` | `0xC24B8B70` | `0xC76C51A3` | `0xD192E819` | `0xD6990624` | `0xF40E3585` | `0x106AA070` |
|   48...55   | `0x19A4C116` | `0x1E376C08` | `0x2748774C` | `0x34B0BCB5` | `0x391C0CB3` | `0x4ED8AA4A` | `0x5B9CCA4F` | `0x682E6FF3` |
|   56...63   | `0x748F82EE` | `0x78A5636F` | `0x84C87814` | `0x8CC70208` | `0x90BEFFFA` | `0xA4506CEB` | `0xBEF9A3F7` | `0xC67178F2` |


### 2.3 Calculo de la salida
Para **SHA256**, la salida se calcula concatenando las palabras A, B, C, D, E, F, G y H empezando por el byte de mayor peso de A y acabando por el byte de menor peso de H.

En el caso de **SHA224** es igual, pero se acaba en la palabra G. La palabra H del buffer se descarta.

## 3. SHA-384 y SHA-512
Entre SHA-384 y SHA-512 unicamente difiere el buffer inicial y en el calculo de la salida.

### Notación
- **SIGMA<sub>0</sub>(X) = ROTR<sup>28</sup>(X) xor ROTR<sup>34</sup>(X) xor ROTR<sup>39</sup>(X)**
- **SIGMA<sub>1</sub>(X) = ROTR<sup>14</sup>(X) xor ROTR<sup>18</sup>(X) xor ROTR<sup>41</sup>(X)**
- **sigma<sub>0</sub>(X) = ROTR<sup>1</sup>(X) xor ROTR<sup>8</sup>(X) xor SHR<sup>7</sup>(X)**
- **sigma<sub>1</sub>(X) = ROTR<sup>19</sup>(X) xor ROTR<sup>61</sup>(X) xor SHR<sup>6</sup>(X)**

### 3.1 Preprocesamiento del mensaje.
La cadena original se expande añadiendo un bit `0b1` seguido de tantos `0b0` hasta que su tamaño en bytes sea congruente con 112 (896 bits) en módulo 128 (1024 bits, tamaño de bloque).

> (message + pading) = 112 mod 128

El primer bit se añade incluso si el tamaño original es congruente. En los 16 bytes (128 bits) finales se almacena el tamaño de la cadena original en bits. En caso de que sea mayor que 2<sup>128</sup>, solo se añaden los 128 bits de menor peso del tamaño.

El tamaño se concatena en formato Big-Endian.


### 3.2 Función de compresión.
Los valores iniciales del buffer son:

| Buffer | SHA384               | SHA512               |
| :----: | -------------------- | -------------------- |
|   A    | `0xCBBB9D5DC1059ED8` | `0x6A09E667F3BCC908` |
|   B    | `0x629A292A367CD507` | `0xBB67AE8584CAA73B` |
|   C    | `0x9159015A3070DD17` | `0x3C6EF372FE94F82B` |
|   D    | `0x152FECD8F70E5939` | `0xA54FF53A5F1D36F1` |
|   E    | `0x67332667FFC00B31` | `0x510E527FADE682D1` |
|   F    | `0x8EB44A8768581511` | `0x9B05688C2B3E6C1F` |
|   G    | `0xDB0C2E0D64F98FA7` | `0x1F83D9ABFB41BD6B` |
|   H    | `0x47B5481DBEFA4FA4` | `0x5BE0CD19137E2179` |

Las constante **K** representan la parte decimal de la raíz cuadrada de los 64 primeros números primos. Son las siguientes:

| Iteraciones |                      |                      |                      |                      |
| :---------: | -------------------- | -------------------- | -------------------- | -------------------- |
|   00...03   | `0x428A2F98D728AE22` | `0x7137449123EF65CD` | `0xB5C0FBCFEC4D3B2F` | `0xE9B5DBA58189DBBC` |
|   04...07   | `0x3956C25BF348B538` | `0x59F111F1B605D019` | `0x923F82A4AF194F9B` | `0xAB1C5ED5DA6D8118` |
|   08...11   | `0xD807AA98A3030242` | `0x12835B0145706FBE` | `0x243185BE4EE4B28C` | `0x550C7DC3D5FFB4E2` |
|   12...15   | `0x72BE5D74F27B896F` | `0x80DEB1FE3B1696B1` | `0x9BDC06A725C71235` | `0xC19BF174CF692694` |
|   16...19   | `0xE49B69C19EF14AD2` | `0xEFBE4786384F25E3` | `0x0FC19DC68B8CD5B5` | `0x240CA1CC77AC9C65` |
|   20...23   | `0x2DE92C6F592B0275` | `0x4A7484AA6EA6E483` | `0x5CB0A9DCBD41FBD4` | `0x76F988DA831153B5` |
|   24...27   | `0x983E5152EE66DFAB` | `0xA831C66D2DB43210` | `0xB00327C898FB213F` | `0xBF597FC7BEEF0EE4` |
|   28...31   | `0xC6E00BF33DA88FC2` | `0xD5A79147930AA725` | `0x06CA6351E003826F` | `0x142929670A0E6E70` |
|   32...35   | `0x27B70A8546D22FFC` | `0x2E1B21385C26C926` | `0x4D2C6DFC5AC42AED` | `0x53380D139D95B3DF` |
|   36...39   | `0x650A73548BAF63DE` | `0x766A0ABB3C77B2A8` | `0x81C2C92E47EDAEE6` | `0x92722C851482353B` |
|   40...43   | `0xA2BFE8A14CF10364` | `0xA81A664BBC423001` | `0xC24B8B70D0F89791` | `0xC76C51A30654BE30` |
|   44...47   | `0xD192E819D6EF5218` | `0xD69906245565A910` | `0xF40E35855771202A` | `0x106AA07032BBD1B8` |
|   48...51   | `0x19A4C116B8D2D0C8` | `0x1E376C085141AB53` | `0x2748774CDF8EEB99` | `0x34B0BCB5E19B48A8` |
|   52...55   | `0x391C0CB3C5C95A63` | `0x4ED8AA4AE3418ACB` | `0x5B9CCA4F7763E373` | `0x682E6FF3D6B2B8A3` |
|   56...59   | `0x748F82EE5DEFB2FC` | `0x78A5636F43172F60` | `0x84C87814A1F0AB72` | `0x8CC702081A6439EC` |
|   60...63   | `0x90BEFFFA23631E28` | `0xA4506CEBDE82BDE9` | `0xBEF9A3F7B2C67915` | `0xC67178F2E372532B` |
|   64...69   | `0xCA273ECEEA26619C` | `0xD186B8C721C0C207` | `0xEADA7DD6CDE0EB1E` | `0xF57D4F7FEE6ED178` |
|   68...71   | `0x06F067AA72176FBA` | `0x0A637DC5A2C898A6` | `0x113F9804BEF90DAE` | `0x1B710B35131C471B` |
|   72...77   | `0x28DB77F523047D84` | `0x32CAAB7B40C72493` | `0x3C9EBE0A15C9BEBC` | `0x431D67C49C100D4C` |
|   76...79   | `0x4CC5D4BECB3E42B6` | `0x597F299CFC657E2A` | `0x5FCB6FAB3AD6FAEC` | `0x6C44198C4A475817` |


### 3.3 Calculo de la salida
Para **SHA512**, la salida se calcula concatenando las palabras A, B, C, D, E, F, G y H empezando por el byte de mayor peso de A y acabando por el byte de menor peso de H.

En el caso de **SHA384** es igual, pero se acaba en la palabra F. Las palabras G y H del buffer se descarta.

###
La salida se calcula concatenando las palabras A, B, C, D, E, F, G y H empezando por el byte de mayor peso de A y acabando por el byte de menor peso de H.

## 4. SHA-512/224 y SHA-512/256
Ambas funciones derivan de **SHA-512** y son parte de un "subgrupo" de funciones SHA2, las *SHA-512/t* (solo SHA-512/224 y SHA-512/256 están aprobadas). Estas funciones usan el mismo algoritmo que SHA-512 salvo que tienen un buffer y calculo de salida propios.

Estas funciones están definidas por el valor de **t**, el cual:
- Tiene que ser menor que 512.
- No puede tomar el valor de 384.

### 4.1 Buffer inicial
Se calcula mediante el algoritmo **SHA-512/t IV Generation Function**.

1. Usando el buffer inicial de **SHA512** (H''), se deriva un buffer auxiliar H'.
    ```
    for i = 0 to 7:
        H''[i] = H'[i] xor 0xA5A5A5A5A5A5A5A5
    ```
2. Se calcula el disgest del string "*SHA512/<t>"* (en ascii) con el buffer auxiliar anterior.
    ```
    // Ejem con t = 256
    H = SHA512("SHA-512/256", H'')
    ```
 3. El digest **H** es el buffer del algoritmo.

Los buffer de SHA-512/224 y SHA-512/256 son:

| Buffer | SHA-512/224          | SHA-512/256          |
| :----: | -------------------- | -------------------- |
|   A    | `0x8C3D37C819544DA2` | `0X22312194FC2BF72C` |
|   B    | `0x73E1996689DCD4D6` | `0X9F555FA3C84C64C2` |
|   C    | `0x1DFAB7AE32FF9C82` | `0X2393B86B6F53B151` |
|   D    | `0x679DD514582F9FCF` | `0X963877195940EABD` |
|   E    | `0x0F6D2B697BD44DA8` | `0X96283EE2A88EFFE3` |
|   F    | `0x77E36F7304C48942` | `0XBE5E1E2553863992` |
|   G    | `0x3F9D85A86A1D36C8` | `0X2B0199FC2C85B8AA` |
|   H    | `0x1112E6AD91D692A1` | `0X0EB72DDC81C52CA2` |

### 4.2 Calculo de la salida.
El digest son los *t* bits más a izquierda que hay al concatenar las palabras A, B, C, D, E, F, G y H del buffer.

