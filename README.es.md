# Jornada — reloj de horarios para ESP32

MVP en C para ESP-IDF 5.0.1, ESP32-WROOM DevKit de 30 pines y DS3231. Interfaz en español, alojada en flash y sin CDN. Cuatro horarios: entrada 08:00, almuerzo 12:00, regreso 13:00 y salida 17:00, lunes a viernes. Zona fija Caracas (UTC−04:00). Pantalla LED preparada mediante una interfaz, todavía sin controlador físico.

## Puesta en marcha

1. Abre esta carpeta como proyecto independiente. No es necesario actualizar ESP-IDF ni editar la configuración global de VS Code.
2. Desde PowerShell, ejecuta `powershell -ExecutionPolicy Bypass -File .\tools\idf.ps1 -Action build`.
3. Con el ESP conectado por USB, usa `powershell -ExecutionPolicy Bypass -File .\tools\idf.ps1 -Action flash -Port COM3`, sustituyendo el puerto. Después ejecuta el mismo comando con `-Action monitor`.
4. Durante el primer arranque se imprime **CLAVE WEB INICIAL**. Guárdala; no se imprime en los siguientes arranques. El monitor también muestra el SSID `Jornada-XXXXXX` y su clave WPA2, distinta de la clave web.
5. Conéctate a esa red y abre **http://192.168.4.1**. El teléfono puede advertir que no tiene Internet: permanece conectado. No hay redirección automática de portal cautivo.
6. Introduce la clave web, cambia la contraseña y comprueba la hora. Si el RTC no tiene hora válida, ajusta fecha y hora desde la web o configura Wi-Fi para sincronizar por NTP.
7. Si guardas una red Wi-Fi, reinicia mediante el botón web en un momento conveniente. También podrás abrir la IP local que muestra el monitor. El punto de acceso sigue disponible.

La primera grabación puede arrancar antes de abrir el monitor. Si no viste la clave inicial, utiliza la recuperación con BOOT descrita a continuación; no hace falta borrar memoria.

El script utiliza las rutas existentes `C:\Espressif\frameworks\esp-idf-v5.0.1` y su entorno Python. Sus cambios de entorno viven en el proceso hijo. Desactiva el gestor de componentes (no hay dependencias externas) y ccache solo en ese proceso. No instala, actualiza ni desinstala herramientas. En otra computadora adapta estas rutas o usa `idf.py build` en una terminal ESP-IDF 5.0.1. Los artefactos quedan en `build/`.

## Recuperar la clave web con BOOT

Esta función se añadió el 13 de septiembre de 2026. Primero graba esta versión actualizada si tu equipo todavía tiene el firmware anterior. Desde esta misma carpeta, cambia `COM3` por tu puerto y ejecuta:

```powershell
powershell.exe -ExecutionPolicy Bypass -File .\tools\idf.ps1 -Action flash -Port COM3
powershell.exe -ExecutionPolicy Bypass -File .\tools\idf.ps1 -Action monitor -Port COM3
```

La acción `flash` compila lo necesario y graba el programa. Esta actualización conserva el formato NVS y la tabla de particiones anteriores; no ejecutes `erase-flash`.

1. Con el monitor abierto, espera a que el equipo arranque. Verás `Recuperacion lista`.
2. Suelta BOOT si lo mantenías pulsado para grabar. Luego, con el programa funcionando, **mantén BOOT durante 5 segundos**. No pulses EN/RESET ni mantengas BOOT al conectar la alimentación: GPIO0 selecciona el cargador de arranque en ese momento.
3. Busca `NUEVA CLAVE WEB: ...`. Copia únicamente los 24 caracteres que siguen a los dos puntos.
4. Suelta BOOT y entra en la página con esa nueva clave. Puedes cambiarla después desde la web.

La clave AP, el SSID y clave de la red de la empresa, las alarmas y el registro de disparos se conservan. Las sesiones web anteriores se invalidan. La recuperación no reinicia el equipo ni activa la campana. Solo genera una clave por pulsación sostenida; si no la capturaste, suelta el botón y repite.

