# Práctica 1 · Plataforma de desarrollo

## 1. Integrantes y roles

| Integrante | Actividades |
|---|---|
| Izquierdo Garnica Lizett | Configurar el ESP32; registrar los datos de plataforma y memoria (Tablas 4 y 5); documentar conexiones y evidencias. |
| Salceda Solórzano Norma Angélica | Implementar y medir el superlazo (Tabla 6); ejecutar el benchmark MAC del ESP32 (Tabla 7) y elaborar la gráfica. |
| Sánchez Cornejo Gael Antonio | Configurar la Raspberry Pi por SSH; ejecutar el benchmark de SBC y completar la Tabla 8. |
| Todo el equipo | Responder el cuestionario, redactar conclusiones individuales, revisar el README y actualizar la bitácora. |

## 2. Objetivo

Configurar y verificar el toolchain de un ESP32 y una SBC, estudiar el uso de memoria, implementar tareas periódicas no bloqueantes y comparar el costo de MAC float32 e int8.

## 3. Plataforma y conexiones

- **ESP32:** ESP32-D0WD-V3, revisión v3.1, dos núcleos a 240 MHz, 4 MiB de flash y sin PSRAM.
- **Toolchain:** Arduino-ESP32 3.3.12 / ESP-IDF v5.5.5.
- **Puerto serie:** `/dev/cu.usbserial-0001`.
- **SBC:** por medir. Agrega modelo, conexiones, diagrama o fotografía y datos de SSH.

## 4. Desarrollo, decisiones y problemas

### Desarrollo

Primero se revisó el material de la práctica y se organizó el proyecto en las carpetas `firmware/` y `datos/`. Después se conectó la placa ESP32 a la computadora mediante USB y se identificó el puerto serie `/dev/cu.usbserial-0001`, que aparece en los registros de carga.

Se utilizó Arduino IDE con el núcleo Arduino-ESP32 versión 3.3.12 y ESP-IDF v5.5.5. Se compiló y cargó el programa de información de la plataforma para identificar el modelo y revisión del chip, la cantidad de núcleos, la frecuencia de CPU, la flash disponible, el heap, la PSRAM y la dirección MAC. Los registros corresponden a un ESP32-D0WD-V3 v3.1, de dos núcleos y 240 MHz, con 4 MiB de flash y sin PSRAM.

Luego se compilaron las tres variantes del programa de memoria. En la primera, el arreglo `pesos` se declaró como `const`; en la segunda, se declaró como variable inicializada; y en la tercera, como variable sin inicializar. Se compararon los resúmenes del IDE y las secciones del archivo ELF para observar cómo cambia el uso de `.flash.rodata`, `.dram0.data` y `.dram0.bss`.

También se compilaron las variantes del superlazo. El programa está diseñado para cambiar el estado del LED cada 250 ms y emitir un reporte por segundo. Para comparar el efecto de las operaciones de espera y de la comunicación serial, la práctica pide medir el lazo sin modificaciones, con `delay(1)` y con un `Serial.printf` en cada iteración. Los registros incluidos muestran que los programas se compilaron, pero no contienen las mediciones de periodo del LED ni de iteraciones por segundo.

Finalmente se ejecutó el benchmark de MAC en el ESP32 para comparar operaciones float32 e int8 a distintas frecuencias. Los registros contienen resultados para N=1024 a 240, 160 y 80 MHz, además de resultados de otros tamaños de vector. Se conservaron las unidades de ns/MAC y ciclos/MAC que imprime el programa.

### Decisiones

Se usaron los registros generados por el IDE y por los programas como fuente para completar las tablas, en lugar de estimar valores que no aparecían en los archivos. Para el análisis de memoria se compararon tanto el resumen de variables globales del IDE como las secciones del ELF, porque cada salida permite observar aspectos distintos del almacenamiento.

El arreglo de pesos se dejó constante en la variante correspondiente para comprobar si podía almacenarse en flash y evitar ocupar RAM durante la ejecución. Para el benchmark se separaron los datos por tamaño de vector y frecuencia, de manera que las comparaciones se hicieran entre configuraciones equivalentes.

Los datos que no aparecen en el ZIP —como los resultados de la SBC, las mediciones del superlazo y algunas ejecuciones del benchmark— se señalaron como pendientes. Así se evita presentar valores estimados como si fueran mediciones realizadas.

### Problemas encontrados

