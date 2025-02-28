# README

# Algoritmos Simétricos
Los algoritmos simétricos de cifrado son los que usan la misma clave para cifrar y descifrar. Se dividen en:
- **Algoritmos simétricos de bloque**: Estos algoritmos funcionan operando grupos de texto claro de una longitud fija llamados bloques (Ejem: El **DES** opera bloques de 64 bits y el **AES** de 128 bits).
- **Algoritmos simétricos de flujo**: Estos algoritmo funcionan generando un flujo pseudo-aleatorio de datos que se operan con el texto claro para cifrarlo.

Los algoritmos simétricos documentados aquí son:
- **De bloque**
	- [DES](Algoritmos/DES.md)

## Información adicional
- Cuando el texto a cifrar no cabe en bloques exactos se le añaden bytes adiciones llamados [padding](Conceptos/Padding.md).
- Los algoritmos simétricos de bloque no son seguros si simplemente se cifra cada bloque y no se hace nada más, por ello es importante conocer los **[modos de operación](Conceptos/Modos%20de%20operaci%C3%B3n.md)**.