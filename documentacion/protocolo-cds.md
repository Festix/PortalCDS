# Protocolo CDS Mastermeter

## Resumen

Este documento resume el protocolo Bluetooth/UART actualmente usado por el proyecto para el equipo `CDS Mastermeter`.

- Interfaz: Bluetooth SPP / UART ASCII
- Baud rate: `9600`
- Datos: `8`
- Paridad: `none`
- Stop bits: `1`
- Inicio de comando: `:`
- Fin de comando: `;`
- Checksum/CRC: no posee

## Reglas generales

- Todos los comandos deben enviarse completos, incluyendo `:` inicial y `;` final.
- El equipo procesa un comando cuando recibe `;`.
- Si el comando no comienza con `:`, se descarta.
- Las respuestas pueden llegar fragmentadas; no debe asumirse que llegan en un solo bloque.
- En general las respuestas comienzan con `:` y terminan con `;`.
- Excepciones conocidas:
  - `:ResetAforador;` responde `AOK`
  - `:L<estado>;` no genera respuesta

## Consulta principal

### Comando

```text
:v;
```

### Respuesta

```text
:v|<masa>|<caudal>|<presion>|<temperatura>|<tension>|<aforador>|<cargas>|<unidades>|<energia>|<auto_tara>|;
```

## Notas de parseo

- El prefijo `v` es parte de la respuesta y no debe tratarse como un dato de negocio.
- El `|;` final genera un campo vacio extra que debe ignorarse.
- El orden actual de los 10 campos de datos es fijo.

## Campos de `:v;`

| Indice | Campo | Tipo | Unidad | Formato | Descripcion |
| --- | --- | --- | --- | --- | --- |
| 0 | `masa` | number | segun `unidades` | 3 decimales | Masa acumulada mostrada por el equipo |
| 1 | `caudal` | number | `g/s` | 1 decimal | Caudal instantaneo |
| 2 | `presion` | number | `bar` | 1 decimal | Presion medida |
| 3 | `temperatura` | number | `C` | 1 decimal | Temperatura informada por el sensor |
| 4 | `tension` | number | `V` | 1 decimal | Tension de alimentacion o bateria |
| 5 | `aforador` | number | `cnt` | entero sin signo | Contador acumulado del aforador |
| 6 | `cargas` | number | `cnt` | entero sin signo | Cantidad de cargas registradas |
| 7 | `unidades` | enum | - | `1` o `2` | Unidad seleccionada para masa |
| 8 | `energia` | enum | - | `0` o `1` | Estado del modo ahorro de energia |
| 9 | `auto_tara` | enum | - | `0` o `1` | Estado del modo auto tara |

## Valores enum

### `unidades`

| Valor | Significado |
| --- | --- |
| `1` | kg |
| `2` | gramos |

### `energia`

| Valor | Significado |
| --- | --- |
| `0` | desactivado |
| `1` | activado |

### `auto_tara`

| Valor | Significado |
| --- | --- |
| `0` | desactivada |
| `1` | activada |

## Ejemplo

### TX

```text
:v;
```

### RX

```text
:v|12.345|150.2|180.0|24.8|12.1|123456|7|1|0|1|;
```

### Interpretacion

- `masa`: `12.345`
- `caudal`: `150.2`
- `presion`: `180.0`
- `temperatura`: `24.8`
- `tension`: `12.1`
- `aforador`: `123456`
- `cargas`: `7`
- `unidades`: `1` -> `kg`
- `energia`: `0` -> `desactivado`
- `auto_tara`: `1` -> `activada`

## Comandos soportados

| Comando | Tipo | Descripcion | Respuesta |
| --- | --- | --- | --- |
| `:v;` | lectura | Consulta variables principales | `:v|...|;` |
| `:M0;` | accion | Reset de masa | `:M0;` |
| `:Z;` | accion | Ajuste de cero | `:Z;` |
| `:U1;` / `:U2;` | escritura | Seleccion de unidades | `:U<unidad>;` |
| `:u;` | lectura | Consulta unidad actual | `:u<unidad>;` |
| `:E0;` / `:E1;` | escritura | Ahorro de energia | `:E<estado>;` |
| `:e;` | lectura | Consulta ahorro de energia | `:e<estado>;` |
| `:T0;` / `:T1;` | escritura | Auto tara | `:T<estado>;` |
| `:t;` | lectura | Consulta auto tara | `:t<estado>;` |
| `:L0;` / `:L1;` | escritura | Backlight | sin respuesta |
| `:ResetAforador;` | accion | Resetea aforador y cargas | `AOK` |

## Comandos de calibracion

Estos comandos modifican parametros persistidos y no deben exponerse en flujos generales salvo pedido explicito.

- `:A<valor>;` / `:a;` - umbral de bateria baja
- `:B<valor>;` / `:b;` - pendiente de tension
- `:C<valor>;` / `:c;` - pendiente de presion
- `:D<valor>;` / `:d;` - ordenada de presion

## Estado actual en la app

- `index.html` usa `cds-protocol.json` como fuente de verdad para renderizar campos.
- El polling por defecto usa `:v;` cada `4000 ms`.
- La conexion usa timeout de apertura de `15000 ms` con retry/backoff.
- La UI mantiene el log RX/TX crudo para depuracion.
- La UI expone `:M0;` como accion directa de reset de masa.