Durante la ejecución de los programas en Arduino IDE, en algunas ocasiones la pantalla de la aplicación quedaba negra o la interfaz se trababa. El IDE dejaba de responder y no era posible continuar normalmente con la compilación o la carga del programa. Cuando ocurría, había que forzar la salida de la aplicación, volver a abrir Arduino IDE y repetir los pasos necesarios: seleccionar la placa y el puerto, abrir el programa, compilarlo y, cuando correspondía, cargarlo nuevamente y consultar el monitor serie.

Este problema interrumpió el flujo de trabajo y podía obligar a repetir operaciones que ya se habían realizado. Para conservar los resultados disponibles, se utilizaron los archivos de salida guardados en la carpeta `datos/`. Cuando no había una salida completa —por ejemplo, para las mediciones del superlazo o de la SBC—, se dejó el dato pendiente en vez de completarlo sin evidencia.

También se observó que algunos registros de compilación no contenían toda la información necesaria para llenar las tablas. En particular, los registros del superlazo muestran datos de compilación, pero no las mediciones de ejecución que solicita la práctica. Asimismo, los registros disponibles del benchmark no incluyen todas las combinaciones requeridas; por eso se deben repetir las pruebas faltantes y guardar la salida completa.

Para evitar perder información en futuras ejecuciones, conviene guardar cada salida del monitor serie y de compilación en un archivo separado, anotar la configuración usada —placa, frecuencia y tamaño del vector— y verificar que el registro incluya el resultado final antes de cerrar Arduino IDE.

## 5. Resultados

### Tabla 4. Datos de la plataforma

| Dato | Reportado por firmware | Hoja de datos / referencia | ¿Coincide? |
|---|---|---|---|
| Modelo y revisión | ESP32-D0WD-V3 v3.1 | ESP32 clásico, doble núcleo Xtensa LX6 | Sí |
| Núcleos y frecuencia | 2 núcleos, 240 MHz | 2 núcleos, hasta 240 MHz | Sí |
| Tamaño de flash | 4,194,304 bytes (4 MiB) | Flash SPI externa; tamaño exacto depende del módulo | Por confirmar |
| Heap total, libre y bloque máximo | 377,696 / 331,676 / 110,580 bytes | Depende del firmware | N/A |
| PSRAM | 0 bytes | No disponible en esta configuración | Sí |
| Firmware / espacio OTA siguiente | 274,368 / 1,310,720 bytes | Partición OTA de 1.25 MiB | Compatible |
| Núcleo Arduino / ESP-IDF | 3.3.12 / v5.5.5 | Versiones del entorno | N/A |
| MAC Wi-Fi (STA) | C8:85:41:E3:70:5C | Identificador de esta placa | N/A |

### Tabla 5. Uso de memoria (bytes)

| Programa | Compilación IDE | .flash.text | .flash.rodata | .dram0.data | .dram0.bss | Memoria dinámica IDE |
|---|---:|---:|---:|---:|---:|---:|
| Listado 1 (base) | 274,216 | Por medir | Por medir | Por medir | Por medir | 22,236 |
| Listado 2 (a) con `const` | 285,616 | 133,320 | 68,568 | 16,849 | 5,264 | 22,116 |
| Listado 2 (b) sin `const` | 285,616 | 133,320 | 52,184 | 33,233 | 5,264 | 38,500 |
| Listado 2 (c) sin inicializar | 269,232 | 133,320 | 52,184 | 16,849 | 21,648 | 38,500 |

Los datos de las secciones provienen de `size -A` en los registros 2a, 2b y 2c. La columna “memoria dinámica IDE” es el resumen de variables globales del compilador; no equivale al heap libre en ejecución.

### Tabla 6. Superlazo

| Condición | Periodo LED medido (ms) | Iteraciones por segundo | Error frente a 500 ms (%) |
|---|---:|---:|---:|
| (a) Listado 3 tal cual | Por medir | Por medir | Por medir |
| (b) Con `delay(1)` | Por medir | Por medir | Por medir |
| (c) Con `Serial.printf` por iteración | Por medir | Por medir | Por medir |

El LED cambia de estado cada 250 ms; un ciclo completo corresponde a 500 ms (2 Hz). Calcula el error con `abs(periodo_medido - 500) / 500 * 100`.

### Tabla 7. Costo de MAC en el ESP32

| Configuración | float32 (ns/MAC) | float32 (ciclos/MAC) | int8 (ns/MAC) | int8 (ciclos/MAC) |
|---|---:|---:|---:|---:|
| 240 MHz, N = 1024 | 25.60 | 6.14 | 59.03 | 14.17 |
| 160 MHz, N = 1024 | 38.45 | 6.15 | 88.72 | 14.20 |
| 80 MHz, N = 1024 | 77.39 | 6.19 | 178.52 | 14.28 |
| 240 MHz, N = 256 | Por medir | Por medir | Por medir | Por medir |
| 240 MHz, N = 4096 | 25.13 | 6.03 | 58.59 | 14.06 |

