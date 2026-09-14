# No aparece CLAVE WEB INICIAL

La versión anterior mostraba esa clave solo una vez, y podía hacerlo antes de abrir el monitor. La clave AP no sirve para entrar a la web.

La actualización añade recuperación con BOOT sin borrar horarios ni redes. Ya está en esta carpeta. Cambia COM3 por tu puerto y, desde la carpeta `jornada`, ejecuta:

```powershell
powershell.exe -ExecutionPolicy Bypass -File .\tools\idf.ps1 -Action flash -Port COM3
powershell.exe -ExecutionPolicy Bypass -File .\tools\idf.ps1 -Action monitor -Port COM3
```

Espera a que arranque y muestre `Recuperacion lista`. Con el ESP32 encendido, suelta BOOT y luego mantenlo presionado durante 5 segundos. No pulses EN/RESET.

El monitor mostrará:

```text
NUEVA CLAVE WEB: <24 caracteres>
```

Copia los 24 caracteres como contraseña de la página y suelta BOOT. Si no viste el mensaje, suelta y vuelve a mantener el botón cinco segundos. La clave Wi-Fi del AP no cambia.

No ejecutes `erase-flash`. El nuevo firmware se compiló y su lógica de pulsación se probó en escritorio; queda por comprobar el botón y la recuperación en tu placa real.
