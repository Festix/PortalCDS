# PortalCDS

Rama de validación técnica para comprobar conexión a un dispositivo Bluetooth SPP/RFCOMM desde Chrome Android 138+ usando Web Serial.

## Demo

- `index.html`: demo estática para abrir puerto, leer y escribir.

## URL de verificación

- `https://festix.github.io/PortalCDS/`

## Criterio de éxito

- Chrome Android 138+ expone `navigator.serial`.
- El selector muestra el dispositivo RFCOMM.
- Se puede abrir el puerto, leer y escribir datos.