El registro de N = 256 no contiene una medición legible a 240 MHz; hay que repetirla.

### Tabla 8. Datos y rendimiento de la SBC

| Dato | Valor |
|---|---|
| Modelo (`/proc/device-tree/model`) | Por medir |
| Arquitectura (`uname -m`) y núcleos (`nproc`) | Por medir |
| RAM total y disponible | Por medir |
| Espacio libre en microSD | Por medir |
| Temperatura en reposo | Por medir |
| Dirección IP | Por medir |
| Python puro (ns/MAC) | Por medir |
| NumPy float32 (ns/MAC) | Por medir |
| NumPy int8 → int32 (ns/MAC) | Por medir |
| ESP32 float32 a 240 MHz | 25.60 ns/MAC |

**Gráfica obligatoria:** ns/MAC contra frecuencia de CPU, con una serie para float32 y otra para int8. Usa los tres resultados de N = 1024 de la Tabla 7.

## 6. Análisis y cuestionario

1. **ISA y compilación cruzada.** El ESP32-D0WD-V3 usa Xtensa LX6, una ISA RISC de 32 bits. Una Raspberry Pi 4 o 5 normalmente usa Arm/AArch64, también RISC; confírmalo con `uname -m` y `lscpu`. Ambas usan instrucciones relativamente simples y regulares. Como la computadora compila para una ISA distinta, se requiere un compilador cruzado; el binario generado para Xtensa no se ejecuta directamente en la computadora.

2. **Ubicación de los pesos.** En (a), `const` coloca el arreglo en `.flash.rodata`, sin consumir RAM. En (b), los datos inicializados ocupan `.dram0.data` y su valor inicial también está en flash; el uso de memoria dinámica sube de 22,116 a 38,500 bytes, una diferencia de 16,384 bytes. En (c), el arreglo sin inicializar ocupa `.dram0.bss`, que aumenta de 5,264 a 21,648 bytes. Para guardar pesos, conviene declararlos como constantes y confirmar su ubicación en el mapa del enlazador.

3. **Temporización y desbordamiento.** `tLed += T_LED_MS` mantiene la fase prevista; asignar `tLed = ahora` incorpora el retraso de ejecución a cada periodo y puede acumular deriva. Por ejemplo, con contador uint32, `t0 = 4,294,967,290`, `T = 10` y `ahora = 4`, la resta modular `ahora - t0` da 10, así que `ahora - t0 >= T` funciona tras el desbordamiento. Comparar tiempos absolutos con `ahora >= t0 + T` puede fallar porque la suma también se desborda. La resta es segura para intervalos menores que el rango del contador.

4. **Escalamiento con frecuencia.** Los tiempos aumentan aproximadamente en proporción inversa a la frecuencia, mientras los ciclos por MAC casi no cambian. Para float32 son 6.14, 6.15 y 6.19 ciclos/MAC a 240, 160 y 80 MHz; para int8 son 14.17, 14.20 y 14.28. Las pequeñas diferencias pueden deberse a la medición y a interrupciones.

5. **int8 frente a float32.** En esta placa, int8 fue más lento: 59.03 ns/MAC frente a 25.60 ns/MAC para float32 a 240 MHz. El ESP32 clásico tiene unidad de punto flotante; el código int8 medido usa operaciones enteras sin una instrucción MAC vectorial dedicada. Aun así, int8 ocupa cuatro veces menos espacio por peso: 1 byte en lugar de 4.

6. **¿Cabe el modelo?** Los pesos de 300 KiB (307,200 bytes) caben en la partición de aplicación de 1,310,720 bytes junto al firmware de 274,368 bytes; quedarían aproximadamente 729,152 bytes. Los pesos deberían vivir en flash como datos constantes. Una arena de 120 KiB (122,880 bytes) no cabe como variable global junto a las variables actuales: 22,236 + 122,880 = 145,116 bytes, por encima del límite aproximado de 124,580 bytes indicado en la guía. Tampoco cabe en un bloque del heap en la medición observada: el bloque máximo es 110,580 bytes. El modelo de 40 MB no cabe en la flash de 4 MiB ni en la partición OTA de 1.25 MiB; habría que reducirlo, usar almacenamiento externo compatible o elegir otra plataforma.

7. **ESP32, Python y NumPy.** Falta la salida del benchmark de la SBC, así que no se puede hacer una comparación numérica todavía. El ESP32 midió 25.60 ns/MAC en float32 a 240 MHz. Agrega los valores de Python puro y NumPy para comparar el efecto del hardware y de la implementación.