La recuperación requiere acceso físico al botón y al monitor USB. No imprime la clave antigua. Si falla el guardado, informa del error y no anuncia una clave nueva. GPIO0 queda reservado para BOOT; si lo asignas al SSR o I²C en la configuración, esta recuperación se deshabilita y lo indica en el monitor.

## Conexiones de baja tensión

| ESP32 | Destino |
| --- | --- |
| GPIO21 | SDA del DS3231 |
| GPIO22 | SCL del DS3231 |
| 3V3 | VCC del DS3231 |
| GND | GND del DS3231 |
| GPIO25 | CH1 del SSR compatible confirmado por el usuario |

32K y SQW no se utilizan. D+ y D− deben alimentarse según la especificación de tu módulo SSR; no se deduce su alimentación de la compatibilidad de CH1. El GPIO25 es activo alto por defecto. Para módulos activos bajos, selecciona `Jornada - hardware` con `-Action menuconfig`. Verifica los niveles con la campana desconectada. Usa una resistencia externa que mantenga CH1 inactivo durante reset (pull-down para activo alto; pull-up a 3,3 V para activo bajo, si corresponde a su interfaz).

Mantén las resistencias pull-up de I²C a 3,3 V. El DS3231 admite esa alimentación; no lleves señales de 5 V a GPIO. La inscripción de batería informada, “CS3031”, no permite confirmar su química ni compatibilidad con posibles circuitos de carga del módulo. Comprueba esa inscripción y el portabatería antes de alimentar el módulo; el firmware no gestiona la carga.

La campana opera a 110 V AC. Instala esa parte en una caja adecuada, con protección y cableado dimensionados para la carga, a cargo de personal competente. El programa no puede garantizar que un SSR averiado deje de conducir. No se incluye una asignación de bornes de potencia sin la documentación específica del módulo.

## Comportamiento temporal

- RTC en **UTC**, presentación y horarios en Caracas. Si antes guardaba hora local, ajusta una vez desde la web.
- Lectura al arrancar y cada 60 segundos. Si difiere más de 2 segundos, se corrige el reloj del sistema. NTP cada 6 horas mediante `pool.ntp.org`; cada respuesta válida actualiza el RTC. Esto ajusta fecha/hora, no el registro de envejecimiento del cristal.
- OSF (oscilador detenido), error I²C o fecha fuera de rango impiden usar el RTC al arrancar. Sin hora válida, las alarmas se suspenden. NTP o ajuste manual habilitan el reloj.
- Si el RTC falla después de adquirir hora, continúa con el reloj del ESP32 y muestra el fallo. En un nuevo corte de energía necesitará de nuevo RTC válido, NTP o ajuste manual.
- Solo se dispara al observar el paso normal de un minuto al siguiente. Arrancar a las 08:00:20 **no** recupera la alarma de las 08:00. El minuto de arranque y el de una corrección detectada se omiten; es una decisión conservadora para no generar avisos tardíos.
- Cada disparo se registra en NVS **antes** de energizar el SSR. Si falla la escritura, se omite. Si se corta la energía justo después de registrar y antes de sonar, puede perderse ese aviso, pero no se repite al volver.
- El registro por alarma conserva el último minuto UTC ejecutado. Un retroceso de hora no reproduce avisos anteriores; si se disparó con una fecha futura incorrecta, se inhiben hasta superar esa fecha. Revisa la hora antes de habilitar el equipo.
- Las alarmas simultáneas se combinan en un solo patrón; una campana ocupada no acumula avisos. Las pruebas manuales tienen una pausa de 30 segundos y no modifican el registro de alarmas.
- Sonido predeterminado: tres pulsos de 1000 ms separados por 1000 ms. Límites: 1–5 pulsos; encendido y pausa de 200–3000 ms. Prueba audibilidad en la planta: el software no certifica cobertura acústica ni vida útil.

## Arquitectura

