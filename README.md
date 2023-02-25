# IOT ESP server

## Servidor web usando ESP32/ESP8266

Usando la placa ESP32/ESP8266 en la carpeta del dispositivo. De forma predeterminada, cuando se graba el firmware de MicroPython,  se crea un archivo [boot.py](https://github.com/ericksc/esp_wifi_server/blob/main/boot.py).

Para este proyecto necesitarás un archivo [boot.py](https://github.com/ericksc/esp_wifi_server/blob/main/boot.py) y un archivo [main.py](https://github.com/ericksc/esp_wifi_server/blob/main/main.py). El  archivo boot.py tiene el código que solo necesita ejecutarse una vez en el arranque. Esto incluye la importación de bibliotecas, credenciales de red, creación de instancias de pines, conexión a la red y otras configuraciones.

El archivo main.py contendrá el código que ejecuta el servidor web para servir archivos y realizar tareas basadas en las solicitudes recibidas por el cliente.