8. **Bootloader y particiones.** El bootloader de segunda etapa lee la tabla de particiones y carga la aplicación seleccionada. En el esquema OTA usado, la partición de aplicación de 1.25 MiB limita el tamaño del firmware. Si el modelo no cabe, se puede optimizar o reducir, cambiar el esquema para asignar una partición mayor, usar almacenamiento externo compatible o ejecutarlo en una SBC.

## Evidencias

### Fotografías y capturas

- [Foto 1](evidencias/1.png)
- [Foto 2](evidencias/2.png)
- [Foto 3](evidencias/3.png)
- [Foto 4](evidencias/4.png)
- [Foto 5](evidencias/5.png)
- [Foto 6](evidencias/6.png)
- [Captura del 19 de octubre, 1:01 p. m.](<evidenciasCaptura de pantalla 2026-10-19 a la(s) 1.01.59 p.m..png>)
- [Captura del 19 de octubre, 1:04 p. m.](<evidencias/Captura de pantalla 2026-10-19 a la(s) 1.04.46 p.m..png>)
- [Captura del 19 de octubre, 5:45 p. m.](<evidencias/Captura de pantalla 2026-10-19 a la(s) 5.45.46 p.m..png>)
- [Captura del 19 de octubre, 12:17 p. m.](<evidencias/Captura de pantalla 2026-10-19 a la(s) 12.17.02 p.m..png>)
- [Captura del 19 de octubre, 12:17 p. m.](<evidencias/Captura de pantalla 2026-10-19 a la(s) 12.17.07 p.m..png>)
- [Captura del 19 de octubre, 12:17 p. m.](<evidencias/Captura de pantalla 2026-10-19 a la(s) 12.17.11 p.m..png>)
- [Captura del 19 de octubre, 12:18 p. m.](<evidencias/Captura de pantalla 2026-10-19 a la(s) 12.18.10 p.m..png>)
- [Captura del 19 de octubre, 12:28 p. m.](<evidencias/Captura de pantalla 2026-10-19 a la(s) 12.28.56 p.m..png>)
- [Captura del 19 de octubre, 12:59 p. m.](<evidencias/Captura de pantalla 2026-10-19 a la(s) 12.59.24 p.m..png>)
- [Captura del 19 de octubre, 12:59 p. m.](<evidencias/Captura de pantalla 2026-10-19 a la(s) 12.59.28 p.m..png>)
- [Captura del 19 de octubre, 12:59 p. m.](<evidencias/Captura de pantalla 2026-10-19 a la(s) 12.59.40 p.m..png>)
- [Captura del 19 de octubre, 12:59 p. m.](<evidencias/Captura de pantalla 2026-10-19 a la(s) 12.59.43 p.m..png>)
- [Captura del 19 de octubre, 1:00 p. m.](<evidencias/Captura de pantalla 2026-10-19 a la(s) 12.59.55 p.m..png>)
- [Captura del 20 de octubre, 1:08 p. m.](<evidencias/Captura de pantalla 2026-10-20 a la(s) 1.08.47 p.m..png>)
- [Captura del 20 de octubre, 1:11 p. m.](<evidencias/Captura de pantalla 2026-10-20 a la(s) 1.10.56 p.m..png>)

### Videos

- [Superlazo sin modificaciones](<evidencias/listado3a_tal cual.mp4>)
- [Superlazo con delay de 1 ms](evidencias/listado3b_delay1.mp4)
- [Video v1](evidencias/v1.mp4)
- [Video v2](evidencias/v2.mp4)

## 7. Conclusiones individuales

[Agrega un párrafo por integrante sobre lo que aprendió y los resultados que verificó.]

## 8. Uso de IA

Se usó ChatGPT/Codex para organizar los datos presentes en los registros, hacer cálculos y preparar explicaciones iniciales. El equipo debe verificar las cifras contra las salidas originales y la documentación. Las mediciones faltantes se dejaron pendientes.

## 9. Referencias

- Lee, E. A. y Seshia, S. A. (2017). *Introduction to Embedded Systems: A Cyber-Physical Systems Approach*, 2.ª ed., capítulos 8 y 9.
- Espressif Systems. *ESP32 Series Datasheet* y *ESP-IDF Programming Guide*.
- Raspberry Pi Ltd. *Raspberry Pi OS Documentation*.
- White, E. (2024). *Making Embedded Systems*, 2.ª ed.
- Warden, P. y Situnayake, D. (2019). *TinyML*.