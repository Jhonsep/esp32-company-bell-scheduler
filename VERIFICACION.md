# Verificación del MVP

Fecha: 10 de septiembre de 2026.

| Comprobación | Resultado |
| --- | --- |
| Compilación ESP-IDF 5.0.1, objetivo ESP32 | Correcta; imagen y bootloader generados |
| Planificador C compilado con GCC y `-Wall -Wextra -Werror` | Correcto |
| Días de semana, desactivación, minuto incorrecto | Correcto |
| Saltos adelante/atrás, duplicados y registro futuro | Correcto |
| Cambio de día a medianoche | Correcto |
| Navegador Chromium: acceso, cuatro alarmas, carga y guardado | Correcto con API simulada |
| Payload de guardado: hora modificada y máscara lunes–viernes | Correcto con API simulada |
| Vista de 390 píxeles y cierre de sesión | Correcto; sin desbordamiento horizontal |
| Errores JavaScript durante la prueba | Ninguno |
| Inspección visual escritorio y móvil | Realizada; capturas incluidas |
| Sustitución del hook SNTP débil de ESP-IDF | Verificada en ELF: símbolo fuerte `sntp_sync_time` |
| Grabación USB y funcionamiento con RTC/SSR/campana reales | Pendiente; no se grabó ningún dispositivo |
| Pruebas HTTP contra firmware ejecutándose en ESP32 | Pendientes |
| Corte de energía, fallo de batería, reconexión Wi-Fi y sincronización NTP real | Pendientes |

La prueba de navegador usa respuestas simuladas y no valida el servidor C en ejecución. Las pruebas del planificador verifican su regla pura; la integración FreeRTOS, NVS y GPIO necesita la placa.

Las capturas presentan datos de demostración. Los horarios del firmware siguen siendo 08:00, 12:00, 13:00 y 17:00.

Para repetir la prueba de interfaz instala Playwright en un entorno de pruebas y ejecuta `node tests/test_ui.cjs`. Puedes indicar `PLAYWRIGHT_MODULE` con la ruta de un paquete ya instalado y `CHROME_PATH` con un navegador compatible. La prueba no se conecta a un ESP32 ni activa la campana.


## Actualización de recuperación — 13 de septiembre de 2026

- Compilación ESP-IDF 5.0.1: correcta con recuperación BOOT y protección de sesión bajo mutex.
- `tests/test_recovery_button.c`, compilado con GCC y `-Wall -Wextra -Werror`: correcto. Cubre botón pulsado al iniciar, umbral de cinco segundos, una sola recuperación por pulsación, liberación estable, pulsaciones cortas y tiempos superiores a 32 bits.
- Conservación del formato y versión del blob NVS y de la tabla de particiones: verificada en código. La recuperación copia la configuración existente y cambia únicamente salt/hash; no escribe el registro de disparos.
- Prueba web repetida en Chromium con API simulada: correcta (acceso, guardado, vista móvil y cierre de sesión).
- Prueba física del botón, persistencia de la nueva clave y rechazo de sesiones antiguas en un ESP32: pendiente. No se grabó ni borró ningún dispositivo.

Para repetir la prueba de pulsación:

```powershell
gcc -std=c11 -Wall -Wextra -Werror -I main main/recovery_button.c tests/test_recovery_button.c -o test_recovery_button.exe
.\test_recovery_button.exe
```