`main.c` coordina NVS, Wi-Fi, la tarea de campana y el planificador. Un mutex protege configuración y estado; la campana consume una cola de capacidad uno con una copia del patrón. Sus tiempos usan FreeRTOS y no dependen de correcciones NTP.

`recovery.c` supervisa BOOT en una tarea independiente; `recovery_button.c` exige una liberación inicial estable y una pulsación continua de cinco segundos. Solo se actualizan el salt y hash de la clave web, bajo el mismo mutex que protege las operaciones web. Se comprueba de nuevo la sesión tras recibir el cuerpo de una petición para rechazar peticiones que estuvieran llegando durante una recuperación.

`clock_service.c` gestiona DS3231 y SNTP; `scheduler.c` contiene la regla temporal pura y comprobable; `web.c` valida solicitudes y sirve la interfaz embebida `index.html`.

`display_tick(utc, valid)` es una implementación débil vacía. Un futuro componente puede reemplazarla con un envío de cuadros a una cola y una tarea RMT para la tira elegida. Debe retornar inmediatamente. Los segmentos de `counter.h` se pueden reutilizar como datos; no se incorpora FastLED ni su cuenta bloqueante. Su función `offCounter()` usa `<= NUM_LEDS` y necesita corregirse a `< NUM_LEDS`; las actualizaciones deben mostrar una trama completa en lugar de llamar `show()` por LED. La futura configuración LED será independiente de horarios y campana.

## Acceso y persistencia

La contraseña web se guarda como SHA-256 con salt aleatorio; la sesión usa un token aleatorio de 256 bits, dura una hora y se conserva solo en memoria del navegador. Un nuevo inicio invalida la sesión anterior. Cambiar clave, salir o reiniciar invalida la sesión. Cada intento de inicio requiere al menos dos segundos. Usa una contraseña larga y exclusiva: esta versión no incorpora un KDF lento contra extracción física de flash.

La API exige JSON y token en `Authorization`; no habilita CORS. No hay cookies de sesión ni secretos en las respuestas de estado. El AP utiliza WPA2 con clave individual aleatoria. Las claves Wi-Fi están en NVS sin cifrado; esta versión no activa secure boot ni flash encryption.

**HTTP local no cifra la contraseña web ni la sesión dentro de la red.** Usa el AP protegido o una red de administración confiable y no expongas el puerto en Internet. TLS, roles, auditoría histórica y actualización OTA quedan fuera del MVP. NTP no está autenticado. La interfaz muestra la última sincronización desde el arranque; ese dato no se persiste.

No se borra automáticamente NVS ante corrupción. Para recuperar una clave web perdida utiliza BOOT como se describe arriba. El borrado completo de flash es una operación de mantenimiento distinta que elimina la configuración; no es necesario para restablecer el acceso con esta versión.

## Verificación

Prueba portable del planificador (GCC):

```powershell
gcc -std=c11 -Wall -Wextra -Werror -I main main/scheduler.c tests/test_scheduler.c -o test_scheduler.exe
.\test_scheduler.exe
```

Antes de usar la campana en producción, verifica en el hardware: polaridad y reposo al reset; marcha de RTC con batería tras un corte; los cuatro horarios y fines de semana; reinicio en el minuto de alarma; ajustes hacia delante/atrás; ausencia de Internet; conexión con credenciales incorrectas; cambio de contraseña; sonido y temperatura del módulo. La compilación y las pruebas de escritorio no sustituyen estas comprobaciones físicas.

## Referencias

- [DS3231, ficha técnica Analog Devices](https://www.analog.com/media/en/technical-documentation/data-sheets/ds3231.pdf)
- [Hora de sistema y SNTP, Espressif](https://docs.espressif.com/projects/esp-idf/en/v5.0.1/esp32/api-reference/system/system_time.html)
- [Servidor HTTP ESP-IDF 5.0.1](https://docs.espressif.com/projects/esp-idf/en/v5.0.1/esp32/api-reference/protocols/esp_http_server.html)
