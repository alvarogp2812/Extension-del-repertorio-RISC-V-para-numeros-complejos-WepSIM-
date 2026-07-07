# Extension del repertorio RISC-V para numeros complejos (WepSIM)

Practica de la asignatura de Estructura de Computadores realizada con [WepSIM](https://wepsim.github.io/wepsim/), un simulador microprogramado de un procesador tipo RISC-V. El proyecto consiste en disenar una extension del repertorio de instrucciones para representar y operar con numeros complejos directamente en microcodigo, y comparar su rendimiento frente a implementar las mismas operaciones con el repertorio de instrucciones base.

Nota: solo se han subido los ficheros de checkpoint (`e1_checkpoint.txt` y `e2_checkpoint.txt`). Son notebooks en formato JSON (estilo nteract/Jupyter) que exporta WepSIM con el estado completo de la sesion: modo, microcodigo (firmware), programa ensamblador, estado de la simulacion y fecha del checkpoint.

## Contenido del repositorio

| Archivo | Descripcion |
|---|---|
| `e1_checkpoint.txt` | Primer checkpoint de la practica en WepSIM: define unicamente la extension de numeros complejos (sin el repertorio base de instrucciones ni la tabla de registros). |
| `e2_checkpoint.txt` | Segundo checkpoint, mas completo: incluye el ciclo de captacion (`begin`/`fetch`), el repertorio base (`rdcycle`, `add`, `sub`, `mul`, `lw`, `sw`, `beq`), la tabla de alias de registros y la misma extension de numeros complejos, con el programa ensamblador revisado. |

## Extension disenada: numeros complejos

Cada numero complejo se representa mediante un par de palabras (parte real, parte imaginaria) almacenadas en memoria de forma contigua. La extension anade las siguientes instrucciones microprogramadas:

| Instruccion | Semantica |
|---|---|
| `la r1, u32` | Carga un valor inmediato de 32 bits en `BR[r1]` (usada para cargar direcciones). |
| `sc r1, r2, (r3)` | Guarda un numero complejo en memoria: `MEM[r3+0] <- BR[r1]` (parte real), `MEM[r3+4] <- BR[r2]` (parte imaginaria). |
| `lc r1, r2, (r3)` | Carga un numero complejo desde memoria: `BR[r1] <- MEM[r3+0]`, `BR[r2] <- MEM[r3+4]`. |
| `addc r1 r2 r3 r4` | Suma de complejos: `BR[r1] <- BR[r1] + BR[r3]` (parte real), `BR[r2] <- BR[r2] + BR[r4]` (parte imaginaria). |
| `mulc r1 r2 r3 r4` | Multiplicacion de complejos: `BR[r1] <- BR[r1]*BR[r3] - BR[r2]*BR[r4]`; `BR[r2] <- BR[r1]*BR[r4] + BR[r2]*BR[r3]`. |
| `beqc r1 r2 r3 r4 s6` | Salto condicional: si `(r1 == r3)` y `(r2 == r4)`, `PC <- PC + s6` (compara dos numeros complejos por igualdad). |
| `call u20` | Llamada a subrutina: `BR[ra] <- PC`, `PC <- u20`. |
| `ret` | Retorno de subrutina: `PC <- BR[ra]`. |
| `hcf` | Detiene la ejecucion (`halt and catch fire`): pone a cero `PC` y el registro de estado. |

## Programa de prueba

El programa ensamblador (seccion `assembly` de cada checkpoint) define en memoria dos numeros complejos, `a = 35 + 15i` y `b = 10 + 20i`, y calcula lo siguiente para ambos:

- **`no_ext`**: misma logica implementada unicamente con instrucciones del repertorio base (`lw`, `beq`, `add`, `mul`, `sub`), operando parte real e imaginaria por separado.
- **`with_ext`**: misma logica usando la extension de numeros complejos (`lc`, `beqc`, `addc`, `mulc`).

En ambos casos, si las dos partes de `a` y `b` son iguales se realiza una suma; si no lo son, se realiza una multiplicacion. El tiempo de ejecucion de cada version se mide con `rdcycle` (contador de ciclos de reloj) antes y despues de cada llamada, y la diferencia se resta para obtener el numero de ciclos consumidos.

### Resultados obtenidos (segun los comentarios del propio programa)

| Caso | Ciclos sin extension | Ciclos con extension | Mejora |
|---|---|---|---|
| A == B (suma) | 131 | 83 | 36.64% |
| A != B (multiplicacion) | 93 | 74 | 20.43% |

## Diferencias entre los dos checkpoints

- `e1_checkpoint.txt` contiene un firmware minimo: solo la extension de numeros complejos, sin el ciclo de captacion ni el repertorio base, y el programa `no_ext` reutiliza los registros `a0`/`a1` (de entrada) para almacenar tambien el resultado.
- `e2_checkpoint.txt` es la version mas avanzada: incluye el bloque `begin`/`fetch` (ciclo de captacion de instrucciones), el repertorio base completo (`rdcycle`, `add`, `sub`, `mul`, `lw`, `sw`, `beq`) y la tabla de alias de registros (`zero`, `ra`, `sp`, `a0`-`a7`, `t0`-`t6`, `s0`-`s11`, etc.), ademas de un programa `no_ext` corregido que usa registros temporales (`t4`, `t5`, `t6`) en lugar de sobrescribir los argumentos de entrada.

## Como reproducirlo

1. Abrir [WepSIM](https://wepsim.github.io/wepsim/) en el navegador.
2. Importar el fichero de checkpoint (`e1_checkpoint.txt` o `e2_checkpoint.txt`) mediante la opcion de carga de sesion/notebook de WepSIM.
3. Ejecutar el programa ensamblador cargado y observar la diferencia de ciclos de reloj entre las versiones `with_ext` y `no_ext` mediante los registros `s0`, `s1` y `s2`.
