# Lab 6 — Modos de Direccionamiento x86
---

## Descripción
Programa en NASM (formato `.COM`) que demuestra cuatro modos de direccionamiento de la arquitectura x86 en modo real de 16 bits. Cada modo accede a los mismos datos de forma distinta y fue verificado instrucción por instrucción con DEBUG dentro de DOSBox.

---
## Requirimientos
- DOSBox 0.74+
- NASM 2.11+ para DOS
- DEBUG.COM.


---
## Estructura de Datos en Memoria

| Símbolo    | Tipo        | Valor inicial | Offset (hex) |
|------------|-------------|---------------|--------------|
| `array`    | 5 × word    | 10,20,30,40,50| 0102h        |
| `nota1`    | word        | 85            | 010Ch        |
| `nota2`    | word        | 73            | 010Eh        |
| `promedio` | word        | 0             | 0110h        |
| `var_x`    | word        | 0FFFFh        | 0112h        |

---

## Tabla Resumen — 4 Modos de Direccionamiento

| # | Modo | Fórmula Dirección Efectiva | Instrucción NASM usada | Valor observado en DEBUG |
|---|------|----------------------------|------------------------|--------------------------|
| 1 | **Inmediato** | No aplica — el dato está dentro del opcode | `MOV ax, 100` | AX = 0064h (100 decimal) |
| 2 | **Directo** | EA = offset del símbolo (fijo en compilación) | `MOV ax, [var_x]` | AX = FFFFh → luego 0000h tras escritura |
| 3 | **Indirecto por registro** | EA = contenido del registro (SI) | `MOV si, nota1` / `MOV ax, [si]` | SI = 010Ch → AX = 0055h (85 decimal) |
| 4 | **Indexado** | EA = BX + SI (base + índice) | `ADD ax, [bx + si]` | AX = 0096h (150 decimal) al finalizar bucle |

---

## Checkpoint 1 — Estructura en Memoria

Comando usado en DEBUG:
```
D CS:100
```

Volcado hexadecimal observado (little-endian):

```
075A:0100  EB 22 0A 00 14 00 1E 00-28 00 32 00 55 00 49 00
```

| Bytes      | Valor decimal | Dato         |
|------------|---------------|------------- |
| `EB 22`    | —             | JMP inicio   |
| `0A 00`    | 10            | array[0]     |
| `14 00`    | 20            | array[1]     |
| `1E 00`    | 30            | array[2]     |
| `28 00`    | 40            | array[3]     |
| `32 00`    | 50            | array[4]     |
| `55 00`    | 85            | nota1        |
| `49 00`    | 73            | nota2        |

---

## Checkpoint 2 — Trazado Modo Indirecto por Registro

Comando usado: `G 142` para llegar al bloque, luego `T` paso a paso.

| Instrucción          | Registro modificado | Valor resultante        |
|----------------------|---------------------|-------------------------|
| `MOV SI, 010C`       | SI                  | 010Ch (dirección nota1) |
| `MOV AX, [SI]`       | AX                  | 0055h = 85 decimal      |
| `MOV SI, 010E`       | SI                  | 010Eh (dirección nota2) |
| `MOV BX, [SI]`       | BX                  | 0049h = 73 decimal      |
| `ADD AX, BX`         | AX                  | 009Eh = 158 decimal     |
| `SHR AX, 1`          | AX                  | 004Fh = 79 decimal      |

---

## Checkpoint 3 — Verificación Modo Indexado + Extensión Inversa

### Bucle original (array[0] → array[4])
```nasm
XOR  si, si
.bucle_array:
    ADD  ax, [bx + si]   ; AX += array[si/2]
    ADD  si, 2
    LOOP .bucle_array
; Resultado: AX = 10+20+30+40+50 = 150 (0096h) 
```

### Extensión: bucle inverso (array[4] → array[0])
```nasm
MOV  si, 8
.bucle_inverso:
    ADD  ax, [bx + si]   ; AX += array[si/2]
    SUB  si, 2
    LOOP .bucle_inverso
; Resultado: AX = 50+40+30+20+10 = 150 (0096h) 
```

Ambos producen `AX = 0096h = 150 decimal`, verificado en DEBUG.

---
