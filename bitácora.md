# Bitácora del Equipo - Proyecto Integrador y Prácticas

## Sesión 1: 14 de Octubre de 2026 (Práctica 1)
* **Fecha:** 14 de Octubre de 2026
* **Duración:** 4.5 horas (1.5 h de laboratorio + 3 h de trabajo autónomo)
* **Integrantes presentes:**
  * Integrante 1 (Lizett): Configuración del toolchain del ESP32, carga del Listado 1 y 2, y medición de memoria.
  * Integrante 2 (Norma): Programación e implementación del superlazo no bloqueante (Listado 3) y pruebas de rendimiento MAC (Listado 4).
  * Integrante 3 (Gael): Instalación, configuración por SSH de la Raspberry Pi, ejecución del benchmark en Python/NumPy (Listado 5 y 6) y creación de la estructura del repositorio.
* **Actividades realizadas:**
  * Creación y clonación del repositorio de GitHub con la estructura de carpetas requerida.
  * Verificación de la compilación cruzada y obtención de los archivos `.elf`, `.bin` y `.map` con la salida verbosa del IDE.
  * Ejecución de pruebas del benchmark MAC a 240 MHz, 160 MHz y 80 MHz.
  * Configuración de la Raspberry Pi OS Lite en la SBC, aislamiento de entorno virtual con Python e instalación de NumPy.
* **Problemas encontrados y soluciones:**
  * *Problema:* Al intentar cargar el firmware al ESP32 por primera vez aparecía el error "A fatal error occurred: Failed to connect to ESP32".
  * *Solución:* Se mantuvo presionado el botón `BOOT` de la tarjeta al momento de iniciar la carga en el IDE de Arduino hasta que el conector serial reconoció el puerto.
  * *Problema:* El optimizador del compilador eliminaba las funciones del benchmark al no detectar un uso directo del retorno.
  * *Solución:* Se implementó la palabra reservada `volatile` y la barrera de compilador `asm volatile(""::: "memory")` tal como se indicaba en el Listado 4.
